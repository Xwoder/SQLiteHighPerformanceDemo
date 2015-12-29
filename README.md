# SQLiteHighPerformanceDemo
SQLite high performance demo.

```objc
NSString *dbPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"student.sqlite"];

NSLog(@"%@", dbPath);

sqlite3 *ppDb = NULL;
if (SQLITE_OK != sqlite3_open(dbPath.UTF8String, &ppDb)) {
    if (NULL != ppDb) {
        sqlite3_close(ppDb);
        return;
    }
}
NSLog(@"1. 成功打开数据库连接");

char const *createTableSql = "CREATE TABLE IF NOT EXISTS t_student(id INTEGER PRIMARY KEY AUTOINCREMENT DEFAULT 1, name TEXT);";
sqlite3_stmt *createTableStmt = NULL;
if (SQLITE_OK != sqlite3_prepare_v2(ppDb, createTableSql, -1, &createTableStmt, NULL)) {
    if (NULL != createTableStmt) {
        sqlite3_finalize(createTableStmt);
        sqlite3_close(ppDb);
        return;
    }
}

if (SQLITE_DONE != sqlite3_step(createTableStmt)) {
    sqlite3_finalize(createTableStmt);
    sqlite3_close(ppDb);
    return;
}
sqlite3_finalize(createTableStmt);
NSLog(@"2. 成功创建数据表");

char const *beginTransactionSql = "BEGIN TRANSACTION";
sqlite3_stmt *beginTransactionStmt = NULL;
if (SQLITE_OK != sqlite3_prepare_v2(ppDb, beginTransactionSql, -1, &beginTransactionStmt, NULL)) {
    sqlite3_finalize(beginTransactionStmt);
    sqlite3_close(ppDb);
    return;
}
if (SQLITE_DONE != sqlite3_step(beginTransactionStmt)) {
    sqlite3_finalize(beginTransactionStmt);
    sqlite3_close(ppDb);
    return;
}
sqlite3_finalize(beginTransactionStmt);
NSLog(@"3. 显式开启一个事务");

char const *insertDataSql = "INSERT INTO t_student(name) VALUES(?);";
sqlite3_stmt *insertDataStmt = NULL;
if (SQLITE_OK != sqlite3_prepare_v2(ppDb, insertDataSql, -1, &insertDataStmt, NULL)) {
    sqlite3_finalize(insertDataStmt);
    sqlite3_close(ppDb);
    return;
}

NSUInteger counter = 1000 * 1000;

for (NSUInteger i = 1; i <= counter; ++i) {
    NSString *name = [NSString stringWithFormat:@"赵剑-%lu", i];
    sqlite3_bind_text(insertDataStmt, 1, name.UTF8String, -1, SQLITE_TRANSIENT);
    if (SQLITE_DONE != sqlite3_step(insertDataStmt)) {
        sqlite3_finalize(insertDataStmt);
        sqlite3_close(ppDb);
        return;
    }
    sqlite3_reset(insertDataStmt);
}
sqlite3_finalize(insertDataStmt);
NSLog(@"4. 插入数据完成");

char const *commitTransactionSql = "COMMIT TRANSACTION";
sqlite3_stmt *commitTransactionStmt = NULL;
if (SQLITE_OK != sqlite3_prepare_v2(ppDb, commitTransactionSql, -1, &commitTransactionStmt, NULL)) {
    sqlite3_finalize(commitTransactionStmt);
    sqlite3_close(ppDb);
    return;
}
if (SQLITE_DONE != sqlite3_step(commitTransactionStmt)) {
    sqlite3_finalize(commitTransactionStmt);
    sqlite3_close(ppDb);
    return;
}
sqlite3_finalize(commitTransactionStmt);
NSLog(@"5. 显式提交一个事务");

char const *dropTableSql = "DROP TABLE t_student;";
sqlite3_stmt *dropTableStmt = NULL;
if (SQLITE_OK != sqlite3_prepare_v2(ppDb, dropTableSql, -1, &dropTableStmt, NULL)) {
    sqlite3_finalize(dropTableStmt);
    sqlite3_close(ppDb);
    return;
}
if (SQLITE_DONE != sqlite3_step(dropTableStmt)) {
    sqlite3_finalize(dropTableStmt);
    sqlite3_close(ppDb);
    return;
}
sqlite3_finalize(dropTableStmt);
NSLog(@"6. 删除数据库");
```