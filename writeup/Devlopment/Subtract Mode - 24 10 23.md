So far, it has only been possible to add densities to the density filed, and so, after enough density has been added, the simulation becomes useless. This would mean that the differences between the levels of density (the only truly useful data) become impossible to see.

The following code adds a method, when the right mouse button is pressed, to remove density:
```cs
ArrayFuncs.paintTo1DArrayAs2D(ref solver.getDensityPrev(), -10000f, cursorY, cursorX, gridSize, gridSize, penSize);
```
One thing to note is that if the right mouse button is held, then the velocity field can be filled with a large negative density. This requires a large positive density be added to get back to a neutral density (somewhere around zero).