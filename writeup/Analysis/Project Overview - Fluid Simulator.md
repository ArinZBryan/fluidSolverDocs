This project is a real-time, fully interactable incompressible fluid simulator that can be saved to disk and resumed at any time. The fluid simulation is able to interact with all kinds of objects within the simulation. From rigidbodies (normal hard objects), to force fields and particle systems. 
## Features of this Solution
- Real-time fluid simulation
	- Customisable simulation aesthetics
		- Attachable particle systems
		- Colour and tint options
	- Exportable to common media formats
		- MP4
		- MOV
		- Image Sequences (PNG / JPG / TGA / BMP)
	- Fully interactive simulation
		- Variety of simulatable objects and interactions
	- Fully deterministic simulation
	- Loadable scenarios from disk to simulation.

## Technical Specifications
This project is an implementation of a fluid simulator, using the methods outlined in the 2003 paper '*Real-Time Fluid Dynamics for Games*' by Jos Stam. This implementation, unlike the original code originally provided with the initial release of the paper in a conference, is written in C#, using Unity as the rendering and 3D engine. It also provides access to the keyboard and an API with which to access the GPU if needed. The version of Unity being used is version `2021.3.23f1`. This mandates the use of either IL2CPP, or Mono 6.4 implementing the .NET standard version 2.1.
#### GPU Support
The GPU API provided by Unity uses standard HLSL shaders, which are dynamically compiled during runtime, and cached for later reuse on the target hardware. Using HLSL, if needed, the GPU can execute fragment, vertex and compute shaders. This project will likely just use compute shaders, as they are the most customisable, and there is no need for the results to be rendered.
#### IL2CPP vs Mono 6.4
IL2CPP is a compiler provided with Unity, that allows for the compilation of the intermediate language that C# compiles to to be further compiled down to machine code. This option not only protects the project's code be significantly obfuscating the output from source. On the other hand, Mono is an open source JIT compiler for C#, supported by Unity. This means that though the code is shipped in a reverse-engineerable format, that is often slower than IL2CPP. Though on the face of it, there seems to only be disadvantages to using Mono, there is one major advantage during development: the lack of a second compile step. IL2CPP must first compile all code to the intermediate language before further compilation to a final binary. On the other hand, Mono has no need for a second compilation step, giving the opportunity for a faster pace of development and testing. For this reason, the majority of the development will be done in Mono, and the final builds will be made using IL2CPP.
#### Other APIs
This program also uses FFMPEG, a video encoding and decoding library to allow for the exporting of animations to other programs. This may also require the inclusion of video codecs with the software to allow for exporting to many different types of file format.