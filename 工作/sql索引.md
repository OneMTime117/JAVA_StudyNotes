# sql索引：



## 要提高SQL语句的执行效率，最常见的方法就是建立索引，以及尽量避免全表扫描

## 索引主要目的是提高了SQL Server系统的性能，加快数据的查询速度与减少系统的响应时间 

### sql索引分为两种，聚集索引、非聚集索引

#### 形象分析索引定义：（就对某个字段名，就行了某种类型的排序，如数字123，字母abc）

- ##### a开头的书，在第一排，b开头的在第二排，这样在找什么书就好说了，这个就是一个聚集索引

- ##### 图书管理员在写一个目录，某某作者的书分别在第几排，第几排，这就是一个非聚集索引

- ##### 聚集索引存储记录是物理上连续存在；而非聚集索引是逻辑上的连续，物理存储并不连续

- ##### 聚集索引一个表只能有一个，而非聚集索引一个表可以存在多个



无索引的表，查询时，是按照顺序存续的方法扫描每个记录来查找符合条件的记录，从头到尾扫描，这样效率十分低下

### 聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致；聚集索引就是这样的，他是和表的物理排列顺序是一样的;而非聚集索引就和表的物理排序顺序不同



### 索引设置原则：

| 动作描述                   | 使用聚集索引 | 使用非聚集索引 |
| -------------------------- | ------------ | -------------- |
| 外键列                     | 应           | 应             |
| 主键列                     | 应           | 应             |
| 列经常被分组排序(order by) | 应           | 应             |
| 返回某范围内的数据         | 应           | 不应           |
| 小数目的不同值             | 应           | 不应           |
| 大数目的不同值             | 不应         | 应             |
| 频繁更新的列               | 不应         | 应             |
| 频繁修改索引列             | 不应         | 应             |
| 一个或极少不同值           | 不应         | 不应           |

1) 定义主键的数据列一定要建立索引。

2) 定义有外键的数据列一定要建立索引。

3) 对于经常查询的数据列最好建立索引。

4) 对于需要在指定范围内的快速或频繁查询的数据列;

5) 经常用在WHERE子句中的数据列。

6) 经常出现在关键字order by、group by、distinct后面的字段，建立索引。如果建立的是复合索引，索引的字段顺序要和这些关键字后面的字段顺序一致，否则索引不会被使用。

7) 对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

8) 对于定义为text、image和bit的数据类型的列不要建立索引。

9) 对于经常存取的列避免建立索引 

9) 限制表上的索引数目。对一个存在大量更新操作的表，所建索引的数目一般不要超过3个，最多不要超过5个。索引虽说提高了访问速度，但太多索引会影响数据的更新操作。

10) 对复合索引，按照字段在查询条件中出现的频度建立索引。在复合索引中，记录首先按照第一个字段排序。对于在第一个字段上取值相同的记录，系统再按照第二个字段的取值排序，以此类推。因此只有复合索引的第一个字段出现在查询条件中，该索引才可能被使用,因此将应用频度高的字段，放置在复合索引的前面，会使系统最大可能地使用此索引，发挥索引的作用。

 

## 创建索引：

CREATE [UNIQUE][CLUSTERED | NONCLUSTERED]  INDEX  index_name  

ON table_name （Index_property，....）

说明：

UNIQUE: 建立唯一索引

CLUSTERED: 建立聚集索引

NONCLUSTERED: 建立非聚集索引

 index_name    索引名

table_name     表名

Index_property: 索引属性(可以设置几个索引属性)

UNIQUE索引既可以采用聚集索引结构，也可以采用非聚集索引的结构，如果不指明采用的索引结构，则SQL Server系统默认为采用非聚集索引结构。



## 删除索引：

DROP INDEX table_name.index_name ，。。。

说明：table_name: 索引所在的表名称。

index_name : 要删除的索引名称。

可以一次删除多个索引

## SQL语句优化：



###   1、避免在where子句中使用 is null 或 is not null 对字段进行判断

如：

select id from table where name is null

在这个查询中，就算我们为 name 字段设置了索引，查询分析器也不会使用，因此查询效率底下。为了避免这样的查询，在数据库设计的时候，尽量将可能会出现 null 值的字段设置默认值，这里如果我们将 name 字段的默认值设置为0，那么我们就可以这样查询：

select id from table where name = 0



### 2、避免在 where 子句中使用 != 或 <> 操作符

如：

select name from table where id <> 0

数据库在查询时，对 != 或 <> 操作符不会使用索引，而对于 < 、 <= 、 = 、 > 、 >= 、 BETWEEN AND，数据库才会使用索引。因此对于上面的查询，正确写法应该是：

select name from table where id < 0

union all    操作符用于合并两个或多个 SELECT 语句的结果集，UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。（union all  和union区别：union all 会显示所有合并结果的值，union 会将相同的值只显示一个）

select name from table where id > 0

这里我们为什么没有使用 or 来链接 where 后的两个条件呢？这就是我们下面要说的第3个优化技巧。

### 3、避免在 where 子句中使用 or来链接条件

如：

select id from tabel where name = 'UncleToo' or name = 'PHP'

这种情况，我们可以这样写：

select id from tabel where name = 'UncleToo'

union all

select id from tabel where name = 'PHP'

### 4、少用 in 或 not in

虽然对于 in 的条件会使用索引，不会全表扫描，但是在某些特定的情况，使用其他方法也许效果更好。如：

select name from tabel where id in(1,2,3,4,5)

像这种连续的数值，我们可以使用 BETWEEN AND，如：

select name from tabel where id between 1 and 5

### 5、注意 like 中通配符的使用

下面的语句会导致全表扫描，尽量少用。如：

select id from tabel where name like'%UncleToo%'

或者

select id from tabel where name like'%UncleToo'

而下面的语句执行效率要快的多，因为它使用了索引：

select id from tabel where name like'UncleToo%'



### 6、避免在 where 子句中对字段进行表达式操作

如：

select name from table where id/2 = 100

正确的写法应该是：

select name from table where id = 100*2



### 7、避免在 where 子句中对字段进行函数操作

如：

select id from table where substring(name,1,8) = 'UncleToo'

或

select id from table where datediff(day,datefield,'2014-07-17') >= 0

这两条语句中都对字段进行了函数处理，这样就是的查询分析器放弃了索引的使用。正确的写法是这样的：

select id from table where name like'UncleToo%'

或

select id from table where datefield <= '2014-07-17'

也就是说，不要在 where 子句中的 = 左边进行函数、算术运算或其他表达式运算。



### 8、在子查询中，用 exists 代替 in 是一个好的选择

如：

select name from a where id in(select id from b) 

如果我们将这条语句换成下面的写法：

select name from a where exists(select 1 from b where id = a.id)

这样，查询出来的结果一样，但是下面这条语句查询的速度要快的多。  