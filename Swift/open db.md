# 打开数据库

## 目标

* 建立存储数据所需的二进制文件

## 代码实现

* 定义全局数据库访问句柄

```swift
/// 全局数据库访问句柄
private var db: COpaquePointer = nil
```

* 实现打开数据库函数

```swift
/// 打开数据库
///
/// - parameter dbName: 数据库文件名
func openDB(dbName: String) {
    
    var path = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true)[0]
    path = (path as NSString).stringByAppendingPathComponent(dbName)
    print(path)
    
    if sqlite3_open(path, &db) != SQLITE_OK {
        print("打开数据库失败")
        return
    }
    
    print("打开数据库成功")
}
```

* 在 `AppDelegate.swift` 中添加以下代码打开数据库

```swift
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {

    DBManager.sharedManager.openDb("my.db")
    
    return true
}
```

## 小结

* 建立数据库需要给定完整的数据库路径
* `sqlite3_open` 函数会打开数据库，如果数据库不存在，会`新建一个空的数据库`，并且返回数据库指针(句柄)
* 后续的所有数据库操作，都基于此 `数据库句柄` 进行
* `SQLite` 数据库是直接保存在沙盒中的一个文件，只有当前应用程序可以使用
* 在移动端开发时，数据库通常是以 `持久式` 连接方式使用的
* 所谓 `持久式连接` 指的是只做一次 `打开数据库` 的操作，永远不做 `关闭` 数据库的操作，从而可以提高数据库的访问效率