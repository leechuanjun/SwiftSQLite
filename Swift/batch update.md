# 批量更新

## 目标

* 掌握`事务`的概念
* 了解数据绑定

## 事务

* 在准备做`大规模数据操作前`，首先开启一个事务，保存操作前的数据库的状态
* 开始数据操作
* 如果数据操作成功，`提交`事务，让数据库更新到数据操作后的状态
* 如果数据操作失败，`回滚`事务，让数据库还原到操作前的状态

## 代码实现

### 准备事务代码

```swift
// MARK: - 事务处理
/// 开启事务
func beginTransaction() {
    sqlite3_exec(db, "BEGIN TRANSACTION;", nil, nil, nil)
}

/// 提交事务
func commitTransaction() {
    sqlite3_exec(db, "COMMIT TRANSACTION;", nil, nil, nil)
}

/// 回滚事务
func rollbackTransaction() {
    sqlite3_exec(db, "ROLLBACK TRANSACTION;", nil, nil, nil)
}
```

### 用事务插入大量数据

* 不使用事务插入数据

```swift
/// 不使用事务插入数据 - 测试结果60秒
func insertManyPerson1() {
    
    let start = CACurrentMediaTime()
    print("开始")
    for i in 0..<100000 {
        Person(dict: ["name": "lisi - \(i)", "age": 21, "height": 1.8]).insertPerson()
    }
    print("结束 \(CACurrentMediaTime() - start)")
}
```

* 使用事务插入数据

```swift
/// 使用事务插入数据 - 测试结果3.6秒
func insertManyPerson2() {
    
    let start = CACurrentMediaTime()
    print("开始")
    SQLiteManager.sharedManager.execSQL("BEGIN TRANSACTION;")
    for i in 0..<100000 {
        Person(dict: ["name": "lisi - \(i)", "age": 21, "height": 1.8]).insertPerson()
    }
    SQLiteManager.sharedManager.execSQL("COMMIT TRANSACTION;")
    print("结束 \(CACurrentMediaTime() - start)")
}
```

> 原因：在 SQLite 中，如果不主动开启事务，每个数据更新操作 (INSERT / UPDATE / DELETE) 都会默认开启一个事务，数据更新结束后自动提交事务

### 利用数据绑定操作数据


