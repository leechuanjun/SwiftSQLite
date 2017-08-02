# 新数据

## 目标

* 新增数据
* 获取自动增长 ID

## 代码实现

### 准备工作

* 建立 `Person.swift` 数据模型

```swift
/// 个人模型
class Person: NSObject {

    // MARK: - 模型属性
    /// 代号
    var id = 0
    /// 姓名
    var name: String?
    /// 年龄
    var age = 0
    /// 身高
    var height: Double = 0
    
    // MARK: - 构造函数
    init(dict: [String: AnyObject]) {
        super.init()
        
        setValuesForKeysWithDictionary(dict)
    }
    
    override var description: String {
        let keys = ["id", "name", "age", "height"]
        
        return dictionaryWithValuesForKeys(keys).description
    }
    
    // MARK: - 数据库操作方法
    
}
```

### 新增记录

* 在 `Person` 模型中添加以下代码

```swift
/// 将当前对象插入到数据库
///
/// - returns: 是否成功
func insertPerson() -> Bool {
    // 0. 断言姓名不为 nil
    assert(name != nil, "姓名不能为 nil")
    
    // 1. 准备 SQL
    let sql = "INSERT INTO T_Person (name, age, height) VALUES ('\(name!)', \(age), \(height));"
    
    // 2. 返回结果
    return SQLiteManager.sharedManager.execSQL(sql)
}
```

* 在 `ViewController` 中增加测试代码

```swift
/// 测试插入数据
func demoInsert() {
    let p = Person(dict: ["name": "zhangsan", "age": 19, "height": 1.7])
    
    if p.insertPerson() {
        print("插入成功 \(p)")
    } else {
        print("插入失败")
    }
}
```

#### 小结

* 如果主键是自动增长的，在插入数据时，不需要指定
* 字符串类型的属性，需要使用单引号

> 问题：如何获取自动增长的主键数值？

* 在 `SQLiteManager` 中增加函数 `execInsert`

```swift
/// 执行插入 SQL
///
/// - parameter sql: sql
///
/// - returns: 返回自动增长 id
func execInsert(sql: String) -> Int {
    if sqlite3_exec(db, sql, nil, nil, nil) != SQLITE_OK {
        return -1
    }
    // 返回自动增长 id
    return Int(sqlite3_last_insert_rowid(db))
}
```

* 修改 insertPerson() 函数

```swift
/// 将当前对象插入到数据库
///
/// - returns: 是否成功
func insertPerson() -> Bool {
    // 0. 断言姓名不为 nil
    assert(name != nil, "姓名不能为 nil")
    
    // 1. 准备 SQL
    let sql = "INSERT INTO T_Person (name, age, height) VALUES ('\(name!)', \(age), \(height));"
    
    // 2. 获得自动增长 id
    id = SQLiteManager.sharedManager.execInsert(sql)
    
    return id > 0
}
```

#### 小结

* 使用 `sqlite3_last_insert_rowid` 可以获得自动增长的主键值
