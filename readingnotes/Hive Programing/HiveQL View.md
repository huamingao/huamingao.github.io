#### 什么是视图

- 视图允许保存一个查询，像对待表一样对这个查询进行操作
- 视图是一个逻辑结构，不像表一样会存储数据
- 当一个查询引用一个视图时，逻辑顺序是，先执行视图，然后使用视图得到的结果进行用户定义的查询。
- 在Hive查询优化器的作用下，视图语句和查询语句可能会合并成一个单一的实际查询语句

#### 使用场景
- 使用视图降低查询复杂度
> 创建视图以替代嵌套子查询

```
--创建一个视图
create view shorter_join as
select * from people join cart
on (cart.people_id=people.id)
where firstname='john';

--查询视图
select lastname from shorter_join where id=3;
```

- 使用视图限制用户访问指定数据
>保护信息不被随意查询，通过视图，只将部分列值暴露出去

```
create view techops_employees as
select firstname,lastname,ssn from userinfo 
where department=='techops';
```

#### 基于同一个物理表，构建多个逻辑表（或视图）
Hive支持array, map, struct数据类型。因此可将一行文本作为一个map而非一组固定的列

#### 视图相关语句
- 按视图创建表
> create table shipments like myViewOfShipments;

- 删除视图
> drop view if exists shipments;

- 查看视图
> show tables语句能查看到视图，没有show views这种语句

- 更改视图
> 视图是只读的，只允许使用 alter view...set tblproperties(...) 修改元数据中tblproperties属性