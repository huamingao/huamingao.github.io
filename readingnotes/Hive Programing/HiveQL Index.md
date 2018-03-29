### 创建索引
除了S3中的数据，对外部表和视图都可以建立索引

```
create index employee_index
--对分区字段country建立索引
on table employees(country)
--指定索引处理器，实际是一个实现了索引接口的Java类
as 'org.apache.hadoop.hive.ql.index.compact.compactIndexHandler'
--表示新索引将呈现空白状态
with deferred rebuild
--索引属性
idxpropertiest('creator'='me','created_at'='some_time')
--并非一定要求索引处理器在一张新表中保留索引数据
in table employees_index_table
partitioned by(country, name)
row format ...
stored as ...
location ...
comment 'employees indexed by country and name.'
```

### bitmap索引
内置的bitmap索引处理器，普遍应用于排重后值较少的列

### 重建索引

```
alter index employees_index
on table employees
partition(country='US')
rebuild;
```


### 显示索引

```
show formatted index on employees;
```

### 删除索引
- drop table之前，不允许drop index
- drop table之后，其对应的索引和索引表也会被删除
- 删除索引后，其对应的索引表也会被删除
- 删除某个分区后，其对应的分区索引和索引表也会被删除

```
drop index if exists employees_index on table employees;
```

### 实现定制化的索引处理器
