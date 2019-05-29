参与对象: 



MapperProxy

1. SqlSession从Configuration的mapperRegistry中获取对应的MapperProxy

2. MapperProxy持有当前SqlSession的引用, 自定义的Mapper接口的Class引用, MapperMethod缓存

MapperMethod



SqlSession

Executor

StatementHandler

ResultSetHandler
