The functions and properties in the simulator class are largely determined by the mathematical methods being used. In this case, that is directly solving the *Navier-Stokes Equations* for various points in space. Thus, at the very least, we need a function to perform the solving for the velocity equation and the density equation.

### Computations
##### Core Computations
- For computing the density, the steps are to compute the diffusion and advection of the 'particles' through the grid after adding any sources of density to the simulation. This may take the form of a density added by the user or this simulation.
- For computing the velocity, we need to diffuse the velocity (acting as a kind of entropy), compute the boundary conditions, and then perform advection.
As should be obvious, both major parts require a function to diffuse and advect a vector field. Thus, such a function should be shared between the two, with any necessary parameter able to be changed as required.
The computing of the boundary conditions is important because else, it is possible for some of the most fundamental laws of the universe to not be obeyed: conservation of mass and conservation of energy.
##### Auxiliary Computations
One of the main steps that is repeated multiple times is needing to solve a set of linear equations for some variables. This requires a function that can work on almost any vector field, and returns a vector field once that is done.
### Externally Visible Functions
As the language being used is C#, it is required that the solver be entirely contained in a class. Thus, the class must have a constructor. This constructor therefore has the responsibility of allocating the memory required for the vector fields given a size. The mathematical model also requires a timestep (also known as 'deltaTime' or in the case of Unity, `UnityEngine.Time.DeltaTime`) which is a single-precision floating point number set as an argument to the constructor.
> In the simulator's current state, deltaTime is maintained from instantiation. This may be changed to allow for a continuously updating deltaTime later.

![[Solver2D_Class_Diagram.png]]