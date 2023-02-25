---
title: SwiftUI：Gesture
---

There are two kinds of gestures: **discrete gesture** and **continuous gesture**. A discrete gesture fires and finishes instantly. There isn't any intermediate state involved. A classic example is single tap. On the contrary, **continuous gesture** lasts for an amount of time. Many common gestures belong to continuous gestures, for example, `MaginificationGesture` and `DragGesture`. In this post, I will show you how to add these gestures into your app, and how to combine two gesture to form a combined gesture.

## SwiftUI built-in gestures

Following texts are from the apple's documentation: [https://developer.apple.com/documentation/swiftui/gestures](https://developer.apple.com/documentation/swiftui/gestures).

> ```swift
> struct TapGesture
> ```

> A gesture that recognizes one or more taps.

> ```swift
> struct SpatialTapGesture
> ```
> A gesture that recognizes one or more taps and reports their location.

> ```swift
> struct LongPressGesture
> ```
> A gesture that succeeds when the user performs a long press.

> ```swift
> struct DragGesture
> ```
> A dragging motion that invokes an action as the drag-event sequence changes.

> ```swift
> struct MagnificationGesture
> ```
> A gesture that recognizes a magnification motion and tracks the amount of magnification.

> ```swift
> struct RotationGesture
> ```
> A gesture that recognizes a rotation motion and tracks the angle of the rotation.


# Discrete Gesture

Among all distrete gestures the most common and simplest one is tapping. It is so commonly-used that Apple defines a handy method on the `View` class - `onTapGesture`. In fact, discrete gesture is very simple, so I will show to how to use `TapGesture`, and you will probably know how other discrete gestures should be used.

To create a `TapGesture`, the simplest way is to appends a `onTapGesture` methods on the view where tapping happens. For example, hypothetically there is a canvas app, and I want to informed users when the canvas is tapped, then following code will suffice:

```swift
struct Canvas: View {
    var body: some View {
        ZStack {
            // Where the canvas elements are placed.
        }
        .onTapGesture {
            print("You have just tapped the canvas")
        }
    }
}
```

Click the *run* button at the top-left corner of XCode, tap inside `ZStack`, and checkout the console. The word *You have just tapped the canvas* should be shown on it.

# Custom Discrete Gesture

Sometimes you have a auxiliary requirement on tapping. for example you want to double tap the text. To do this, you can create `TapGesture` class directly with the `count` argument set to `2`, set the `onEnded`'s callback, pass it to the `gesture` method of the target view, then it's OK. Here is an example:

```swift
struct Canvas: View {
    var body: some View {
        Text("hello, world").gesture(gestureDoubleTapToResizeCanvas())
    }

    private func gestureDoubleTapToResizeCanvas() -> some Gesture {
        TapGesture(count: 2).onEnded {
            print("tap twice")
        }
    }
}
```

Other discrete gestures are as simple as this. You customize it by creating an instance and specify its argument and callbacks.

# Continuous Gesture

Gesture like `DraggingGesture` and `MagnificationGesture` are typically continous gesture. They runs for an amount of time and response touch events from users. There are two methods on gestures that can help use to customize a continuous gesture, which are `onChange(_ action:)` and `updating(_ action:)`.

## `onChange(_ action:)`

On change it the simplier one. It will give you a gesture's newest changed information and some assisted information. Take `DragGesture` as the example, it will give you the newest finger location to you, and the the location where you first pressed down your finger and start dragging.

## `@GestureState` and `updating(_ action):`

In practise updating is more commonly-used. Because we need to store some additional and temporally states for a gesture to make it work, so we will use `@GestureState` in combination with `updating(_ action:)`. `@GestureState` is designed for recording these temporally state and `updating(_ action:)` is the proper way to change a `@GestureState` (we cannot directly change a `@GestureState` property by assigning a value to it).

In the following example we define a gesture `gestureDragImage` to drag an image on the canvas that is loaded from [picsum.photos](https://https://picsum.photos/id/237/536/354), and we add a state (`changingOffset`) to remember the offset of the image relative to the position before it is dragged.


```swift
import Combine
import SwiftUI

class ViewModel: ObservableObject {
    @Published var uiImage: UIImage? = nil
    
    private var imageFetchingAnyCancellable: AnyCancellable? = nil
    
    init() {
        if let imageURL = URL(string: "https://picsum.photos/id/237/536/354") {
            imageFetchingAnyCancellable = URLSession.shared.dataTaskPublisher(for: imageURL)
                .map { (data, response) in UIImage(data: data) }
                .replaceError(with: nil)
                .assign(to: \Self.uiImage, on: self)
        }
    }
}

struct ContentView: View {
    @StateObject var viewModel = ViewModel()
    
    @State var offset: (x: CGFloat, y: CGFloat) = (x: 0, y: 0)
    
    var offsetWithDragging: (x: CGFloat, y: CGFloat) {
        (offset.x + changingOffset.x, offset.y + changingOffset.y)
    }
    
    @GestureState var changingOffset: (x: CGFloat, y: CGFloat) = (x: 0, y: 0)
    
    var body: some View {
        if let image = viewModel.uiImage {
            Image(uiImage: image)
                .position(x: offsetWithDragging.x, y: offsetWithDragging.y)
                .gesture(gestureDragImage())
        }
    }
    
    private func gestureDragImage() -> some Gesture {
        DragGesture(minimumDistance: 0)
            .updating($changingOffset) { newOffset, inOutChangingOffset, _ in
                inOutChangingOffset = (
                    x: newOffset.location.x - newOffset.startLocation.x,
                    y: newOffset.location.y - newOffset.startLocation.y
                )
            }
            .onEnded { finalPosition in
                offset = (x: offset.x + finalPosition.location.x - finalPosition.startLocation.x,
                          y: offset.y + finalPosition.location.y - finalPosition.startLocation.y)
            }
    }
}
```

The property wrapper `@GestureState`, is used for recording intermediate state while a gesture is performing. In the example above, it records the image's dragging offset.

There are three important points that you should know when using `@GestureState`:

First is that, assigning to a `@GestureState` is to set the state's initial value, the value when the gesture isn't performing. You cannot change its current value while gesture is performing by assigning (i.e. via `=` operator).

Second, the only way to update a `@GestureState` is to assign a value to the second argument of `updating`'s callback. Specifically for the example above, we have to set the `inOutChangeOffset` in updating's callback to update the gesture state `changingOffset`'s value.

```swift
theView.updating($changingOffset) { newOffset, inOutChangingOffset, _ in
    inOutChangingOffset = (
        x: newOffset.location.x - newOffset.startLocation.x,
        y: newOffset.location.y - newOffset.startLocation.y
    )
}
```

Third, when the gesture is finished, it will automatically reset itself to the initial value. This happens before callback `.onEnded(_)` is called. So there is no way to know the last value before gesture ends.

# `onStart(_ action:)`?

There isn't such function on gesture, though I think it is very useful and hope Apple can add it in the future. Currently I cannot know when a gesture starts without hacks. If there is a `onStart(_ action:)` method, in some situation, it can make the code simpler.

# Composing Gestures

## Adding multiple `.gesture(_)`s on a view

Except for `simultanouslyGesture(_)`, multiple gestures are performed one by one, each of which will wait and listen to the incomming low-level touch event for a little amount of time (roughly tens of milliseconds). If a gestures's triggering criteria isn't met, for example, a `DoubleTapGesture` requires double taps but user have just tapped once, the next gesture will be performed. If it does met, depends on the way you combine gestures, the underlying touch event will be either be consumed or passed through to the next gesture. Every gesture has a priority, a.k.a percedence. A gesture with higher priority will be performed first. Gestures tha attaches to the view first (and its sub-view) will have higher priority than the one that attaches later. You can use `.highPriorityGesture(_)` to "break" this rule: the new-added gesture will have higher priority than all gestures that have defined on the view (and its sub-view). Here is a example that demonstrate this rule:

```swift
struct ContentView: View {
    var body: some View {
        ZStack {
            Text("Hello, world")
                .gesture(gestureInSubView())
        }
        .gesture(gestureOne())
        .gesture(gestureTwo())
        .highPriorityGesture(gestureTree())
    }

    private func gestureInSubView() -> some Gesture { /* ... */ }
    private func gestureOne() -> some Gesture { /* ... */ }
    private func gestureTwo() -> some Gesture { /* ... */ }
    private func gestureThree() -> some Gesture { /* ... */ }
}
```

If all gestures are triggered, `gestureThree` will performs first, followed by `gestureInSubView`, followed by `gestureOne`, then `gestureTwo` at last. (to be confirmed). In practise, it is very rare to add more than two gestures to a view, and it will be very unlikely to successfully trigger all these four gestures. Mostly only the gesture with highest priority runs, and others will not response.

Why? because for a chained gesture on a view, after it finishes, it will consume the touch events it received. As the result, the next gesture will not be performed because it cannot get the events. Let's make another little example. In this example, there is a `gestureDoubleTap` and a `gestureSingleTap`, and we will rearrange in different ways to demostrate the idea above. First lets define these gestures in a dummy view `DemoView`:

```swift
struct DemoView: View {
    var body: some View {
        Text("hello, world")
            .gesture(gestureDoubleTap())
            .gesture(gestureSingleTap())
    }
    
    func gestureSingleTap() -> some Gesture {
        TapGesture().onEnded { print("tap once") }
    }
    
    func gestureDoubleTap() -> some Gesture {
        TapGesture(count: 2).onEnded { print("tap twice") }
    }
}
```

For the first trial, I put the `gestureDoubleTap` before `gestureSingleTap`. Therefore `gestureDoubleTap` has a higher priority than `gestureSingleTap`, and should be performed first when touch events come. Let's try out the demo and check the result.

![GIF that I tap the text once, twice, three times in a row respectively]()

![GIF that what console prints at each time]()

It turns out that, when I tap the text once, the console prints `tap once`, but rather doing so instantly, it delays for a while. When I tap twice, the console prints `tap twice` immediately. And when I tap three times, the console print `tap twice` and `tap once` sequentially but promptly.

This confirms that my assumption above is correct. `tap twice` always shows before `tap once`, proving that gesture with high priority does perform first. The reason why `tap once` doesn't get printed immediately, is because `gestureDoubleTap` is waiting for another touch event, yet failed to come, so this event passes through to the second gesture, namely `gestureSingleTap`. When I tap three times in a row, `gestureTapTwice` performs sucessfully and consumes two tapping, leaving the last tapping for the `gestureSingleTap`, which just right suffice it to trigger the callback, print `tap once` on the console.

If we swap these two's order, let `gestureSingleTap` preceed `gestureDoubleTap`, then run the app. Why will happen? Well, you may be suprised. `gestureDoubleTap` will never trigger! No matter how many times you tap the text, what outputs on the console is always `tap once`. The reason is actually simple, and also evidence that the assumption is right - every single tap is consumed by `gestureSingleTap`. Under the hooks, there is no remain touch events for `gestureDoubleTap`.

To make it more convincing, I will makes auxiliary example later.

We can say that gesture cannot response immediately sometimes, because it has to wait for the previous failing gesture to finish, which leads to a unpleasant user experiences. In my opinion, this is a limitation of SwiftUI's gesture system. If you encounter it, try finding alternaitve solution and see if you can get around it.

## `simultanously(with:)`, `exclusively(before:)` and `sequenced(before:)` on `Gesture`

There are three methods define on a `Gesture` that can combine another `Gesture` to form a composed gesture, which are `simultanously`, `exclusively` and `sequentially`. Their names are quite self-explantory, and they all generates a new gestures that handles two gestures in following manner, respectively.

1. `simultanously(with:)`: Performs two gestures at the same time.
2. `exclusively(before:)`: Only perform one of the two gestures. Try the first one first, if failed, try the later one, otherwise the later one won't be performed.
3. `sequenced(before:)`: The later gesture will only happens if the first one successfully finished.

And here is the explanation from the apple documentation ([original link]()): 

> ```swift
> func exclusively<Other>(before: Other) -> ExclusiveGesture<Self, Other>
> ``` 
> Combines two gestures exclusively to create a new gesture where only one gesture succeeds, giving precedence to the first gesture.

> ```swift
> func sequenced<Other>(before: Other) -> SequenceGesture<Self, Other>
> ```
> Sequences a gesture with another one to create a new gesture, which results in the second gesture only receiving events after the first gesture succeeds.
> 
> ```swift
> func simultaneously<Other>(with: Other) -> SimultaneousGesture<Self, Other>
> ```
> Combines a gesture with another gesture to create a new gesture that recognizes both gestures at the same time.


`simultanously(with:)`、`exclusively(before:)` and `sequenced(before:)` are three functions that generate combined gestures. They are not gestures themself. But for simplicity, in the following post their names refer to the gesture they generate too. Please distinguish them by context. And I will named the gestures that attach to views via method `gesture(_)` as *chained gesture*, mainly for separate them from gestures that are generated from three combining functions just mentioned above.

As the documentation says, `exclusively(before:)` will create a new gesture that only one gesture success, and the first one preceeds the second one, which means that it has higher priority and SwiftUI will try running it first. It has some kinds of similarity to chained gestures, those that attaches directly on the view: it will try two gesture one by one. But unlike chianed gestures, if the first one succeed, it stops, and the second gesture will not get performed, whereas when SwiftUI finsihes running a *chained gestures*, it will find the next one on the view.

The `simultanously(with:)` is special, same as `View`'s `simultanousGesture(_)`, which is a sugaric syntax to the former one. It will try to recognizing two gestures at the same time, and will performs gestures once it receives enough touch events (and of cause, all of them should meet the triggering criteria). Moreover, the touch events are accumulated and shared for both events, instead of throwing them away like what *chained gestures* does. Since they are recognized at the same time, two gestures of a simultanous gesture has a same priority.

For the last one, `sequenced(before:)`, will compose two gesture in a way that the second gesture will only get run if the first one succeed. It is kind of like the opposite of `exclusive(before:)`, and it is similar with the *chained gesture* in that they are run one by one. The biggest difference between gesture created from`sequenced(before:)` and *chained gesture* is that, the first gesture of `sequence(before:)` will preserve touch events for the second one, while *chained gesture* will not. 

Following examples reveals the differences, hope they can help you to understand the concept furthermore. First is a `simultanously(with:)` example:

```swift
struct DemoView: View {
    var body: some View {
        Text("hello, world")
            .gesture(gestureDoubleTap().simultaneously(with: gestureSingleTap()))
    }

    // gestureDoubleTap and gestureSingleTap has been defined above.
}
```

When I first tap the text, the `gestureSingleTap` will perform, and the underlying events get reserved. Wwhen I tap again, there will be two touch events in total, thus causing `gestureDoubleTap` to perform. The result will be same if I swap these two gesture's order, that is `gestureSingleTap().simultaneously(with:gestureDoubleTap())`.

```swift
struct DemoView: View {
    var body: some View {
        Text("hello, world")
            .gesture(gestureDoubleTap().exclusively(before: gestureSingleTap()))
    }

    // gestureDoubleTap and gestureSingleTap has been defined above.
}
```

If I change `simultaneously(with:)` to `exclusively(before:)`, the result will be quite different. When I only tap once, `tap once` will be printed, but with some delay. That is because the first gesture that SwiftUI tried to recognize is `gestureDoubleTap` and it will persist waiting for the second tap about a hundred of miliseconds. When it gave up finally, it will decide to recognize the second gesture `gestureSingleTap`, which is just able to performs. If we swap the order, that is `gestureSingleTap().exclusively(before:)`, then no matter how fast and how many times I tap, only `gestureSingleTap` will performs, printing `tap once` on the screen. That is because every single tap can trigger `gestureSingleTap` successfully, and from the definition we know that only one gesture of two will be performed.

```swift
struct DemoView: View {
    var body: some View {
        Text("hello, world")
            .gesture(gestureDoubleTap().sequenced(before: gestureSingleTap()))
    }

    // gestureDoubleTap and gestureSingleTap has been defined above.
}
```

The last kind of gesture is `sequence(before:)`. Its example is shown above. The result is kind of interesting. After the first tap nothing is printed out in the console, that is expected since `gestureDoubleTap`, which must be recognized before `gestureSingleTap`, needs two taps to performs. However subsequently when I tap again, not only `tap twice` is printed, but also `tap once` (right after `tap twice`). Why does this happen? Well, because `sequence(before:)` gesture accumulates touch events. Hence, when it finishes, unlike *chained gestures*, which consumes two touch events, it keeps them so that the `gestureSingleTap` can receive them. Obviously `gestureSingleTap` will perform because it only need one tap. You may ask why shouldn't print `tap once` twice, since two accumulated tap can let `gestureSingleTap` performs twice. That is a gesture is a one-time thing. It gets enough events, then it performs. When it's done, it's done, really. No more event can make it do more thing. If you combine one more `gestureSingleTap`, forming:

```swift
gestureDoubleTap()
    .sequenced(before: gestureSingleTap())
    .sequenced(before: gestureSingleTap())
```

That will result in one `tap twice` followed by two `tap once` after double tap.

# limitations

In my opinion SwiftUI's gesture API is suitable for many situation, but not all - just like a bag of all-purpose flour isn't quite suitable for making cake. If the application you build is gesture-rich, for instance a professional drawing app like *procrate*, soon you'll reach the edge of its capability. For example, I tried to write a gesture that can drag, rotate and pinch an image in the same time, but I didn't success, none of three combined gesture can help. For this situation I think I should consider using UIKit or event SpriteKit, but that will be more complicated. Maybe as new WWDC is held more new features will come, and the gesture API will become more capable, Well, let's see.

