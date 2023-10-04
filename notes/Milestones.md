# Development Milestones

- Set up Unity 3D (not URP) project
- Create simulation object 
- Set simulation object to start simulation
- Learn HLSL
	- Dispatch compute shader
	- Write to texture
	- Write to camera texture
	- Take in parameters from external gameobject
	- Read from texture
	- Read from input sources
	- Implement simulation
	- Optimise simulation
		- Reduce branching
	- Dispatch simulation from external gameobject
- User Interface for controlling basic simulation params
	- Simulation speed
	- Simulation size
	- Add density to simulation with mouse
    - Add forces to simulation with mouse
- 3D Cursor
	- Size / Shape manipulation
	- Movement
	- 'Wireframe' view of cursor object
- Create 3D version of compute shader
	- Alter for 3D
	- Input from 3D cursor
	- Create volumetric mesh from density field
- save / load simulation
	- serialise simulation state and settings
- export simulation
	- run commands in terminal
	- set compute shader2D to export texture to image sequence
	- record camera texture to image sequence 
	- use ffmpeg to stitch together image sequences
	- move file
- User interface for interacting with simulation more complexly
	- Place density sources in simuation
    - Place force fields in simulation
	- Place objects in simulation