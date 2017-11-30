## Configuring SDL Logging
SDL iOS v5.0 includes a powerful new built-in logging framework designed to make debugging easier. It provides many of the features common to other 3rd party logging frameworks for iOS, and can be used by your own app as well. We recommend that your app's integration with SDL provide logging using this framework rather than any other 3rd party framework your app may be using or NSLog. This will consolidate all SDL related logs in a common format and to common destinations.

### Configuring SDL Logging
SDL will configure its logging into a production-friendly configuration by default. If you wish to use a debug or a custom configuration, then you will have to specify this yourself. `SDLConfiguration` allows you to pass a `SDLLogConfiguration` with custom values. A few of these values will be covered in this section, the others are in their own sections below.

When setting up your `SDLConfiguration` you can pass a different log configuration:

###### Objective-C
```objc
SDLConfiguration* configuration = [SDLConfiguration configurationWithLifecycle:lifecycleConfiguration lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration debugConfiguration]];
```

###### Swift
```swift
let configuration = SDLConfiguration(lifecycle: lifecycleConfiguration, lockScreen: .enabled(), logging: .debug())
```

#### Format Type
Currently, SDL provides three output formats for logs (for example into the console or file log), these are "Simple", "Default", and "Detailed".

Simple:
```
09:52:07:324 🔹 (SDL)Protocol – I'm a log!
```

Default:
```
09:52:07:324 🔹 (SDL)Protocol:SDLV2ProtocolHeader:25 – I'm also a log!
```

Detailed:
```
09:52:07:324 🔹 DEBUG com.apple.main-thread:(SDL)Protocol:[SDLV2ProtocolHeader parse:]:74 – Me three!
```

#### Synchronicity
The configuration provides two properties, `asynchronous` and `errorsAsynchronous`. By default `asynchronous` is true and `errorsAsynchronous` is false. What this means is that any logs which are not logged at the error log level will be logged asynchronously on a separate serial queue, while those on the error log level will be logged synchronously on the separate queue (but the thread that logged it will be blocked until that log completes).

#### Log level
The `globalLogLevel` defines which logs will be logged to the target outputs. For example, if you set the log level to `debug`, all error, warning, and debug level logs will be logged, but verbose level logs will not be logged.

!!! NOTE
Although the `default` log level is defined in the SDLLogLevel enum, it should not be used as a global log level. See the [API documentation](https://smartdevicelink.com/en/docs/iOS/master/Enums/SDLLogLevel/) for more detail.
!!!

### Targets
Targets are the output locations where the log will appear. By default, in both default and debug configurations, only the Apple System Logger target (iOS 9 and below) or OSLog (iOS 10+) will be enabled.

#### SDLLogTargetAppleSystemLogger
The Apple System Logger target is the default log target in both default and debug configurations for devices running iOS 9 or older. This will log to the Xcode console and the devices Console.

#### SDLLogTargetOSLog
The OSLog target is the default log target in both default and debug configurations for devices running iOS 10 or newer. For more information on this logging system see [Apple's documentation](https://developer.apple.com/reference/os/logging). SDL's OSLog target will take advantage of subsystems and levels to allow you powerful runtime filtering capabilities through MacOS Sierra's Console app with a connected device.

#### SDLLogTargetFile
The File target allows you to log messages to a rolling set of files which will be stored on the device, specifically in the `Documents/smartdevicelink/log/` folder. The file names will be timestamped with the run start time.

#### Custom log targets
The protocol all log targets conform to, `SDLLogTarget`, is public. If you wish to make a custom log target in order to, for example, log to a server, it should be fairly easy to do so. If it can be used by other developers and is not specific to your app, then submit it back to the SmartDeviceLink iOS library project! If you want to add targets *in addition* to the default target that will output to the console:

###### Objective-C
```objc
logConfig.targets = [logConfig.targets setByAddingObjectsFromArray:@[[SDLLogTargetFile logger]]];
```

###### Swift
```swift
let _ = logConfig.targets.insert(SDLLogTargetFile())
```

### Modules
Modules are a concept of a set of files packaged together. A module is created using the `SDLLogFileModule` class and passed along with the configuration. This is used when outputting a log message. The log message may specify the module instead of the specific file name for clarity's sake. SDL will automatically add the modules corresponding to its own files after you submit your configuration. For your specific use case, you may wish to provide a module corresponding to your whole app's integration and simply name it with your app's name, or, you could split it up further if desired. To add modules to the configuration:

###### Objective-C
```objc
logConfig.modules = [logConfig.modules setByAddingObjectsFromArray:@[[SDLLogFileModule moduleWithName:@"Test" files:[NSSet setWithArray:@[@"File1", @"File2"]]]]];
```

###### Swift
```swift
logConfig.modules.insert(SDLLogFileModule(name: "Test", files: ["File1, File2"]))
```

### Filters
Filters are a compile-time concept of filtering in or out specific log messages based on a variety of possible factors. Call `SDLLogFilter` to easily set up one of the default filters or to create your own using a custom `SDLLogFilterBlock`. You can filter to only allow certain files or modules to log, only allow logs with a certain string contained in the message, or use regular expressions.

###### Objective-C
```objc
SDLLogFilter *filter = [SDLLogFilter filterByDisallowingString:@"Test" caseSensitive:NO];
```

###### Swift
```swift
let filter = SDLLogFilter(disallowingString: "Test", caseSensitive: false)
```
