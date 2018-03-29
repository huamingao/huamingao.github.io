## HiveQL数据定义与装载
### 数据类型和文件格式
#### 基本数据类型: 
- tinyint, smalint, int, bigint, float, boolean, string, double, binary, timestamp
- 显示数据类型转换：cast(字段名or具体值 as 指定类型)
```
select cast(1 as double) from table;
```


#### 集合数据类型
- struct: 类似于“对象”，通过点号访问元素内容, eg struct.item
- map：键值对（key-value pair），通过map['key']访问value值
- array: 数组结构，通过索引访问数组中的元素，索引从零开始编号，eg array[0]

#### 默认的字段分隔符

分隔符 | 描述 | 八进制编码
---|---| ---
\n | 分隔行 | 
^A | 分隔字段（列）| \001
^B | 用于array或struct中的元素，或map键值对之间的分隔 | \002
^C | 用于map中键和值之间的分隔 | \003
问题：如何在vim中输入^A？答：先crtl+V，再输入A

#### 显示指定字段分隔符

```
create table mytable(
    name string,
    salary float,
    ...
)
--指定字段间的分隔符，row format这一句必须写在所有子句之前
row format delimited
fields terminated by '\001',
--指定元素间的分隔符
collection items terminated by '\002',
--指定map中键与值之间的分隔符
map keys terminated by '\003',
--指定行之间的分隔符
lines terminated by '\n',
--缺省情况下默认使用textfile文件格式存储表信息
stored as textfile;

```

### 数据定义
#### 数据库操作
##### 创建数据库
```
create database mydb;
create database if not exists mydb;

create database mydb
--数据库默认存放在/user/hive/warehouse/中，其文件目录名以.db结尾
location '/my/preferred/directory'
comment 'holds all mydb tables'
```
##### 查看数据库
```
--列出数据库名
show databases;
show databases like 'my*';

--显示指定数据库的基本信息
desc database mydb;
--显示更详细的信息
desc database extended mydb;
```

##### 删除数据库

```
drop database mydb;

--先自行删除mydb库中的所有表，才能执行下面的命令删除mydb库
drop database if exists mydb;

--或者使用cascade关键字，将自行先删除mydb中的表，再删除mydb库
drop database if exists mydb cascade;
```

##### 修改数据库属性dbproperties
数据库名、数据所在的目录位置是不可更改的；可以设置键值对作为数据库的属性
```
alter database mydb set dbproperties('edited-by'='kaye');
```

#### 查看表

如果不指定数据库，hive会把表创建在default数据库下。指定数据库后查看表信息：

```
use mydb;
show tables;
show tables in mydb;
show tables in 'my*';
desc mytable;

--输出内容详细，可读性差
desc extended mytable;

--输出内容详细，可读性强
desc formatted mytable;

--只查看mytable表中的name字段的信息
desc mydb.mytable name;
```

#### 创建表

- **管理表**：Managed Table, 又称内部表。表数据存储在hive-site.xml的配置项hive.metastore.warehouse.dir定义的目录的子目录下。
- **外部表**：使用external关键字创建的表，就是外部表，否则就是内部表。drop内部表时会从HDFS上删除数据，而drop外部表不会。
- **分区表**：一个表可以拥有一个或多个分区，每个分区以目录的形式单独存放在表目录中。可以在创建表时创建分区，也可以在装载数据时创建分区
- **外部分区表**：外部表也可以分区

##### 创建管理表，以country和state分区，注意集合数据类型在表中的使用
```
create table if not exists employees(
    name string,
    salary float,
    subordinates array<string>,
    deductions map<string,float>,
    address struct<street:string,city:string,state:string,zip:int>
)
--设置表分区，按country和state分区
partitioned by (country string,state string)
row format delimited 
fields terminated by ','
location '/user/hive/warehouse/mydb.db/employees';
```

##### 对已存在的表进行表结构复制（而不会复制数据），创建外部表
```
create external table if not exists mydb.employees3
like mydb.employees
location '/path/to/data'
```
##### 使用show partitions 命令查询表分区
```
show partitions mydb.employees;
show partitions mydb.employees partition(country='US');
show partitions mydb.employees partition(country='US',state='CA');
```

##### 在装载数据时创建表分区，注意例子中对环境变量的引用

```
load data local inpath '${env:home}'/california-employees'
into table employees
partition (country='US',state='CA');
```

##### 查看分区数据所在路径

```
desc extended mydb.employees partition(country='US',state='CA');
```

#### 删除表
- 对于**管理表（内部表）**，表的元数据信息和表内数据**都会被删除**
- 对于**外部表**，表的元数据信息会被删除，**表数据本身不会被删除**
```
drop table if exists employees;
```

#### 修改表
大多数表属性通过alter table命令修改，该命令只能修改表的元数据，不能修改表数据本身

- 表重命名

```
alter table log_messages rename to logmsgs;
```


- 增加、删除、修改表分区，均需要为每个分区键指定值
```
--增加分区，必须为每个分区键指定值
alter table mydb.log_messages add if not exists
partition(year=2012,month=1,day=12) location '/logs/2012/1/12'
partition(year=2012,month=1,day=13) location '/logs/2012/1/13'
...;

--重新设置分区数据的存储路径，把不常用的旧日志放到廉价的AWS S3存储桶中
alter table mydb.log_messages partition(year=2012,month=1,day=12)
set location 's3n：//ourbucket/logs/2012/1/12';

--删除指定分区
alter table log_messages drop if exists partition(year=2012,month=1,day=12);
```

- 修改列信息：**alter table...change column**... after some_column
- 增加列：**alter table...add columns**(new_column string,new_column2 string)
- 删除或替换列：**alter table...replace columns**(new_column string,new_column2 string)
- 修改表属性： **alter table...set tblproperties**(...);
- 修改存储格式： **alter table...set fileformat** [TEXTFILE|SEUENCEFILE|...];
- 修改SerDe属性：**alter table...set SerDe**...
- 其他修改表的语句：touch, archive, unarchive,enable no_drop, enable offline

### 数据装载
#### 向管理表装载数据
- load data... 装载来自hdfs文件系统的数据
- load data local... 装载来自本地文件系统的数据
- 使用overwrite关键字，目标目录下之前存在的数据会被删除；如果不使用，则会保留原有的数据
```
load data local inpath '/export/data/dividends' 
overwrite into table dividends;
```
- 通过装载数据的方式创建分区
```
load data local inpath '/export/data/employees' 
overwrite into table employees
partition (country='US',state='CA'); 
```
#### 通过查询语句装载数据
- insert overwrite table ...  覆盖之前表或分区中已存在的内容
- insert into table ...     追加写入数据，不会覆盖之前已经存在的内容
- 静态分区插入：显示地标明每一个分区名称
- 动态分区插入：基于查询参数推断出需要创建的分区名称

##### 静态分区：单个分区
```
insert overwrite table employees
partition(country='US',state='CA')
select * from staged_employees se
where se.cnty='US' and se.st='CA';
```
上例中可能存在的问题：如果要对美国65个州依次执行上面类似的语句，就意味着要扫描staged_employees表65次！如果表很大，将十分低效！

##### 静态分区：多个分区

下面用另一种insert语法，只扫描一次staged_employees表，就可以完成所有州的数据装载。在这种语法中，insert overwrite 和 insert into 可以混合使用。

```
--只扫描staged_employees表一次，从该表读取的每条数据，都会经过一条select...where子句进行判断
from staged_employees se
insert overwrite table employees
    partition(country='US',state='CA')
    select * where se.cnty='US' and se.st='CA'
insert into table employees
    partition(country='US',state='IL')
    select * where se.cnty='US' and se.st='IL'
...
insert overwrite table employees
    partition(country='US',state='OR')
    select * where se.cnty='US' and se.st='OR';
```

##### 动态分区

根据select子句中字段的位置（而非命名）来匹配源表字段值与输出分区值
```
--根据select子句中的最后两列，确定分区字段country和state的值 
insert overwrite table employees
partition(country,state)
select...,se.cnty,se.st
from staged_employees se;
```

##### 混合模式
下面例子中country是静态分区字段，state是动态分区字段。注意，静态分区键必须出险在动态分区键之前。

```
insert overwrite table employees
partition(country = 'US',state)
select ...,se.cnty,se.st
from staged_employees se
where se.country='US';
```

#### 一句话完成创建表与加载数据
此功能只能用于内部表。常见的使用场景是：从一个大宽表中只抽取部分需要的数据集。

```
create table ca_employees
as select name, salary, address
from employees
where se.state='CA;
```

#### 从hive导出数据

##### 直接拷贝
hadoop fs -cp source_path target_path

##### 只需要包含特定字段的数据集

```
--overwrite换成into表示追加写入；local表示本地目录，去掉则指定外部url路径
insert overwrite local directory '/tmp/ca_employees'
select name, salary, address
from emoloyees se
where se.state='CA'
```

##### 一次性导出多个数据集文件

```
from staged_employees se
insert overwrite directory '/tmp/or_employees'
    select * where se.cty='US' and se.st='OR'
insert overwrite directory '/tmp/ca_employees'
    select * where se.cty='US' and se.st='CA'
...
insert overwrite directory '/tmp/il_employees'
    select * where se.cty='US' and se.st='IL';
```

##### 处理默认分隔符^A ^B ^C
- 以期望的分隔符（如，制表符）定义一个“临时表”
- 从源表抽取数据到“临时表”
- 通过insert overwrite directory将“临时表”中的数据导出到本地
- 删除“临时表”（Hive本身没有临时表，需要手动删除创建了的但又不想长期保留的表）