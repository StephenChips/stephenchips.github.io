---
title: SwiftUI：在App中使用相机
---

# 本文内容

本文将通过一个小例子，展示如何在App中使用相机拍摄照片，并展示在App主界面上。

# 步骤

首先新建一个Swift文件，然后写入以下代码：

```swift
import Foundation
import SwiftUI

// UIViewControllerRepresentable将一个UIViewController包装成一个SwiftUI视图
struct Camera: UIViewControllerRepresentable {
    typealias UIViewControllerType = UIImagePickerController
    
    private var delegate: Delegate
    
    static var isAvailable: Bool {
        UIImagePickerController.isSourceTypeAvailable(.camera)
    }
    
    init(onPhotoIsTaken: @escaping (UIImage?) -> Void) {
        self.delegate = Delegate(onPhotoIsTaken: onPhotoIsTaken)
    }
    
    // 创建一个UIImagePickerController，这个控制器能够控制原生相机App，实现拍照功能。
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let controller = UIImagePickerController()
        // 这里只能选 .camera，其他选项如.photoLibrary都已经弃用。如果需要在App中打开相册，
        // 则应该使用PhotosUI中的PHPickerViewController，而不是这里的UIImagePickerController.
        controller.sourceType = .camera 
        // 在这里将委托对象设置给控制器，注意这个引用是弱引用，所以直接`controller.delegate = Delegate(...)`
        // 是不行的。
        controller.delegate = delegate
        return controller
    }
    
    // 更新UIImagePickerController的函数
    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {
        // 创建之后我们不需要修改任何东西
    }
    
    // 我们需要一个委托对象（Delegate）来处理回调。
    private class Delegate: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        var onPhotoIsTaken: (UIImage?) -> Void
        
        fileprivate init(onPhotoIsTaken: @escaping (UIImage?) -> Void) {
            self.onPhotoIsTaken = onPhotoIsTaken
        }
        
        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            onPhotoIsTaken(nil)
        }
        
        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let editedImage = info[.editedImage] as? UIImage {
                onPhotoIsTaken(editedImage)
            } else if let originalImage = info[.originalImage] as? UIImage {
                onPhotoIsTaken(originalImage)
            } else {
                onPhotoIsTaken(nil)
            }
        }
    }
}
```

然后，在View Builder中添加`Camera`。注意，如果直接在View中添加`Camera`将在视图中，那么照相机应用就会直接嵌入到视图中。如果要实现“打开照相机”功能，则需要将`Camera`移至一个`sheet`或者`if..else`语句中，然后使用一个变量决定它是否显示：

```swift
struct ContentView: View {
    @ObservedObject var viewModel = CameraDemoViewModel()
    @State var showCamera = false
    
    var body: some View {
        VStack {
            if Camera.isAvailable {
                if let uiImage = viewModel.photoTakenFromCamera {
                    Image(uiImage: uiImage)
                        .resizable()
                }
                
                Button() {
                    showCamera = true
                } label: {
                    Label("Take a picture", systemImage: "camera.fill")
                }
            } else {
                Text("Camera is not available on this device.")
            }
        }
        .sheet(isPresented: $showCamera) {
            Camera() { photoJustTaken in
                viewModel.photoTakenFromCamera = photoJustTaken
                showCamera = false
            }
        }
    }
}

class CameraDemoViewModel: ObservableObject {
    @Published var photoTakenFromCamera: UIImage? = nil
}
```

然后，打开工程的Info，添加条目*Privacy - Camera Usage Description*，注明App需要使用照相机的原因。这是必须的，如果没有，当你尝试打开照相机时，程序就会抛出异常。

![Privacy: Camera Usage Description](/images/info-plist-privacy-camera-usage-description.png)


# 一些笔记

1. SwiftUI现在还没有一个View能够直接显示照相机，所以我们需要借助UIKit来实现这个功能。由于SwiftUI模块已经包含了UIKit，所以我们并不需要额外写`import UIKit`了。

2. 怎么在SwiftUI中引入UIKit？就要说到`UIViewControllerRepresentable`协议.这个协议将一个UIViewController包装成一个SwiftUI视图（适配器模式）。体现在这个例子中，就是`Camera`结构体通过实现`UIViewControllerRepresentable`协议，“包装”了`UIImagePickerController`，从而实现相机的功能。

3. 实现`UIViewControllerRepresentable`协议需要满足以下的要求：
    1. 指定被包装的UIViewController类型：`typealias UIViewControllerType = ...`
    2. 实现创建UIViewController的函数，既`makeUIViewController`
    3. 实现修改UIViewController的函数：`updateUIViewController`
    4. 根据情况，我们实现相应的委托对象：`private class Delegate ...`

4. `Image().resizable()`，让图片大小自适应。如果你想改变图片大小，也需要添加这个View Modifier。