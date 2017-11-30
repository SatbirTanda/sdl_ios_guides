## Knowing the In-Car UI Status
Once your app is connected to Core, most of the interaction you will be doing requires knowledge of the current In-Car UI, or HMI, Status. For a refresher on how to get your app integrated with SDL, and to connect to Core, head to [Getting Started > Integration Basics](Getting Started/Integration Basics). The HMI Status informs you of where the user is within the head unit in a general sense. 

Refer to the table below of all possible HMI States:

HMI State   | What does this mean?
------------|------------------------------------------------------------
NONE        | The user has not been opened your app, or it has been Exited via the "Menu" button.
BACKGROUND  | The user has opened your app, but is currently in another part of the Head Unit. If you have a Media app, this means that another Media app has been selected.
LIMITED     | For Media apps, this means that a user has opened your app, but is in another part of the Head Unit.
FULL        | Your app is currently in focus on the screen.

!!! NOTE
Be careful with sending RPCs in the NONE and BACKGROUND states; some head units may limit the number of RPCs you may send in these states before blocking your requests. It is recommended that you wait until your app reaches HMI FULL to set up your app UI.
!!!

### Monitoring HMI Status
Monitoring HMI Status is provided through a required delegate callback of `SDLManagerDelegate`. The function `hmiLevel:didChangeToLevel:` will give you information relating the previous and new HMI levels your app is progressing through.

#### Objective-C
```objc
- (void)hmiLevel:(SDLHMILevel *)hmiLevel didChangeToNewLevel:(SDLHMILevel *)newLevel {
    <# code #>
}
```

#### Swift
```swift
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeToLevel newLevel: SDLHMILevel) {
    <# code #>
}
```

### More Detailed HMI Information
When an interaction occurs relating to your application, there is some additional pieces of information that can be observed that help figure out a more descriptive picture of what is going on with the Head Unit.

#### Audio Streaming State
The Audio Streaming State informs your app whether any currently streaming audio is audible to user (AUDIBLE) or not (NOT_AUDIBLE). A value of NOT_AUDIBLE means that either the application's audio will not be audible to the user, or that the application's audio should not be audible to the user (i.e. some other application on the mobile device may be streaming audio and the application's audio would be blended with that other audio).

You will see this come in for things such as Alert, PerformAudioPassThru, Speaks, etc.

Audio Streaming State   | What does this mean?
------------------------|------------------------------------------------------------
AUDIBLE     			| Any audio you are streaming will be audible to the user. 
ATTENUATED  			| Some kind of audio mixing is occuring between what you are streaming, if anything, and some system level sound. This can be visible is displaying an Alert with `playTone` set to `true`.
NOT_AUDIBLE 			| Your streaming audio is not audible. This could occur during a VRSESSSION System Context.

#### Objective-C
```objc
- (void)audioStreamingState:(nullable SDLAudioStreamingState)oldState didChangeToState:(SDLAudioStreamingState)newState {
    <# code #>
}
```

#### Swift
```swift
func audioStreamingState(_ oldState: SDLAudioStreamingState?, didChangeToState newState: SDLAudioStreamingState) {
    <# code #>
}
```

#### System Context
System Context informs your app if there is potentially a blocking HMI component while your app is still visible. An example of this would be if your application is open, and you display an Alert. Your app will receive a System Context of ALERT while it is presented on the screen, followed by MAIN when it is dismissed.

System Context State   | What does this mean?
-----------------------|------------------------------------------------------------
MAIN        		   | No user interaction is in progress that could be blocking your app's visibility.
VRSESSION  			   | Voice Recognition is currently in progress.
MENU     			   | A menu interaction is currently in-progress. 
HMI_OBSCURED    	   | The app's display HMI is being blocked by either a system or other app's overlay (another app's Alert, for instance).
ALERT 				   | An alert that you have sent is currently visible (Other apps will not receive this).

#### Objective-C
```objc
- (void)systemContext:(nullable SDLSystemContext)oldContext didChangeToContext:(SDLSystemContext)newContext {
    <# code #>
}
```

#### Swift
```swift
func systemContext(_ oldContext: SDLSystemContext?, didChangeToContext newContext: SDLSystemContext) {
    <# code #>
}
```