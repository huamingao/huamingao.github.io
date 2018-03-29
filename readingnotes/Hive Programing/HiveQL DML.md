## 数据查询示例

### where子句

```
select a.ymd,a.price_close,b.price_close
from stocks a 
JOIN stocks b 
on a.ymd=b.ymd
where a.symbol = 'IBCP' 
and b.symbol='IBM';
```

```
select name,address.street 
from employees 
where address.city like 'Chi%';
```

谓词操作符
- **Like**：SQL下的正则表达式，x% 表示必须以x开头，%x表示必须以x结尾，%x%表示包含x，可位于开头、结尾或字符串中间
- **Rlike**：Java接口实现的正则表达式

### join...on...语句
总是先执行Join语句进行表连接，然后再把结果通过where语句进行过滤。基于次特性，思考如何进行HiveQL语句的性能优化。
- **join**(inner join): 交集，匹配的数据在两个表中都存在，才会保留在结果集中
- **full outer join**: 并集，返回左表和右表中所有符合条件的数据，不匹配的字段值用null填充
- **left outer join**: 并集，只返回左表中所有符合条件的数据，右表不匹配的字段值用null填充
- **right outer join**：并集，只返回右表中所有符合条件的数据，左表不匹配的字段值用null填充
- **left semi join**：左半开连接,在右边表满足on语句的判定条件后，返回左边表的记录； select和where不能引用到右边表的字段
- **笛卡尔积**(cross join)：默认不允许执行，除非设置配置项hive.mapred.mode=unstrict。 eg. select * from stocks join dividends
- **mapjoin**(tablename)：要连接的表中有一张表很小，可以放到内存中，只在map端执行连接过程，省略掉reduce过程，从而实现查询语句的优化
- reducejoin(tablename)

### 分组、排序
- **group by**：通常与聚合函数一起使用，按一个列或多个列进行分组，然后对每个组执行聚合操作。后接having语句，可少写一个嵌套子查询
- **order by**：全局排序，意味着所有数据都通过一个reducer处理。在数据量大的情况下，执行时间漫长，低效
- **sort by**：局部排序，在每个reducer内部执行排序。这样可以保证每个reducer的输出结果是有序的，之后再进行全局排序，会更有效率
- **distribute by**：控制maper的输出被分发到同一个reducer中进行处理。通常后面接着sort by语句，避免不同reducer的输出内容发生重叠，从而保证全局有序。distribute语句必须放在sort by语句之前。
- **cluster by**：如果distribute by...sort by...这两个语句中涉及的列完全相同，而且采用的是默认的升序排序，就可以用cluster by语句代替这两个语句。两种形式是等价的。这两种形式的语句会剥夺sort by的并行性，但能够保证输出数据是全局排序的。


### 分桶表与抽样查询：tablesample
对于非常大的数据集，只想求得一个具有代表性的查询结果，而非对全部数据操作。这种情形，可以进行分桶抽样

分桶语句中的分母M表示，数据会被散列在M个桶中；分子N表示，将会选择N个桶中的数据进行查询。

可以使用rand()函数进行抽样，该函数会返回一个随机值，使得每次执行同一查询语句，得到的结果不同。如果用指定的列替代rand(),则每次执行得到的结果是相同的。

```
--分10个桶，抽取2个桶中数据
select * from numbers tablesample(bucket 2 out 10 on rand()) s;

--抽取百分之十的数据作为样本
select * from numbers tablesample(0.1 percent) s;
```

### union all
将2个或更多个表合并成一个表。每一个Union子查询语句都必须具有相同的列，并且各字段的字段类型一致



## 示例说明

```
--outer join
select s.ymd,s.symbol,s.price_close,d.dividend
from dividends d
full outer join stocks s
on d.ymd = s.ymd
and d.symbol = s.symbol
where s.symbol='IBM';
```

```
-- left semi join
select s.ymd, s.symbol, s.price_close
from stocks s
left semi join dividends d
on d.ymd = s.ymd
and d.symbol = s.symbol
where s.symbol='IBM';
```

```
--map side join
select /*+ mapjoin(d) */ s.ymd, s.symbol, s.price_close, d.dividend
from stocks s 
join dividends d 
on s.ymd = d.ymd and s.symbol=d.symbol
where s.symbol='IBM';
```

```
--嵌套select语句
select s.ymd,s.symbol,s.price_close,d.dividend from
(select * from dividends where symbol = 'AHGP' and exchanger = 'NASDAQ') d
left outer join
(select * from stocks where symbol = 'AHGP' and exchanger = 'NASDAQ') s
on d.ymd = s.ymd;
```

```
--orderby 全局排序（降序）
select d.ymd, d.symbol, d.dividend
from dividends d
where d.symbol like 'IB%'
order by d.ymd,d.symbol;
```

```
--sort by局部排序（降序），由于只有一个reducer,其查询结果与上面的orderby类似
select d.ymd, d.symbol, d.dividend
from dividends d
where d.symbol like 'IB%'
distribute by d.ymd,d.symbol
sort by d.ymd,d.symbol;
```


```
--clusterby在升序排列并且涉及列完全相同的情况下，等价于distributeby...sort By...
select d.ymd, d.symbol, d.dividend
from dividends d
where d.symbol like 'IB%'
cluster by d.ymd,d.symbol;
```

