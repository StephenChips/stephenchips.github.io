# 在项目中添加Core Data

首先，新建一个数据模型文件（文件 -> File -> New -> File... -> Core Data -> Data Model），并在里边定义应用的数据模型。

然后，在项目中创建代码文件`Persistence.swift`，并填入下面代码：

```swift
import CoreData

struct PersistenceController {
    // 这个类的单例
    static let shared = PersistenceController()

    // 创建一个内存数据库用于预览App
    static var preview: PersistenceController = {
        let previewController = PersistenceController(inMemory: true)
        // 在这里插入一些假数据
        return previewController
    }()

    // NSPersistenContainer代表了一个Core Data数据库
    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "CoreDataDemo")
        if inMemory {
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.

                /*
                 Typical reasons for an error here include:
                 * The parent directory does not exist, cannot be created, or disallows writing.
                 * The persistent store is not accessible, due to permissions or data protection when the device is locked.
                 * The device is out of space.
                 * The store could not be migrated to the current model version.
                 Check the error message to determine what the actual problem was.
                 */
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        container.viewContext.automaticallyMergesChangesFromParent = true
    }
}
```

然后，我们需要将CoreData的上下文对象`PersistenceController.shared.container.viewContext`添加到根视图的环境变量中：

```swift
@main
struct YourAwesomeApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(
                    \.managedObjectContext,
                    PersistenceController.shared.container.viewContext)
        }
    }
}
```

然后，在需要使用Core Data的视图中，添加属性：`@Environment(\.managedObjectContext) var viewContext`，就可以获取到这个对象，并使用它来操作数据库。

```swift
struct YourView: View {
    @Environment(\.managedObjectContext) private var viewContext
}
```

# 使用FetchRequest获取数据

示例代码：

```swift
struct YourView: View {
    // 获取笔记本对象（Notebook），结果集按照用户定义的次序（userOrder字段）升序排列
    @FetchRequest(sortDescriptors: [SortDescriptor(\Notebook.userOrder, order: .forward)])
    private var notebooks: FetchedResults<Notebook>

    var body: some View {
        NavigationView {
            List {
                ForEach(notebooks) { notebook in
                    NavigationLink(notebook.name ?? "") {
                        NotebookView(notebook: notebook)
                    }
                }
            }
        }
    }
}
```

1. `sortDescriptors:`参数表明了结果集的先后次序，而`order:`决定了结果集是升序还是降序排列。结果集的类型为`FetchResults<CoreDataModelType>`，其中的泛型参数为应该是`NSManagedObject`的子类，表示一个定义在数据模型中的实体。
2. Always declare properties that have a fetch request wrapper as private. This lets the compiler help you avoid accidentally setting the property from the memberwise initializer of the enclosing view.
3. 带`@FetchRequest`同时是一个`ObservableObject`。在结果集变更后，`@FetchRequest`会自动获取新数据并更新视图。
3. 因为需要监听数据库的变动，所以`@FetchRequest`内部会用到Core Data上下文对象，所以在使用`@FetchRequest`之前，需要确保Core Data上下文对象已经设置到环境变量`.managedObjectContext`中。

`FetchResult`对象与普通数组别无二致，能够遍历，支持随机访问。

# 为什么CoreData对象字段都是可选的

Swift编程语言中，如果一个字段不是可选的，那么它永远不会为`nil`。这一点对于Core Data对象来说太过于严苛了，他只需要在保存的时候（调用`viewContext.save()`时），确保所有非可选字段都有值就行，其他时候有没有值对它并不重要。

你可以把Core Data对象理解成临时手稿，而不是真实的数据库数据。所有的修改都会先写到这份手稿上，只有当你调用`viewContext.save()`，他才会将这些修改更新到数据库中。

# Core Data对象都是`ObservableObject`

根据[这里](https://developer.apple.com/forums/thread/121897)，所有`NSManagedObject`都是`ObservableObject`和`Publisher`，所以可以这么做：

```swift
struct NotebookView: View {
    @Environment(\.editMode) var editMode
    @Environment(\.managedObjectContext) var viewContext
    
    @ObservedObject var notebook: Notebook

    var body: some View {
        if editMode.wrappedValue.isEditing {
            TextField("name", text: $notebook.name)
            Button("Save") { viewContext.save() }
        } else {
            Text(notebook.name)
        }
    }
}
```

（待验证）

参考资料：

[https://developer.apple.com/videos/play/wwdc2019/230](https://developer.apple.com/videos/play/wwdc2019/230)

# 新建Core Data对象

下面例子展示了如何创建笔记本。

注意新建CoreData对象并不意味着往数据库插入数据，你需要调用`viewContext.save()`才能将新建的对象存到数据库中

```swift
struct NotebookView: View {
    @Environment(\.managedObjectContext) var viewContext

    func createNotebook(name: String) {
        let notebook = Notebook(context: viewContext)
        notebook.name = "New notebook"
        try? viewContext.save()
    }
}
```

# 修改Core Data对象

```swift
struct NotebookView: View {
    @Environment(\.managedObjectContext) var viewContext

    func changeNotebookName(notebook: Notebook, newName: String) {
        notebook.name = newName
        try? viewContext.save()
    }
}
```

# 设置Core Data一对一关系（1:1 Relationship）

```swift
struct NoteDetailView: View {
    @Environment(\.managedObjectContext) var viewContext
    @ObservedObject var note: Note

    func changeNoteCategory(note: Note, newCategoryName: String) {
        let category = Category(context: viewContext)
        category.name = newCategoryName
        note.category = category
    }
}
```

如果Category不存在，那么它会先被创建。

# 获取、添加、删除Core Data一对多关系

通过强制类型转换，将`NSSet`转换成`Set`或者`Array`，然后再遍历；


# Core Data一对多关系

## 获取

一对多关系CoreData使用`NSSet`表示，你可以通过以下强制转换，将`NSSet`变成标准的`Set`或者`Array`，然后就能够愉快使用了。

```swift
let setOfObj = nsSet as! Set<Obj>
let arrOfObj = nsSet.allObjects as! [Obj]
```

## 添加、删除

使用生成的`removeFromXXX`和`addToXXX`来添加、删除一对多关系。其中`XXX`为关系名称。比如`note`实体存在一对多关系`tag`，那么在`Note`对象上面就有`removeFromTag(tag:Tag)`和`addToTag(tag:Tag)`，专门用来往`note`添加、删除tag。

添加：

```swift
note.addToTag(tag)
```

删除：

```swift
note.removeFromTag(tag)
```

## 一个完整的例子

```swift
struct NoteView: View {
    @Environment(\.managedObjectContext) var viewContext
    @ObservedObject var note: Note
    @State var newTagName: String = ""

    var noteTag: [Tag] {
        note.tag!.allObjects as! [Tag] 
    }

    var body: some View {
        VStack {
            TextField("New Tag Name", $newTagName)
            Button("Add New Tag", action: addNewTag)

            List {
                ForEach(noteTag) { tag in
                    Text(tag.name ?? "N/A")
                }
            }
            .onDelete { indexSet in
                for index in indexSet {
                    note.removeFromTag(tag[index])
                }

                try? viewContext.save()
            }
        }
    }

    func addNewTag() {
        let newTag = Tag(context: viewContext)
        newTag.name = newTagName
        note.addToTag(newTag)
        saveCoreData(managedObjectContext: viewContext)
    }
}
```

# 将CoreData对象绑定到TextField中

如果你想达到“修改后立刻保存”的效果，那么可以使用如下的方法，将CoreData对象绑定到`TextField`中：

```swift
TextField("Title", text: Binding {
    note.title ?? ""
} set: { newValue in
    note.title = newValue
    saveCoreData(managedObjectContext: viewContext)
})
```


# 将`Binding<T?>`变成`Binding<T>`?

使用构造函数Binding(_:)，比如将`Binding<Category?>`变成`Binding<Category>?`，就可以这么做：

```swift
// $note.category has a type of Binding<Category?>
let optionalCategoryBinding = Binding($note.category)
```
