Added 1 New File:
- `ScreenDrawWithEffect.cs`
This file contains a working implementation of a viewport pen, that draws directly to the screen. It also provides an interface `ITexture2DEffect`, which can be inherited from, and allows for different effects on the drawn image. 

```cs
using System;
using UnityEngine;

public class ScreenDrawWithEffect : MonoBehaviour
{
    public int texWidth = 32;
    public int texHeight = 32;
    
    public Color penColor;
    public Color baseColor;
    Vector4 penVector;
    Vector4 baseVector;

    public int penSize = 1;

    public static Texture2D drawTex;
    public static Vector4[,] drawVecs;

    int mouseX;
    int mouseY;
    public ITexture2DEffect selectedImplementation;

    // Start is called before the first frame update
    void Start()
    {

        //force pen texture to always be visible
        if (penColor.a != 1)
        {
            penColor.a = 1;
        }

        drawTex = new Texture2D(texWidth, texHeight);
        drawTex.filterMode = FilterMode.Point;

        drawVecs = new Vector4[texWidth, texHeight];
        penVector = (Vector4)penColor;
        baseVector = (Vector4)baseColor;
        for (int i = 0; i < texWidth; i++) for (int j = 0; j < texHeight; j++) drawVecs[i, j] = baseVector;

        selectedImplementation = new Texture2DEffects.NoEffect();
    }

    // Update is called once per frame
    void Update()
    {
        mouseX = (int)Input.mousePosition.x;
        mouseY = (int)Input.mousePosition.y - (Screen.height - texWidth);

        mouseX = Math.Clamp(mouseX, 0, texWidth-1);
        mouseY = Math.Clamp(mouseY, 0, texHeight - 1);

        if (Input.GetMouseButton(0))
        {
            vector4Paint(ref drawVecs, penVector, mouseX, mouseY, penSize);
        }

        if (Input.GetMouseButton(1))
        {
            vector4Paint(ref drawVecs, baseVector, mouseX, mouseY, penSize);
        }

        selectedImplementation.runEffect(ref drawVecs);
        drawTex = vector4sToTexture(drawVecs);
    }

    void OnGUI()
    {
        if (Event.current.type.Equals(EventType.Repaint))
        {
            Graphics.DrawTexture(new Rect(0, 0, texWidth, texHeight), drawTex);
        }
    }

    Texture2D vector4sToTexture(Vector4[,] vecs)
    {
        int vecWidth = vecs.GetLength(0);
        int vecHeight = vecs.GetLength(1);

        Color[] cols = new Color[vecWidth * vecHeight];

        for (int i = 0; i < vecWidth; i++)
        {
            for (int j = 0; j < vecHeight; j++)
            {
                cols[i + j * vecWidth].r = vecs[i, j].x;
                cols[i + j * vecWidth].g = vecs[i, j].y;
                cols[i + j * vecWidth].b = vecs[i, j].z;
                cols[i + j * vecWidth].a = vecs[i, j].w;
            }
        }

        Texture2D tex = new Texture2D(vecWidth, vecHeight);
        tex.SetPixels(cols);
        tex.Apply();
        return tex; 
    }

    void vector4Paint(ref Vector4[,] vec, Vector4 value, int x, int y, int brushSize)
    {
        int vecWidth = vec.GetLength(0);
        int vecHeight = vec.GetLength(1);
        for (int i = x-(brushSize-1); i < x + brushSize; i++ )
        {
            if (i >= vecWidth || i < 0) { continue; }
            for (int j = y-(brushSize-1); j < y + brushSize; j++)
            {
                if (j >= vecHeight || j < 0) { continue; }
                vec[i, j] = value;
            }
        }
    }
}
```
This file contains the following methods:
- `vector4sToTexture()`
- `vector4Paint()`
These two methods are used to convert a 2D array of `UnityEngine.Vector4`, to a `UnityEngine.Texture2D`, and to 'paint' a value to a 2D array of `UnityEngine.Vector4`.  
# Finished ScreenDrawWithEffect 13-08-23
Added 1 New File:
- `Texture2DEffects.cs`
This adds two example effects for use within the interface of `ITexture2DEffect`, and can be chosen as options when using the `ScreenDrawWithEffect` Monobehaviour and script.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public interface ITexture2DEffect
{
    void runEffect(ref Vector4[,] texture);
}

public class Texture2DEffects
{ 
    public class NoEffect : ITexture2DEffect
    {
        public void runEffect(ref Vector4[,] vectors)
        {
            //Do Nothing
        }
    }

    public class Blur : ITexture2DEffect
    {
        public void runEffect(ref Vector4[,] vectors)
        {
            int imageWidth = vectors.GetLength(0);
            int imageHeight = vectors.GetLength(1);
            for (int x = 1; x < imageWidth - 1; x++)
            {
                for (int y = 1; y < imageHeight - 1; y++)
                {
                    vectors[x, y] = (vectors[x - 1, y + 1] + vectors[x, y + 1] + vectors[x + 1, y + 1] + vectors[x - 1, y] + vectors[x, y] + vectors[x + 1, y] +        vectors[x - 1, y - 1] + vectors[x, y - 1] + vectors[x + 1, y - 1] ) / 9;
                }
            } 
        }
    } 
}
```