## Touch Input

Navigation applications have support for touch events, including both single and multitouch events. This includes interactions such as panning and pinch. A developer may use the included `SDLTouchManager` class, or yourself by listening to the `SDLDidReceiveTouchEventNotification` notification.

### 1. Using `SDLTouchManager`

`SDLTouchManager` has multiple callbacks that will ease the implementation of touch events. 

!!! IMPORTANT
The view passed from the following callbacks are dependent on using the built-in focusable item manager to send haptic rects. See [supporting haptic input](Mobile Navigation/Supporting Haptic Input) "Automatic Focusable Rects" for more information.
!!!

The following callbacks are provided:

#### Objective-C
```objc
- (void)touchManager:(SDLTouchManager *)manager didReceiveSingleTapForView:(nullable UIView *)view atPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager didReceiveDoubleTapForView:(nullable UIView *)view atPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager panningDidStartInView:(nullable UIView *)view atPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager didReceivePanningFromPoint:(CGPoint)fromPoint toPoint:(CGPoint)toPoint;
- (void)touchManager:(SDLTouchManager *)manager panningDidEndInView:(nullable UIView *)view atPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager panningCanceledAtPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager pinchDidStartInView:(nullable UIView *)view atCenterPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager didReceivePinchAtCenterPoint:(CGPoint)point withScale:(CGFloat)scale;
- (void)touchManager:(SDLTouchManager *)manager didReceivePinchInView:(nullable UIView *)view atCenterPoint:(CGPoint)point withScale:(CGFloat)scale;
- (void)touchManager:(SDLTouchManager *)manager pinchDidEndInView:(nullable UIView *)view atCenterPoint:(CGPoint)point;
- (void)touchManager:(SDLTouchManager *)manager pinchCanceledAtCenterPoint:(CGPoint)point;
```

#### Swift
```swift
    optional public func touchManager(_ manager: SDLTouchManager!, didReceiveSingleTapForView view: UIView?, atPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, didReceiveDoubleTapForView view: UIView?, atPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, panningDidStartInView view: UIView?, atPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, didReceivePanningFromPoint fromPoint: Any!, toPoint: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, panningDidEndInView view: UIView?, atPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, panningCanceledAtPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, pinchDidStartInView view: UIView?, atCenterPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, didReceivePinchAtCenterPoint point: CGPoint, withScale scale: CGFloat)
    optional public func touchManager(_ manager: SDLTouchManager!, didReceivePinchInView view: UIView?, atCenterPoint point: CGPoint, withScale scale: CGFloat)
    optional public func touchManager(_ manager: SDLTouchManager!, pinchDidEndInView view: UIView?, atCenterPoint point: CGPoint)
    optional public func touchManager(_ manager: SDLTouchManager!, pinchCanceledAtCenterPoint point: CGPoint)
```

!!! note
Points that are provided via these callbacks are in the head unit's coordinate space. This is likely to correspond to your own streaming coordinate space. You can retrieve the head unit dimensions from `SDLStreamingMediaManager.screenSize`.
!!!

### 2. Self Implementation of `onTouchEvent`

If apps want to have access to the raw touch data, the `SDLDidReceiveTouchEventNotification` notification can be evaluated. This callback will be fired for every touch of the user and contains the following data:

#### Type

BEGIN
: Sent for the first touch event.

MOVE
: Sent if the touch moved.

END
: Sent when the touch is lifted.

CANCEL
: Sent when the touch is canceled (for example, if a dialog appeared over the touchable screen while the touch was in progress).

#### Event

touchEventId
: Unique ID of the touch. Increases for multiple touches (0, 1, 2, ...).

timeStamp
: Timestamp of the head unit time. Can be used to compare time passed between touches.

coord
: X and Y coordinates in the head unit coordinate system. (0, 0) is the top left.

#### Example

#### Objective-C
```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(touchEventAvailable:) name:SDLDidReceiveTouchEventNotification object:nil];

- (void)touchEventAvailable:(SDLRPCNotificationNotification *)notification {
    if (![notification.notification isKindOfClass:SDLOnTouchEvent.class]) {
      return;
    }
    SDLOnTouchEvent *touchEvent = (SDLOnTouchEvent *)notification.notification;

    // Grab something like type
    SDLTouchType* type = touchEvent.type;
}

```

#### Swift
```swift
// To Register
NotificationCenter.default.addObserver(self, selector: #selector(touchEventAvailable(_:)), name: .SDLDidReceiveTouchEvent, object: nil)

// On Receive
@objc private func touchEventAvailable(_ notification: SDLRPCNotificationNotification) {
    guard let touchEvent = notification.notification as? SDLOnTouchEvent else {
        print("Error retrieving onTouchEvent object")
        return
    }

    // Grab something like type
    let type = touchEvent.type
}
```