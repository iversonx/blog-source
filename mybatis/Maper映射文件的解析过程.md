## Mapper映射文件的解析过程

### XMLMapperBuilder

Mapper映射文件的解析工作是由XMLMapperBuilder对象来完成的。





MapperBuilderAssistant
ParameterMap
ParameterMapping
ResultMap
ResultMapping
XMLStatementBuilder

XML映射文件顶级元素 Configuration的属性对应
<cache> - Map<String, Cache> caches
<cache-ref> - Map<String, String> cacheRefMap
<resultMap> - Map<String, ResultMap> resultMaps 
<parameterMap> - Map<String, ParameterMap> parameterMaps
<sql> - Map<String, XNode> sqlFragments
<insert> - Map<String, MappedStatement> mappedStatements
<update> - Map<String, MappedStatement> mappedStatements
<delete> - Map<String, MappedStatement> mappedStatements
<select> - Map<String, MappedStatement> mappedStatements

对cache元素的解析

对cache-ref元素的解析

#### 对resultMap元素的解析

对应`Configuration`的`Map<String, ResultMap> resultMaps`属性

入口：`XMLMapperBuilder.resultMapElements(List<XNode> list)`

大致过程:

1. 获取`<resultMap>`元素的`id`, `type`, `extends`, `autoMapping`属性
2. 遍历`<resultMap>`所有的子元素
3. 根据解析得到的`id`, `typeClass`, `extend`, `discriminator`, `resultMappings`, `autoMapping`实例化`ResultMapResolver`
4. 在`MapperBuilderAssistant`的`addResultMap`方法中根据解析的数据，调用`ResultMap.Builder`得到一个`ResultMap`。最后将`ResultMap`添加到`Configuration`中的`resultMaps`属性中。

#### 对sql元素的解析

对应`Configuration`的`sqlFragments`属性

入口: `sqlElement(List<XNode> list)`

过程: 对`<sql>`的解析比较简单，先遍历所有的`<sql>`，接着添加到`sqlFragments`,`sqlFragments`是个`map`，`key`是`<sql>`的id属性, `value`是`<sql>`的内容。

#### 对insert, update, delete, selelct的解析

对应`Configuration`的`sqlFragments`属性`mappedStatements`

入口: `buildStatementFromContext(List<XNode> list)`

过程:

mybatis使用`XMLStatementBuilder`来完成这个任务。


