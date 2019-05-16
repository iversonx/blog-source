## 14.8 InnoDB表和索引

### 14.8.1 InnoDB表

#### 14.8.1.1 创建InnoDB表

```sql
CREATE TABLE t1 (a INT, b CHAR (20), PRIMARY KEY (a)) ENGINE=InnoDB;
```

- 在每个表的文件表空间创建表
  
  1. 当启用`innodb_file_per_table`时，会在每个表的单个文件表空间中隐式创建一个innoDB表。
  
  2. 对于每个表文件表空间中创建的表，MySQL默认会在数据库目录中创建.ibd表空间文件。

- 在InnoDB系统表空间创建表
  
  1. 当`innodb_file_per_table`被禁用时，在innodb系统表空间中隐式创建一个innodb表。
  
  2. 在InnoDB系统表空间中创建的表是在现有的ibdata文件中创建的，该文件驻留在MySQL数据目录中。

- 在通用表空间中创建表
  
  1. 要在通用表空间中创建表，请使用CREATE TABLE ... TABLESPACE语法。
  
  2. 在通用表空间中创建的表在现有的通用表空间.ibd文件中创建。

##### InnoDB表和.frm文件

MySQL在数据库目录中的.frm文件中存储表的数据字典信息。InnoDB还在系统表空间内的内部数据字典中对表信息进行编码。当MySQL删除表或数据库时，它会删除一个或多个.frm文件以及InnoDB数据字典中的相应条目。

##### InnoDB表和行格式

InnoDB表的默认行格式由innodb_default_row_format配置选项定义，其默认值为DYNAMIC。动态和压缩行格式允许您利用InnoDB特性，例如表压缩和长列值的高效页外存储。 要使用这些行格式，必须启用innodb_file_per_table，innodb_file_format必须设置为Barracuda。

或者可以使用`CREATE TABLE ... TABLESPACE`语法在通用表空间中创建InnoDB表。 通用表空间支持所有行格式。

##### InnoDB表和主键

始终为innoDB表创建主键，可以指定以下列为主键：

- 被重要的查询引用

- 永远不会为空

- 永远不会有重复的值

- 插入后很少改变值

如果没有明显的列用可以作为主键，就创建一个自增长的id作为主键。

##### 查看InnoDB表属性

使用`SHOW TABLE STATUS FROM 数据库名称 WHERE 条件`可以查看表属性。

```sql
SHOW TABLE STATUS FROM 数据库名称 WHERE 条件;
```

也可以使用`INFORMATION_SCHEMA.INNODB_SYS_TABLES`查询InnoDB表属性。

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES WHERE 条件
```

#### 14.8.1.2 InnoDB表的物理行结构

InnoDB表的物理行结构取决于创建表时指定的行格式。

##### Redundant行格式特点

REDUNDANT格式可用于保持与旧版MySQL的兼容性。

- 每个索引记录包含一个6字节的标头。标头用于将连续记录链接在一起，也用于行级锁定。

- 聚簇索引中的记录包含所有用户定义列的字段。此外，还有一个6字节的事务ID字段和一个7字节的滚动指针字段。

- 如果没有定义主键，则每个聚簇索引还包含一个6字节的row id字段。

- 每个次索引包含对聚簇索引定义的所有主键字段，这些字段不在次索引中。

- 记录包含指向记录的每个字段的指针。如果记录中字段的总长度小于128字节，则指针是一个字节;否则，两个字节。这些指针的数组称为记录目录。这些指针指向的区域称为记录的数据部分。

- 以固定长度格式存储固定长度的字符列。

- InnoDB将长度大于或等于768字节的固定长度字段编码为可变长度字段，可以在页外存储。

- NULL值在记录目录中保留一个或两个字节。SQL NULL值在记录目录中保留一个或两个字节。 除此之外，如果存储在可变长度列中，则SQL NULL值在记录的数据部分中保留零个字节。 在固定长度的列中，它在记录的数据部分中保留列的固定长度。 为NULL值保留固定空间可以将列的更新从NULL更新为非NULL值，而不会导致索引页碎片化。

##### COMPACT行格式特点

与REDUNDANT格式相比，COMPACT行格式减少了约20%的行存储空间，同时增加了某些操作的CPU使用。如果您的工作负载是受缓存命中率和磁盘速度限制的典型工作负载，则COMPACT格式可能会更快。 如果工作负载是受CPU速度限制的罕见情况，则紧凑格式可能会更慢。

- 每个索引记录包含一个5字节的头，可以在可变长度头之前。标头用于将连续记录链接在一起，也用于行级锁定。

#### 14.8.1.3 移动或复制InnoDB表

#### 14.8.1.4 将表从MyISAM转换为InnoDB

#### 14.8.1.5 InnoDB中的AUTO_INCREMENT处理

#### 14.8.1.6 InnoDB和FOREIGN KEY约束

#### 14.8.1.7 InnoDB表的限制

### 14.8.2 InnoDB索引

#### 14.8.2.1 集群和二级索引

#### 14.8.2.2  InnoDB索引的物理结构

#### 14.8.2.3 排序索引构建

#### 14.8.2.4 InnoDB FULLTEXT索引
