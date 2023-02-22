# 使用UserDefaults存储数据

`UserDefaults`是一个简单的Key-Value数据库，通常用它来储存用户偏好、应用设置。当然如果你的应用比较简单，那么也可以把用户数据也保存在这里。如果应用的数据模型比较复杂，那么使用CoreData会更加合适。

# 使用方法

一般我们使用其默认实例`UserDefaults.standards`来获取、保存数据。下面是所有数据写入的方法：

```swift
// Sets the value of the specified default key.
// The value to be set should only be a property list object, i.e.
// NSData, NSString, NSNumber, NSDate, NSArray, or NSDictionary
func set(Any?, forKey: String)

// Sets the value of the specified default key to the specified float value.
func set(Float, forKey: String)

//  Sets the value of the specified default key to the double value.
func set(Double, forKey: String)

// Sets the value of the specified default key to the specified integer value.
func set(Int, forKey: String)

// Sets the value of the specified default key to the specified Boolean value.
func set(Bool, forKey: String)

// Sets the value of the specified default key to the specified URL.
func set(URL?, forKey: String)
```

以及所有数据获取方法。

```swift

// Returns the object associated with the specified key.
// Notice the "object" here means the proeprty list object,
// i.e.  NSData, NSString, NSNumber, NSDate, NSArray, or NSDictionary
// Actually you cannot save and extract an custom object directly into
// UserDefaults.
func object(forKey: String) -> Any?

// Returns the URL associated with the specified key.
func url(forKey: String) -> URL?

// Returns the array associated with the specified key.
func array(forKey: String) -> [Any]?

// Returns the dictionary object associated with the specified key.
func dictionary(forKey: String) -> [String : Any]?

// Returns the string associated with the specified key.
func string(forKey: String) -> String?

// Returns the array of strings associated with the specified key.
func stringArray(forKey: String) -> [String]?

// Returns the data object associated with the specified key.
func data(forKey: String) -> Data?

// Returns the Boolean value associated with the specified key.
func bool(forKey: String) -> Bool

// Returns the integer value associated with the specified key.
func integer(forKey: String) -> Int

// Returns the float value associated with the specified key.
func float(forKey: String) -> Float

// Returns the double value associated with the specified key.
func double(forKey: String) -> Double

// Returns a dictionary that contains a union of all key-value pairs in the domains in the search list.
func dictionaryRepresentation() -> [String : Any]
```

# 存储自定义对象

对于自定义对象，你可以把它转换成`Data`对象再存进`UserDefaults`。最常见的做法就是使用`JSONEncoder`生成包含对象`JSON`的`Data`对象，然后再把该对象存入`UserDefaults`

```swift
struct UserPreference: Codable {
    let theme: String
    let username: String
    let fontSize: Int
    let enableAutoSaving: Bool

    static let userDefaultsKey = "UserPreference"
}

func saveUserPreference(userPreference: UserPreference) {
    let jsonData = JSONEncoder().encode(userPreference)
    UserDefaults.standard.set(jsonData, forKey: UserPreference.userDefaultsKey)
}

func retreiveUserPreference() throws -> UserPreference? {
    if let jsonData = UserDefaults.standard.data(forKey: UserPreference.userDefaultsKey) {
        return try JSONDecoder().decode(UserPreference.self, from: jsonData)
    } else {
        return nil
    }
}
```

# 注意要点

为了防止Typo，还有方便以后修改`key`，建议将`key`值保存在一个常量中，比如上文的`UserPreference.userDefaultsKey`。如果`key`数量很多，那么建议添加如下代码：

```swift
protocol UserDefaultValue {
    associatedtype defaultKeys: RawRepresentable
}

extension UserDefaultValue where defaultKeys.RawValue==String {
    static func set(value: Any?, forKey key: defaultKeys) {
        UserDefaults.standard.set(value, forKey: key.rawValue)
    }
    
    static func string(forKey key: defaultKeys) -> String? {
        UserDefaults.standard.string(forKey: key.rawValue)
    }
    
    static func double(forKey key: defaultKeys) -> Double {
        UserDefaults.standard.double(forKey: key.rawValue)
    }
    
    static func float(forKey key: defaultKeys) -> Float {
        UserDefaults.standard.float(forKey: key.rawValue)
    }
    
    static func bool(forKey key: defaultKeys) -> Bool {
        UserDefaults.standard.bool(forKey: key.rawValue)
    }
    
    static func object(forKey key: defaultKeys) -> Any? {
        UserDefaults.standard.object(forKey: key.rawValue)
    }
    
    static func integer(forKey key: defaultKeys) -> Int {
        UserDefaults.standard.integer(forKey: key.rawValue)
    }
    
    static func array(forKey key: defaultKeys) -> [Any]? {
        UserDefaults.standard.array(forKey: key.rawValue)
    }
    
    static func dictionary(forKey key: defaultKeys) -> [String : Any]? {
        UserDefaults.standard.dictionary(forKey: key.rawValue)
    }
}
```

然后将所有`UserDefaults`的`key`想下面这样组织到一个文件中：

```swift
extension UserDefaults {
    // 账户信息
    struct AccountInfo: UserDefaultsSettable {
        enum defaultKeys: String {
            case userName
            case age
        }
    }
    
    // 登录信息
    struct LoginInfo: UserDefaultsSettable {
        enum defaultKeys: String {
            case token
            case userId
        }
    }
}
```

然后我们就能够轻松设置`UserDefaults`，不用担心`typo`造成的bug，也能够享受到XCode的智能提示了。

```swift
UserDefaults.AccountInfo.set(value: "chilli cheng", forKey: .userName)
UserDefaults.AccountInfo.string(forKey: .userName)
        
UserDefaults.LoginInfo.set(value: "ahdsjhad", forKey: .token)
UserDefaults.LoginInfo.string(forKey: .token)
```

这个主意来自[https://www.jianshu.com/p/3796886b4953](https://www.jianshu.com/p/3796886b4953)。感谢这位小伙伴的聪明想法💡。