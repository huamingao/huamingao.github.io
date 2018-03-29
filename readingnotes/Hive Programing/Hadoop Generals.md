## Hadoop生态
Apache Hadoop 软件库是一个框架，是在集群服务器上使用简单的编程模型（MapReduce）对大数据集进行**分布式**处理。Hadoop能够从单台服务器扩展到数以千计的服务器，每台服务器都有本地的计算和存储资源。
![image](https://dn-anything-about-doc.qbox.me/userid29778labid764time1427382197256)

#### HDFS架构

- 高吞吐量：分布式存储数据，使用离用户最近、访问量最小的服务器提供用户访问，并行从集群中的多台服务器上读写数据，增加了访问带宽

- 高容错性： 每个数据块（block）都被冗余多份（默认3份），分布在不同的磁盘和服务器上

- 可线性扩展：HDFS block信息存在namenode上，文件block存在datanode上；数据量增加，添加datanode即可，实现在线扩展，无需宕机

![image](https://dn-anything-about-doc.qbox.me/userid29778labid1032time1433385988267?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

#### MapReduce分布式计算框架

MapReduce是一个分布式、并行处理的编程模型，把任务分为 map(映射)阶段和 reduce(化简)阶段。

当你向MapReduce框架提交一个计算作业时，它会首先**把计算作业拆分成若干个Map任务，分配到不同的节点上去执行，每一个Map任务处理输入数据中的一部分**，当Map任务完成后，它会生成一些中间文件，这些中间文件将会作为Reduce任务的输入数据。**Reduce任务的主要目标就是把前面若干个Map的输出汇总到一起并输出。**

![image](https://dn-anything-about-doc.qbox.me/userid29778labid1033time1433404183969?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

##### MapReduce流程分析

Input -> Mappers -> Sort（排序）, Shuffle（重新分发）-> Reducers -> Output

![image](https://dn-anything-about-doc.qbox.me/userid29778labid1033time1433404196962?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

## Hadoop中的Hive
Hive是构建于Hadoop集群之上的数据仓库应用，它提供了类似于SQL语法的HQL语句作为数据访问接口。Hive将结构化的数据文件映射为一张数据库表，并使用sql语句转换为MapReduce任务进行运行，实现对数据的提取转化加载（ETL）。

#### Hive vs RMDB
- Hive使用HDFS（Hadoop分布式文件系统），关系数据库使用服务器本地文件系统
- Hive使用的计算模型是Mapreduce，关系数据库则是自身的计算模型
- 关系数据库是为实时查询业务设计的，而Hive为海量数据做数据挖掘设计，实时性很差
- Hive很容易扩展自身的存储和计算能力，这个是继承Hadoop的，而关系数据库则较难

#### Hive架构与组件
##### 服务端组件
- **Driver**：该组件包括Complier、Optimizer和Executor，它的作用是将HiveQL（类SQL）语句进行解析、编译优化，生成执行计划，然后调用底层的mapreduce计算框架；
- **Metastore**：元数据服务组件，存储Hive的元数据，Hive的元数据存储在关系数据库里，支持的关系数据库有derby、mysql等。元数据对于Hive十分重要，因此Hive支持把metastore服务独立出来，安装到远程的服务器集群里，从而解耦Hive服务和metastore服务，保证Hive运行的健壮性；
**Thrift**：用来进行可扩展且跨语言的开发，能让不同的编程语言调用hive接口。

##### 客户端组件
- **CLI**：command line interface，命令行接口。
- **Thrift Client**：下面的架构图里没有写上Thrift客户端，但Hive架构的许多客户端接口是建立在thrift客户端之上，包括JDBC和ODBC接口。
- **WEBGUI**：H一种通过网页方式访问hive的服务。这个接口对应Hive的hwi组件（hive web interface），使用前要启动hwi服务

![image](https://dn-anything-about-doc.qbox.me/document-uid29778labid1042timestamp1433943328371.png?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)

[Apache Hive 官网上的架构介绍，戳这里](https://cwiki.apache.org/confluence/display/Hive/Design)

![image](https://cwiki.apache.org/confluence/download/attachments/27362072/system_architecture.png?version=1&modificationDate=1414560669000&api=v2)





