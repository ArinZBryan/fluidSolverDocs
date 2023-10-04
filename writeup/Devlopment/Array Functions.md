Added 1 New File:
- `ArrayFuncs.cs`

As this program deals very heavily with arrays, and may need to convert between 1D, 2D and eventually 3D arrays.

Facilitate this, several functions have been developed:
`T[] array2Dto1D<T>(T[,])`
`T[,] array1Dto2D<T>(T[], int, int)`
`void printArray2D<T>(T[,], string)`
`void paintTo1DArrayAs2D<T>(ref T[], T, int, int, int, int, int)`

### `T[] array2Dto1D<T>(T[,])`
This function takes a 2D array of a generic type (This is a reference type), and returns a 1D array (Also a reference type). This function is useful when using algorithms designed for a 1D array, or when iterating through all the objects in the collection, where a cast to `Span<T>` can yield significant performance gains. This function should not be called in the `update()` function, as it will allocate a large chunk of memory on the heap, which will increase garbage collection times significantly.
```cs
T[] array2Dto1D<T>(T[,] A)
{
    int len = A.GetLength(0) * A.GetLength(1);
    T[] ret = new T[len];
    for (int i = 0; i < A.GetLength(0); i++) for (int j = 0; j < A.GetLength(1); j++)
    {
        ret[i + j *A.GetLength(0)] = A[i,j];
    }
    return ret;
}
```

### `T[,] array1Dto2D<T>(T[], int, int)`
This function takes a 1D array and maps it to a 2D array given a width and height. This function may throw an `ArgumentOutOfRangeException` if the `width` times the `height` does not equal the length of the array `A`. It is important not to call this function often, as it can allocate a large quantity of memory on the heap. Though the actual allocation is not particularly expensive, the garbage collection that is needed afterwards *is* quite expensive in terms of CPU time. 
```CS
T[,] array1Dto2D<T>(T[] A, int width, int height)
{
    int len = width * height;
    if (len != A.Length) {
        StringBuilder ExceptionMessage = new StringBuilder();
        ExceptionMessage.Append("Expected length of A to be equal to width*height. A's lenth was: ");
        ExceptionMessage.Append(A.Length.ToString());
        ExceptionMessage.Append(" width*height was: ");
        ExceptionMessage.Append((width*height).ToString());
        throw new ArgumentOutOfRangeException("T[] A", ExceptionMessage.ToString());
    }
    T[,] ret = new T[width, height];
    for (int i = 0; i < width; i++) for (int j = 0; j < height; j++)
    {
        ret[i,j] = A[i + j * width];
    }
    return ret;
}
```

### `printArray2D<T>(T[,], string)`
This function will print a 2D array to the terminal, or to the debug console in a 'pretty' form, ie. as a grid.
```CS
void printArray2D<T>(T[,] arr, string delimiter = "\t")
{
    int rowLength = arr.GetLength(0);
    int colLength = arr.GetLength(1);
    for (int i = 0; i < rowLength; i++)
    {
        for (int j = 0; j < colLength; j++)
        {
            Console.Write(string.Format("{0} ", arr[i, j]));
        }
        Console.Write(Environment.NewLine);
    }
}
```

### `paintTo1DArrayAs2D<T>(ref T[], T, int, int, int, int, int)` 
This function is the largest and most complex. Given an array to 'draw' into as a reference, and a value to 'draw' with. It can draw to an x and y position on the 1D array as if it were a 2D array. It can also draw more than one cell at a time, being able to change the size of it's square 'brush'.
```CS
void paintTo1DArrayAs2D<T>(ref T[] arr, T value, int x, int y, int arrW, int arrH, int brushSize)
{
    int a = (brushSize - 1) / 2;
    List<int> validIndicies = new List<int>(brushSize*brushSize);
    for (int i = -a; i <= a; i++) for (int j = -a; j <= a; j++)
    {
            int index = (y + i) + (x + j) * arrW;
            if (y + i >= arrW || y + i < 0) { continue; }
            if (x + j >= arrH || x + j < 0) { continue; }
            if (index > arr.Length) { continue; }
            validIndicies.Add(index);
    }
    foreach (int index in validIndicies)
    {
        arr[index] = value;
    }
}
```
As this is a relatively complex function, that would not be easy to fix in the future, it was important, at least for this function, to create unit tests for this. One of the seven tests used is shown below:
```cs
paintTo1DArrayAs2D(ref vecs, 1, 0, 0, width, height, 1);
testArr = new int[10,10]{   {1,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                            {0,0,0,0,0,0,0,0,0,0},
                        };
Asserter.assert(
	Enumerable.SequenceEqual(array2Dto1D(testArr), vecs),
	"Did not paint one brush correctly",
	() => {
		Console.Write("Got:\n");
		printArray2D(array1Dto2D(vecs, 10, 10));
		Console.Write("Wanted:\n");
		printArray2D(testArr);
	}
);
Array.Clear(vecs);
```
This specific test draws a '1' to a 10x10 array of integers using a known brush size and coordinates. Using the result, we can check against an expected array (shown), and fail the test if it does not work.
> Note that `Asserter.assert` is a custom function to assert test values. The standard `Debug.Assert` was not used so more debug information could be printed to the screen on failure. As it would not be optimised away when building a release build, these tests have been removed for the final implementation.