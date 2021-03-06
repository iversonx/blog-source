InnoDB是一种通用存储引擎，可以平衡高可靠性和高性能。 在MySQL 5.7中，InnoDB是默认的MySQL存储引擎。 除非您已配置其他默认存储引擎，否则发出不带ENGINE =子句的CREATE TABLE语句会创建InnoDB表。



### 主要优势

- DML操作遵循ACID模型，具有提交，回滚和崩溃恢复功能的事务来保护用户数据。

- 行级锁定和Oracle风格的一致性读取可提高多用户并发性和性能。

- InnoDB表将数据排列在磁盘上，以根据主键优化查询。InnoDB的主键索引是聚簇索引，用于组织数据以最大限度地减少主键查找的IO。

- 为了保持数据完整性，InnoDB支持FOREIGN KEY约束。对于外键，将检查插入、更新和删除，以确保它们不会导致不同表之间的不一致。

### 使用InnoDB表的好处

1. 如果服务器因硬件或软件问题崩溃，在重启数据库后，InnoDB会完成在崩溃之前提交的所有更改，并撤销正在进行但未提交的所有更改。

2. InnoDB存储引擎维护自己的缓冲池，当访问数据时，该缓冲池将表和索引数据缓存在主内存中。经常使用的数据直接从内存中处理。此
   
   缓存应用于多种类型的信息并加快处理速度。在专用数据库服务器上，高达80%的物理内存通常分配给缓冲池。

3. 如果将相关数据拆分为不同的表，则可以设置强制引用完整性的外键。更新或删除数据，其他表中的相关数据将自动更新或删除。

4. 如果数据在磁盘或内存中损坏，校验和机制会在使用前提醒您使用伪造的数据。

5. 当为每个表设计具有适当主键列的数据库时，涉及这些列的操作将自动优化。在where子句、order by子句、group by子句和join操作中引用主键列非常快。

6. 插入、更新和删除由一种称为更改缓冲的自动机制优化。InnoDB不仅允许对同一个表进行并发读写访问，它还缓存更改的数据以简化磁盘I/O。

7. 性能优势不仅限于具有长时间运行查询的大型表。当从一个表中反复访问相同的行时，一个称为自适应散列索引的功能将接管这些查询，使它们的查找速度更快，就好像它们是从散列表中出来的一样。

8. 可以压缩表和关联的索引。

9. 可以创建和删除索引，对性能和可用性的影响要小得多。

10. 截断每个表的文件表空间非常快，并且可以释放磁盘空间以供操作系统重用，而不是释放系统表空间中只有InnoDB可以重用的空间。

11. 对于具有动态行格式的BLOB和长文本字段，表数据的存储布局更高效。

12. 可以通过查询INFORMATION_SCHEMA表来监视存储引擎的内部工作方式。

13. 可以通过查询Performance Schema 表来监控存储引擎的性能详细信息

14. 可以自由地将InnoDB表与其他MySQL存储引擎中的表混合，甚至在同一语句中也是如此。 例如，您可以使用连接操作在单个查询中组合InnoDB和MEMORY表中的数据

15. 专为处理大量数据时的CPU效率和最高性能而设计

16. 在文件大小限制为2GB的操作系统上，InnoDB表也可以处理大量数据。
