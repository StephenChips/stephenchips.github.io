---
title: SwiftUI：把对象保存成JSON
---

# 涉及知识点

1. 如何使用`Codable`编码、解码对象；
2. 如何使用`JSONEncoder`将`Codable`对象变成`Data`对象；
3. 如何将`Data`对象保存到文件系统中；
4. 如何从`URL`（文件路径或网络地址）加载`Codable`对象；
5. 四个编译器自动生成代码的`protocol`。

# 文章内容

这篇文章以`Shape`对象为例子，说明怎么讲对象保存成`JSON`。`Shape`对象定义如下：

```swift
struct Shape {
    var color: ShapeColor
    var type: ShapeType
    var isPicked: Bool
}

enum ShapeColor {
    case red, green, blue, none
}

enum ShapeType {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
    case square(edgeLength: Double)
}
```

`Shape`对象只是为了这篇文章而专门构造的，并不来自某段程序，但你可以把它想成画板工具的一个数据模型。

# `Codable`、`Decodable`和`Encodable`协议

要想让`Shape`变成JSON，以及想从`JSON`中构建`Shape`，我们需要先让`Shape`对象遵从`Codable`协议。该协议的定义如下：

```swift
typealias Codable = Decodable & Encodable
```

我们可以看看`Encodable`和`Decodale`分别是怎么定义的：

```swift
/// A type that can encode itself to an external representation.
public protocol Encodable {

    /// Encodes this value into the given encoder.
    ///
    /// If the value fails to encode anything, `encoder` will encode an empty
    /// keyed container in its place.
    ///
    /// This function throws an error if any values are invalid for the given
    /// encoder's format.
    ///
    /// - Parameter encoder: The encoder to write data to.
    func encode(to encoder: Encoder) throws
}

/// A type that can decode itself from an external representation.
public protocol Decodable {

    /// Creates a new instance by decoding from the given decoder.
    ///
    /// This initializer throws an error if reading from the decoder fails, or
    /// if the data read is corrupted or otherwise invalid.
    ///
    /// - Parameter decoder: The decoder to read data from.
    init(from decoder: Decoder) throws
}
```

只要对象中所有的属性都遵守`Codable`协议，那么Swift编译器就会自动帮你生成这两个函数，所以除非你有特殊要求，我们并不需要编写这两个函数。Swift中所有常见的类型，如`Int`、`String`、`URL`、数组、字典都遵从`Codable`协议。对于`enum`，只要枚举值的参数都是`Codable`的，或者所有枚举值都没有参数，那么我们也不需要自己编写这两个函数。我们的`ShapeColor`和`ShapeType`都满足这样的条件，所以我们直接加上`Codable`协议即可：

```swift
struct Shape: Codable {
    var color: ShapeColor
    var type: ShapeType
    var isPicked: Bool
}

enum ShapeColor: Codable {
    case red, green, blue, none
}

enum ShapeType: Codable {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
    case square(edgeLength: Double)
}
```

顺便说说，在Swift中有四个特殊的协议，如果满足条件，那么Swift编译器就会自动生成默认实现，用户无需自己定义。

1. `Encodable`：自动生成`func encode(to encoder: Encoder) throws`；
2. `Decodable`：自动生成`init(from decoder: Decoder) throws`；
3. `Equatable`：自动生成`static func ==(a: Type, b: Type)`，其中`Type`是对象类型；
4. `Hashable`： 自动生成`var hashValue: Int { get }`。

你可以从这里看到更详细的规则：[https://github.com/apple/swift-evolution/blob/main/proposals/0185-synthesize-equatable-hashable.md](https://github.com/apple/swift-evolution/blob/main/proposals/0185-synthesize-equatable-hashable.md)

关于怎么使用`Codable`，可以阅读这篇文章：[https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)

# 控制输出JSON的内容

通过自己编写`func encode(to encoder: Encoder) throws`和`init(from decoder: Decoder)`，我们就能控制输出JSON的内容和格式。比如，我们规定`Shape`输出成JSON时，需要满足以下几点要求：

1. `Shape.isPicked`不应该出现在JSON中；
2. 如果`Shape.color = none`，那么color字段也不应该出现在JSON中。

那么，我可以通过自己定义`encode(to encoder:)`与`init(from decoder:)`，满足上面的要求。

```swift
struct Shape: Codable {
    var color: ShapeColor
    var type: ShapeType
    var isPicked: Bool
    
    enum CodingKeys: String, CodingKey {
        case color, type
    }
    
    init(color: ShapeColor, type: ShapeType, isPicked: Bool = false) {
        self.color = color
        self.type = type
        self.isPicked = isPicked
    }
    
    init(from decoder: Decoder) throws {
        var container = try decoder.container(keyedBy: CodingKeys.self)
        
        let color = try container.decodeIfPresent(ShapeColor.self, forKey: .color)
        self.color = color ?? .none
        try self.type = container.decode(ShapeType.self, forKey: .type)
        
        self.isPicked = false
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        
        if case .none = color {} else {
            try container.encode(color, forKey: .color)
        }
        
        try container.encode(type, forKey: .type)
    }

}
```

# 将`Codable`对象变成JSON文件

要将`Codable`对象变成变成JSON文件，需要用到`JSONEncoder`。顾名思义，它就是用来将`Encodable`对象编码成JSON的。它的用法很简单：创建一个`JSONEncoder`对象，调用其`write`方法，然后就可以得到一个包含JSON数据的`Data`对象了。注意在使用前需要`import Foundation`。

```swift
import Foundation

let shape = Shape(color: .red, type: .circle(radius: 5))
let data = try JSONEncoder().encode(shape)
```

# `Data`对象

我们可以将`Data`对象变成字符串打印出来，这样我们就得到`Shape`对象编码成JSON的样子。

```swift
let jsonString = String(data: data, encoding: .utf8)
pirnt(jsonString)

// in the console: {"type":{"circle":{"radius":5}}}
```

当然，我们也可以将`Data`存到文件系统中，要做到这一步，只需一步：

```swift
try data.write(theURLWhereTheFileIsStored)
```

# 获取保存的文件路径

通常我们会把文件存放到特定的目录下，比如说“documentDirectory”，在iOS下它是“文件”App的本地根目录，在macOS下是用户的“文稿”目录。下面函数可以用来获取到“documentDirectory”的URL：

```swift
func getDocumentDirectoryURL() -> URL? {
    FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first
}
```

然后我们可以使用`URL.appending`方法，把文件名`filename`加上去，以获得完整的保存路径：

```swift
let theURLWhereTheFileIsStored = getDocumentDirectoryURL()?.appending(path: filename)
```

有了完整的保存路径，我们只需要调用`data.write`，就成功将`Shape`对象存到文件系统中了。我们可以整理整理上边的代码，然后在`Shape`对象上添加一个公开的对象方法。调用它，就能保存`Shape`对象。

```swift
struct Shape: Codable {
    // ...

    func saveToFile(filename: String) throws {
        guard let documentDirectory = getDocumentDirectoryURL() else { return }
        let theURLWhereTheFileIsStored = documentDirectory.appending(path: filename)
        let data = try JSONEncoder().encode(self)
        try data.write(to: theURLWhereTheFileIsStored)
    }

    private func getDocumentDirectoryURL() -> URL? {
        FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first
    }
}
```

# 从文件中读取`Shape`

我们可以写一个构造器，让`Shape`从一个`URL`中读取初始化数据。这样我们就能够从文件中读取并初始化`Shape`了。（不仅仅如此，由于输入的是URL，所以我们也能够从网络下载数据，并使用这些数据初始化`Shape`！）

```swift
struct Shape {
    // ...

    init(fromURL url: URL) throws {
        let data = try Data(contentsOf: url)
        let shape = try JSONDecoder().decode(Shape.self, from: data)
    }
}
```

# 完整例子


```swift
import Foundation

struct Shape: Codable {
    var color: ShapeColor
    var type: ShapeType
    var isPicked: Bool
    
    enum CodingKeys: String, CodingKey {
        case color, type
    }
    
    init(color: ShapeColor, type: ShapeType, isPicked: Bool = false) {
        self.color = color
        self.type = type
        self.isPicked = isPicked
    }
    
    init(from decoder: Decoder) throws {
        var container = try decoder.container(keyedBy: CodingKeys.self)
        
        let color = try container.decodeIfPresent(ShapeColor.self, forKey: .color)
        self.color = color ?? .none
        try self.type = container.decode(ShapeType.self, forKey: .type)
        
        self.isPicked = false
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        
        if case .none = color {} else {
            try container.encode(color, forKey: .color)
        }
        
        try container.encode(type, forKey: .type)
    }

    init(fromURL url: URL) throws {
        let data = try Data(contentsOf: url)
        let shape = try JSONDecoder().decode(Shape.self, from: data)
    }

    func saveToFile(filename: String) throws {
        guard let documentDirectory = getDocumentDirectoryURL() else { return }
        let theURLWhereTheFileIsStored = documentDirectory.appending(path: filename)
        let data = try JSONEncoder().encode(self)
        try data.write(to: theURLWhereTheFileIsStored)
    }

    private func getDocumentDirectoryURL() -> URL? {
        FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first
    }
}

enum ShapeColor: Codable {
    case red, green, blue, none
}

enum ShapeType: Codable {
    case circle(radius: Double)
    case rectangle(width: Double, height: Double)
    case square(edgeLength: Double)
}
```
