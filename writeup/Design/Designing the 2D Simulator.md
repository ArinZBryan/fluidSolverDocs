The 2D simulator is encompassed by the `Solver2D21.cs` file. It contains one class: `Solver2D21`. This is derived from Unity's built in `MonoBehaviour` class. This is a class that allows for scripts to be 
attached to gameobjects, and thus run in the context of the gameobject.
## Computations
This simulator has two main computations that it has to do:
- Drawing Density
- Drawing Velocity
It also has to create and dispatch the 2D solver, as well as any simulation objects to compute the simulation. 
#### Drawing Density
To draw density, the value of the density at every grid cell needs to be looked up, and the red, green and blue channels of the output texture set to the same value. Though this seems quite simple, it does run into one major roadblock: the 'real' grid that the simulation runs on is larger than the area designed to be shown to the viewer. This is because the border cells (the cells adjacent to the edge) are used to ensure the simulation is continuous around borders Finally, the texture is scaled up by a scaling factor, so it can be viewed at a reasonable scale.
#### Drawing Velocity
To draw the velocity field, each cell excluding the border cells must have its horizontal and vertical velocities loaded from the respective memory spans that they are contained in. Then two coordinates need to be found. One is the center of each cell after it has been scaled up. The other coordinate needed is a point within a circle with a radius the size of the scaling factor. The direction of the line formed is to be the same direction of the original velocity and a length that is a scaled version of the magnitude of the velocity at that point, This can then be added to the first coordinate to form a line that can be used to visualise the velocity at that point.
![VelocityDraw1.drawio.svg](VelocityDraw1.drawio.svg)
> Above: The process of finding the second coordinate.
> Below: The vector field viewed by the user.

![VelocityDraw2.drawio.svg](VelocityDraw2.drawio.svg)
### Dispatching the Solver
When the gameobject this `MonoBehaviour` is attached to is instantiated during scene creation, the `Start()` method of this class will be run. Here we will allocate the memory used in the solver and also the memory used to store the texture data that will be written to the scene. 
Every *tick*, the `FixedUpdate()` method is called. Here, we take user input and process the results by calling the solver's `densStep()` and `velStep()` functions. Then we can compute the next image to show the user and place it in memory. Normally, taking user input and computing graphics would be done every frame (`Update()` rather than `FixedUpdate()`), but as physics calculations are also being done here, we need this to be consistent, so it must be placed in `FixedUpdate()`.
Every time the UI is redrawn, the `OnGUI()` function is called, this redraws the relevant image to the viewport and adjusts scaling as needed. The `OnGUI()` function is generally analogous to the `Update()` function in that it is called once per frame, however, it is called at the end of the frame, rather than at the start, so it is expected all calculations are done by the time this function is called.

### Creating Simulation Objects
To create a simulation object, first an instance of the derived class must be instantiated, and appended to a `List<SimulationObject>`. This allows for the simulator to complete all the required calculations for all the simulation objects, no matter how many there are. However, even though the limit for the number of simulation objects is theoretically infinite, this does not mean it is advisable to use too many.
In the Unity editor, a special macro is used to allow for such simulation objects to be created, with individually tweakable parameters at runtime. This is the `[Button("Text")]` macro provided by the [Advanced Editor Tools](./../Development/Advanced%20Editor%20Tools%20-%2024%2010%2023.md) extension
### Dispatching Simulation Objects
All simulation objects require different information when dispatching them, so making an interface that's shared between them would seem to be impossible at first glance. Though the obvious solution to this is simply to pass in a reference to the data upon instantiation, this doesn't work for a few reasons:
1. `ref` Fields are only supported on `ref struct` structs
	This requires that all instances of this class is stored on the stack. As this class is being stored in a linked list, C# forbids this to prevent a stack overflow.
2. C# 9.0 Does not support `ref` fields
	This one is a bit more simple. Unity runs all C# code within the editor using an open-source C# runtime called Mono. It is also possible to ship a version of Mono with the project, and this is often needed when attempting to support platforms that do not have support by IL2CPP. Thus, C# requires all code to be compatible with Mono. As of Unity's current build (Unity 2023.2.3f1), and including the version being used for this project (Unity 2021.3.23f1 LTS), the version of Mono in use only supports up to C# 9.0. It is planned for new releases of Unity to use the new CoreCLR, an alternative offered by Microsoft sometime in the next few years. This will then bring support for C# 11. Once this update happens, `ref` fields will be possible, but until then, `ref` fields remain a language feature I cannot use.