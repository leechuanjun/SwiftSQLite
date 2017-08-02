# 修改和删除数据

## 目标

* 修改数据
* 删除数据

## 代码实现

* 更新记录

```swift
/// 将当前对象的信息更新至数据库
///
/// - returns: 是否成功
func updatePerson() -> Bool {
    // 0. 断言
    assert(name != nil, "姓名不能为 nil")
    assert(id > 0, "ID 必须 > 0")
    
    // 1. 准备 SQL
    let sql = "UPDATE T_Person SET name = '\(name!)', age = \(age), height = \(height) \n" +
        "WHERE id = \(id);"
    
    print(sql)
    
    // 2. 返回结果
    return SQLiteManager.sharedManager.execSQL(sql)
}
```

* 删除记录

```swift
/// 将当前对象从数据库中删除
///
/// - returns: 是否成功
func deletePerson() -> Bool {
    // 0. 断言
    assert(id > 0, "ID 必须 > 0")
    
    // 1. 准备 SQL
    let sql = "DELETE FROM T_Person WHERE id = \(id);"
    
    // 2. 返回结果
    return SQLiteManager.sharedManager.execSQL(sql)
}
```

* 测试代码

```swift
/// 测试更新数据
func demoUpdate() {
    let p = Person(dict: ["id": 19999, "name": "lisi", "age": 21, "height": 1.8])
    
    if p.updatePerson() {
        print("更新成功 \(p)")
    } else {
        print("更新失败")
    }
}

/// 测试删除数据
func demoDelete() {
    let p = Person(dict: ["id": 1, "name": "lisi", "age": 21, "height": 1.8])
    
    if p.deletePerson() {
        print("删除成功 \(p)")
    } else {
        print("删除失败")
    }
}
```

## 小结

* 创表 / 新增 / 修改 / 删除 本质上都是执行一条 `SQL 指令` 
* 删除记录时，如果数据不存在，不会返回错误
* 更新记录时，如果指定的 id 不存在，不会返回错误

### 数据操作方法扩展

* 增加 `execUpdate` 函数

```swift
/// 执行 更新 / 删除 SQL
///
/// - parameter sql: sql
///
/// - returns: 返回 更新 / 删除 的数据行数
func execUpdate(sql: String) -> Int {
    if sqlite3_exec(db, sql, nil, nil, nil) != SQLITE_OK {
        return -1
    }
    // 返回影响的数据行数
    return Int(sqlite3_changes(db))
}
```

* 修改更新和删除按钮，再次测试

```swift
/// 将当前对象的信息更新至数据库
///
/// - returns: 是否成功
func updatePerson() -> Bool {
    // 0. 断言
    assert(name != nil, "姓名不能为 nil")
    assert(id > 0, "ID 必须 > 0")
    
    // 1. 准备 SQL
    let sql = "UPDATE T_Person SET name = '\(name!)', age = \(age), height = \(height) \n" +
        "WHERE id = \(id);"
    
    // 2. 返回结果
    let rows = SQLiteManager.sharedManager.execUpdate(sql)
    print("更新了 \(rows) 条记录")
    
    return rows > 0
}

/// 将当前对象从数据库中删除
    ///
    /// - returns: 是否成功
    func deletePerson() -> Bool {
        // 0. 断言
        assert(id > 0, "ID 必须 > 0")
        
        // 1. 准备 SQL
        let sql = "DELETE FROM T_Person WHERE id = \(id);"
        
        // 2. 返回结果
        let rows = SQLiteManager.sharedManager.execUpdate(sql)
        print("删除了 \(rows) 条记录")
        
        return rows > 0
    }
```

### 小结

* 新增记录可以使用 `sqlite3_last_insert_rowid` 获取最后插入的主键数值
* 更新 / 删除记录可以使用 `sqlite3_changes` 获取影响数据行数

