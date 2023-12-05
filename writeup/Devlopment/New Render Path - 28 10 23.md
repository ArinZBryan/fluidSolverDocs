Previously, the path that was taken for an image to render was as follows:

| Old Render Path        | New Render path                                     |
| ---------------------- | ------------------------------------ |
| ![OldRenderPath.png](OldRenderPath.png) | ![NewRenderPath.png](NewRenderPath.png) |

Put to words, where before, the image, as a `UnityEngine.Texture2D` output by the simulator was directly piped straight to the viewport, now it, along with any other images output by the simulation are piped through a 'Result Dispatcher' first.
This allows for, say, an image containing a representation of the vector field to only be shown to the user, while just the density field, being moved through the velocity field is saved to disk as a sequence of images.
One important measure to note is that the incoming images are no longer instances of `UnityEngine.Texture2D`, but rather of `UnityEngine.RenderTexture`. This allows for more complex operations to be performed on these images, and makes it faster to display to the screen, however, comes with a performance penalty when converting back to a `UnityEngine.Texture2D` for saving. On the whole, this is minimal, but is still worth noting that saving a record of the data as it comes in is slower than not.