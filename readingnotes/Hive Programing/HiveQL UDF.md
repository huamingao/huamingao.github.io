- 内置函数
- 用户自定义函数（UDF，User Defined Function）

一个不成文习惯是，人们常常用 UDF 指代任何函数，包括内置的和自定义的。

#### 发现和描述函数

```
--显示所有函数名称，包括内置函数和用户自定义函数
show functions;

--可简写为desc
describe function concat;

--更详细的描述信息，包括一个examaple
describe function extended concat;
```

#### 调用函数
```
select concat(address.street,' ',address.city) as city_address from employees;
```

#### 函数分类
- 标准函数
> 以每行数据中的一个或多个字段值作为传入参数，最终返回一个值的函数。如大多数数学函数：abs(), round(), floor()...又如许多字符串操作函数: reverse(), concat() 

- 聚合函数
> UDAF，User Defined Aggregation Function, 用户自定义聚合函数，接受从零行到多行的零个到多个列，然后返回单一值。如数学函数 sum(), avg(), min(), max()... 通常和 group by 一起使用。

- 表生成函数
> UDTF, User Defined Table Function, 用户自定义表生成函数，接受零个或多个输入，然后产生多列或多行输出。例如，array()将一列输入转换为一个数组输出, explode()以array 类型数据作为输入，对数组中的数据迭代，逐行返回每个数组元素值。注意，UDTF 不支持从表中产生其他的列或行。

```
--以下查询将报错
select name, explode(subordinates) from employees;

--可用Laterral view 来实现类似功能
select name, sub
from employees
lateral view explode(subordinates) subView as sub;
```
> Lateral view 可以将 explode 得到的行转列的结果集合在一起，需要指定视图别名，和生成的新列的别名，在本例中分别是 subView 和 sub 。
