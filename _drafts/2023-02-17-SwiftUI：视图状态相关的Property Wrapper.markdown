---
title: SwiftUI：视图状态相关的Property Wrapper
---

# `@State`

`@State`用于定义视图的私有状态，只能用在值类型上，如`Int`、`Bool`、`String`、`struct`对象。如下边的计数器例子，就使用了`@State`来定义计数器的变量`counter`。

```swift
class MyView: View {
    @State counter: Int = 0

    var body: some View {
        Text("count: \(counter)")
        Button("Plus One") {
            counter += 1
        }
    }
}
```

如果将`@State`用在`class`对象上边，那么当对象更新后，视图将不会更新。这是`class`是引用类型，当我修改对象属性后，对象引用并没有发生变化，所以在SwiftUI看来，这个对象并没有发生改变；而如果是`struct`，那么在修改后，SwiftUI其实是暗地里重新生成了一个对象，然后再赋值给`@State`包裹的对象，如此一来，SwiftUI就能够观察到变化了。 

因为只能用在值类型上，所以使用是会有一些限制。首先你不能把它用在一个可选类型上，如下边的代码会导致编译错误：

```swift
class MyClass: View {
    @State content: String?
}
```

其次，因为是值类型，所以当你把状态传给子视图后，子视图得到的是该状态的一份拷贝，而非两个指向相同状态的引用。当子视图修改传入的状态，它其实是修改原状态的拷贝，而原状态并不会被修改。如下代码，点击`ChildView`中按钮后，`ParentView`中`Text(store.count)`并不会改变。

```swift
import SwiftUI

struct ParentView: View {
    @State var state = CounterState(count: 0)
    
    var body: some View {
        Text(String(state.count))
        ChildView(state: state)
    }
}

struct ChildView: View {
    @State var state: CounterState

    var body: some View {
        Button("plus one") { state.count += 1 }
    }
}

class CounterState: ObservableObject {
    @Published var count: Int
    
    init(count: Int) {
        self.count = count
    }
}
```

那么正确的方法是什么呢，那就是使用`@Observedbject`或`@StateObject`

# `@ObservedObject`

只需要让`CounterState`继承`ObservableObject`协议，然后把`@State`替换成`@ObservedObject`，并将`CounterState.count`属性标记成`@Published`，视图就能够正确更新了。修改后的代码如下：

```swift
import SwiftUI

struct ParentView: View {
    @ObservedObject var state = CounterState(count: 0)
    
    var body: some View {
        Text(String(state.count))
        ChildView(state: state)
    }
}

struct ChildView: View {
    @ObservedObject var state: CounterState

    var body: some View {
        Button("plus one") { state.count += 1 }
    }
}

class CounterState: ObservableObject {
    @Published var count: Int
    
    init(count: Int) {
        self.count = count
    }
}
```

# `@StateObject`

> @ObservedObject和@StateObject的区别，这篇文章讲得不错：
> [https://onevcat.com/2020/06/stateobject/](https://onevcat.com/2020/06/stateobject/)
>
> 另外StackOverflow上也有一篇答案说明了两者的不同：
> [https://stackoverflow.com/questions/62544115/what-is-the-difference-between-observedobject-and-stateobject-in-swiftui](https://stackoverflow.com/questions/62544115/what-is-the-difference-between-observedobject-and-stateobject-in-swiftui)

简单来说，`@ObservedObject`会随着视图重建而重置，而`@State`和`@StateObject`则不会——他们只会在视图新建的时候创建一次，然后在视图的整个生命周期中复用。下面这个例子就展现两者的区别，当我点击`ParentView`的`Button("toggle font color")`时，`ChildView`中的`state.count`就会归零。这是因为修改`isRed`导致整个`ParentView`失效，然后SwiftUI就会刷新`ParentView`的`body`，生产新的 `ChildView`以及其中的`@ObservedObject var state`。

要解决这个问题，只需要将`ObservedObject`改成`@StateObject`即可。

```swift
import SwiftUI
  
struct ParentView: View {
    @State var isRed: Bool = false

    var body: some View {
        VStack {
            Button("toggle font color") { isRed = !isRed }
                .buttonStyle(.bordered)
            ChildView()
        }
    }
}

struct ChildView: View {
    @ObservedObject var state = CounterState(count: 0)

    var body: some View {
        VStack {
            Text("count : \(state.count)")
            Button("plus one") { state.count += 1 }
                .buttonStyle(.bordered)
        }
    }
}

class CounterState: ObservableObject {
    @Published var count: Int
    
    init(count: Int) {
        self.count = count
    }
}
```

# `@Binding`

使用`@Binding`最典型的例子就是`TextField`：

```swift
class MyView: View {
    @State var state = ""

    var body: some View {
        TextField("name", $state)
    }
}
```

# `@Environment`

向子视图注入环境变量

# `@EnvironmentObject`

向子视图注入`ObservableObject`

# 自定义Property Wrapper

