## The Problem
In 3D-Graphics workloads, such as in CGI, one of the most complex and computationally expensive actions is the simulation of fluids in a realistic manner. Further, it is highly complex to set up such a simulation, as highly accurate simulations have a tendency to 'blow up', or otherwise act erratically, wasting much time computing simulations that will ultimately be scrapped. It is for this reason, that it is highly useful to use a dedicated program to view and tweak the simulation, in real-time. This allows for the later composition of the resulting images into such projects.

## My Solution
Using the simulation method outlined in the 2003 paper 'Real-Time Fluid Dynamics for games', I can simulate almost any situation involving incompressible fluids. The simulation method outlined in the paper is optimised for both speed of computation, and also stability. This makes it exceedingly difficult to create a simulation where the final output does not look plausibly realistic. The main program will consist of a rendered view, using a lower resolution version of the simulation. To allow for easy rendering of 2D/3D scenes, the Unity game engine will be used to provide easy CPU and GPU rendering capabilities. This also allows for significant portability, which is important, as artists may choose to work on their preferred platform, without disrupting their workflow. The simulation can finally be exported, at any size and file format using integrated command-line tools (FFMPEG).

## Main features of the solution
My solution should be able to at a baseline, simulate stable fluid dynamics in 2D on the CPU, using managed C# code as part of Unity. This simulation should also be able to be tweaked in semi-real-time. This should then be able to be saved as an image sequence or video that can be used elsewhere. However, I intend to implement both 3D simulation and GPU acceleration using compute shaders that can be run in parallel. I would also like to have a fully interactable and possibly scriptable simulation, such that velocities and densities can be added and changed as the simulation runs.

## Additional features of the solution
On top of the mentioned features, there may be room to implement other features. For example, drawing the information to the screen in other ways such as with arrows for the vectors, and rendering multiple passes of information to the screen at once. Another major possible feature is the ability to modify the boundary parameters of the simulation to allow for obstructions and objects within the simulation. This could then be extended to three dimensions, where models might be able to be imported into the scene and converted to volumetric voxels for simulation.