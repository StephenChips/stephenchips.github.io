---
title: 写一个基于文档的App
tags: SwiftUI
---

# 概述

这篇文章源自苹果的一片官方教程：[https://developer.apple.com/documentation/swiftui/building_a_document-based_app_with_swiftui](https://developer.apple.com/documentation/swiftui/building_a_document-based_app_with_swiftui)。

在这篇教程中，我们将编写一个基于文档的清单（Checklist）应用。用户可以创建清单，然后往清单添加条目，选中条目或删除条目。在下文我们讲使用[DocumentGroup](https://developer.apple.com/documentation/swiftui/documentgroup)创建基于文本的应用，实现[ReferenceFileDocument](https://developer.apple.com/documentation/swiftui/referencefiledocument)协议，以及回撤、重做的功能。

完成之后，我们的应用将拥有一个和iOS"文件"应用差不多的界面。在这个界面上，你能新建清单，打开清单，将清单保存到本地或iCloud上。

（图片）

# 创建应用的数据模型

一个代办清单由一系列清单条目组成，一个代办条目有两种状态：已完成和未完成。因此，这个应用的数据模型由两个`struct`组成：`ChecklistItem`和`Checklist`，分别代表清单条目与清单本身。这两个`struct`遵循了`Codable`协议，以方便序列化；另外还遵循了`Identifiable`协议，这样每个对象都会有一个为一个标识，以方便枚举。

```swift
struct ChecklistItem: Identifiable, Codable {
    var id = UUID()
    var isChecked = false
    var title: String
}

struct Checklist: Identifiable, Codable {
    var id = UUID()
    var items: [ChecklistItem]
}
```

# 定义应用的文档类型（UTType）

我们还需要为我们的清单文档定一个新的文档类型，这样设备才会知道使用我们这个App打开清单文档。首先，我们需要在Information Property List中

# 使用DocumentGroup Scene

一个基于文档的程序，其入口的`body`属性会返回一个`DocumentGroup`。我们需要使用其构造器`init(newDocument:editor)`来创建`DocumentGroup`, 它有两个参数：
1. `newDocument`：为一个函数，用于创建文档的数据模型。返回的数据模型必须满足`FileDocument`或者`ReferenceFileDocument`协议。
2. `editor`：也是一个函数，用于创建文档视图。

```swift
@main
struct DocumentBasedApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: { ChecklistDocument() }) { configuration in
            ChecklistView()
        }
    }
}
```

# 实现ReferenceFileDocument协议

# 实现回撤和重做

