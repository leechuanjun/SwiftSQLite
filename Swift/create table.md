# 创建数据表

## 目标

* 建立数据表，保证数据库中存在保存数据的 `表`

### 提示

* 如果是第一次运行，打开数据库之后，只能得到一个空的数据，没有任何的数据表
* 为了让数据库正常使用，在第一次打开数据库后，需要执行 `创表` 操作
* 注意：创表操作本质上是通过执行 `SQL` 语句实现的

## 代码实现

* 执行 `SQL` 语句函数

```swift
/// 执行 SQL
///
/// - parameter sql: SQL
///
/// - returns: 是否成功
func execSQL(sql: String) -> Bool {
    /**
        参数
        1. 数据库句柄
        2. 要执行的 SQL 语句
        3. 执行完成后的回调，通常为 nil
        4. 回调函数第一个参数的地址，通常为 nil
        5. 错误信息地址，通常为 nil
    */
    return sqlite3_exec(db, sql, nil, nil, nil) == SQLITE_OK
}
```

* 创建数据表

```swift
/// 创建数据表
///
/// - returns: 是否成功
private func createTable() -> Bool {
    // 1. 准备 SQL
    let sql = "CREATE TABLE IF NOT EXISTS 'T_Person' ( \n" +
    "'id' INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT, \n" +
    "'name' TEXT, \n" +
    "'age' INTEGER, \n" +
    "'height' REAL \n" +
    ");"
    
    // 2. 返回执行 SQL 的结果
    return execSQL(sql)
}
```

* 调整 `openDB` 函数

```swift
if createTable() {
    print("创建数据表成功")
} else {
    print("创建数据表失败")
}
```

### 小结

* 创表 `SQL` 可以从 `Navicat` 中粘贴，然后做一些处理
    * 将 `"` 替换成 `'`
    * 在每一行后面增加一个 `\n` 防止字符串拼接因为缺少空格造成 `SQL` 语句错误
    * 在 `表名` 前添加 `IF NOT EXISTS` 防止因为数据表存在出现错误

## 通过 SQL 文件直接创表

* 在 Xcode 中建立 `db.sql` 文件
* 将 Navicat 中的 DDL 创表文件粘贴到 db.sql 中
    * 每个表名前添加 `IF NOT EXISTS` 防止因为数据表存在出现错误
    * 每段 SQL 末尾添加 `;`
    * 使用 `-- 描述信息` 增加数据表备注

```sql
-- 创建个人信息表
CREATE TABLE IF NOT EXISTS "T_Person" (
"id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
"name" TEXT,
"age" INTEGER,
"height" REAL
);

-- 创建图书表
CREATE TABLE IF NOT EXISTS "T_Books" (
"id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
"name" TEXT
);
```

* 修改 `createTable` 函数

```swift
/// 创建数据表 - 从 bundle 加载并执行 db.sql
///
/// - returns: 是否成功
private func createTable() -> Bool {
    // 1. 准备 SQL
    let path = NSBundle.mainBundle().pathForResource("db.sql", ofType: nil)!
    guard let sql = try? NSString(contentsOfFile: path, encoding: NSUTF8StringEncoding) else {
        print("加载 SQL 失败")
        return false
    }
    
    // 2. 执行 SQL
    return execSQL(sql as String)
}
```

### 小结

* 通过 sql 脚本创建数据表的方式灵活性更大，被广泛应用在商业应用中
* 如果要保护数据安全，可以对 sql 脚本进行一些简单的加密处理