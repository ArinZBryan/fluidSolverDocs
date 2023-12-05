Previously, as part of the drawing and compositing of densities, the density texture needed to be scaled up, as it is drawn pixel by pixel, versus drawing large squares, which can be error prone.

### The Problem
As a result, the method used was to simply convert the `UnityEngine.Color[]` which held the output texture into a `UnityEngine.Texture2D` which can be sent to the GPU, where dedicated hardware *should* be able to do the scaling operation rather quickly. Finally, the scaled texture is sent back to the CPU for conversion back to a `UnityEngine.Color[]` where it can be worked on to draw velocity or rendered to the screen.

However, there two parts of this that are really quite slow: getting data to the GPU and getting data back from the GPU. Despite the speed at which PCIE devices *can* talk to the CPU, most of the bandwidth is taken by other things, and any large data transfers have to wait until the end of the frame before starting. These transfers then prevent the GPU from starting to render the next frame, causing a very low framerate.

### A Solution
The simple yet counter-intuitive solution to this is to move the calculations from hardware that should be optimised from it back to the place where the rest of the calculations are performed (the CPU)

This however, requires writing an algorithm to perform the scaling. There are many such algorithms that could be used, but the simplest by far is *integer scaling* combined with [nearest neighbour interpolation](https://en.wikipedia.org/wiki/Nearest-neighbor_interpolation). My method, more commonly just called integer scaling, is as follows:
```cs
public static T[,] scaleArray2D<T>(T[,] arr, int scale)
    {
        T[,] dest = new T[arr.GetLength(0) * scale, arr.GetLength(1) * scale];
        for (int inputX = 0; inputX < arr.GetLength(0); inputX++) 
        for (int inputY = 0; inputY < arr.GetLength(1); inputY++)
            {
                T item = arr[inputX, inputY];
                for (int outputX = 0; outputX < scale; outputX++) 
                for (int outputY = 0; outputY < scale; outputY++)
                    {
                        int destX = inputX * scale + outputX;
                        int destY = inputY * scale + outputY;
                        dest[destX, destY] = item;
                    }
            }
        return dest;
    }
```
This, as should be plainly obvious only operates on 2D arrays (of any [reference or value type](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types)). A second version of this algorithm was made by modifying the above code to function on single dimensional arrays:
```cs
public static T[] scaleArray1Das2D<T>(T[] arr, int scale, int originalWidth, int originalHeight)
    {
        T[] dest = new T[originalHeight * scale * originalWidth * scale];
        for (int inputX = 0; inputX < originalWidth; inputX++) 
        for (int inputY = 0; inputY < originalHeight; inputY++)
            {
                T item = arr[inputX + inputY * originalWidth];
                for (int outputX = 0; outputX < scale; outputX++) 
                for (int outputY = 0; outputY < scale; outputY++)
                    {
                        int destX = inputX * scale + outputX;
                        int destY = inputY * scale + outputY;
                        dest[destX + destY * originalWidth * scale] = item;
                    }
            }
        return dest;
    }
```

### Finding the right solution
This code performs a vital function in the drawing and updating of the simulation, thus it is important that it be fast, maintainable and most importantly, **not slower than the previous implementation**. This is a fairly easy thing to test: by looking at the Unity profiler, I can see that for each frame this function is called, it takes roughly 12-14 milliseconds to complete. By comparison, the new version takes ~8ms to complete. Seemingly this is a straight forward conclusion, however, there is one more thing to contemplate: 1% lows and highs. In the new version, we see a more consistent framerate, with the old version however, there were occasional highs of ~60fps, a huge improvement over the average.
In general, this behaviour is undesirable, and large swings in framerate can look quite unnatural. Finally, this brings us to a conclusion: the new CPU-side scaling is a better fit for this project.
