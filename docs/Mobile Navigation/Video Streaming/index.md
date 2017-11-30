## Video Streaming
In order to stream video from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from `SDLManager`.

### Video Stream Lifecycle
Currently, the lifecycle of the video stream must be maintained by the developer. Below is a set of guidelines for when a device should stream frames, and when it should not. The main players in whether or not we should be streaming are HMI State, and your app's state. Due to an iOS limitation, we must stop streaming when the device moves to the background state. 

The lifecycle of the video stream is maintained by the SDL library. The `SDLManager.streamingMediaManager` will exist by the time the `start` method of `SDLManager` calls back. `SDLStreamingMediaManager` will automatically take care of determining screen size and encoding to the correct video format.

!!! NOTE
It is not recommended to alter the default video format and resolution behavior but that option is available to you using `SDLStreamingMediaConfiguration.dataSource`.
!!!

### Sending Data to the Stream
To check whether or not you are ready to start sending data to the video stream, watch for the `SDLVideoStreamDidStartNotification` and `SDLVideoStreamDidStopNotification` notifications. When you receive the start notification, start sending data; stop when you receive the stop notification. There are parallel notifications for audio streaming.

Sending video data to the head unit must be provided to `SDLStreamingMediaManager` as a `CVImageBufferRef` (Apple documentation [here](https://developer.apple.com/library/mac/documentation/QuartzCore/Reference/CVImageBufferRef/)). Once the video stream has started, you will not see video appear until a few frames have been received. To send a frame, refer to the snippet below:

#### Objective-C
```objective-c
CVPixelBufferRef imageBuffer = <#Acquire Image Buffer#>;

if ([self.sdlManager.streamManager sendVideoData:imageBuffer] == NO) {
  NSLog(@"Could not send Video Data");
}
```

#### Swift
```swift
let imageBuffer = <#Acquire Image Buffer#>;

guard let streamManager = self.sdlManager.streamManager, !streamManager.videoStreamPaused else {
    return
}

if streamManager.sendVideoData(imageBuffer) == false {
    print("Could not send Video Data")
}
```

### Best Practices
* A constant stream of map frames is not necessary to maintain an image on the screen. Because of this, we advise that a batch of frames are only sent on map movement or location movement. This will keep the application's memory consumption lower.
* For an ideal user experience, we recommend sending 30 frames per second.
