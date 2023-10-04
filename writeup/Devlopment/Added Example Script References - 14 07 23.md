Added 4 New Files:
- `DeployComputeShader.cs`
- `DrawImage.cs`
- `ExampleComputeShader.compute`
- `ScreenDraw.cs`

### DeployComputerShader.cs
Compute shaders are one of the more complex topics when it comes to Unity development. Commonly, they are used to speed up tasks otherwise slow to run in serial, buy offloading them to the GPU. This allows for the CPU to only have to run the complex branching code, with large mathematical computation allocated to hardware built for this kind of program.
The first hurdle when it comes to using compute shaders is setting up communication between the GPU and the CPU. This has to be done by allocating a buffer of memory that can be passes between the CPU and GPU. As this is the only way to provide data to a compute shader, miscellaneous parameters must also be passed this way.
### ExampleComputeShader.compute
This is an example implementation of a compute shader, that is compatible with the code used in `DeployComputeShader.cs` . This particular shader draws the UV coordinates of the texture being rendered to the screen.
### DrawImage.cs
This script gives an example implementation for software rendering of 2D graphics straight to the screen. To do this, a buffer of memory is prepared (a `Vector4[]`) into which graphics can actually be rendered. This can then be passed into a native Unity function which converts this to an array of `UnityEngine.Color` objects, and passes those to the GPU for rendering. Also shown in this file is how to draw an arbitrary image to the screen from a file.

### ScreenDraw.cs
This script builds upon the work done in `DrawImage.cs`, and provides an example implementation of a 'pen' that the player can draw on the screen with. The colour, opacity and brush size can all be changed as an example of possible parameters.