The previous array drawing algorithm was often quite slow, and often proved unreliable near the boundaries of the simulation. Thus, a replacement was needed, at least in the interim. Therefore, this, much simpler algorithm is used.
```cs
public static void edit1DArrayAs2D<T>(ref T[] arr, T value, int x, int y, int arrW, int arrH)
    {
        T[,] values = array1Dto2D(arr, arrW, arrH);
        values[x,y] = value;
        arr = array2Dto1D(values);
    }
```
This algorithm **does not** support variable 'brush sizes', unlike its predecessor, and is significantly more costly in memory usage, as it allocates and frees effectively a complete copy of one of the simulation's fields. This was the primary reason for the over-designing of the function this is replacing, however, in testing of this function, the performance impact this has is minimal. One of the main goals in the design of the code in this project is to minimise memory allocations and frees; this is because memory operations are very slow, as they must go through the operating system.

To test the function above, a new method of drawing 2D arrays was used. This code is adapted from code meant to render mathematical matrices to the terminal in a 'pretty print' format. The project of mine this was adapted from can be found here: https://github.com/ArinZBryan/InverseMatrix

```cs
public static string printArray2DMatrix<T>(T[,] matrix)
    {
        StringBuilder stringBuilder = new StringBuilder();
        int longestChar = 0;
        for (int row = 0; row < matrix.GetLength(0); row++)
        {
            for (int column = 0; column < matrix.GetLength(1); column++)
            {
                if (matrix[row, column].ToString().Length > longestChar)
                {
                    longestChar = matrix[row, column].ToString().Length;
                }

            }
        }
        int xPadding = matrix.GetLength(1) * 2 + matrix.GetLength(1) * longestChar;
        for (int xDisplay = 0; xDisplay < matrix.GetLength(0); xDisplay++)
        {
            for (int yDisplay = 0; yDisplay < matrix.GetLength(1); yDisplay++)
            {
                if (yDisplay == 0)
                {
                    stringBuilder.Append(" ");
                }
                int requiredPadding = longestChar - matrix[xDisplay, yDisplay].ToString().Length;
                if (matrix[xDisplay, yDisplay].ToString().Length < longestChar)
                {
                    if (requiredPadding % 2 != 0)
                    {
                        stringBuilder.Append(" ");
                        requiredPadding--;
                    }
                    for (int padding = 0; padding < (requiredPadding) / 2; padding++)
                    {
                        stringBuilder.Append(" ");
                    }
                }
                stringBuilder.Append(" ");

                stringBuilder.Append(matrix[xDisplay, yDisplay].ToString());
                stringBuilder.Append(" ");
                if (yDisplay == matrix.GetLength(1) - 1)
                {
                    stringBuilder.Append(" ");
                }
                for (int padding = 0; padding < requiredPadding / 2; padding++)
                {
                    stringBuilder.Append(" ");
                }
            }
            stringBuilder.Append(" \n");
        }
        return stringBuilder.ToString();
    }
```

This code is so much longer than the original code used because it accounts for one more thing: the text representation of elements in the array may be of a variable length, and so the column width may need to be adjusted to account for this. One other notable difference is the move from using string concatenation to using the `System.Text.StringBuilder` class provided with the standard library. This allows for the memory usage of these strings to be minimised, and prevents significant amounts of memory moves during string concatenation.