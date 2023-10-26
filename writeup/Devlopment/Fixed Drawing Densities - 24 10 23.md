The method used to draw the densities in the simulation thus far has an interesting quirk: there appear to be diagonal 'bands' in the rendering, where diffusion of density do not penetrate. This effect is not shown when printing out the values to a file or to the console, so the effect can be isolated here. This function was re-written to provide a more accurate method of rendering.

To do this, we lean on a built-in feature of Unity's `Graphics.DrawTexture` API: If we provide it with a texture smaller than the size of the `UnityEngine.Rect` that we are drawing to, it will automatically upscale the image to suit the size. This means we can render the densities as if each cell were one pixel, and let Unity offload this to the GPU, rather than dealing with this computationally expensive operation on the CPU, where the lack of hardware acceleration gives an $O(N^2)$ operation.
```cs
densTex = new Texture2D(gridSize, gridSize)
```

The new version of `void drawDensity` is as follows:
```cs
void drawDensity(in float[] density, ref Texture2D drawTex)
{
	Color[] colors = new Color[gridSize * gridSize];
    for (int simCellX = 1; simCellX <= gridSize; simCellX++) for (int simCellY = 1; simCellY <= gridSize; simCellY++)
	{
		int colIndex = ArrayFuncs.accessArray1DAs2D(simCellX - 1, simCellY - 1, gridSize, gridSize); 
		int denIndex = ArrayFuncs.accessArray1DAs2D(simCellX, simCellY, gridSize + 2, gridSize + 2);
        colors[colIndex].r = density[denIndex];
        colors[colIndex].g = density[denIndex];
        colors[colIndex].b = density[denIndex];
        colors[colIndex].a = 1f;
	}
	drawTex.SetPixels(colors);
	drawTex.Apply();
```