So far, the display of velocities per cell has been broken, and the lines meant to show the direction and magnitude of the velocities have been drawing incorrectly, a small update to a mathematical formula passed into the line-drawing algorithm fixes this issue.
```cs
line(ref drawTex, xCoord, yCoord, (int)Math.Round((normalised * maxLength).x) + xCoord, (int)Math.Round((normalised * maxLength).y) + yCoord, foreground);
```
The addition of the `xCoord` and `yCoord` variables fixes the issues, preventing the lines from rendering one point at the origin.

Another fix to the way directions of velocities were calculated is as follows

```cs
void drawVelocity(in float[] velocityX, in float[] velocityY, ref Texture2D drawTex, Color background, Color foreground)
{
    int maxLength = (scale - 1) / 2;
    //Makes a texture of one colour
    Color[] pixels = Enumerable.Repeat(background, Screen.width * Screen.height).ToArray();
    drawTex.SetPixels(pixels);
    Vector2 normalised;
    Vector2 velocity;
    int xCoord, yCoord;
    for (int i = 1; i < N; i++)
    {
        for (int j = 1; j < N; j++)
        {
            velocity.x = ArrayFuncs.accessArray1DAs2D(i, j, N, N, velocityX);
            velocity.y = ArrayFuncs.accessArray1DAs2D(i, j, N, N, velocityY);
            normalised = velocity.normalized;
            xCoord = (i - 1) * scale + 2;
            yCoord = (j - 1) * scale + 2;
            line(ref drawTex, xCoord, yCoord, (int)Math.Round((normalised * maxLength).x) + xCoord, (int)Math.Round((normalised * maxLength).y) + yCoord, foreground);
		}
	}
}
```

### Combined Drawing
Another feature added here was the ability to draw the background of the velocity screen as transparent. This allows for the ability to overlay it on top of the density display. This means that the velocities can be manipulated while the user is able to see their effects on the density field.
A new key binding was made to enable this mode of drawing using the following code:
```cs
bool drawBoth = false;

if (Input.GetKeyUp(KeyCode.T))
{
    drawBoth = !drawBoth;
}

if (drawBoth)
{
	drawDensity(solver.getDensity(), ref densTex);
    drawVelocity(solver.getVelocityX(), solver.getVelocityY(), ref velTex, Color.clear, Color.green);
} else {
    drawVelocity(solver.getVelocityX(), solver.getVelocityY(), ref velTex, Color.black, Color.green);
}
```
This means that pressing the 'T' key toggles whether this mode is used for displaying velocities. Note that as this requires doing the calculations for drawing densities as well, this can come with a small performance hit. This however, is negligible to the hit taken by viewing velocities in the first place, which can be quite expensive to draw all the required lines using *Bresenham's Line Drawing Algorithm*. Such algorithms allow for efficient computation of which pixels should be drawn when rasterising a vector line of an arbitrary slope.