# Gesture

There are two kinds of gestures: **Discrete Gesture** and **Continuous Gesture**. A discrete finishes instantly after it starts. There isn't intermediate state involved. One of the most classic example is tapping. Other discrete gestures include *double tapping* and *long pressing*. Of course, there are more. On the contrary, **continuous gesture** lasts for an amount of time. Many common gestures are continuous gesture, for example, zooming (`MaginificationGesture`), panning (`DragGesture`) and rotating (`RotationGesture`). In this article, I will show you have to add these gestures into your app, and how to combine multiple gesture together, letting them happens simultaneously, exclusively, or sequentially (a.k.a, one by one).

# Discrete Gesture

Among all distrete gestures the most common and simplest one must be the tapping. It is so commonly-used that Apple defines a handy method on the `View` class, letting you to add tapping to your view quickly - `onTapGesture`. In fact, using discrete gesture is quite simple, so I will use `TapGesture` in this section to demonstrate all discrete gesture's usage, instead of showing all SwiftUI built-in gestures boringly.

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

Sometimes you have a auxiliary requirement on tapping. for example you want the app to automatically scale the canvas to a reasonably size by double tapping. To do this, you can create `TapGesture` class directly with the `count` argument be set to `2`, and sets the callback to method `onEnded` (fires when a double tap finishes), then pass it to the `gesture` method of the target view. To make the code more tidy, put the creation code in a dedicate method.

```swift
struct Canvas: View {
    var body: some View {
        ZStack {
            // Where the canvas elements are placed.
        }
        .gesture(gestureDoubleTapToResizeCanvas())
    }

    private func gestureDoubleTapToResizeCanvas() -> some Gesture {
        TapGesture(count: 2).onEnded {
            // codes that will resize the canvas to an ideal size
        }
    }
}
```

That is it. That is how you customize a built-in SwiftUI gesture - just create an instance by yourself and specify its arguments and callback functions. Nothing could be more complicated than this.

# Continuous Gesture

Here I will use 

## `onChange`

## `@GestureState` and `updating`

As a example, let's write the `gestureDragCanvas` that can pan the canvas.

```swift

```

# Composing Gestures

## `simultanously(with:)`, `exclusively(before:)` and `sequenced(before:)` on `Gesture`

There are three methods define on a `Gesture` that can combine another `Gesture` to form a composed gesture, which are `simultanously`, `exclusively` and `sequentially`. Their names are quite self-explantory, and they all generates a new gestures that handles two gestures in following manner, respectively.

1. `simultanously(with:)`: Performs two gestures at the same time.
2. `exclusively(before:)`: Only perform one of the two gestures. Try the first one first, if failed, try the later one, otherwise the later one won't be performed.
3. `sequenced(before:)`: The later gesture will only happens if the first one successfully finished.

Here is a simple example that combines `gestureDragCanvas` and `gestureDoubleTapToResizeCanvas` together (defined above)

```swift

```

After the app runs, you can either drag the canvas, or double tap the cnavas to resize it to a reasonable size (where all elements can be display).

For the first though, it makes more sense to use `exclusively(before:)` instead of `simultanously(with:)`, since these two gesture do not really happens in the same time. But actually no, like I just said, `exclusively(before:)` will try the first gesture, and the second gesture will only be performed if the first one failed.

## `simultanousGesture()` on `View`

## Avoiding Conflicts

If the gesture delays after you've dragged/tapped the screen, probably because SwiftUI is waiting for another higher-priority gesture to finish.

Should the oddity of three combined gestures

Example single tap and double tap together

Except for simultanously gesture, multiple gestures are performed one by one, each of which will wait and listen to the incomming low-level touch event for a little amount of time (roughly tens of milliseconds). If a gestures's triggering criteria isn't met, for example, a `DoubleTapGesture` requires double taps but user have just tapped once, the next gesture will be performed. If it does met, depends on the way you combine gestures, the underlying touch event will be either be consumed or passed through to the next gesture. Every gesture has a priority, a.k.a percedence. A gesture with higher priority will be performed first. For a view, Gestures that have been added to the view (and its sub-view) will have higher priority than the one that have just added with the modifier `.gesture(_)`. Of cause, if you use `.highPriorityGesture(_)` modifier, the situation will be inverted: the new-added gesture will have higher priority than all gestures that have defined on the view (and its sub-view). Here is a example that demonstrate this rule:

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

If all gestures are triggered, `gestureThree` will performs first, followed by `gestureInSubView`, followed by `gestureOne`, then `gestureTwo` at last. (to be confirmed). But in practise, it is very rare to adds more than two gestures onto a view, and it will be very unlikely to successfully trigger four events at a time. Mostly only the highest-priority-gesture will runs, leaving others no response.

Why? because for a chained gesture on a view, after it finishes, it will consume the touch events it received. As the result, the next gesture will not be performed because it cannot get those events and hence cannot reach the triggering requirement. Let's make another little example to shows this. In this example, there is a `gestureDoubleTap` and a `gestureSingleTap`, and we will rearrange in different ways to demostrate the idea above. First lets define these gestures in a dummy view `DemoView`:

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

The triplet on the `Gesture`, i.e. `simultanously(with:)`, `exclusively(before:)` and `sequenced(before:)` can combine two gestures together to form a compound gesture. Since there are two gestures involves, it got to have a rule about how to performs two gestures. First, let's quote from Apple's documentation, and have a brief looks on their original explanation ([original link]()): 

> ```swift
> func exclusively<Other>(before: Other) -> ExclusiveGesture<Self, Other>
> ``` 
> Combines two gestures exclusively to create a new gesture where only one gesture succeeds, giving precedence to the first gesture.
> 
> ```swift
> func sequenced<Other>(before: Other) -> SequenceGesture<Self, Other>
> ```
> Sequences a gesture with another one to create a new gesture, which results in the second gesture only receiving events after the first gesture succeeds.
>
> ```swift
> func simultaneously<Other>(with: Other) -> SimultaneousGesture<Self, Other>
> ```
> Combines a gesture with another gesture to create a new gesture that recognizes both gestures at the same time.

From the documetnation we know that `simultanously(with:)`、`exclusively(before:)` and `sequenced(before:)` are functions generate a combined gesture. They are not gestures themself. But for simplicity, in the following post these names refer to the combined gesture it generates too. Please distinguish them by context. And I will named the gestures that attach to a view as *chained gesture*, mainly for separate them from gestures inside and generated from three combining functions just mentioned above.

As the documentation says, `exclusively(before:)` will create a new gesture that only one gesture success, and the first one preceeds the second one, which means that it has higher priority and SwiftUI will try running it first. It has some kinds of similarity to chained gestures, those that attaches directly on the view: it will try two gesture one by one. But unlike chianed gestures, if the first one succeed, it stops, and the second gesture will not get performed, whereas when SwiftUI finsihes running a *chained gestures*, it will find the next one on the view.

`simultanously(with:)` is special, same as `View`'s `simultanousGesture(_)`, which can be consider a sugaric synax to the former one. It will try two gestures at the same time, as long as the touch events is able to trigger the gesture, the gesture's callback will run. Moreover, the touch events is accumulate and is shared for both events. The event will not be discarded after one gesture is triggers. Since they are recognized at the same time, two gestures share same priority.

For the last one, `sequenced(before:)`, will compose two gesture in a way that the second gesture will only get run if the first one succeed. It is kind of like the opposite of `exclusive(before:)`, and it is similar with the *chained gesture* in that they are run one by one. The biggest difference between gesture created from`sequenced(before:)` and *chained gesture* is that, the first gesture of `sequence(before:)` will preserve touch events for the second one, while *chained gesture* will not. 

Following examples revealsthe differences, hope they can help you to understand the concept furthermore:

```swift
// sequenced

// exclusively

// simultanously

```

# The limitations

