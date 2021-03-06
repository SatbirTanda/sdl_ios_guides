## Text, Images, and Buttons
All text, images, and soft buttons on the HMI screen must be sent as part of a `SDLShow` RPC. Subscribing hard buttons are sent as part of a `SDLSubscribeButton` RPC.

### Text
A maximum of four lines of text can be set in `SDLShow` RPC, however, some templates may only support 1, 2, or 3 lines of text. If all four lines of text are set in the `SDLShow` RPC, but the template only supports three lines of text, then the fourth line will simply be ignored.

To determine the number of lines available on a given head unit, check the `RegisterAppInterfaceResponse.displayCapabilities.textFields` for `mainField(1-4)`.

#### Objective-C
```objc
SDLShow* show = [[SDLShow alloc] initWithMainField1:@"<Line 1 of Text#>" mainField2:@"<Line 2 of Text#>" mainField3:@"<Line 3 of Text#>" mainField4:@"<Line 4 of Text#>" alignment:SDLTextAlignmentCentered];
self.sdlManager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      // The text has been set successfully
    }
}];
```

#### Swift
```swift
let show = SDLShow(mainField1: "<#Line 1 of Text#>", mainField2: "<#Line 2 of Text#>", mainField3: "<#Line 3 of Text#>", mainField4: "<#Line 4 of Text#>", alignment: .centered)
sdlManager.send(show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode.isEqual(to: .success) {
        // The text has been set successfully
    }
}
```

### Images
The position and size of images on the screen is determined by the currently set template. All images must first be uploaded to the remote system using the SDLManager’s [file manager](Uploading Files and Graphics) before being used in a `SDLShow` RPC. Once the image has been successfully uploaded, the app will be notified in the upload method's completion handler. For information relating to how to upload images, go to the [Uploading Files and Graphics](Uploading Files and Graphics) section.

!!! NOTE
Some head units you may be connected to may not support images at all. Please consult the `RegisterAppInterfaceResponse.displayCapabilities.graphicSupported` [documentation](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLDisplayCapabilities/).
!!!

#### Show the Image on a Head Unit
Once the image has been uploaded to the head unit, you can show the image on the user interface. To send the image, create a `SDLImage`, and send it using the `SDLShow` RPC. The `value` property should be set to the same string used in the `name` property of the uploaded `SDLArtwork`. Since you are uploading your own image, the `imageType` property should be set to `dynamic`.

#### Objective-C
```objc
SDLImage* image = [[SDLImage alloc] initWithName:@"<#Uploaded as Name#>" ofType:SDLImageTypeDynamic];

SDLShow* show = [[SDLShow alloc] init];
show.graphic = image;

self.sdlManager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      // The text has been set successfully
    }
}];
```

#### Swift
```swift
let sdlImage = SDLImage(name: "<#Uploaded As Name", of: .dynamic())

let show = SDLShow()
show.graphic = image

sdlManager.send(show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode.isEqual(to: .success) {
      // Success
    }
}
```

### Soft & Subscribe Buttons
Buttons on the HMI screen are referred to as soft buttons to distinguish them from hard buttons, which are physical buttons on the head unit. Don’t confuse soft buttons with subscribe buttons, which are buttons that can detect user selection on hard buttons (or built-in soft buttons).

#### Soft Buttons
Soft buttons can be created with text, images or both text and images. The location, size, and number of soft buttons visible on the screen depends on the template. If the button has an image, remember to upload the image first to the head unit before setting the image in the `SDLSoftButton` instance.

!!! IMPORTANT
If a button type is set to `image` or `both` but no image is stored on the head unit, the button might not show up on the HMI screen.
!!!

#### Objective-C
```objc
SDLSoftButton* softButton = [[SDLSoftButton alloc] init];

// Button Id
softButton.softButtonID = @(<#Unique Button Id#>);

// Button handler - This is called when user presses the button
softButton.handler = ^(SDLOnButtonPress *_Nullable buttonPress,  SDLOnButtonEvent *_Nullable buttonEvent) {
    if (buttonPress != nil) {
        if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeShort]) {
            // Short button press
        } else if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeLong]) {
            // Long button press
        }
    } else if (buttonEvent != nil) {
        if ([buttonEvent.buttonEventMode isEqualToEnum:SDLButtonEventModeButtonUp]) {
            // Button up
        } else if ([buttonEvent.buttonEventMode isEqualToEnum:SDLButtonEventModeButtonDown]) {
            // Button down
        }
    }
};

// Button type can be text, image, or both text and image
softButton.type = SDLSoftButtonTypeBoth;

// Button text
softButton.text = @"<#Button Text#>";

// Button image
softButton.image = [[SDLImage alloc] initWithName:@"<#Save As Name#>" ofType:SDLImageTypeDynamic];

SDLShow *show = [[SDLShow alloc] init];

// The buttons are set as part of an array
show.softButtons = @[softButton];

// Send the request
[self.sdlManager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      // The button was created successfully
    }
}];
```

#### Swift
```swift
let softButton = SDLSoftButton()

// Button Id
softButton.softButtonID = <#Unique Button Id#>

// Button handler - This is called when user presses the button
softButton.handler = { (press, event) in
    if let buttonPress = press {
        if buttonPress.buttonPressMode == .short {
            // Short button press
        } else if buttonPress.buttonPressMode == .long {
            // Long button press
        }
    } else if let buttonEvent = event {
        if onButtonEvent.buttonEventMode == .buttonUp {
            // Button up
        } else if onButtonEvent.buttonEventMode == .buttonDown {
            // Button down
        }
    }
}

// Button type can be text, image, or both text and image
softButton.type = .both

// Button text
softButton.text = "<#Button Text#>"

// Button image
softButton.image = SDLImage(name: "<#Save As Name#>", of: .dynamic)

let show = SDLShow()

// The buttons are set as part of an array
show.softButtons = [softButton]

// Send the request
sdlManager.send(show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode.isEqual(to: .success) {
        // The button was created successfully
    }
}
```

#### Subscribe Buttons
Subscribe buttons are used to detect changes to hard buttons. You can subscribe to the following hard buttons:

| Button  | Template | Button Type |
| ------------- | ------------- | ------------- |
| Ok (play/pause) | media template only | soft button and hard button |
| Seek left | media template only | soft button and hard button |
| Seek right | media template only | soft button and hard button |
| Tune up | media template only | hard button |
| Tune down | media template only | hard button |
| Preset 0-9 | any template | hard button |
| Search | any template |hard button |
| Custom | any template | hard button |

Audio buttons like the OK (i.e. the `play/pause` button), seek left, seek right, tune up, and tune down buttons can only be used with a media template. The OK, seek left, and seek right buttons will also show up on the screen as a soft button in a predefined location dictated by the media template on touchscreens. The app will be notified when the user selects the subscribe button on the screen or when the user manipulates the corresponding hard button.

!!! NOTE
You will automatically be assigned the media template if you set your configuration app type as `MEDIA`.
!!!
