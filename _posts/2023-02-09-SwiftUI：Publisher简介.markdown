---
title: SwiftUI：Publisher
tag: SwiftUI
---

# 本文内容

本文将介绍SwiftUI中的Publisher。

# 什么是Publisher

Publisher对象能够发布数据。作为一个协议，它的的签名是: `Publisher<Output, Failure>`，其中`Output`是Publisher发布的数据的类型，而`Failure`是Publisher抛出的异常的类型。

# 有哪些Publisher

Publisher在SwiftUI中可谓无处不在，可以说它是SwiftUI的一个核心机制。在一个`ObservableObject`对象中，每个标注了`@Published`的属性，都会有一个对应的Publisher，其名称为：`$原属性名`。比如下边的一个`ObservableObject`

```swift
class MyViewModel: ObservableObject {
    @Published var foo: Int
}
```

其属性`self.foo`对应的Publisher，就是`self.$foo`。

其他的一些常见Publisher包括：

1. `URLSession`的`dataTaskPublisher`：发布从一个URL获取到的数据
2. `Timer`的`publish(every:)`：定期发布当前日期和时间，类型为`Date`
3. `NotificationCenter`的`publisher(for:)`：订阅系统事件，在收到一个系统事件后发布通知。

# 订阅Publisher

要订阅Publisher，只需要调用Publisher的公有方法：`sink(receiveCompletion:receiveValue:)`。该方法的两个参数均是闭包，当Publisher产生了异常或关闭了，就会触发`receiveComplection`，而当Publisher发布了新的数据，就会触发`receiveValue`。

如果这个Publisher不会抛出异常（类型为`Publisher<Output, Never>`，也就是说`Failure = Never`），那么你就可以用`sink(receiveValue:)`重载方法，省去注册`receiveCompletion`函数。如果你在一个会抛出异常的函数的Publisher中使用该方法，会导致编译错误。

下面是一个小例子，它定义了一个视图，它能从某个URL下载图片，并将该图片展示出来。

```swift
class URLImageView: View {
    private var imageFetchCancleable: AnyCancelable?
    
    var body: some View {
        Image(from: UIImage(data: data))
    }

    init(from url: String) {
        downloadFrom(url: url)
    }

    func downloadDataFrom(url: String): Data {
        URLSession.shared.dataTaskPublisher(for: url)
            .sink(receiveCompletion: { result in

            }, receiveValue: { [weak self] data in
                self.data = data
            })
    }
}
```

# 其他常用方法

1. `assign(to:on)`
4. `replaceError(with:)`
5.  `map(_ transform:)`
6. `mapError(_ transform:)`
7. `receive(on:)`：决定订阅函数在哪个任务队列执行，默认是后台队列（Background Queue）。如果你想让Publisher别的任务队列，就需要用到这个方法。`receive(on:Dispatch.main)`就指定在主队列中，通常是需要在订阅函数中更新UI是会这么做。


# 取消侦听

`sink`函数会返回一个`AnyCancelable`对象，我们可以调用其中的`cancel()`方法来手动取消侦听：

```swift
let cancellable = publisher.sink({ value in ... })
cancellable.cancel()
```

不过，我们不必总是这么做，因为`AnyCancellable`对象会在其销毁时，会自动帮我们调用`cancel()`，取消侦听Publisher（[doc](https://developer.apple.com/documentation/combine/anycancellable)）。

我们可以利用这一个特点，将`AnyCancellable`定义为对象属性，这样只要对象还存活，那么这个对象就可以获取Publisher的最新数据：

```swift
class Program {
    let cancellable: AnyCancellable?

    func listenChanges() {
        cancellable = createPublisherFoo()
            .sink { error in
                // 处理错误
            } receiveValue: { value in
                // 接收更新
            }
    }
}
```

# `onReceive`

如果`AnyCancellable`赋值给函数的一个局部变量，那么当离开变量的作用域后，侦听随即停止。这是一个坑：当我们调用完`sink`函数就结束了，随即`AnyCancellable`取消订阅Publisher，导致`sink`里边的代码永远不会被执行。如果里边代码是用来更新试视图的，那么视图将不会更新。

```swift
class Program {
    func listenChanges() {
        let cancellable = createPublisherFoo()
            .sink { error in
                // 这个函数永远不会执行！
            } receiveValue: { value in
                // 这个函数也永远不会执行！
            }
        // 出了这个函数，订阅就没了！
    }
}
```

# View订阅Publisher

```swift
someView.onReceive { thingsPublisherPublishes in
    // 实现与thingsPublisherPublishes相关的代码
}
```

当回调执行后，将会更新视图。

# 相关资料

1. [Medium: Efficient resource handling — resource acquisition is initialization (RAII) in Swift](https://medium.com/@redhotbits/resource-acquisition-is-initialization-raii-in-swift-62f856063a74)
2. [Swift By Sundell: Managing self and cancellable references when using Combine](https://www.swiftbysundell.com/articles/combine-self-cancellable-memory-management/)
3. [苹果开发文档：Publisher](https://developer.apple.com/documentation/combine/publisher)
4. [苹果开发文档：receive(on:options:)](https://developer.apple.com/documentation/combine/publisher/receive(on:options:))
5. [苹果开发文档：AnyCanellable](https://developer.apple.com/documentation/combine/anycancellable)
6. [苹果开发文档：Receiving and Handling Events with Combine](https://developer.apple.com/documentation/combine/receiving-and-handling-events-with-combine)

