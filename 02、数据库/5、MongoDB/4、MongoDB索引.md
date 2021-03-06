[TOC]



索引提高查询速度，降低写入速度，权衡常用的查询字段，不建议在太多列上建索引。在mongodb中，索引可以按字段升序/降序来创建，便于排序。默认是用btree来组织索引文件，也允许建立hash索引。

```
# 在test库下stu下创建10000行数据的成绩表
for (var i=1;i<=10000;i++){
	db.stu.insert({sn:i,name:'stu'+i,email:'stu'+i+'@126.com',score:{yuwen:i%80,shuxue:i%90,yingyu:i%100}})
}
```

# 1. 普通索引

## 1. 单列索引

在表stu创建sn列索引

> ```
> db.stu.ensureIndex({sn:1})
> ```

1表示升序，-1表示降序

## 2. 多列索引

在表stu创建sn列和name列共同索引

> ```
> db.stu.ensureIndex({sn:1,name:1})
> ```

1表示升序，-1表示降序

## 3. 子文档索引

在表stu的score列下的yuwen字段创建索引

> ```
> db.stu.ensureIndex({‘score.yuwen’:1})
> ```

1表示升序，-1表示降序

# 2. 唯一索引

创建唯一索引后字段值都是唯一的

在表stu创建email列索引

> ```
> db.stu.ensureIndex({email:1},{unique:true})
> ```

# 3. 稀疏索引

稀疏索引的特点：如果针对field做索引，针对不含field列的文档，将不建立索引。与之相对的普通索引会把该文档的field列的值认为NULL，并建索引。

使用场景：小部分文档含有某列时。

在表stu创建phone列稀疏索引

> ```shell
> db.stu.ensureIndex({age:1},{sparse:true})
> ```

# 4. 哈希索引

哈希索引速度比普通索引快，缺点是不能对范围查询进行优化。使用场景：随机性强的散列

在表stu创建email列哈希索引

> ```shell
> db.stu.ensureIndex({email:‘hashed’})
> ```

# 5. 重建索引

一个表经过很多次修改后，导致表的文件产生空洞，索引文件也如此。可以通过索引的重建，减少索引文件碎片，并提高索引的效率，类似mysql中的optimize table

在表stu重建索引

> ```shell
> db.stu.reIndex()
> ```

# 6. 删除索引

```shell
# 语法
db.collection.dropIndex({filed:1/-1});

# 示例
db.stu.dropIndex({sn:1})
db.stu.dropIndex ({email:'hashed'})
```

# 7. 查看索引和执行计划

```shell
# 查看表索引
db.stu.getIndexes()

# 查看执行计划
db.stu.find({sn:5555}).explain() # 默认只输出queryPlanner

# 其中explain()参数有三个，分别是'queryPlanner'、'executionStats'、'allPlansExecution'
db.stu.find({sn:5555}).explain('executionStats')

# explain分析结果的几个重要字段，通过结果分析可以判断是否需要优化执行语句
```

executionStats属性下的字段：

- executionTimeMillis：查询耗时，单位(ms)
- totalDocsExamined：扫描文档数
- executionStages.stage：”COLLSCAN”表示全表扫描，”FETCH”表示索引扫描
- executionStages. executionTimeMillisEstimate：索引扫描耗时，单位(ms)

winningPlan.inputStage属性下的字段：

- indexName：索引名字