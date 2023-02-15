# 简介

CoreData是一个ORM框架，它将数据库表映射到对象，通过新建、删除、修改ORM对象，完成数据库增删改查。

# 创建表，定义Schema

[Creating a Managed Object Model](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/KeyConcepts.html#//apple_ref/doc/uid/TP40001075-CH30-SW1)

基本上是图形界面，简单易懂。

## 一些概念

## Inverse Relationship

只有当两个实体互相关联时方可设置他们之间的反向关系。设置反向关系能够让Core Data帮我们维护两个实体间的数据完整性。

打个比方，我们定义了两个实体：员工（Employee）和部门（Department）。他们之间互相关联：员工“知道”他所在的部门，而部门记录了属于它的所有员工。

Employee
1. (Property) employeeName
2. (Relationship) department (1:1)

Department
1. (Property) departmentName
2. (Relationship) employees (1:n)

当设置好反向关系后，修改实体的关系将变得非常简单，比如我想要修改员工所属的属性，那么只需一行代码：

```swift
employee.department = newDepartment
```

如果我们不设置反向关系，当我修改员工所属部门后，我还需要手动将该员工移除出其原部门，并将其添加到新部门中，不然数据将不完整：老部门的员工列表中还记着这位员工，而新部门的员工列表中却没有。

相关资料：
1. (苹果官方文档：Inverse Relationship, Creating Managed Object Relationships)[https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/HowManagedObjectsarerelated.html#//apple_ref/doc/uid/TP40001075-CH17-SW4]
2. (StackOverflow: Does every Core Data Relationship have to have an Inverse)[https://stackoverflow.com/questions/764125/does-every-core-data-relationship-have-to-have-an-inverse]

## Abstract Entity

## Fetched Properties

# 在代码中引入Core Data

(Setting Up a Core Data Stack)[https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack#//apple_ref/doc/uid/TP40001075-CH4-SW1]

# 创建、保存对象

创建对象
```swift
let employee = NSEntityDescription.insertNewObjectForEntityForName("Employee", inManagedObjectContext: managedObjectContext) as! EmployeeMO
```

保存对象
```swift
do {
    try managedObjectContext.save()
} catch {
    fatalError("Failure to save context: \(error)")
}
```

(Creating Managed Object)[https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/CreatingObjects.html#//apple_ref/doc/uid/TP40001075-CH5-SW1]

# 相关资料

1. [苹果官方文档： Setting Up Core Data with CloudKit](httpsdeveloper.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit/setting_up_core_data_with_cloudkit)
2. [苹果官方文档：Core Data](https://developer.apple.com/documentation/coredata)
3. [苹果官方文档：Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075-CH2-SW1)

# 坑

如果Codegen中选择了`Class Definition`，就不用手动生成代码（Editor -> Create NSManagedObject Subclass）。当你在Data Model Editor增删改实体后，CoreData会立即生成对应数据库类，而你只需要在文件中`import CoreData`，就能够使用它们。

如果要使用Create NSManagerObject Subclass，就需要把Codegen设置为`Manual/None`.