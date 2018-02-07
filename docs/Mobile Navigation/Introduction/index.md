## Mobile Navigation

Mobile Navigation allows map partners to bring their applications into the car and display their maps and turn by turn easily for the user. This feature has a different behavior on the head unit than normal applications. The main differences are:

* Navigation Apps don't use base screen templates. Their main view is the video stream sent from the device.
* Navigation Apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands.
* Navigation Apps can receive touch events from the video stream.

### Connecting an app

The basic connection is similar for all apps. Please follow [Getting Started > Integration Basics](Getting Started/Integration Basics) for more information.

The first difference for a navigation app is the `appHMIType` of `SDLAppHMITypeNavigation` that has to be set in the `SDLLifecycleConfiguration`. Navigation apps are also non-media apps.

The second difference is that a `SDLStreamingMediaConfiguration` must be created and passed to the `SDLConfiguration`. A property called `securityManagers` must be set if connecting to a version of Core that requires secure video & audio streaming. This property requires an array of classes of Security Managers, which will conform to the `SDLSecurityType` protocol. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.

#### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfig = [SDLLifecycleConfiguration defaultConfigurationWithAppName:@"<#App Name#>" appId:@"<#App Id#>"];
lifecycleConfig.appType = SDLAppHMITypeNavigation;

SDLStreamingMediaConfiguration *streamingConfig = [SDLStreamingMediaConfiguration secureConfigurationWithSecurityManagers:@[OEMSecurityManager.class]];

SDLConfiguration *config = [SDLConfiguration configurationWithLifecycle:lifecycleConfig lockScreen:nil logging:nil streamingMedia:streamingConfig];
```

#### Swift
```swift
let lifecycleConfig = SDLLifecycleConfiguration(appName: "<#App Name#>", appId: "<#App Id#>")
lifecycleConfig.appType = .navigation

let streamingConfig = SDLStreamingMediaConfiguration(securityManagers: [OEMSecurityManager.self])
let config = SDLConfiguration(lifecycle: lifecycleConfig, lockScreen: nil, logging: nil, streamingMedia: streamingConfig)
```

!!! IMPORTANT
When compiling, you must make sure to include all possible OEM's security managers that you wish to support.
!!!