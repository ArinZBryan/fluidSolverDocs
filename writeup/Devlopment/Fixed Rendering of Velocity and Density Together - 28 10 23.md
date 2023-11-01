With the move to use `UnityEngine.RenderTexture` wherever possible, some functions must be swapped out. Chief among them is the move from:
```c#
-- Texture1.SetPixels(Texture2.GetPixels())
++ Graphics.CopyTexture(Texture1, Texture2)
```
This seemingly small change actually results in quite a large change when we look at what these functions do: The first is entirely on the CPU, whereas the second is entirely on the GPU. This is because now, the textures are `UnityEngine.RenderTexture`s, which are objects which can only live on the GPU, and can't really be used on the CPU.

Another major change was the move from:
```cs
-- Graphics.drawTexture(Texture1);
-- Graphics.drawTexture(Texture2);
++ Graphics.Blit(RenderTexture, Texture2D);
++ Graphics.Blit(RenderTexture, Texture2D);
```
However, this results in a problem. `Graphics.Blit()` is a *full copy* of the data from the `UnityEngine.Texture2D`, including transparency. This means we lose out on the automatic alpha blending (In essence, if some part of the second image was transparent, the transparency would overwrite the first image, rather than letting the first image through.) performed when just rendering out to the viewport, as it is now essential to return a `UnityEngine.RenderTexture`.  To fix this, a far more basic approach was used.

### The Approach
As both textures start off as a `UnityEngine.Color[]`, we can just perform the calculations to make the second image on the same array, only converting to a `UnityEngine.Texture2D` when finally necessary.

> One note is that the original arrays were different sizes, so in actuality, the first colour array was converted to an image, scaled up on the GPU, and converted back to a colour array for further drawing to. This just so happens to be the easiest way to do this, and allows for the use of different blending modes if the need arises. 

