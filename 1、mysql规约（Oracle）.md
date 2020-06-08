# 1、mysql规约，基于mysql5.7.25

## 1、mysql的库名、表名、字段名、字符类型字段值大小写区分

mysql默认情况： 

在windows下，所有库名、表名、字段名、字符类型字段值不区分大小写

在linux下，库名、表名严格区分大小写；字段名、字符类型字段值不区分大小写

mysql不区分大小写，其实是对于数据在硬盘中的储存使用小写，然后强制将sql中的所有内容转化为小写进行匹配

mysql规范要求：对于mysql建库、建表时，库名、表名、字段名全部使用小写；对于字符类型字段值，一般应区分大小写，因此通过设置字符集的排序规则来控制：

要求使用utf-8编码，排序规则为utf8_bin、utf8_genral_cs(在mysql5.6.10后就不再支持）从而区分字符类型字段值的大小写。当需要强行要求不区分大小写时，排序规则使用utf-8_genral_ci

**####Oracle**：

**Oracle数据库表名和字段名全部大写（因为Oracl默认会将sql语句全部使用大写，除被“ ”括起来的；若强行使用小写命名，表名或字段名在使用时需要添加“ ”，并JDBC中拼写sql语句时，使用 \\" 转义字符来代替 “）**

**Oracle数据库默认所有字段数据中的字符区分大小写**

## 2、mysq常用字段的数据类型及使用场景

mysql分为三大类数据类型：数值类型、时间/日期类型、字符串类型

​	1、整数数值类型：分为有符号和无符号，即是否有负数（默认为有符号，取值范围减半）

![由此可知1566201801526](C:\Users\OneMTime\Desktop\Typora图片\1566201801526.png)

由此可知整数数值类型的大小是固定的，设置的其长度并不是限制数据的大小，而是用于设置数值显示宽度（显示宽度会在数据位数小于该值时，在左侧使用0填充；但是需要设置该字段填充0选项，并且只适用于无符号数值字段）

**一般情况下，对于整数数值类型的长度使用0，在保存时会默认转化为对于数值类型位数大小**

**对于字段实际中为非负数时，一定设置为unsigned**（无符号）

根据需要的整数大小进行选择，一般默认使用int类型（默认有符号），取值范围为+- 21亿

在实际应用中，常见场景：人寿命使用 tinyint 无符号类型 0-255；超过21亿的整数数值使用bigint

​	2、浮点数&定点数数据类型

![1566204814441](C:\Users\OneMTime\Desktop\Typora图片\1566204814441.png)

当浮点数有效位数在7位以内时，可以使用float；当浮点数有效位数在16位以内时，可以使用double；因此长度也需要小于等于7或16，小数点小于等于长度
**由于float、double都存在固定有效位的精度，因此禁止使用浮点数**

decimal定点数数据类型，根据指定的长度M（精度，总长度）和小数点D（标度，小数点后的长度），来精确指定数值的精度。

**decimal的数字最大长度M为65，当存储数字小数点前的长度大于M-D，则会出现数据超超出最大值的警告**

**一般出现较大长度的小数时，使用decimal来确保数据计算的精度（如货币，一般使用decimal（14，2））**

​	3、时间/日期类型

year：1901-2155   长度默认为4

date：‘YYYY-MM-DD’ ，长度默认为0（无实际意义）

time：‘HH:MM:SS’,长度自定义，用于限制s后面的小数点位数，一般设置为0（最多6位）

**datetime：‘YYYY-MM-DD HH:MM:SS’，最常使用的日期格式，8个字节存储，长度一般为0（和time一致）**

timestamp：格式和datetime一样，4个字节存储，存储时间范围小一点，并且存储的时间会根据当前时区将其转化为UTC（时间标准时间）格式保存，进行查询时，会根据查询系统的当前时区进行转化。

​	4、常用字符串类型

**char   固定长度字符串，长度最大值为255；且不会根据编码的不同最大长度改变（长度是指的的字符长度）；若存储数据的长度不够，则使用空格在后面填充。**

**varchar 可变长度字符串，支持65535个字节，但实际只有65533，需要两个字节记录数据大小**

varchar根据编码格式的不同，可设置的最大长度也不一样，一般常用：

utf8中，一个字符占3个字节，所以L = 65533/3 = 21844
gbk编码中一个字符占2个字节，所以L = 65533 / 2 = 32766

1. 如果数据长度都一样，就用char，比如身份证号、手机号
2. 如果数据长度不一样，就用varchar，比如名字、地址，但要保证不越界
3. 定长的磁盘空间比较浪费，但效率高
4. 变长的磁盘空间比较节省，但效率低

由于Varchar查询效率低，因此当最大长度超过5000时，应使用text文本类型，并且为独立的一张表，通过主键进行对应，避免影响其他字段的索引效率

​	5、文本字符串类型

![1566269730401](C:\Users\OneMTime\Desktop\Typora图片\1566269730401.png)

text类型基本和varchar类型相同，但它不能指定最大长度，而是默认使用固定的最大长度。因此一般能用varchar时，不用text。

只有当不知道数据长度，或者长度超过varchar的最大值时，就使用text类型

​	6、二进制类型

bit  位二进制，指定二进制的位数，最大64位

binary、varbinary 对应char、varchar，只是它们是以二进制的形式储存，并且指定的长度为字节数（最大为255，65535）

BLOB二进制大对象，和text类型类似，指可变的二进制存储，但不能指定最大长度字节数，而是使用固定的最大长度；一般使用于二进制大小无法确定。

![1566270966243](C:\Users\OneMTime\Desktop\Typora图片\1566270966243.png)

binary、varbinary设计是为了代替char、varchar对字符串进行二进制的存储，但实际中没有这种需求，并且二进制无法体现数据的可视性。

而BLOB可用于文件的存储，如多媒体文件（图片、音乐、视频）：blob 56kb，mediumblob 16MB，longblob 4GB

**合理选择数据类型长度，可以节约数据库表空间、对应索引存储，最重要是提升检索速度**

## 3、数据库库名、表名、字段名命名规范

- 库名应体现对于项目名称
- 表名命名格式为  业务名称_表的作用，并且不能出现复数名词
- 建表时，必须包含id, gmt_create, gmt_modified三个字段：id为主键，单表时自增；gmt_create，代表表建立的时间；gmt_modified,代表改行数据最近修改时间
- 对于表达是与否概念的字段，使用is_xxx命名，并且数据类型为unsigned tinyint（1代表是，0代表否）

- 对于库名、表名、字段名尽量不用使用myslq的保留字（内置sql关键字），不然可能出现sql歧义而报错，无法避免时使用 反引号（`）扩起来；因此推荐使用 xxx_xx的格式命名，这样有效减少可能出现使用保留字的概率
- 若表为后台创建时，表名添加 _b的后缀，代表back（后台）
- 唯一索引名为uk _ 字段名；普通索引名为idx _ 字段名；组合索引为  字段名_字段名

## 4、数据库存储引擎

mysql 5.5版本之前，默认使用 MyISAM，不支持事务管理，提供高速存储和检索，支持全文搜索，只支持表级锁

mysql5.5版本后，默认使用 InnoDB，提供了事务、行级锁机制和外键约束的功能，但对比MyISAM的存储引擎，InnoDB写的处理效率差一些，并且会占用更多 的磁盘空间以保留数据和索引

## 5、数据库表约束建立规范

- 数据库唯一约束（unique）：

  1、建立字段唯一约束时，会自动创建字段的唯一索引，通过唯一索引来实现唯一约束。因此唯一约束就是创建唯一索引，不需要额外对唯一约束的字段添加索引，默认使用BTree索引方式；

  2、唯一约束后，还是可以存在多个null值

  3、**唯一约束实际中需要时，应该直接在数据库中加上**；虽然可以通过“先查后插”的代码方式实现，但会影响速度，并且在高并发时，无法保证插入数据的唯一性。

  **###在开发中，我们常常使用“先查后插”的代码方式，但只是为了提供用户交互性（如用户注册，我们会先判断用户名是否存在，这样不需要分析重复插入时，数据库由于唯一约束报出的异常）**

- 数据库主键约束（primary key）：

   	1、不能为null，数据唯一的字段才能标记为主键，用于表示表每行数据的唯一性，来限制数据存储的约束。

      	2、主键在一张表中只能存在一个，但可以由多个字段组合而成（组合主键），此时可以允许多个字段出现相同的值，但绝对不能出现所有字段出现两组相同数据。

  ​     3、主键会默认建立唯一索引（也叫做主键索引），当使用主键作为查询条件、排序时，会加快查询速度；因此不需要对主键添加额外的索引，索引默认使用BTree索引方式。

- 数据库非空约束和默认约束（not null 、Default）：

  ​	1、insert语句跳过某个字段时，会使用默认值（null）填充；当使用非空约束后，无默认值，需要手动指定默认值，否则insert语句无法跳过该字段；因此使用非空约束后，一定要搭配默认约束（防止添加数据时，该字段无法跳过）

  ​	2、**表字段应尽量设置非空约束，并使用默认约束**，默认值为：0、特殊字符或者空字符；原因有：

  ​		1、可null的字段在建立索引时，每行数据的索引多一个字节来存储是否为null，增加索引占用空间

  ​		2、null数据在计算时，都会返回null，如在not in（null）查询结果永远为空，做为查询条件时，容易出错

  ​		3、当使用null后，在进行条件判断时，就会用到 is null、is not null的查询条件，此时查询会由于数据中 null的占比，从而有一方不容易命中索引（全表效率和走索引差不多）；

- 数据库是否要建立外键约束：

  ​	1、外键的作用是确保数据的完整性，当主键数据变化时，若出现外键数据找不到对应的主键数据时，数据库就会自动阻止主键数据的修改
  ​	2、外键的最大作用是，外键数据和主键数据相同值使用的一个物理空间，在表数据非常多时，创建外键可以节省物理空间
  ​    3、外键由于被数据库维护，每次修改主键都需要对另一个表检查数据，需要额外的锁，因此在高并发下会出现死锁

  **因此为了减少数据库维护外键的开销，一般不使用外键**

## 6、数据库索引建立规范

优点：数据库索引的建立可以大大提高数据库的检索数据

缺点：1、建立索引需要额外的表空间

​			2、在更新数据时，数据库需要花费额外的时间对索引进行更新，保证数据和索引的一致性

**在以下条件下，不推荐建立字段的索引：**

- **表数据量很少时**（低于300条数据），因为此时全表扫描的成本和使用索引的成本相近，多此一举

- **表字段数据区分度不高时**（根据使用场景不同来判断，一般需要高于0.1才使用索引），因为此时索引需要扫描的数据依然很多（大量相同数据导致索引的数量很少，而对应数据块中需要匹配扫描的数据很多）。如性别，当使用索引时，还是需要扫描一半的数据。
- **表字段不作为或很少作为查询条件时**，因为此时该字段索引不会被使用，反而加大数据更新的成本
- **经常更新的字段**，此时实际对于写入速度需求远大于查询速度需求
- **对于数据量太多的字段，**此时建立索引的成本太高

**常见建立索引的字段有：**主键、连接查询时的关联字段（外键）、排序、分组和去重字段、经常需要作为条件查询的字段

**对于需要进行大量更新的表，但又有一定的查询需求时，应合理规划索引个数，尽量不超过3-5个**

## 7、数据库索引类型选择

**按照索引类型分类有：**

- NORMAL   普通索引        提高查询数据速度

- UNIQUE     唯一索引        提高查询速度的同时，保证数据的唯一性（用于创建唯一约束），可以为null

- FULLTEXT  全文索引        普通索引对于like查询，只支持左边前缀查询（xxx%），因此对于其他形式的like查询都是进行全表扫描，速度很慢；此时就需要使用全文索引来完成类似于 like %xx%的查询，有更快的查询速度
  全文搜索的语法：MATCH(col1,col2,…) AGAINST (“xxx”)，match中的字段需要时建立了FULLTEXT索引

  缺点：全文索引建立所消耗的时间、空间很大，因此不适合大数据量的表；推荐不要一开始就建立，当数据量稳定、进行项目使用时，再创建

- FULLTEXT  全文索引        

- SPATIAL      空间索引       只能用于创建空间数据的索引，InnoDB引擎不支持空间数据，无法使用

**mysql索引方式有（默认InnoDB引擎）：**

BTree（B+数索引）：自定义索引时，默认只能使用该索引

Hash（哈希索引）：该方式是自适应的，不能手动指定使用哈希索引；只有当该索引使用非常频繁时，才会在BTree索引基础上创建哈希表（使查询速度更快）

全文索引：用于在FULLTEXT索引类型中使用

**组合索引：相当于是对于单列索引的扩展，适用于多条件查询**

## 8、数据库索引使用规范

- **索引长度**

索引总长度和编码方式、字段是否可null、字段索引长度、字段类型有关

**在utf-8下，索引总长度=字段索引长度*3+1（可为nll时，null标识符）+2（varchar字段长度可变时，2个标识符）**

强制在varchar字段建立索引时，应设置字段的索引长度（默认字段索引长度和字段最大长度一致）；合理的设置索引长度，能有效的在使用索引增加查询速度的同时，减少索引建立带来的负面影响。

**即索引在保证区分度的情况下，长度尽可能小一点，**varchar字段一般要保证在最小索引长度下，区分度达到90%

**通过使用sql语句：select count(distinct left(name, 20))/count(*) from test     来判断此时name字段索引长度为20的区分度**

- **索引使用**

**1、mysql一个查询只能使用一个索引，因此多条件查询时应使用组合索引**

**2、对应于索引字段进行运算（或函数）时，会导致索引失效**

**3、< , <= , = , >, >= ,between,in，not in,以及某些时候的like(不以通配符%或_开头的情形)才能使用索引，<>（ !=都是不等于 ）不能使用**

**4、当多个索引（单列、组合）同时存在该查询时，优化器会自动选择最有效的索引**

**5、尽量不要使用or，这样会导致索引失效，进行全表扫描**

**6、当可以使用索引的成本低于全表扫描时，优化器会自动不使用索引，而是进行全表扫描（如in、not in、is null  、is not null）**

单列索引的正确使用：

**对于排序、分组和去重字段，最好和优化器选择使用的索引字段一致，否则无法使用索引来优化排序、分组和去重**

组合索引的正确使用：

1、组合索引可以有由多种索引组合，满足最左前缀原则，如建立name-password-age三列的组合索引，可有组合：

index（name）、index（name，password）、index（name，password，age）

**因此组合索引可以替代单列索引的建立，从而满足单条件和多条件的查询**

2、组合索引的建立是有有序性的，并满足左前缀原理：

- 只有条件中出现左边全部字段，后面的字段才能使用组合索引；

- 当前面字段出现范围查询后，后面字段也应该为范围查询（否则后面字段就不能使用）；因此进行单独值匹配的字段应在前面，范围查询在后面；
- 由于组合索引可以满足其他组合形式，因此频繁查询的字段应放在前面
- 当前面字段出现范围查询时，后面字段就无法利用组合索引的有序性（但前面都为独立值匹配时，此时该字段可以进行范围查询，并且使用索引排序、分组、去重）

3、组合索引在使用是，优化器会按照组合索引建立的字段顺序来调整where条件顺序（但是推荐按照索引建立的顺序编写where条件，提高sql的可阅读性（展现where条件的优先级））

4、组合索引一般不用超过4个字段，否则就考虑建立多个索引

## 9、数据库表设计三范式

第一范式：

1、保证每一列字段数据的原子性：在应用中不需要对其进行进一步的分割，来获取需要的数据

​	如对于地址，可以分为省、市、区、街道、详细地址

2、当两个字段数据类型相近或一样，且具有原子性时，应将它们合并一起保存，以某种格式进行分割显示

​	如购物物品，会出现物品1、物品2...其实可以存储在一个字段中，用逗号分隔

第二范式：

1、表的每一行数据，都是关系紧密，不可分割的。它们共同描述了一个事物。当多行数据出现多个字段属性重复时，应考虑使用另一张表来描述这些字段。

如订单表，会出现一个人有多个订单，此时就会出现该用户所有信息字段重复，造成数据冗余；此时应创建一个用户表来保存所有用户信息，使用其用户ID来对于订单中的用户编号进行关联

这样既可以减少数据直接的复杂关系逻辑，又避免了数据冗余，便于数据库维护更新

第三范式：

表所有字段都是和主键有直接关系而不是间接关系，否则应将间接关系的字段另外建表

如学生表，姓名、学号、学校、年龄是直接关系，而学校地址、学校电话是间接关系。因此需要建立学生表和学校信息表两张表进行关联查询

三大范式只是一般设计数据库的基本理念，可以建立冗余较小、结构合理的数据库。但在实际中，我们需要优先从需求和性能出发，不能一味满足数据库三范式。

如为了保证第三范式的代价太大时，可以牺牲数据冗余代码的存储增加，从而减少联合查询的性能损失

# 2、sql语法

## 1、使用mysql数据库常用命令行：

登入数据库(可以直接获取数据库版本信息)：     mysql    -u  root   -p   （下一步输入密码）

查看所有数据库：    show databases

连接使用一个数据库：   use  数据库名

设置客户端查询数据的字符集：  set names utf8（控制台打印使用的GBK编码，因此每次进入数据库默认GBK，修改为utf8时乱码）

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION  允许任意主机IP使用root账号访问（mysql默认root账号只能为localhost；也可以注释mysql.cnf文件中的bind-address字段）

通过 select user,host from user 可以查看当前所有账号，允许使用的ip；%代表所有

修改密码：mysqladmin -u用户名 -p旧密码 password 新密码

启动、停止mysql服务：  （在管理员权限下） net stop mysql    ,net  start mysql

## 2、sql基本语法

### 1、sql语句中的所有库名、表名、字段名、别名应该小写；所有sql关键字应该大写

#### 2、简单sql语句：

- 简单查询语句：  SELECT    *   FROM     table_name     

- 简单条件查询语句:   SELECT *  FROM table_name  WHERE   column_name ='CN'

  where子句使用的运算符有：=、<>(不等于，可写为!=)、>、<、>=、<=、BETWEEN、like、in（not in）、is null（is not  null）；逻辑运算符有：and、or

- 数据插入语句：  INSERT INTO table_name(column1,column2)   VALUES (value1,value2,value3)

  mysql支持  INSERT INTO table_name(column1,column2)   VALUES (value1,value2,value3)，（value1,value2,value3）语句，来实现批量插入（oracle不支持该语句）

- 数据更新语句：  UPDATE  table_name  SET   column1=value1，column2=value2  WHERE  some_column=some_values

- 数据删除语句：   DELETE  FROM table_name  WHERE some_column=some_values

  **数据更新、删除语句可以不加where条件，这样会对全表数据进行操作**

#### 3、常用功能查询语句：

- 去重查询语句：   SELECT   DISTINCT   column_name1,column_name2   FROM  table_name

  distinct不会忽略null；

  对多个字段进行操作时，相当于筛选出所有字段组合在一起不会重复的数据；

  可以配合count（）、SUM()等聚合函数使用，对所有去重数据进行函数操作；但只要出现去重字段中有null的，就会不对该组数据统计

- 排序查询语句：  SELECT *   FROM  table_name   ORDER  BY   column   DESC

  asc  为升序，desc  为降序；当不明写asc、desc 时，默认使用asc；

  可以对多个字段进行排序，根据字段顺序来确定排序的优先级

- 数据行数限定查询语句：

  对于不同数据库，使用的sql语句和实现方式各有不同：

  mysql：SELECT *  FROM  table_name LIMIT   number1,number2   number1 限定起始行数，number2  限定从起始行数开始，可获取的最大数据条数（不指定时，默认获取起始行数后的所有数据）

  Oracle：SELECT * FROM table_name WHERE rownum<number  直接对查询数据的行数（rownum）进行where条件限定，来选择需要的行数数据

  SQL Server/MS Access : SELECT TOP 50 *  FROM table_name 获取前50条的数据

- 模糊查询语句： SELECT * FROM table_name WHERE column  like '%ni%'

  模糊查询使用通配符来实现：%  代表（0到多个字符）、_代表一个字符、[a,b] 代表它们其中的一个字符、[! a,b]或者[ ^ a,b]  代表不为它们其中的一个字符

- 分组查询语句 ：SELECT name,COUNT(name) FROM table_name GROUP BY name having  name =value

  group by 一般查询  分组的字段和分组字段的count（）函数，但是还可以使用其他聚合函数（关于其他字段的聚合函数，聚合函数只会计算对应组的该字段数据），但一定不能查询其他字段（这样不符合分组机制）

- 函数查询语句：

  sql函数分为两种：Aggregate（聚合） 函数、Scalar（标量，非聚合） 函数

  1、聚合函数：计算一列中的所有数据，返回一个单一数据;**当查询语句中使用聚合函数时，就不能查询表字段（但是group by 除外，可以查询分组字段）；聚合函数不能直接在where子句中使用，而是通过子查询来配合使用**

  AVG()  、MAX()、MIN()、SUM()   不会计算为null的行 ，但当字段所有值都为null时，会返回null；因此应注意这些聚合函数的NEP问题：可以通过sql中的IF、ISNULL函数避免返回null  ：

  ````java
SELECT IF(ISNULL(SUM(age)),0,SUM(age))  FROM test 
  ````

  COUNT() 返回行数，COUNT(column)时，不计算为null的行，当字段值都为null时，会返回0；

  COUNT(1)、COUNT(*)返回所有行，即使所有列为null；
  
  **强制使用COUNT(*)来统计所有行，为了符合sql定义的标准语法**
  
  2、非聚合函数：对单个值进行计算
  
  -  日期时间类函数（）：
  
    - 获取当前日期时间：now（）、sysdate（）  格式为：2019-08-29 09:46:24
  
    **两种区分在于：now（）是在执行sql开始时获取，sysdate（）是执行函数时获取；因此sysdate（）可能在一个sql中出现多次时，值不一样；但是now（）一样**
  
    - 获取当前日期：curdate（）、current_date()
    
    - 获取当前时间：curtime（）、current_time()
    
    - **获取当前时间戳：TIMESTAMP(date/time），当为date时，会自动将时分秒补0**
    
    - 获取时间、日期、时间日期表达式中的信息：
    
      day（d） 日期中的day（该月的第几天）
    
      month（d）日期中的month（第几个月，从1-12，java是0-11）
    
      year（d）日期中的year
    
      weekday（d）该日期为星期几（0代表星期一）
    
      weekofyear（d）该日期为本年的第几个星期
    
      hour（t）获取时间中的小时值
    
      minute（t）获取时间中的分钟值
    
      second（t）获取时间中的秒值
    
    - 对日期时间进行计算：
    
      datediff(d1,d2)   计算两日期差值 d1-d2 单位为天数    当使用datetime时，只计算日期
    
      timediff（t1,t2）计算两时间差值 t1-t2  单位为time    当使用datetime时，会将日期一起计算
    
      对时间日期增加一个时间间隔：
    
      ````java
      select date_add(d, interval 1 day); -- add 1 day
      select date_add(d, interval 1 hour); -- add 1 hour
      select date_add(d, interval 1 minute); -- ...
      select date_add(d, interval 1 second);
      select date_add(d, interval 1 microsecond);
      select date_add(d, interval 1 week);
      select date_add(d, interval 1 month);
      select date_add(d, interval 1 quarter);  一季：3个月
      select date_add(d, interval 1 year);
      ````
    
      可以加负数来实现减法
    
    - **mysql可以自动将符合datetime格式的字符串转化为datetime类型（默认使用2019-11-11 23：22：22格式），而oracle必须进行格式转化**
    
  - 字符串处理函数
  
    -  length（s）获取字符数
    -  concat（s1,s2）拼接多个字符串
    -  lower（s）、upper（s）转化大小写
    -  trim（s） 去空格
    -  mid（s，n，len）从指定位置截取固定长度
  
  - 数值处理函数
  
    - abs（x）  绝对值
    -  floor（x）返回小于x的最大整数
    -  round（x）对数字进行四舍五入

#### 4、连接查询：

- 自然连接查询：SELECT column1，column2 FROM table_name1  t1, table_name2 t2 WHERE  t1.some_column=t2.some_column

  先进行笛卡尔积连接（两张表所有数据互相组合成一组新数据，新数据总数为两表数据的乘积），然后通过where语句中的条件进行过滤；因此自然连接查询效率非常低

- 内连接查询： SELECT column1，column2   FROM table_name1  t1 INNER JOIN table_name2 t2  ON  t1.some_column=t2.some_column

  通过满足条件的数据，来筛选需要连接的数据；inner join可以省略为 join

- 左外连接查询、右外连接查询：SELECT column1，column2   FROM table_name1  t1 LEFT  JOIN (RIGHT JOIN)table_name2 t2  ON  t1.some_column=t2.some_column

  相对于内连接，外连接会保存其中一个表的所有数据，无法匹配的另一个表的字段值通过null填充

#### 5、合并查询结果：

 SELECT  column1，column2 FROM table1  UNION  SELECT    SELECT  column1，column2 FROM table2

UNION会过滤一组数据完全相同的；而UNION ALL 不会

UNION语句要保证合并的结果集的字段一一对应

#### 6、exists查询：

用于相关子查询，即子查询的查询条件依赖于外层父查询的某个属性

如：SELECT * FROM table_name1  WHERE  id IN (SELECT id FROM table_name2  WHERE  name ='yh')

可以通过exists改写为：SELECT * FROM table_name1  t1 WHERE  EXISTS (SELECT 1 FROM table_name2 t2 WHERE t1.id=t2.id AND t2.name='yh')

该sql语句的执行过程为：获取外层父查询表中的第一行数据，然后使用该行数据带入执行子查询语句，若返回true（EXISTS+ 子查询 语句会判断该次查询结果集是否为null，非null，则返回true）就保存该外层父查询的数据中需要的字段数据，放到结果集中。重复执行该过程，遍历所有父查询表中的数据

exists+ 子查询 语句不会在乎查询的结果集，因此一般使用 1常量来代替

exists  （select null） 返回ture，可以结果集会保存所有父查询中的数据

使用in 不容易走索引（理论会使用了索引全表扫描，但由于效率不高，查询优化器默认使用数据全表扫描）

因此使用exists可以优化in语句的查询效率；但是exists会遍历外表的所有数据，因此只适用于外表小，内表大的查询；外部太大时，还是只能使用in

## 3、建表（库）、删表（库）语句

- 建库：CREATE DATABASE  database_name

- 建表：CREATE TABLE  table_name (

​			colum_name1   data_type(size),

​			colum_name2   data_type(size)

​           )

 

- 在建表时添加约束、索引：

​           CREATE TABLE  table_name (

​			colum_name1   data_type(size)   NOT NULL   AUTO_INCREMENT,     非空、自动递增

​			colum_name2   data_type(size)   NOT NULL   DEFAULT ' ',    非空，默认

​			PRIMARY KEY  (column_name1) ,    主键

​			UNIQUE KEY  uk_column_name2   (column_name2)  USING BTREE   唯一约束（唯一索引）

​			KEY   idx_column_name3(column_name3)     普通索引

​           )

- 删除DROP语句

  DROP DATABASE database_name  删除数据库

  DROP TABLE table_name  删除表

  TRUNCATE TABLE table_name   清空表 （效果和没有where子句的DELETE类似；速度更快，但是无事务，不触发trigger，一般不使用在开发代码中）

  DROP INDEX   index_name ON table_name   删除索引

- 修改ALTER  TABLE语句，用于修改、添加、删除表中的列

  ALTER TABLE table_name  ADD column_name datatype  添加字段

  ALTER TABLE table_name  DROP COLUMN column_name  删除字段

  ALTER TABLE table_name    MODIFY COLUMN column_name datatype   修改字段属性

# 3、sql使用技巧

## 1、使用EXPLAIN分析sql查询执行效率

在正常sql语句前添加 explain关键字：explain select * from test 

通过该解释器结果来分析，mysql该查询语句的性能，表数据字段含义如下：

- **id**

sql语句中，每个select语句都会有对应查询性能数据：

当使用连接查询时，所有连接的表都会有一组数据，并且ID一样

当进行子查询时，每次查询都会有一组数据，id排序代表优先级，由大到小

- **select_type  查询类型**

SIMPLE   简单查询

PRIMARY  复合查询中的最外层查询（最外层为连接查询时，所有连接表的查询都为PRIMARY）

SUBQUERY   查询结果作为条件的查询（子查询）

DERIVED  查询结果作为临时表，用于进行下一步查询

UNION、UNION RESULT     查询结果进行合并的查询、查询使用的表为合并表的查询

- **table  查询的表名**
- **type   查询类型**

所有查询类型，按查询性能排序

**all <  index < range < ref < eq_ref < const <s ystem**

all  对于数据进行全表扫描

index  对于索引树进行全部扫描

range  对于索引树进行范围扫描

ref   对索引进行单独值的匹配，然后扫描获取对应索引key的数据

eq_ref   唯一性索引，每个索引值只对应一条数据，因此只需要进行索引单独值匹配（只用于连接查询，将唯一索引列作为连接条件）

const   通过索引只匹配到一行数据（和eq_ref类似，但不适用于连接查询）

system  查询的表只有一条数据

- possible_keys  可能被用到的索引
- key   被使用的索引
- key_len  索引长度（字节数）
- Extra   额外重要信息，常见有：

Using filesort   使用了额外外部文件排序，即没有使用索引的排序

Using temporary    使用了额外临时表排序，即没有使用索引的排序

Using index   使用了索引进行查询

Using  where  使用了where条件过滤数据

Using index condition  使用索引进行了数据过滤

**对于查询sql语句的性能优化应保证：至少达到rang级别，要求是ref级别，最好为consts**

## 2、sql查询优化

**关于避免不走索引的优化：**

不使用null，用默认值代替；

不使用<>、!=、or操作符，使用union all合并查询来代替；

in 尽量使用between代替；

like 不使用左通配符；

对于where条件字段不进行表达式操作、函数操作；

组合索引使用时，必须满足左前缀原理

满足相关子查询并且外表小，内表大时，使用exists代替in

**其他优化：**

尽量使用数字类型字段；数字比较时只用一次，而字符需要多次

尽量使用varchar代替char；varchar节省存储空间，查询对于短字符也快一点

尽量不要使用*，指定需要的字段；减少后台从数据库获取数据的量；防止表在字段扩展时，对代码带来的影响  

**能在数据库完成的数据处理操作，绝不在后台做处理（理论上数据库操作比后台操作快很多）**‘