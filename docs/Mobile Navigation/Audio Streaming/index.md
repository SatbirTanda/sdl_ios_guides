## Audio Streaming
Navigation apps are allowed to stream raw audio to be played by the head unit. The audio received this way is played immediately, and the current audio source will be attenuated. The raw audio has to be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

In order to stream audio from a SDL app, we focus on the `SDLStreamingMediaManager` class. A reference to this class is available from an `SDLProxy` property `streamingMediaManager`.

#### Audio Stream Lifecycle
Like the lifecycle of the video stream, the lifecycle of the audio stream is maintained by the SDL library. When you recieve the `SDLAudioStreamDidStartNotification`, you can begin streaming audio.

#### Sending Data to the Stream
Once the audio stream is connected, data may be easily passed to the Head Unit. The function `sendAudioData:` provides us with whether or not the PCM Audio Data was successfully transferred to the Head Unit. If your app is in a state that it is unable to send audio data, this method will return a failure.

#### Objective-C
```objective-c

NSData* audioData = <#Acquire Audio Data#>;

if ([self.sdlManager.streamManager sendAudioData:audioData] == NO) {
  NSLog(@"Could not send Audio Data");
}
```

#### Swift
```swift
let audioData = <#Acquire Audio Data#>;

guard let streamManager = self.sdlManager.streamManager, streamManager.audioConnected else {
    return
}

if streamManager.sendAudioData(audioData) == false {
    print("Could not send Audio Data")
}
```