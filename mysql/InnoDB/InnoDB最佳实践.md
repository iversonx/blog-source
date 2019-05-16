1. 使用最频繁查询的一个或多个列作为表的主键，如果没有，就使用自增id作为主键。

2. 根据多个表的相同ID值从多个表中提取数据时，使用join。在连接列上定义外键，提高join性能。

3. 关闭自动提交，每秒数百次提交会影响性能。

4. 将一组相关的DML操作和 START TRANSACTION 和 COMMIT语句括起来，这样可以将相关的DML操作会分组到事务中。 

5. 不使用LOCK TABLES语句。InnoDB可以同时处理多个会话，同时读取和写入同一个表，而不会牺牲可靠性和高性能。要获得对一组行的独占写访问权，请使用SELECT ... FOR UPDATE语法仅锁定要更新的行。

6. 启用innodb_file_per_table选项或使用常规表空间将表的数据和索引放入单独的文件中，而不是系统表空间。默认情况下，innodb_file_per_table选项是启用的。

7. 使用选项--sql_mode = NO_ENGINE_SUBSTITUTION运行服务器，以防止在CREATE TABLE的ENGINE =子句中指定的引擎出现问题时使用其他存储引擎创建表。
