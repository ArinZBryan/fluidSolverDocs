This is one of the more major parts of this project: saving simulations to disk, so they can be used by other programs.
The first export option is the ability to save every output image to a folder in a numbered order, such that they can be pieced back together later.

By leveraging the `IImageDestination` interface, we can create a class that will accept an image every frame, and encode it as a PNG, JPG or TGA. This leaves us with an array of bytes, per image, which we can collate into a list, and save to disk after simulating. This ensures we do not slow down the simulation by issuing lots of file IO operations to the operating system, which may be slow to complete.

```cs
public class ImageSequence : IImageDestination
{
    Texture2D texture;
    int imageWidth, imageHeight;
    Destinations.FileFormat fmt;
    string folder, name;
    List<byte[]> unsavedImages = new List<byte[]>();
    RenderTexture tempRT;
    public int lifetimeRemaining { get; set; } = int.MaxValue;
    public ImageSequence(string folder, string name, Destinations.FileFormat format)
    {
        this.folder = folder;
        this.name = name;
        this.fmt = format;
    }

    public void setImage(RenderTexture img)
    {
        if (texture == null)
        {
            texture = new Texture2D(img.width, img.height, TextureFormat.RGBA32, 1, true);
        }

        //This might be a bit slow;
        tempRT = RenderTexture.active;
        RenderTexture.active = img;
        texture.ReadPixels(new Rect(0, 0, imageWidth, imageHeight), 0, 0);
        texture.Apply();
        RenderTexture.active = tempRT;

        imageWidth = img.width;
        imageHeight = img.height;
        switch (fmt)
        {
            case FileFormat.PNG: unsavedImages.Add(texture.EncodeToPNG()); break;
            case FileFormat.JPG: unsavedImages.Add(texture.EncodeToJPG()); break;
            case FileFormat.TGA: unsavedImages.Add(texture.EncodeToTGA()); break;
            default: Debug.LogError("Cannot Use Fileformat with ImageSequence"); break;
        }
    }
    public string destroy()
    {
        for (int imageIndex = 1; imageIndex < unsavedImages.Count; imageIndex++)
        {
            string path = folder + name + (imageIndex - 1).ToString().PadLeft(4, '0');
            switch (fmt)
            {
                case FileFormat.PNG: path += ".png"; break;
                case FileFormat.JPG: path += ".jpg"; break;
                case FileFormat.TGA: path += ".tga"; break;
                default: Debug.LogError("You changed the file format during execution - Cannot save correctly"); break;
            }
            File.WriteAllBytes(path, unsavedImages[imageIndex]);
        }
        return folder;
    }
```