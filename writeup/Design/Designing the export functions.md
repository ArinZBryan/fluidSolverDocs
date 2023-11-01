The exporter class is a class that needs to be able to export the simulation in a variety of ways
- As an encoded video
	- MP4
	- MOV
	- GIF
- As an image sequence
	- PNG
	- JPEG
- As a reloadable format

## Implementation
#### The Result Dispatcher
To accomplish this, we are using composition to inject a class implementing the `IImageDestination` interface into a new object, the `ResultDispatcher`. This object should never be instantiated more than once, and thus can be considered a singleton. However, as the only place this will be instantiated is as a gameobject upon scene load, it should be unnecessary to implement protections against this and the desyncs and overwriting that would occur as a result. This object takes in a `UnityEngine.RenderTexture` from a valid `ISimulator` and dispatches it to the relevant `IImageDestination`s. This will almost always include `Destination.Viewport`, but may also include `Destination.ImageFile`, `Destination.VideoFile` or `Destination.ImageSequence`. Note that `Destination.ImageFile` is designed for taking snapshots  of the simulation at *one* point in time, rather than a running view (use `Destination.ImageSequence` or `Destination.VideoFile` for this).
#### The Image Destinations
All image destinations are bundled together under the `Destination` class. This class has no function other than code organisation and holding the `FileFormat` enumerator. Each sub-class must implement the `IImageDestination` interface to be passed to the [result dispatcher](#The%20Result%20Dispatcher). Each of these must implement the functions `public void sendCurrentImage()`, `public void init(string, Destination.FileFormat)` and `public string destroy()`
![[Simplan-dark.svg]]


