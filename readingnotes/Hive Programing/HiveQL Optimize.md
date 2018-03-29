##### 使用 explain 查看执行过程

##### 使用 explain extended 查看更详细的执行过程
> 增加了文件系统信息

##### 确保 limit 抽样
即使使用 limit 语句，仍然需要执行整个查询语句，再返回部分结果。为确保使用 limit 语句时，是对数据源抽样，可以设置配置项如下：
- hive.limit.optimize.enable = true
- hive.limit.row.max.size = 100000
- hive.limit.optimize.limit.file = 10

##### join 优化
- 大表放在最后，或直接使用 /*streamtable(table_name)*/ 标记大表
- 使用 /*mapjoin(table_name)*/ 标记小表，触发map side join。注意需要更改配置项hive.auto.convert.join = true, 并设置 hive.mapjoin.smalltable.filesize

##### 使用本地模式
对小数据集使用本地模式，而非分布式模式处理
> 设置配置项 hive.exec.mode.local.auto = true , 让hive在合适时机自动开启本地模式  

##### 设置并行执行
> 设置配置项 hive.exec.parallel = true

##### 开启严格模式
> 设置配置项 hive.mapred.mode = strict 可以禁止三种类型的查询
- where语句必须含有分区字段，通过过滤条件来限制数据范围，避免扫描所有分区
- order by 必须与 limit 搭配使用，因为order by 是将所有数据放到一个reducer中处理
- 限制笛卡尔积查询。join后面必须有on，来过滤条件

##### 调整 mapper 和 reducer 的个数
- 每个reducer包含的数据量字节数：hive.exec.reducers.bytes.per.reducer
- 设置固定数值的reducer个数，默认是3个：mapred.reduce.tasks = 3
- 设置一个job最多可以使用多少个reducer：hive.exec.reducers.max
> 一个hadoop集群中的 mapper 和 reducer 的资源个数（也称“插槽”）是固定的，要避免某个大job(某个查询)占用太多reducer，导致其他job（查询）无法执行。

##### 开启 JVM 重用
hadoop默认使用派生 JVM 来执行 map 和 reduce 任务。JVM 重用适合小文件场景和task 特别多的情况。更改下面配置项，以设置 JVM 实例在同一个job中重新使用N次
> mapred.job.reuse.jvm.num.tasks = N

##### 索引可加快含 group by 的查询语句计算速度

##### 动态分区调整