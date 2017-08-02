# 利用绑定参数插入数据

## 目标

* 了解 STMT 绑定更新数据的方式
* 这种方式有些繁琐，不过是其他第三方工具使用的方式，有必要了解

## 代码实现

* 准备函数

```swift
/// 利用绑定参数插入数据
///
/// - parameter sql:  SQL
/// - parameter args: 参数列表
func execInsert(sql: String, args: CVarArgType...) {
    print(args)
}
```

* 增加测试函数

```swift
/// 绑定参数插入数据
func bindInsert() {
    let sql = "INSERT INTO T_Person (name, age, height) VALUES (?, ?, ?);"
    
    SQLiteManager.sharedManager.execInsert(sql, args: "张三", 18, 1.7)
}
```

### 预编译 SQL

```swift
func execInsert(sql: String, args: CVarArgType...) -> Bool {

    // 1. 预编译 SQL
    var stmt: COpaquePointer = nil
    if sqlite3_prepare_v2(db, sql, -1, &stmt, nil) != SQLITE_OK {
        print("SQL 错误")
        sqlite3_finalize(stmt)
        
        return false
    }

    // 3. 释放 stmt
    sqlite3_finalize(stmt)
    
    return true
}
```

* 绑定参数

```swift
// 2. 绑定参数
var index: Int32 = 1
for arg in args {
    
    if arg is Double {
        sqlite3_bind_double(stmt, index, arg as! Double)
    } else if arg is Int {
        sqlite3_bind_int64(stmt, index, sqlite3_int64(arg as! Int))
    } else if arg is String {
        let str = (arg as! String).cStringUsingEncoding(NSUTF8StringEncoding)
        sqlite3_bind_text(stmt, index, str!, -1, nil)
    }
    
    index++
}
```

* 单步执行 SQL 以及复位

```swift
// 3. 单步执行
if sqlite3_step(stmt) != SQLITE_DONE {
    print("插入失败")
}

// 4. 释放 stmt
sqlite3_finalize(stmt)

return Int(sqlite3_last_insert_rowid(db))
```

#### 第5个参数

* 如果第5个参数传递 `NULL` 或者 `SQLITE_STATIC` 常量，SQlite 会假定这块 buffer 是静态内存，或者客户应用程序会小心的管理和释放这块 buffer，所以SQlite放手不管
* 如果第5个参数传递的是 `SQLITE_TRANSIENT` 常量，则 SQlite 会在内部复制这块 buffer 的内容。这就允许客户应用程序在调用完 bind 函数之后，立刻释放这块 buffer（或者是一块栈上的 buffer 在离开作用域之后自动销毁）。SQlite会自动在合适的时机释放它内部复制的这块 buffer

由于在 SQLite.h 中 `SQLITE_TRANSIENT` 是以宏的形式定义的，而在 swift 中无法直接利用宏传递函数指针，因此需要使用以下代码转换一下

```swift
/*
typedef void (*sqlite3_destructor_type)(void*);
#define SQLITE_STATIC      ((sqlite3_destructor_type)0)
#define SQLITE_TRANSIENT   ((sqlite3_destructor_type)-1)
*/
private let SQLITE_STATIC = unsafeBitCast(0, sqlite3_destructor_type.self)
private let SQLITE_TRANSIENT = unsafeBitCast(-1, sqlite3_destructor_type.self)
```

* 修改字符串绑定代码

```swift
sqlite3_bind_text(stmt, col, str, -1, SQLITE_TRANSIENT)
```

