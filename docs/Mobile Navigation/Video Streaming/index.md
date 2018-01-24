## Video Streaming
In order to stream video from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from `SDLManager`.

### Video Stream Lifecycle
Currently, the lifecycle of the video stream must be maintained by the developer. Below is a set of guidelines for when a device should stream frames, and when it should not. The main players in whether or not we should be streaming are HMI State, and your app's state. Due to an iOS limitation, we must stop streaming when the device moves to the background state. 

The lifecycle of the video stream is maintained by the SDL library. The `SDLManager.streamingMediaManager` will exist by the time the `start` method of `SDLManager` calls back. `SDLStreamingMediaManager` will automatically take care of determining screen size and encoding to the correct video format.

!!! NOTE
It is not recommended to alter the default video format and resolution behavior but that option is available to you using `SDLStreamingMediaConfiguration.dataSource`.
!!!

### CarWindow
CarWindow is a system for automatically video streaming a view controller's frame to the head unit. It will automatically set the view controller passed to the correct frame and start sending the data when the video service has completed setup. There are a few important `SDLStreamingMediaConfiguration` parameters. To start, you will have to set a `rootViewController`, and there are initializers to help in doing so. You can also choose how CarWindow captures and renders the screen using the `carWindowRenderingType` enum. You can use the initializers `+ (instancetype)autostreamingInsecureConfigurationWithInitialViewController:(UIViewController *)initialViewController;` and `+ (instancetype)autostreamingSecureConfigurationWithSecurityManagers:(NSArray<Class<SDLSecurityType>> *)securityManagers initialViewController:(UIViewController *)initialViewController;`.

By default, when using `SDLCarWindow`, the `SDLTouchManager` will sync it's touch updates to the framerate. To disable this, set `SDLTouchManager.enableSyncedPanning` to `NO`.

Note that CarWindow will hard-dictate the framerate of the app. To change it and other parameters, update `SDLStreamingMediaConfiguration.customVideoEncoderSettings`.

These are the current defaults:
```objc
@{
    (__bridge NSString *)kVTCompressionPropertyKey_ProfileLevel: (__bridge NSString *)kVTProfileLevel_H264_Baseline_AutoLevel,
    (__bridge NSString *)kVTCompressionPropertyKey_RealTime: @YES,
    (__bridge NSString *)kVTCompressionPropertyKey_ExpectedFrameRate: @15,
    (__bridge NSString *)kVTCompressionPropertyKey_AverageBitRate: @600000
};
```

#### Replacing the view controller
Simply update `self.sdlManager.streamManager.rootViewController` to the new view controller. This will also update the haptic parser.

#### App UI vs. Off-Screen UI
It is generally recommended to pass a non-on-device-screen view controller to display via CarWindow, that is, to instantiate a new view controller and pass it. This will then appear on-screen in the car, while remaining off-screen on the device. It is also possible to display your on-device-screen UI to the car screen by passing `UIApplication.sharedApplication.keyWindow.rootViewController`. However, if you use the app UI, the app's UI will have to resize to accomidate the head unit's screen size.

!!! NOTE
If the `rootViewController` is app UI and is set from the `UIViewController` class, it should only be set after `viewDidAppear:animated` is called. Setting the `rootViewController` in `viewDidLoad` or `viewWillAppear:animated` can cause weird behavior when setting the new frame.
!!!

!!! NOTE
If setting the `rootViewController` when the app returns to the foreground, the app should register for the `UIApplicationDidBecomeActive` notification and not the `UIApplicationWillEnterForeground` notification. Setting the frame after a notification from the latter can also cause weird behavior when setting the new frame.
!!!

### Manually Sending Data to the Stream
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
