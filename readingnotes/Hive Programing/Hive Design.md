#### Hive 模式设计
- 按天划分表：使用分区表来实现

```
create table supply(id int,part string,quantity int)
partitioned by(day int);

alter table supply add partition(day=20110102);
alter table supply add partition(day=20110103);

...装载数据...

select part, quantity from supply
where day>=20110102 and day<20110301 and quantity<4;
```

- 分区不宜过小
HDFS用于存储数百万大文件，而非数十亿小文件。过多的分区将导致创建大量非必须的Hadoop文件夹和文件。由于namenode是将所有系统文件的元数据存储在内存中，最终可能导致namenode对元数据信息的处理能力。