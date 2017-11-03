## Keyboard Input
Keyboard input is available via the `SDLPerformInteraction` RPC. For a general understanding of how this RPC works, reference the [Displaying Information > Menus](Displaying Information/Menus). As opposed to the normal process for using `SDLPerformInteraction` with a required `SDLCreateInteractionChoiceSet`, using the keyboard requires no interaction choice sets to be created beforehand. It does, however, require an empty array to be passed in. To show the perform interaction as a keyboard, we modify the `interactionLayout` property to be `KEYBOARD`. Note that while the vehicle is in motion, keyboard input will be unavailable (resulting in a grayed out keyboard).

#### Objective-C
```objc
SDLPerformInteraction* performInteraction = [[SDLPerformInteraction alloc] init];
performInteraction.initialText = @"Find Location";
performInteraction.interactionChoiceSetIDList = @[];
performInteraction.timeout = @(100000);
performInteraction.interactionMode = SDLInteractionModeManualOnly;
performInteraction.interactionLayout = SDLLayoutModeKeyboard;
[self.sdlManager sendRequest:performInteraction withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (![response.resultCode isEqualToEnum:SDLResultSuccess] || ![response isKindOfClass:SDLPerformInteractionResponse.class]) {
        NSLog(@"Error sending perform interaction.");
        return;
    }

    SDLPerformInteractionResponse* performInteractionResponse = (SDLPerformInteractionResponse*)response;
    if (performInteractionResponse.manualTextEntry.length != 0) {
        // text entered
    }
}];
```

#### Swift
```swift
let performInteraction = SDLPerformInteraction()
performInteraction.initialText = "Find Location"
performInteraction.interactionChoiceSetIDList = []
performInteraction.timeout = 100000
performInteraction.interactionMode = .manualOnly
performInteraction.interactionLayout = .keyboard
sdlManager.send(performInteraction) { (request, response, error) in
    if response?.resultCode.isEqual(to: .success) == false {
        print("Error sending perform interaction.")
        return
    }

    guard let performInteractionResponse = response as? SDLPerformInteractionResponse, let textEntry = performInteractionResponse.manualTextEntry else {
        return
    }

    // text entered
}
```

!!! note
In Ford's current SYNC 3 implementation of SmartDeviceLink, there is a bug resulting in the need for an interaction choice array to be set in the RPC call.
!!!