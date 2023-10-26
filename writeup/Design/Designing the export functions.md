The exporter class is a class that needs to be able to export the simulation in a variety of ways
- As an encoded video
	- MP4
	- MOV
	- GIF
- As an image sequence
	- PNG
	- JPEG
- As a reloadable format

It is important that this class be re-usable no matter the simulation, and there should be scope for adding formats to export to in the future. Should a new type of video format become available in the future, there should be minimal modifications needed to support it. To make this work, I will be using an open-source library called FFMpeg, or more specifically, a library called . This is a C# wrapper for the FFMpeg executable, letting me interact with the native code functions without having to spin up a terminal and interact with it via text.

### Class Structure

