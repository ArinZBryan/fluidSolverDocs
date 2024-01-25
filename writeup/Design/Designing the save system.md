One of the primary functions of this program is the ability to record and save playbacks of previous simulations. These then need to be able to be edited or recorded over to allow for fine tuning of the simulation.

### Saving a file
There are seven buffers that need to be kept track of:
- Velocity This Frame (x-component)
- Velocity Last Frame (x-component)
- Velocity This Frame (y-component)
- Velocity Last Frame (y-component)
- Density This Frame 
- Density Last Frame
- Current Simulation Objects
In theory, it would be sufficient to save the state of each of them each tick, however, this is far more data saved than is necessary. If we want to access a frame, we can source the previous data buffers by simply looking at the tick before the one being loaded. This allows us to cut save file sizes by half (a not insignificant saving). However, we must still include all buffers in the first frame, as there is no previous frame to load from.
Thus, we have the following buffers:
- Velocity This Frame (x-component)
- Velocity This Frame (y-component)
- Density This Frame
- Current Simulation Objects
This is good, but it overlooks a critical feature, by saving these buffers, it is impossible make changes to the simulation within the saved frames. We can kill two birds with one stone here by leveraging the fact that we still have access to all the functions of the fluid solver. This allows us to use a trick that will lower the final file size of the saved playback, but will also allow for modification and playback.
##### Game Demos
Though not a common process nowadays, it used to be common for games such as *Doom*, *Quake* and *Half-Life* to support recording 'demos'. These were files that specified the exact user input. As these games had either deterministic random functions (eg. if the game generated 3 now, it will always generate 179 next) or could record the results of the random function. This allowed for the game engine to behave exactly as if it were being played, and generate the computationally expensive graphics on-the-fly. This meant that the sizes of 'demofiles' to be small enough to be shared on mid-nineties internet in a reasonable time, something video files could not do. 
##### The Final Saved Data
This leaves us with the following fields to be saved:
- Density updated by user this frame
- Velocity updated by user this frame (x-component)
- Velocity updated by user this frame (y-component)
- Current Simulation Objects
##### The process of saving
A new instance of the `Savefile` class is created. At initialisation, the constant simulation values are saved into this class. Then, each frame is copied into a new instance of either the `SolveState.Updateframe` or `SolvesState.Keyframe` classes. If the frame is the first one to be saved, then a `SolveState.Keyframe` is used, and the instance copied into the save file. All subsequent frames are stored into a list of type `List<SolveState.Updateframe>`. Finally, the `Savefile` class can be serialised to a binary stream, and saved to disk. This would be done as follows:
```cs
if (savefile.isFull)
{
	var f = System.IO.File.Create("./saves/save.simsave");
    var b = new System.Runtime.Serialization.Binary.BinaryFormatter();
    b.Serialize(f, savefile);
}
```
However, to be able to use the serialisation functions provided by the language, each class that must be serialised must be marked as `[System.Serializable]`. This must be applied to all custom classes that need serialising. Thus, this means that the following must have this property applied: 
- `Savefile`
- `SolveState.Keyframe`
- `SolveState.Updateframe`
- `PackedArray<T>`
### Loading a file 
To load a file, first the save must be loaded from disk. Then begins the process of de-serialising the binary blob. This should be done as follows:
```c#
var f = System.IO.File.Open("./saves/save.simsave");
var b = new System.Runtime.Serialisation.Binary.BinaryFormatter();
Savefile s = (Savefile) b.Deserialize();
```
This leaves us with a complete recreation of the original save file. From here, we can choose to load the required elements to reconstruct the simulation at any given point in time.
##### Restoring to any point in time
1. Load the save file
2. Instantiate the fluid solver with the required parameters from the save file
3. Load the required saved ticks
4. Load values from the saved ticks into the required fields.
5. Continue simulating from this time.