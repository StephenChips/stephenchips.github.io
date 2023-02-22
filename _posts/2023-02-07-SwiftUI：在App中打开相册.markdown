---
title: SwiftUI：在App中使用相册
---

# 本文内容

本文将通过一个小例子，展示如何在App中打开相册选择图片，将它展示在App主界面上。

# 步骤

首先新建一个Swift文件，然后写入以下代码：

```swift
//
//  PhotoLibary.swift
//  EmojiArt
//
//  Created by 黄栋材 on 2023/2/5.
//

import Foundation
import SwiftUI
import PhotosUI

struct PhotoLibrary: UIViewControllerRepresentable {
    typealias UIViewControllerType = PHPickerViewController
    
    private var delegate: PHPickerViewControllerDelegate
    
    init(handlePickedImage: @escaping (UIImage?) -> Void) {
        delegate = Delegate(handlePickedImage: handlePickedImage)
    }
    
    class Delegate: PHPickerViewControllerDelegate {
        private var handlePickedImage: (UIImage?) -> Void
        
        init(handlePickedImage: @escaping (UIImage?) -> Void) {
            self.handlePickedImage = handlePickedImage
        }
        
        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            guard let pickedPhotoItemProvider = results.first?.itemProvider,
                    pickedPhotoItemProvider.canLoadObject(ofClass: UIImage.self)
            else { return handlePickedImage(nil) }
            
            pickedPhotoItemProvider.loadObject(ofClass: UIImage.self) { [] photo, error in
                // 注意loadObject会在后台线程异步执行，而更新图片属于UI操作，所以我们需要将相关代码包在
                // `DispatchQueue.main.async`中，不然虽然图片会设置成功，但是在应用退出之后就会消失，还原会旧图片。
                DispatchQueue.main.async {
                    self.handlePickedImage(photo as? UIImage)
                }
            }
        }
    }
    
    func makeUIViewController(context: Context) -> PHPickerViewController {
        var config = PHPickerConfiguration(photoLibrary: .shared())
        config.filter = PHPickerFilter.images
        
        let controller = PHPickerViewController(configuration: config)
        controller.delegate = delegate
        
        return controller
    }
    
    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {
        // do nothing
    }
}
```

然后，我们就能在View Builder中调用`PhotoLibary`打开相册选择图片了。

注意，如果直接将`PhotoLibary`添加到视图中，那么它就会直接嵌入到视图中。如果要实现“打开相册”功能，则需要将`PhotoLibary`写在一个`sheet`或者`if..else`语句中，然后使用一个变量决定它是否显示：

```swift
struct ContentView: View {
    @ObservedObject var viewModel = PhotoLibaryViewModel()
    @State var showPhotoLibary = false
    
    var body: some View {
        VStack {
            if let uiImage = viewModel.handlePickedImage {
                Image(uiImage: uiImage).resizable()
            }
            
            Button() {
                showPhotoLibary = true
            } label: {
                Label("Open photo libary", systemImage: "photo.on.rectangle")
            }
        }
        .sheet(isPresented: $showPhotoLibary) {
            PhotoLibary() { photoJustTaken in
                viewModel.handlePickedImage = photoJustTaken
                showPhotoLibary = false
            }
        }
    }
}

class PhotoLibaryViewModel: ObservableObject {
    @Published var handlePickedImage: UIImage? = nil
}
```

# 一些笔记

1. SwiftUI现在还没有一个View能够直接显示照相机，所以我们需要借助UIKit来实现这个功能。由于SwiftUI模块已经包含了UIKit，所以我们并不需要额外写`import UIKit`了。

2. 怎么在SwiftUI中引入UIKit？就要说到`UIViewControllerRepresentable`协议.这个协议将一个UIViewController包装成一个SwiftUI视图（适配器模式）。体现在这个例子中，就是`PhotoLibary`结构体通过实现`UIViewControllerRepresentable`协议，“包装”了`PHPickerViewController`，从而实现相机的功能。

3. 实现`UIViewControllerRepresentable`协议需要满足以下的要求：
    1. 指定被包装的UIViewController类型：`typealias UIViewControllerType = ...`
    2. 实现创建UIViewController的函数，既`makeUIViewController`
    3. 实现修改UIViewController的函数：`updateUIViewController`
    4. 根据情况，我们实现相应的委托对象：`private class Delegate ...`

4. `Image().resizable()`，让图片大小自适应。如果你想改变图片大小，也需要添加这个View Modifier。
    