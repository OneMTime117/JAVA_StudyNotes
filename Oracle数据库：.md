# Oracle数据库：

## 1、mysql和oracle的区别

#### 一、并发性

并发性是oltp数据库最重要的特性，但并发涉及到资源的获取、共享与锁定。

mysql:
mysql以表级锁为主，对资源锁定的粒度很大，如果一个session对一个表加锁时间过长，会让其他session无法更新此表中的数据。
虽然InnoDB引擎的表可以用行级锁，但这个行级锁的机制依赖于表的索引，如果表没有索引，或者sql语句没有使用索引，那么仍然使用表级锁。

oracle:
oracle使用行级锁，对资源锁定的粒度要小很多，只是锁定sql需要的资源，并且加锁是在数据库中的数据行上，不依赖与索引。所以oracle对并发性的支持要好很多。

#### 二、一致性

oracle:
oracle支持serializable的隔离级别，可以实现最高级别的读一致性。每个session提交后其他session才能看到提交的更改。oracle通过在undo表空间中构造多版本数据块来实现读一致性，
每个session查询时，如果对应的数据块发生变化，oracle会在undo表空间中为这个session构造它查询时的旧的数据块。

mysql:
mysql没有类似oracle的构造多版本数据块的机制，只支持read commited的隔离级别。一个session读取数据时，其他session不能更改数据，但可以在表最后插入数据。
session更新数据时，要加上排它锁，其他session无法访问数据。

#### 三、事务

oracle很早就完全支持事务。

mysql在innodb存储引擎的行级锁的情况下才支持事务。

#### 四、数据持久性

oracle
保证提交的数据均可恢复，因为oracle把提交的sql操作线写入了在线联机日志文件中，保持到了磁盘上，
如果出现数据库或主机异常重启，重启后oracle可以考联机在线日志恢复客户提交的数据。
mysql:
默认提交sql语句，但如果更新过程中出现db或主机重启的问题，也许会丢失数据。

#### 五、提交方式

oracle默认不自动提交，需要用户手动提交。
mysql默认是自动提交。

#### 六、逻辑备份

oracle逻辑备份时不锁定数据，且备份的数据是一致的。

mysql逻辑备份时要锁定数据，才能保证备份的数据是一致的，影响业务正常的dml使用。

#### 七、热备份

oracle有成熟的热备工具rman，热备时，不影响用户使用数据库。即使备份的数据库不一致，也可以在恢复时通过归档日志和联机重做日志进行一致的回复。
mysql:
myisam的引擎，用mysql自带的mysqlhostcopy热备时，需要给表加读锁，影响dml操作。
innodb的引擎，它会备份innodb的表和索引，但是不会备份.frm文件。用ibbackup备份时，会有一个日志文件记录备份期间的数据变化，因此可以不用锁表，不影响其他用户使用数据库。但此工具是收费的。
innobackup是结合ibbackup使用的一个脚本，他会协助对.frm文件的备份。

#### 八、sql语句的扩展和灵活性

mysql对sql语句有很多非常实用而方便的扩展，比如limit功能，insert可以一次插入多行数据，select某些管理数据可以不加from。
oracle在这方面感觉更加稳重传统一些。

#### 九、复制

oracle:既有推或拉式的传统数据复制，也有dataguard的双机或多机容灾机制，主库出现问题是，可以自动切换备库到主库，但配置管理较复杂。
mysql:复制服务器配置简单，但主库出问题时，丛库有可能丢失一定的数据。且需要手工切换丛库到主库。

#### 十、性能诊断

oracle有各种成熟的性能诊断调优工具，能实现很多自动分析、诊断功能。比如awr、addm、sqltrace、tkproof等
mysql的诊断调优方法较少，主要有慢查询日志。

#### 十一、权限与安全

mysql的用户与主机有关，感觉没有什么意义，另外更容易被仿冒主机及ip有[可乘之机](https://www.baidu.com/s?wd=%E5%8F%AF%E4%B9%98%E4%B9%8B%E6%9C%BA&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)。
oracle的权限与安全概念比较传统，[中规中矩](https://www.baidu.com/s?wd=%E4%B8%AD%E8%A7%84%E4%B8%AD%E7%9F%A9&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)。

#### 十二、分区表和分区索引

oracle的分区表和分区索引功能很成熟，可以提高用户访问db的体验。
mysql的分区表还不太成熟稳定。

#### 十三、管理工具

oracle有多种成熟的命令行、图形界面、web管理工具，还有很多第三方的管理工具，管理极其方便高效。
mysql管理工具较少，在linux下的管理工具的安装有时要安装额外的包（phpmyadmin， etc)，有一定复杂性。

#### 十四、技术支持

oracle出问题可以找客服

mysq出问题自己解决

#### 十五、授权

#### 

oracle收费

mysq开源-免费

#### 十六、选择

有钱用建议用oracle

没钱且能满足需求建议用mysq。（[阿里巴巴](https://www.baidu.com/s?wd=%E9%98%BF%E9%87%8C%E5%B7%B4%E5%B7%B4&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，wiki百科等大型项目也用了mysql，人家主要用了分布式存储、缓存、分表分库等技术）

## Oracle数据库与MySQL数据库的区别：

1.组函数用法规则

~~mysql中组函数在select语句中可以随意使用~~，但在oracle中如果查询语句中有组函数，那其他列名必须是组函数处理过的，或者是group by子句中的列否则报错

```
eg：

select name,count(money) from user；这个放在mysql中没有问题在oracle中就有问题了。
而select name,count(money) from user group by name或者select max(name),count(money) from user；
在oracle就不会报错，同样这两种情况在mysql也不会报错
```

2.自动增长的数据类型处理

MYSQL有自动增长的数据类型，插入记录时不用操作此字段，会自动获得数据值。ORACLE没有自动增长的数据类型，需要建立一个自动增长的序列号，插入记录时要把序列号的下一个值赋于此字段。

CREATE SEQUENCE序列号的名称(最好是表名+序列号标记)INCREMENT BY 1 START WITH 1 MAXVALUE 99999 CYCLE NOCACHE;

其中最大的值按字段的长度来定，如果定义的自动增长的序列号NUMBER(6)，最大值为999999

INSERT语句插入这个字段值为：序列号的名称.NEXTVAL

3.单引号的处理

MYSQL里可以用双引号包起字符串，ORACLE里只可以用单引号包起字符串。在插入和修改字符串前必须做单引号的替换：把所有出现的一个单引号替换成两个单引号。

4.翻页的SQL语句的处理

MYSQL 处理翻页的SQL语句比较简单，用LIMIT开始位置，记录个数；PHP里还可以用SEEK定位到结果集的位置。ORACLE处理翻页的 SQL语句就比较繁琐了。每个结果集只有一个ROWNUM字段标明它的位置，并且只能用ROWNUM<100，不能用ROWNUM>80。

以下是经过分析后较好的两种ORACLE翻页SQL语句(ID是唯一关键字的字段名)：

语句一：

SELECT ID, [FIELD_NAME,...] FROM TABLE_NAME WHERE ID IN ( SELECT ID FROM (SELECT ROWNUM AS NUMROW, ID FROM TABLE_NAME WHERE 条件1 ORDER BY 条件2) WHERE NUMROW > 80 AND NUMROW < 100 ) ORDER BY 条件3;

语句二：

SELECT * FROM (( SELECT ROWNUM AS NUMROW, c.* from (select [FIELD_NAME,...] FROM TABLE_NAME WHERE 条件1 ORDER BY 条件2) c) WHERE NUMROW > 80 AND NUMROW < 100 ) ORDER BY 条件3;

5.长字符串的处理

长 字符串的处理ORACLE也有它特殊的地方。INSERT和UPDATE时最大可操作的字符串长度小于等于4000个单字节，如果要插入更长的字 符串，请考虑字段用CLOB类型，方法借用ORACLE里自带的DBMS_LOB程序包。插入修改记录前一定要做进行非空和长度判断，不能为空的字段值和 超出长度字段值都应该提出警告，返回上次操作。

6.日期字段的处理

MYSQL日期字段 分DATE和TIME两种，ORACLE日期字段只有DATE，包含年月日时分秒信息，用当前数据库的系统时间为 SYSDATE，精确到秒，或者用字符串转换成日期型函数TO_DATE(‘2001-08-01’,’YYYY-MM-DD’)年-月-日24小时:分 钟:秒的格式YYYY-MM-DD HH24:MI:SS TO_DATE()还有很多种日期格式，可以参看ORACLE DOC.日期型字段转换成字符串函数TO_CHAR(‘2001-08-01’,’YYYY-MM-DD HH24:MI:SS’)

日期字 段的数学运算公式有很大的不同。MYSQL找到离当前时间7天用DATE_FIELD_NAME > SUBDATE(NOW()，INTERVAL 7 DAY)ORACLE找到离当前时间7天用 DATE_FIELD_NAME >SYSDATE - 7;

MYSQL中插入当前时间的几个函数是：NOW()函数以`'YYYY-MM-DD HH:MM:SS'返回当前的日期时间，可以直接存到DATETIME字段中。CURDATE()以’YYYY-MM-DD’的格式返回今天的日期，可以 直接存到DATE字段中。CURTIME()以’HH:MM:SS’的格式返回当前的时间，可以直接存到TIME字段中。例：insert into tablename (fieldname) values (now())

而oracle中当前时间是sysdate

7.空字符的处理

MYSQL的非空字段也有空的内容，ORACLE里定义了非空字段就不容许有空的内容。按MYSQL的NOT NULL来定义ORACLE表结构，导数据的时候会产生错误。因此导数据时要对空字符进行判断，如果为NULL或空字符，需要把它改成一个空格的字符串。

8.字符串的模糊比较

MYSQL里用字段名like%‘字符串%’，ORACLE里也可以用字段名like%‘字符串%’但这种方法不能使用索引，速度不快，用字符串比较函数instr(字段名，‘字符串’)>0会得到更精确的查找结果。

9.程序和函数里，操作数据库的工作完成后请注意结果集和指针的释放。

## 数据库原理：

### 1、数据库出现原因

对数据的存储需求一直存在。保存数据的方式，经历了手工管理、文件管理等阶段，直至数据库管理阶段。

文件存储方式保存数据的弊端：

- 缺乏对数据的整体管理,数据不便修改；
- 不利于数据分析和共享;
- 数据量急剧增长,大量数据不可能长期保存在文件中。

数据库[应运而生](https://www.baidu.com/s?wd=%E5%BA%94%E8%BF%90%E8%80%8C%E7%94%9F&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，是人们存放数据、访问数据、操作数据的存储仓库。

### 2、数据库

- 数据库（Database,简称DB）是按照数据结构来组织、存储和管理数据的仓库。

- 数据库管理系统(Database Management System,简称DBMS)：管理数据库的软件。

- 数据库的典型特征包括：数据的结构化，数据间的共享，减少数据的冗余度，以及数据的独立性。

### 3、关系数据库简介

​       关系是一个数学概念，描述两个元素间的关联或对应关系。所以关系型数据库，即是使用关系模型把数据组织到数据表(Table)中

主流数据库产品：

- Oracle（Oracle）
- DB2（IBM）
- SQL Server（MS）
- MySQL（Oracle）

#### 3.1、关系数据库表

- 在关系数据库中，数据被存放于二维数据表(Table)中。

- 一个关系数据库由多个数据表组成，数据表是关系数据库的基本存储结构，由行和列组成，行（Row）也就是横排数据，也经常被称作记录(Record)，列（Column）就是纵列数据，也被称作字段(Field)。表和表之间是存在关联关系的。

####      3.2、主流关系数据库简介

#####            3.2.1. Oracle数据库概述

Oracle是当今著名的Oracle(甲骨文)公司的数据库产品，它是世界上第一个商品化的关系型数据库管理系统，也是第一个推出和数据库结合的第四代语言开发工具的数据库产品。

Oracle采用标准的SQL结构化查询语言，支持多种数据类型，提供面向对象的数据支持，具有第四代语言开发工具，支持UNIX、WINDOWS、OS/2等多种平台。Oracle公司的软件产品丰富，包括Oracle服务器产品，Oracle开发工具和Oracle应用软件。其中最著名的就是Oracle数据库，目前最新的版本是Oracle 12c。

##### 3.2.2. DB2数据库概述

DB2是IBM的关系型数据库管理系统，DB2有很多不同的版本，可以运行在从掌上产品到大型机不同的终端机器上。DB2 Universal Database Personal Edition和DB2 Universal Database Workgroup Edition分别是为OS/2和Windows系统的单用户和多用户提供的数据库管理系统。

DB2在高端数据库的主要竞争对手是Oracle。

##### 3.2.3. Sybase数据库

Sybase是美国Sybase公司研制的一种关系型数据库系统，是较早采用C/S技术的数据库厂商，是一种典型的UNIX或Windows NT平台上客户机/服务器环境下的大型数据库系统。 Sybase通常与Sybase SQL Anywhere用于客户机/服务器环境，前者作为服务器数据库，后者为客户机数据库，采用该公司研制的PowerBuilder为开发工具，在国内大中型系统中具有广泛的应用。

SYBASE主要有三种版本，一是UNIX操作系统下运行的版本，二是Novell Netware环境下运行的版本，三是Windows NT环境下运行的版本。对UNIX操作系统目前广泛应用的为SYBASE 10 及SYABSE 11 for SCO UNIX。

2010年Sybase被SAP收购。

##### 3.2.4. SQL Server数据库概述

Microsoft SQL Server是运行在Windows NT服务器上，支持C/S结构的数据库管理系统。它采用标准SQL语言，微软公司对它进行了部分扩充而成为事务SQL（Transact-SQL）。

SQL Server最早是微软为了要和IBM竞争时，与Sybase合作所产生的，其最早的发展者是Sybase，和Sybase数据库完全兼容。在与Sybase终止合作关系后，微软自主开发出SQL Server 6.0版，往后的SQL Server即均由微软自行研发。最新的版本是SQL Server 2012，上一版本是2008。

Microsoft SQL Server几个初始版本适用于中小企业的数据库管理，但是后来它的应用范围有所扩展，已经触及到大型、跨国企

##### 3.2.5.MySQL数据库概述

MySQL是一个开放源码的小型关系型数据库管理系统，开发者为瑞典MySQL AB公司。目前MySQL被广泛地应用在Internet上的中小型网站中。

与其它的大型数据库例如Oracle、IBM DB2等相比，MySQL自有它的不足之处，如规模小、功能有限等，但对于一般个人使用者和中小型企业来说，MySQL提供的功能已经绰绰有余，而且由于MySQL是开放源码软件，因此可以大大降低总体拥有成本，许多中小型网站为了降低网站总体拥有成本而选择了MySQL作为网站数据库。

2008年1月16日，Sun正式收购MySQL。2009年4月20日，SUN被Oracle公司收购。目前的最新版本是MySQL5.6.



### 4、 SQL概述

#### 4.1、结构化查询语言sql

SQL(Structured Query Language) 是结构化查询语言的缩写。

SQL是在关系数据库上执行数据操作、检索及维护所使用的标准语言,可以用来查询数据，操纵数据，定义数据，控制数据，所有数据库都使用相同或者相似的语言。

SQL可分为:

数据定义语言（DDL） : Data Definition Language
数据操纵语言（DML） : Data Manipulation Language
事务控制语言（TCL）：Transaction Control Language)
数据查询语言（DQL）：Data Query Language
数据控制语言（DCL） : Data Control Language
执行SQL语句时，用户只需要知道其逻辑含义，而不需要知道SQL语句的具体执行步骤

#### 4.2、sql分类

##### 4.2.1. 数据定义语言（DDL）

用于建立、修改、删除数据库对象，包括创建语句(CREATE)、修改语句(ALTER)、删除语句(DROP)，比如使用CREATE TABLE创建表，使用ALTER TABLE修改表，使用DROPTABLE删除表等动作。这类语言不需要事务的参与，自动提交。

##### 4.2.2. 数据操作语言（DML）

用于改变数据库数据，包括INSERT、UPDATE、DELETE三条语句。其中，INSERT语句用于将数据插入到数据库中，UPDATE语句用于更新数据库中已存在的数据，DELETE用于删除数据库中已存在的数据。DML语言和事务是相关的，执行完DML操作后必须经过事务控制语句提交后才真正的将改变应用到数据库中。

##### 4.2.3. 事务控制语言（TCL）

用来维护数据一致性的语句，包括提交(COMMIT)、回滚(ROLLBACK)、保存点(SAVEPOINT)三条语句，其中COMMIT用来确认已经进行的数据库改变， ROLLBACK语句用来取消已经进行的数据库改变，当执行DML操作后(也就是上面说的增加、修改、删除等动作)，可以使用COMMIT语句来确认这种改变，或者使用ROLLBACK取消这种改变。SAVEPOINT语句用来设置保存点，使当前的事务可以回退到指定的保存点，便于取消部分改变。

##### 4.2.4. 数据查询语言（DQL）

用来查询所需要的数据。使用最广泛，语法灵活复杂。

##### 4.2.5. 数据控制语言（DCL）

用于执行权限的授予和收回操作、创建用户等，包括授予(GRANT)语句，收回(REVOKE)语句，CREATE USER语句，其中GRANT用于给用户或角色授予权限， REVOKE用于收回用户或角色已有的权限。DCL语句也不需要事务的参与，是自动提交的。



## Oracle启动服务：

默认oracle安装后会有7个服务：

**1、OracleServiceORCL：**

数据库服务（数据库实例），是Oracle数据库核心服务，是数据库启动基础

*****ORCL是数据库实例名，默认的数据库是ORCL，你可以创建其他的，即OracleService+数据库名。

**2、OracleOraDb11g_home1TNSListener:**

监听器服务，在oracle数据库需要远程访问时需要启动（本地访问（sqlpuls）不需要启动，但本地使用工具进行连接时，还是需要启动，在oracle9i版本后，默认包含XDB特性，当oracle监听服务启动时，XDB的Http服务会默认占用8080端口）

解决方式：

​	1、禁用XDB服务：

```
在ORACLE_HOME/dbs/initSID.ora文件，去除如下行：
dispatchers='(PROTOCOL=TCP) (SERVICE=XDB)'
```

**3、OracleOraDb11g_home1ClrAgent：**

Oracle数据库的.NET扩展服务的一部分（非必须启动）

**4、OracleMTSRecoveryService：**

服务端控制服务，作用是允许数据库充当一个微软事务服务器MTS、COM/COM+对象和分布式环境下的事务的资源管理器（非必需启动）

**5、OracleDBConsoleorcl：**

Oracle数据库控制台服务，在运行Enterprise Manager（企业管理器OEM）的时候，需要启动这个服务。（非必须启动）

**6、Oracle ORCL VSS Writer Service：**
Oracle卷映射拷贝写入服务，它可以在多卷或者单个卷上创建映射拷贝，同时不会影响到系统的系统能。（非必须启动）

**7、OraclJobSchedulerORCL:**
Oracle作业调度（定时器）服务，ORCL是Oracle实例标识。（非必须启动）

********cmd启动服务方式：**      sc start  服务名



## Oracle数据库体系结构：

实例（Instance）：一组Oracle后台进程以及在服务器中分配的共享内存区域

数据库（Database）：是由磁盘的数据文件、控制文件、日志文件、参数文件和归档日志文件等组成的物理文件集合

数据库服务器（Database Server）:管理数据库的软件工具（如sqlPLUS）

**其关系：**

实例用于管理控制数据库，数据库为实例提供数据

一个数据库可以被多个实例装载使用

一个实例在生命周期中只能装置打开一个数据库



#### orcale数据储存结构：

从逻辑上：一个数据库（database）下面可以分为多个表空间（tablespace）；一个表空间下面又分为多个段（segment）；一个段又分为多个区间（extent）；一个区间又有一组连续的数据块（data block）组成

**段：一个数据表占一个段、一个索引占一个段，段是指占用数据文件空间的通称，或数据库对象使用的空间的集合；段可以有表段、索引段、回滚段、临时段和高速缓存段等。**

**区：一个区中的数据块是连续的，但在物理磁盘上可能是分散的，分配给对象（如表）的任何连续块叫区间；区间也叫扩展，因为当它用完已经分配的区间后，再有新的记录插入就必须在分配新的区间（即扩展一些块）；一旦区间分配给某个对象（表、索引及簇），则该区间就不能再分配给其它的对象.**

从物理上：一个表空间由多个数据文件构成，数据文件是实实在在存在的磁盘上的文件，这些数据文件由系统磁盘的块组成



#### 表空间对区间的管理：

**词典管理表空间**(Dictionary-managed tablespaces)

​    在表空间里，有的区间被占用了，有的没被占用，这些数据是放在数据字典里的。当你对这个表空间进行分配或释放的时候，数据文件里相关的表就会做修改。

 

**本地管理表空间**（locally managed tablespace）

​      本地管理表空间不是在数据词典里存储表空间的，由自由区管理的表空间。用位图来自由的管理区间。一个区间对一个位，如果这个位是1表示已经被占用，0表示未被占用。

　　词典管理空间表示“中央集权治”，本地管理表空间表示“省市自治区”，一个databases表示中国，tablespaces表示一个省或直辖市。词典管理统一由中央调配。而本地管理表示有高度的自治权利，自已各种资源的分配不用上报中央。

#### 表空间类型：

**Undo  tablespace**

　　Undo 类型的表空间（还原表空间），当你对一张表或一条记录进行修改的时候，它会对修改之前的信息进行保存，这样可以保证数据的回滚。Undo 只包含undo类型的对象，不能包含任何其他对象，只适合于数据文件和区间管理。

创建undo 类型的表空间：

 SQL>create undo tablespace  undo1 datafile '/ora10/product/oradata/ora10/paul01.dbf' size 20m;

**Temporary  Tablespaces**

 　　临时表空间，相当于一个临时的垃圾场。用于排序操作，比如你要做一次大数据量的查询，但在内存无法存储这么大量的数据，然后会在磁盘上建立一个临时的表空间用记存放这些数据。Oracle就会用这个临时表空间做排序，存储中间结果。

一个全局的临时表空间，可以由多个用户共享，谁需要谁使用。但它只能存放临时的数据，不能包含任何永久性对象。 建议用本地管理方式创建这个表空间。

创建临时表空间：

 SQL>create temporary tablespace  temp datafile '/ora10/product/oradata/ora10/paul01.dbf' size 20m  extent management local uniform size 4m;

**Tablespaces**

​         用户数据表空间， 最重要,也是用于存放用户数据表空间

SQL>create undo tablespace  tablespace datafile '/ora10/product/oradata/ora10/paul01.dbf' size 20m;

#### 表空间的创建：

create  tablespace(表空间类型)   tablespacename（表空间名）  datafile  '/ora10/product/oradata/ora10/paul01.dbf'（表空间数据文件及路径） size 20m（数据文件最大值，即表空间大小）

*****表空间数据文件路径一定要是本来存在的，否则部分创建

除了最基本的属性外，还可以设置其他属性：

表空间的区大小   uniform  size  128k

表空间的数据文件大小自动增长（并可以设置最大值） autoextend on next 10m maxsize 500m

**这些属性出来在创建表空间时添加外，还可以之后通过修改表空间属性来设置：**

如：alter tablespace sp01（表空间名） 'D:\dev\oracle\product\10.2.0\dada01.dbf' autoextend on next 10m maxsize 500m;

#### 表空间状态：

当建立表空间时，表空间处于联机的(online)状态，此时该表空间是可以访问的，并且该表空间是可以读写的，即可以查询该表空间的数据，而且还可以在表空间执行各种语句。但是在进行系统维护或是数据维护时，可能需要改变表空间的状态。一般情况下，由特权用户或是dba来操作。

1)、使表空间脱机
alter tablespace 表空间名 offline;
eg、alter tablespace data01 offline;--表空间名不能加单引号
2)、使表空间联机
alter tablespace 表空间名 online;
eg、alter tablespace data01 online;
3)、只读表空间
当建立表空间时，表空间可以读写，如果不希望在该表空间上执行update，delete，insert操作，那么可以将表空间修改为只读
alter tablespace 表空间名 read only;

4)、可读写表空间

alter tablespace 表空间名 read write;

**只读或可读写状态时，表空间一定也为online**

#### 删除表空间：

一般情况下，由特权用户或是dba来操作，如果是其它用户操作，那么要求用户具有drop tablespace 系统权限。
drop tablespace ‘表空间’ including contents and datafiles;

**including contents表示删除表空间时，删除该空间的所有数据库对象，而datafiles表示将数据库文件也删除。**

#### 扩展表空间：

表空间是由数据文件组成的，表空间的大小实际上就是数据文件相加后的大小。

因此扩展表空间，就是扩展表空间的数据文件，有三种方式：

1、设置表空间数据文件自动增长

```
SQL> alter tablespace sp01 'D:\dev\oracle\product\10.2.0\dada01.dbf' autoextend on next 10m maxsize 500m;
```

2、修改表空间数据文件的大小

```
SQL> alter tablespace sp01 'D:\dev\oracle\product\10.2.0\dada01.dbf' resize 4m
```

3、添加表空间的数据文件

```
SQL> alter tablespace sp01 add datafile 'D:\dev\oracle\product\10.2.0\dada02.dbf' size 1m;
```

#### 表空间数据文件移动

有时，如果你的数据文件所在的磁盘损坏时，该数据文件将不能再使用，为了能够重新使用，需要将这些文件的副本移动到其它的磁盘，然后恢复。

1. 确定数据文件所在的表空间

```
select tablespace_name from dba_data_files where file_name=upper('D:\dev\oracle\product\10.2.0\dada01.dbf');
```

2. 使表空间脱机

--确保数据文件的一致性，将表空间转变为offline的状态。

alter tablespace sp01 offline;

3. 使用命令移动数据文件到指定的目标位置

host move D:\dev\oracle\product\10.2.0\dada01.dbf （要移动的文件）    c:\dada01.dbf（移动后的文件）

4. 执行alter tablespace 命令
    在物理上移动了数据后，还必须执行alter tablespace命令对数据库文件进行逻辑修改：
    alter tablespace sp01 rename datafile 'D:\dev\oracle\product\10.2.0\dada01.dbf' to 'c:\sp01.dbf';
5. 使得表空间联机
    在移动了数据文件后，为了使用户可以访问该表空间，必须将其转变为online状态。
    alter tablespace sp01 online;

#### 在表空间中创建表

create table mypart(
   deptno number(4), 
   dname varchar2(14), 
   loc varchar2(13)
) tablespace sp01;



## Oracle数据库特征：

### 1、维护数据的完整性

**数据的完整性用于确保数据库数据遵从一定的商业和逻辑规则，在oracle中，数据完整性可以使用约束、触发器、应用程序（过程、函数）三种方法来实现，在这三种方法中，因为约束易于维护，并且具有最好的性能，所以作为维护数据完整性的首选**

### 1、1约束：

约束用于确保数据库数据满足特定的商业规则。在oracle中，约束包括：not null、 unique， primary key， foreign key和check 五种。
1)、not null(非空)
如果在列上定义了not null，那么当插入数据时，必须为列提供数据。
2)、unique(唯一)
当定义了唯一约束后，该列值是不能重复的，但是可以为null。
3)、primary key(主键)
用于唯一的标示表行的数据，当定义主键约束后，该列不但不能重复而且不能为null。
需要说明的是：一张表最多只能有一个主键，但是可以有多个unqiue约束。
4)、foreign key(外键)
用于定义主表和从表之间的关系。外键约束要定义在从表上，主表则必须具有主键约束或是unique 约束，当定义外键约束后，要求外键列数据必须在主表的主键列存在或是为null。
5)、check
用于强制行数据必须满足的条件，假定在sal列上定义了check约束，并要求sal列值在1000-2000之间如果不在1000-2000之间就会提示出错。

#### 添加约束方式：

1、在建表时对列添加约束

```
   customerId char(8) primary key, --主键
   name varchar2(50) not null, --不为空
   address varchar2(50), foreign key  --外键
   email varchar2(50) unique, --唯一
   sex char(2) default '男' check(sex in ('男','女')), -- 一个char能存半个汉字，两位char能存一个汉字
   cardId char(18)
```

2、在建表后添加约束

1)、增加商品名也不能为空
SQL> alter table goods modify goodsName not null;
2)、增加身份证也不能重复
SQL> alter table customer add constraint xxxxxx unique(cardId);

**增加not null约束时，需要使用modify选项，而增加其它四种约束使用add选项。**

#### 删除或显示表约束：

当不再需要某个约束时，可以删除：
alter table 表名 drop constraint 约束名称；

*****在删除主键约束的时候，可能有错误，比如：alter table 表名 drop primary key；这是因为如果在两张表存在主从关系，那么在删除主表的主键约束时，必须带上cascade选项 如像：alter table 表名 drop primary key cascade;

显示表约束信息

select constraint_name, constraint_type, status, validated from user_constraints where table_name = '表名';

#### 约束定义分类：

1、列级定义：（在列定义时添加约束）

```
create table department4(
   dept_id number(12) constraint pk_department（约束名） primary key,
   name varchar2(12), 
   loc varchar2(12)
);
```

2、表级定义：（在列定义好后，添加约束）

```
create table employee2(
   emp_id number(4), 
   name varchar2(15),
   dept_id number(2), 
   constraint pk_employee（约束名）  primary key (emp_id),
   constraint fk_department（约束名） foreign key (dept_id) references（参照；对应） department4(dept_id)    定义该列为外键（对应对于epartment4的主键dept_id）
);
```

### 1、2 触发器

  触发器的定义就是说某个条件成立的时候，触发器里面所定义的语句就会被自动的执行。因此触发器不需要人为的去调用，也不能调用。然后，触发器的触发条件其实在你定义的时候就已经设定好了。

触发器可以分为语句级触发器和行级触发器。简单的说就是**语句级的触发器可以在某些语句执行前或执行后被触发**。**而行级触发器则是在定义的了触发的表中的行数据改变时就会被触发一次**。

#### 触发器语句：

create or replace  tigger 触发器名 触发时间 触发事件
on 表名
[for each row]
begin
  pl/sql语句
end

触发器名：触发器对象的名称。由于触发器是数据库自动执行的，因此该名称只是一个名称，没有实质的用途。
触发时间：指明触发器何时执行，该值可取：
before：表示在数据库动作之前触发器执行;
after：表示在数据库动作之后触发器执行。
触发事件：指明哪些数据库动作会触发此触发器：
insert：数据库插入会触发此触发器;
update：数据库修改会触发此触发器;
delete：数据库删除会触发此触发器。
表 名：数据库触发器所在的表。
for each row：对表的每一行触发器执行一次。如果没有这一选项，则只对整个表执行一次。

触发器能实现如下功能：

功能：
1、 允许/限制对表的修改
2、 自动生成派生列，比如自增字段
3、 强制数据一致性
4、 提供审计和日志记录
5、 防止无效的事务处理
6、 启用复杂的业务逻辑

### 1、3 存储过程和存储函数

**存储在数据库中，供所有用户程序调用的子程序，叫做存储过程和存储函数**

#### 存储过程：

create  [or replace] (当过程名可能存在时添加)  procedure  过程名 （参数列表 ： 参数名    in/out/in out  数据类型（不能定义大小））

AS/IS

变量列表：变量名    变量类型

begin

存储过程sql语句

end



执行存储过程

call  过程名（参数列表）；

execute   过程名 （参数列表）；  没有参数时 （）可以省略



*****参数未指明时，默认为IN

IN 定义输入参数变量，用于传递参数给存储过程

OUT 定义输出参数变量，用于从存储过程中获取数据

IN  OUT  即为输入参数，又为输出参数



存储过程在java中的调用：

无返回值：

```
            // 3.创建CallableStatement（用于执行存储过程）
            CallableStatement cs = ct.prepareCall("{call sp_update(?,?)}");
            // 4.给?赋值
            cs.setString(1, "SMITH");
            cs.setInt(2, 4444);
            // 5.执行
            cs.execute();
```

有返回值

            // 3.创建CallableStatement（用于执行存储过程）
            CallableStatement cs = ct.prepareCall("{call sp_update(?,?)}");
            // 4.给?赋值
            cs.setString(1, "SMITH");
            cs.registerOutParameter(2,Types.VARCHAR)
            // 5.执行
                cs.execute();
            // 6.获取返回值
                String time=cs.getString(2);
#### 存储函数：

```
CREATE FUNCTION annual_incomec（函数名） (参数列表： uname VARCHAR2)
RETURN NUMBER IS （返回值：）
annual_salazy NUMBER(7,2);
BEGIN 
   SELECT a.sal*13 INTO annual_salazy FROM emp a WHERE a.ename=uname;
   RETURN annual_salazy;
END;
```



**存储函数直接在sql语句中进行调用**



*****使用存储过程和使用存储函数的一条简单非必要原则：如果只有一个返回值，使用存储函数的return子句返回；如果有多个返回值，则使用存储过程通过out参数返回。

### 2、 包

**包用于在逻辑上组合过程和函数，它由包规范和包体两部分组成。**

相当于java中的类，包包括数据类型、游标、变量、常量、函数、过程

创建一个包：

1、声明该包有一个过程和函数（可以有多个）

```
create package sp_package is
   procedure update_sal(name varchar2, newsal number);
   function annual_income(name varchar2) return number;
end;
```

2、创建包体

```
CREATE OR REPLACE PACKAGE BODY SP_PACKAGE IS
  --存储过程
  PROCEDURE UPDATE_SAL(NAME VARCHAR2, NEWSAL NUMBER) IS
  BEGIN
     UPDATE EMP SET SAL = NEWSAL WHERE ENAME = NAME;
     COMMIT;
  END;
  
  --函数
  FUNCTION ANNUAL_INCOME(NAME VARCHAR2) RETURN NUMBER IS
     ANNUAL_SALARY NUMBER;
  BEGIN
     SELECT SAL * 12 + NVL(COMM, 0) INTO ANNUAL_SALARY FROM EMP WHERE ENAME = NAME;
     RETURN ANNUAL_SALARY;
  END;
END;
```

3、调用包中的函数或过程

call   包名.函数（过程）表达式



### 3、视图：

视图是一张虚拟表，其内容由查询定义，同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。(视图不是真实存在磁盘上的)

二、视图与表的区别
1、表需要占用磁盘空间，视图不需要
2、视图不能添加索引(所以查询速度略微慢点)
3、使用视图可以简化，复杂查询
4、视图的使用利于提高安全性
比如：不同用户查看不同视图

三、创建/修改视图
1、创建视图
create view 视图名 as select 语句 [with read only]
2、创建或修改视图
create or replace view 视图名 as select 语句 [with read only]
3、删除视图
drop view 视图名

### 4、索引

一、管理索引-原理介绍
索引是用于加速数据存取的数据对象。合理的使用索引可以大大降低i/o次数，从而提高数据访问性能。索引有很种
1)、单列索引
单列索引是基于单个列所建立的索引
语法：create index 索引名 on 表名(列名);
eg、create index nameIndex on custor(name);
2)、复合索引
复合索引是基于两列或是多列的索引。在同一张表上可以有多个索引，但是要求列的组合必须不同，比如：
create index emp_idx1 on emp(ename, job);
create index emp_idx1 on emp(job, ename);
以上这两个索引是两个不同的索引。

三、使用原则
1)、在大表上建立索引才有意义
2)、在where子句或是连接条件上经常引用的列上建立索引
3)、索引的层次不要超过4层

四、索引的缺点
索引有一些先天不足：
1)、建立索引，系统要占用大约为表1.2倍的硬盘和内存空间来保存索引。
2)、更新数据的时候，系统必须要有额外的时间来同时对索引进行更新，以维持数据和索引的一致性。
实践表明，不恰当的索引不但于事无补，反而会降低系统性能。因为大量的索引在进行插入、修改和删除操作时比没有索引花费更多的系统时间。
比如在如下字段建立索引应该是不恰当的：

1. 很少或从不引用的字段；
2. 逻辑型的字段，如男或女(是或否)等。
综上所述，提高查询效率是以消耗一定的系统资源为代价的，索引不能盲目的建立，这是考验一个DBA是否优秀的很重要的指标

五、其它索引
按照数据存储方式，可以分为B*树、反向索引、位图索引；
按照索引列的个数分类，可以分为单列索引、复合索引；
按照索引列值的唯一性，可以分为唯一索引和非唯一索引。
此外还有函数索引，全局索引，分区索引...

对于索引我还要说：
在不同的情况，我们会在不同的列上建立索引，甚至建立不同种类的索引，请记住，技术是死的，人是活的。
比如：B*树索引建立在重复值很少的列上，而位图索引则建立在重复值很多、不同值相对固定的列上。

六、显示索引信息
1)、在同一张表上可以有多个索引，通过查询数据字典视图dba_indexs和user_indexs，可以显示索引信息。其中dba_indexs用于显示数据库所有的索引信息，而user_indexs用于显示当前用户的索引信息：select index_name, index_type from user_indexes where table_name = '表名';
2)、显示索引列
通过查询数据字典视图user_ind_columns,可以显示索引对应的列的信息
select table_name, column_name from user_ind_columns where index_name ='IND_ENAME';
你也可以通过pl/sql developer工具查看索引信息

### 5、权限角色

#### 1、系统权限

1)、**系统权限是指执行特定类型sql命令的权利。它用于控制用户可以执行的一个或是一组数据库操作。**比如当用户具有create table权限时，可以在其方案中建表，当用户具有create any table权限时，可以在任何方案中建表。oracle提供了100多种系统权限。
常用的有：
create session 连接数据库 
create table 建表
create view 建视图 
create public synonym 建同义词
create procedure 建过程、函数、包 
create trigger 建触发器
create cluster 建簇

2、显示系统权限
oracle提供了100多种系统权限，而且oracle的版本越高，提供的系统权限就越多，我们可以查询数据字典视图system_privilege_map，可以显示所有系统权限。
select * from system_privilege_map order by name;

3、授予系统权限

一般情况，授予系统权限是由dba完成的，如果用其他用户来授予系统权限，则要求该用户必须具有grant any privilege的系统权限。

4、回收系统权限
一般情况下，回收系统权限是dba来完成的，如果其它的用户来回收系统权限，要求该用户必须具有相应系统权限及转授系统权限的选项(with admin option)。

#### 2、对象权限

**指访问其它方案对象的权利，用户可以直接访问自己方案的对象，但是如果要访问别的方案的对象，则必须具有对象的权限。**
比如smith用户要访问scott.emp表(scott：方案，emp：表)
常用的有：
insert 添加
delete 删除 
alter 修改 
select 查询 
index 索引 
references 引用 
execute 执行

2)、显示对象权限
通过数据字段视图可以显示用户或是角色所具有的对象权限。视图为dba_tab_privs
SQL> conn system/manager;
SQL> select distinct privilege from dba_tab_privs;
SQL> select grantor, owner, table_name, privilege from dba_tab_privs where grantee = 'BLAKE';

3)、授予对象权限
在oracle9i前，授予对象权限是由对象的所有者来完成的，如果用其它的用户来操作，则需要用户具有相应的(with grant option)权限，从oracle9i 开始，dba用户(sys，system)可以将任何对象上的对象权限授予其它用户。授予对象权限是用grant 命令来完成的。对象权限可以授予用户，角色，和public。在授予权限时，如果带有with grantoption 选项，则可以将该权限转授给其它用户。但是要注意with grant option选项不能被授予角色。
1.monkey 用户要操作scott.emp 表，则必须授予相应的对象权限
1). 希望monkey可以查询scott.emp 表的数据，怎样操作？
grant select on emp to monkey;
2). 希望monkey可以修改scott.emp 的表数据，怎样操作？
grant update on emp to monkey;
3). 希望monkey可以删除scott.emp 的表数据，怎样操作？
grant delete on emp to monkey;
4). 有没有更加简单的方法，一次把所有权限赋给monkey？
grant all on emp to monkey;

2.能否对monkey访问权限更加精细控制。(授予列权限)
1). 希望monkey只可以修改scott.emp的表的sal字段，怎样操作？
grant update on emp(sal) to monkey
2).希望monkey 只可以查询scott.emp 的表的ename，sal 数据，怎样操作？
grant select on emp(ename,sal) to monkey

3.授予alter权限
如果black用户要修改scott.emp表的结构，则必须授予alter对象权限
SQL> conn scott/tiger
SQL> grant alter on emp to blake;
当然也可以用system，sys 来完成这件事。
4.授予execute权限
如果用户想要执行其它方案的包/过程/函数，则须有execute权限。
比如为了让ken可以执行包dbms_transaction，可以授予execute 权限。
SQL> conn system/manager
SQL> grant execute on dbms_transaction to ken;
5.授予index权限
如果想在别的方案的表上建立索引，则必须具有index 对象权限。
如果为了让black 可以在scott.emp 表上建立索引，就给其index 的对象权限
SQL> conn scott/tiger
SQL> grant index on scott.emp to blake;
6.使用with grant option 选项
该选项用于转授对象权限。但是该选项只能被授予用户，而不能授予角色
SQL> conn scott/tiger;
SQL> grant select on emp to blake with grant option;
SQL> conn black/shunping
SQL> grant select on scott.emp to jones;

4)、回收对象权限
在oracle9i 中，收回对象的权限可以由对象的所有者来完成，也可以用dba用户(sys，system)来完成。
这里要说明的是：收回对象权限后，用户就不能执行相应的sql命令，但是要注意的是对象的权限是否会被级联收回？【级联回收】
如：scott------------->blake-------------->jones
select on emp select on emp select on emp
SQL> conn [scott/tiger@accp](mailto:scott/tiger@accp)
SQL> revoke select on emp from blake
请大家思考，jones能否查询scott.emp表数据。
答案：查不了了(级联回收，和系统权限不一样，刚好1相反)

#### 3、角色

**角色。角色是一组权限的集合，将角色赋给一个用户，这个用户就拥有了这个角色中的所有权限。不同的角色有不同的权限，分为预定义角色和自定义角色**

**预定义角色**：是指oracle所提供的角色，每种角色都用于执行一些特定的管理任务，下面我们介绍常用的预定义角色connect、resource、dba。

1)、connect角色
connect角色具有一般应用开发人员需要的大部分权限，当建立了一个用户后，多数情况下，只要给用户授予connect和resource角色就够了，那么connect角色具有哪些系统权限呢？
create cluster
create database link
create session
alter session
create table
create view
create sequence

2)、resource角色
resource角色具有应用开发人员所需要的其它权限，比如建立存储过程，触发器等。这里需要注意的是resource角色隐含unlimited tablespace系统权限。
resource角色包含以下系统权限：
create cluster
create indextype
create table
create sequence
create type
create procedure
create trigger

3)、dba角色
dba角色具有所有的系统权限，及with admin option选项，默认的dba用户为sys和system，它们可以将任何系统权限授予其他用户。但是要注意的是dba角色不具备sysdba和sysoper的特权（启动和关闭数据库）。

1)、建立角色（不验证）
如果角色是公用的角色，可以采用不验证的方式建立角色。
create role 角色名 not identified;
2)、建立角色（数据库验证）
采用这样的方式时，角色名、口令存放在数据库中。当激活该角色时，必须提供口令。在建立这种角色时，需要为其提供口令。
create role 角色名 identified by 密码;

2、角色授权
1)、给角色授权
给角色授予权限和给用户授权没有太多区别，但是要注意，系统权限的unlimited tablespace和对象权限的with grant option选项是不能授予角色的。
SQL> conn system/oracle;
SQL> grant create session to 角色名 with admin option
SQL> conn [scott/oracle@orcl](mailto:scott/oracle@orcl);
SQL> grant select on scott.emp to 角色名;
SQL> grant insert, update, delete on scott.emp to 角色名;
通过上面的步骤，就给角色授权了。

2)、分配角色给某个用户
一般分配角色是由dba来完成的，如果要以其它用户身份分配角色，则要求用户必须具有grant any role的系统权限。
SQL> conn system/oracle;
SQL> grant 角色名 to blake with admin option;
因为我给了with admin option选项，所以，blake可以把system分配给它的角色分配给别的用户。

3、删除角色
使用drop role，一般是dba来执行，如果其它用户则要求该用户具有drop any role系统权限。
SQL> conn system/oracle;
SQL> drop role 角色名;
问题：如果角色被删除，那么被授予角色的用户是否还具有之前角色里的权限？
答案：不具有了

4、显示角色信息
1)、显示所有角色
SQL> select * from dba_roles;
2)、显示角色具有的系统权限
SQL> select privilege, admin_option from role_sys_privs where role='角色名';
3)、显示角色具有的对象权限
通过查询数据字典视图dba_tab_privs可以查看角色具有的对象权限或是列的权限。
4)、显示用户具有的角色，及默认角色
当以用户的身份连接到数据库时，oracle 会自动的激活默认的角色，通过查询数据字典视图dba_role_privs 可以显示某个用户具有的所有角色及当前默认的角色。
SQL> select granted_role, default_role from dba_role_privs where grantee = ‘用户名’;

五、精细访问控制
精细访问控制是指用户可以使用函数，策略实现更加细微的安全访问控制。如果使用精细访问控制，则当在客户端发出sql语句(select，insert，update，delete)时，oracle会自动在sql语句后追加谓词(where子句)，并执行新的sql语句，通过这样的控制，可以使得不同的数据库用户在访问相同表时，返回不同的数据信息，如：
用户 scott blake jones
策略 emp_access
数据库表 emp
如上图所示，通过策略emp_access，用户scott，black，jones在执行相同的sql语句时，可以返回不同的结果。
例如：当执行select ename from emp时，根据实际情况可以返回不同的结果。

**自定义角色**：顾名思义就是自己定义的角色，根据自己的需要来定义。一般是dba来建立，如果用别的用户来建立，则需要具有create role的系统权限。在建立角色时可以指定验证方式(不验证，数据库验证等)。



## 数据库基本使用：

1、Oracle安装默认自动生成sys用户和system用户

sqlPLUS常用命令：

#### 1、连接命令：

**登入数据库（可以在登入后继续使用，来切换不同用户）：**

conn   用户名（本地连接）      /        conn 密码@网络用户名 （远程连接）   

**显示该数据库的用户名 ：**

show   用户名      

**该命令用来断开与当前数据库的连接：**

disc/disconn/disconnect

**修改用户密码：**

passw   （修改当前用户密码）      password   用户名   （可以修改其他用户）

**清空sqlPLUS屏幕：**

clear screen

#### 2、文件操作命令

**运行sql脚本**

@/start    脚本路径

将sqlPLUS内容输出到指定文件

spool   文件路径

sql代码

spool   off



**sqlPLUS工具作用**：主要用于执行sql语句，pl\sql块

*****PL\SQL：过程化sql语句，是Oracle数据库对sql语句的扩展，在普通sql语句上增加使用了编程语言的特点，

通过一些逻辑语言（条件判断、循环等）来执行sql语句，实现复杂功能的数据库查询

*****Mysql不支持PL/SQL,但支持Navicat Premium

**PL\SQL和存储过程的区别：**

功能实现一样，但PL\SQL本质上还是sql语句，只能一次性使用；存储过程是将一系列语句保存在数据库中，可以重复调用



# Oralce数据库命名规则和数据类型：

一、表名和列名的命名规则
1)、必须以字母开头
2)、长度不能超过30个字符
3)、不能使用oracle的保留字
4)、只能使用如下字符 a-z，a-z，0-9，$,#等

二、数据类型
1)、字符类
char 长度固定，最多容纳2000个字符。
例子：char(10) ‘小韩’前四个字符放‘小韩’，后添6个空格补全，如‘小韩      ’
varchar2(20) 长度可变，最多容纳4000个字符。
例子：varchar2（10） ‘小韩’ oracle分配四个字符。这样可以节省空间。
clob(character large object) 字符型大对象，最多容纳4g
char 查询的速度极快浪费空间，适合查询比较频繁的数据字段。
varchar 节省空间

Nvarchar 根据字符集格式来处理数据长度，utf-8汉字按一个算

2)、数字型
number范围-10的38次方到10的38次方，可以表示整数，也可以表示小数
number(5,2)表示一位小数有5位有效数，2位小数；范围：-999.99 到999.99
number(5)表示一个5位整数；范围99999到-99999
3)、日期类型
date 包含年月日和时分秒 oracle默认格式1-1月-1999
timestamp 这是oracle9i对date数据类型的扩展。可以精确到毫秒。
4)、图片
blob 二进制数据，可以存放图片/声音4g；一般来讲，在真实项目中是不会把图片和声音真的往数据库里存放，一般存放图片、视频的路径，如果安全需要比较高的话，则放入数据库。



# Oracle数据库基本语句：

1、创建表
--学生表
create table student (
   xh number(4), --学号
   xm varchar2(20), --姓名
   sex char(2), --性别
   birthday date, --出生日期
   sal number(7,2) --奖学金
);

2、修改表
--添加一个字段
sql>alter table student add (classid number(2));
--修改一个字段的长度
sql>alter table student modify (xm varchar2(30));
--修改字段的类型或是名字（不能有数据） 不建议做
sql>alter table student modify (xm char(30));
--删除一个字段 不建议做(删了之后，顺序就变了。加就没问题，应该是加在后面)
sql>alter table student drop column sal;
--修改表的名字 很少有这种需求
sql>rename student to stu;

3、删除表
sql>drop table student;

基本Oracle数据对表数据的处理：

1、添加数据
--所有字段都插入数据
insert into student values ('a001', '张三', '男', '01-5 月-05', 10);
--oracle中默认的日期格式‘dd-mon-yy’ dd 天 mon 月份 yy 2位的年 ‘09-6 月-99’ 1999年6月9日
--修改日期的默认格式（临时修改，数据库重启后仍为默认；如要修改需要修改注册表）
alter session set nls_date_format ='yyyy-mm-dd';
--修改后，可以用我们熟悉的格式添加日期类型：
insert into student values ('a002', 'mike', '男', '1905-05-06', 10);
--插入部分字段
insert into student(xh, xm, sex) values ('a003', 'john', '女');
--插入空值
insert into student(xh, xm, sex, birthday) values ('a004', 'martin', '男', null);
--问题来了，如果你要查询student表里birthday为null的记录，怎么写sql呢？
--错误写法：select * from student where birthday = null;
--正确写法：select * from student where birthday is null;
--如果要查询birthday不为null,则应该这样写：
select * from student where birthday is not null;

2、修改数据
--修改一个字段
update student set sex = '女' where xh = 'a001';
--修改多个字段
update student set sex = '男', birthday = '1984-04-01' where xh = 'a001';
--修改含有null值的数据
不要用 = null 而是用 is null；
select * from student where birthday is null;

3、删除数据
delete from student; --删除所有记录，表结构还在，写日志，可以恢复的，速度慢。
--delete的数据可以恢复。
savepoint a; --创建保存点
delete from student;
rollback to a; --恢复到保存点
一个有经验的dba，在确保完成无误的情况下要定期创建还原点。

drop table student; --删除表的结构和数据；
delete from student where xh = 'a001'; --删除一条记录；
truncate table student; --删除表中的所有记录，表结构还在，不写日志，无法找回删除的记录，速度快。

3、查询数据

和mysql基本相同

4、聚合函数使用：

1、当聚合函数作为条件语句时：

```
错误写法：select ename, sal from emp where sal=max(sal);

正确写法：select ename, sal from emp where sal=(select max(sal) from emp);
```

2、当聚合函数在查询字段中时：

```
	注意：select ename, max(sal) from emp;这语句执行的时候会报错，说ora-00937：非单组分组函数。因为max是分组函数，而ename不是分组函数.......

	但是select min(sal), max(sal) from emp;这句是可以执行的。因为min和max都是分组函数，就是说：如果列里面有一个分组函数，其它的都必须是分组函数，否则就出错。这是语法规定的
```



常见查询：
1、查询工资高于500或者是岗位为manager的雇员，同时还要满足他们的姓名首字母为大写的J？

```
select * from emp where (sal > 500 or job = 'MANAGER') and ename like 'J%';
```

2、使用order by字句 默认asc 

```
select * from emp order by sal;
```

3、如何显示所有员工的平均工资和工资总和？

```
	select sum(e.sal), avg(e.sal)  from emp e;
```

4、显示工资高于平均工资的员工信息

```
select * from emp e where sal > (select avg(sal) from emp);
```

5、group by 和 having 子句 group by 用于对查询的结果分组统计， having 子句用于限制分组显示结果

```
select avg(sal), max(sal), deptno from emp group by deptno;
```

如果你要分组查询的话，分组的字段deptno一定要出现在查询的列表里面

显示每个部门的每种岗位的平均工资和最低工资？(多字段分组）

```
select min(sal), avg(sal), deptno, job from emp group by deptno, job;
```

显示平均工资低于2000的部门号和它的平均工资？（分组添加条件）

```
select avg(sal), max(sal), deptno from emp group by deptno having avg(sal)< 2000;
```

6、多表查询

显示部门号为10的部门名、员工名和工资？

```
SELECT d.dname, e.ename, e.sal FROM emp e, dept d WHERE e.deptno = d.deptno and e.deptno = 10;
```

7、子查询

*****使用聚合函数的效率更高（比使用all，any）

SELECT * FROM emp WHERE deptno = (select deptno from emp WHERE ename = 'SMITH');

如何显示高于自己部门平均工资的员工的信息 (进行分组操作时，查询字段可以有聚合函数和其他字段)

```
SELECT e.ename, e.deptno, e.sal, ds.mysal 

	FROM emp e, (SELECT deptno, AVG(sal)mysal FROM emp GROUP by deptno) ds 

	WHERE e.deptno = ds.deptno AND e.sal > ds.mysal;
```

查询所有部门的平均工资，部门；通过与员工信息表连接查询，条件是员工所在部门，和其工资大于对于部门的平均工资，得出对应条件的员工信息

**当在from 子句中使用子查询时，必须给子查询指定别名，但表别名不能使用as  ，列名可以使用**

8、用查询结果创建新表，这个命令是一种快捷的建表方式

```
CREATE TABLE mytable (id, name, sal, job, deptno) as SELECT empno, ename, sal, job, deptno FROM emp;
```

合并查询 有时在实际应用中，为了合并多个select语句的结果，可以使用集合操作符号union，union all，intersect，minus。 多用于数据量比较大的数据局库，运行速度快。 1). union 该操作符用于取得两个结果集的并集。当使用该操作符时，会自动去掉结果集中重复行。

```
SELECT ename, sal, job FROM emp WHERE sal >2500

	UNION

	SELECT ename, sal, job FROM emp WHERE job = 'MANAGER';
```

 

2).union all 该操作符与union相似，但是它不会取消重复行，而且不会排序。

```
SELECT ename, sal, job FROM emp WHERE sal >2500

	UNION ALL

	SELECT ename, sal, job FROM emp WHERE job = 'MANAGER';

	该操作符用于取得两个结果集的并集。当使用该操作符时，不会自动去掉结果集中重复行。
```

3). intersect 使用该操作符用于取得两个结果集的交集。

```
SELECT ename, sal, job FROM emp WHERE sal >2500

	INTERSECT

	SELECT ename, sal, job FROM emp WHERE job = 'MANAGER';
```

4). minus 使用该操作符用于取得两个结果集的差集，他只会显示存在第一个集合中，而不存在第二个集合中的数据。

```
SELECT ename, sal, job FROM emp WHERE sal >2500

	MINUS

	SELECT ename, sal, job FROM emp WHERE job = 'MANAGER';

	（MINUS就是减法的意思）
```



5）、查询前10条数据

SELECT  *  from table WHERE ROWNUM<=10



## Oracle数据库分页方法：

oracle的分页一共有三种方式

方法一 根据rowid来分

```
SELECT *
  FROM EMP
 WHERE ROWID IN
       (SELECT RID
          FROM (SELECT ROWNUM RN, RID
                  FROM (SELECT ROWID RID, EMPNO FROM EMP ORDER BY EMPNO DESC)
                 WHERE ROWNUM <= ( (currentPage-1) * pageSize + pageSize )) --每页显示几条
         WHERE RN > ((currentPage-1) * pageSize) ) --当前页数
 ORDER BY EMPNO DESC;
 
eg、
-- 5 = (currentPage-1) * pageSize + pageSize   每页显示几条
-- 0 = (currentPage-1) * pageSize              当前页数
SELECT *
  FROM EMP
 WHERE ROWID IN
       (SELECT RID
          FROM (SELECT ROWNUM RN, RID
                  FROM (SELECT ROWID RID, EMPNO FROM EMP ORDER BY EMPNO DESC)
                 WHERE ROWNUM <= ( (1-1) * 5 + 5 )) --每页显示几条
         WHERE RN > ((1-1) * 5) ) --当前页数
 ORDER BY EMPNO DESC;
```

方法二 按分析函数来分

```
SELECT *
FROM (SELECT T.*, ROW_NUMBER() OVER(ORDER BY empno DESC) RK FROM emp T)
WHERE RK <= ( (currentPage-1) * pageSize + pageSize ) --每页显示几条
AND RK > ( (currentPage-1) * pageSize ); --当前页数
eg、
-- 5 = (currentPage-1) * pageSize + pageSize   每页显示几条
-- 0 = (currentPage-1) * pageSize              当前页数
SELECT *
FROM (SELECT T.*, ROW_NUMBER() OVER(ORDER BY empno DESC) RK FROM emp T)
WHERE RK <= 5
AND RK > 0;
```

方法三 按rownum 来分

```
SELECT *
  FROM (SELECT T.*, ROWNUM RN
          FROM (SELECT * FROM EMP ORDER BY EMPNO DESC) T
         WHERE ROWNUM <= ( (currentPage-1) * pageSize + pageSize )) --每页显示几条
 WHERE RN > ( (currentPage-1) * pageSize ); --当前页数
           
eg、
-- 5 = (currentPage-1) * pageSize + pageSize   每页显示几条
-- 0 = (currentPage-1) * pageSize              当前页数
SELECT *
  FROM (SELECT T.*, ROWNUM RN
          FROM (SELECT * FROM EMP ORDER BY EMPNO DESC) T
         WHERE ROWNUM <= 5)
 WHERE RN > 0;
```

其中emp为表名称，empno 为表的主键id，获取按empno降序排序后的第1-5条记录，emp表有70000 多条记录。
个人感觉方法一的效率最好，方法三 次之，方法二 最差。

下面通过方法三来分析oracle怎么通过rownum分页的

```
1、
SELECT * FROM emp;
2、显示rownum，由oracle分配的
SELECT e.*, ROWNUM rn FROM (SELECT * FROM emp) e; --rn相当于Oracle分配的行的ID号 
3、先查出1-10条记录
正确的: SELECT e.*, ROWNUM rn FROM (SELECT * FROM emp) e WHERE ROWNUM<=10;
错误的：SELECT e.*, ROWNUM rn FROM (SELECT * FROM emp) e WHERE rn<=10;
4、然后查出6-10条记录
SELECT * FROM (SELECT e.*, ROWNUM rn FROM (SELECT * FROM emp) e WHERE ROWNUM<=10) WHERE rn>=6;
```

**rownum和rowid都是伪列，但是两者的根本是不同的，rownum是根据sql查询出的结果给每行分配一个逻辑编号，所以你的sql不同也就会导致最终rownum不同，但是rowid是物理结构上的，在每条记录insert到数据库中时，都会有一个唯一的物理记录 ，这个记录是不会随着sql的改变而改变。**
**因此，这就导致了他们的使用场景不同了，通常在sql分页时或是查找某一范围内的记录时，我们会使用rownum。**

*****直接用rownum查询的数据，要包含rownum=1，因为rownum要从1开始查询记录

因此到查询如3到5行时，需要放在一个虚表中进行子查询（先在虚表中查询5行以内的，rownum从1-5，之后在查询该虚表中的rownum中的3-5，此时的rownum还是原来从1开始的记录的）

*****rowid可以用来作为处理其他数据相同的判断条件（如删除表中的重复数据，在子查询表数据中作为条件查询对应主表的数据）



## Oracle数据库的常用函数：

1、字符函数：

lower(char)：将字符串转化为小写的格式。
upper(char)：将字符串转化为大写的格式。
length(char)：返回字符串的长度。
substr(char, m, n)：截取字符串的子串，n代表取n个字符的意思，不是代表取到第n个
replace(char1, search_string, replace_string)  在char1字段中，进行字符串替换
instr(C1,C2,I,J) -->判断某字符或字符串是否存在，存在返回出现的位置的索引，否则返回小于1;在一个字符串中搜索指定的字符,返回发现指定的字符的位置;
C1 被搜索的字符串
C2 希望搜索的字符串
I 搜索的开始位置,默认为1
J 出现的位置,默认为1

```
问题：将所有员工的名字按小写的方式显示

	SQL> select lower(ename) from emp;

	问题：将所有员工的名字按大写的方式显示。

	SQL> select upper(ename) from emp;

	问题：显示正好为5个字符的员工的姓名。

	SQL> select * from emp where length(ename)=5;

	问题：显示所有员工姓名的前三个字符。

	SQL> select substr(ename, 1, 3) from emp;

	问题：以首字母大写,后面小写的方式显示所有员工的姓名。

	SQL> select upper(substr(ename,1,1)) || lower(substr(ename,2,length(ename)-1)) from emp;

	问题：以首字母小写,后面大写的方式显示所有员工的姓名。

	SQL> select lower(substr(ename,1,1)) || upper(substr(ename,2,length(ename)-1)) from emp;

	问题：显示所有员工的姓名，用“我是老虎”替换所有“A”

	SQL> select replace(ename,'A', '我是老虎') from emp;

	问题：instr(char1,char2,[,n[,m]])用法

	SQL> select instr('azhangsanbcd', 'zhangsan') from dual; --返回2

	SQL> select instr('oracle traning', 'ra', 1, 1) instring from dual; --返回2

	SQL> select instr('oracle traning', 'ra', 1, 2) instring from dual; --返回9

	SQL> select instr('oracle traning', 'ra', 1, 3) instring from dual; --返回0
```



2、数学函数

数学函数的输入参数和返回值的数据类型都是数字类型的，包含常用数学函数

3、日期函数 

日期函数用于处理date类型的数据。默认情况下日期格式是dd-mon-yy 即“12-7 -19”

（根据数据库系统的字符集格式变更，可以手动修改默认值）

sysdate  返回当前系统时间（格式为默认值）

sysdate+1  返回当前系统时间加一天的时间

last_day(date) 返回指定时间的当月最后一天的此时时间

NEXT_DAY(SYSDATE,1) 返回下个星期时间，1代表星期日

ADD_MONTHS（sysdate，1）返回当前时间加一个月的时间

**不止sysdate，对于数据库中的时间字段，也可以直接进行+ 、-操作**

**常使用sysdate来作为数据插入时间字键的默认值**

4、转化函数

to_char()  将时间类型、数字类型转化为字符串

````java
to_char(sysdate,'yy-mm-dd hh24:mi:ss')
````

to_date()   把字符串转化为date类型

trunc（datetime） 将获取当前时间的0点时间

to_number(VALUE, '999999.999') 将字符串转为数值类型

```
 to_date('2012-02-18 09:25:30','yyyy-mm-dd hh24:mi:ss')//将字符串转化为date
```

# Oracle日常问题集合

## 1、pl/sql工具对表的属性设置：

**一般情况下，都是用默认值**

  %free 10 --块保留10%的空间留给更新该块数据使用 
  initrans 1 --初始化事务槽的个数
  maxtrans 255 --最大事务槽的个数

```
initial 64k --区段(extent)一次扩展64k
minextents 1 --最小区段数
maxextents unlimited --最大区段无限制 
```

## 2、Oracle客户端连接注意事项：

查询当前oracle服务端的字符集编码：

select * from nls_database_parameters where parameter='NLS_CHARACTERSET';

查询当前连接oracle客户端的字符集编码：

select   userenv('language')  from   dual;不配置环境变量，默认为 SIMPLIFIED CHINESE_CHINA.AL32UTF8

1、数据库连接工具进行oracle数据库连接时，需要使用instantclient（oracle即时客户端），并且配置该客户端使用的环境变量（保证所有路径都能执行该客户端）：

Path：   instantclient路径

**由于数据库连接工具都可以直接配置oracle客户端环境，因此不需要该配置**

2、为保证oracle客户端连接所使用的字符集编码与服务器数据库一致，应配置环境变量

NLS_LANG： oracle服务器对应的字符集编码

3、当使用TNS连接时，需要在instantclient安装目录下，创建Network\Admin目录（必须与oracle原客户端目录一致，否则连接工具无法检测到该文件），并添加配置文件tnsnames.ora；（**也可以通过配置TNS_ADMIN环境变量来指定tnsnames.ora文件的路径：只对plsqDeveloper有效**）

4、一定要保证instantclient版本和数据库连接工具的版本一致

1. plsqlDeveloper使用TNS作为连接方式，它会自动扫描本地oracle安装文件中的客户端配置文件；因此在进行oracl远程连接时，需要进行TNS连接配置，此时就会扫描配置好的tnsnames.ora；当然也可以直接使用正常连接：ip:端口/oracle服务名  （oracle8i之后，一般数据库远程连接，对外都使用服务名）

   **plsqDeveloper在help->support info，可以指定所有配置路径**

2. Navicat Premium 12，自带一个精简版instantclient_10_2，但该版本不支持GBK编码，只支持Unicode（AL32UTF8属于unicode编码）、ASCII以及西欧字符集；因此需要下载一个完整版的instantclient客户端

## 3、Oracle创建自增字段：

1、创建oracle序列

create sequence 序列名  
       increment by 1 --每次增加几个，我这里是每次增加1 
       start with 1   --从1开始计数 
       nomaxvalue      --不设置最大值 
       nocycle         --一直累加，不循环 
       nocache        --不建缓冲区 

2、创建触发器

CREATE TRIGGER ContestDB_trigger 
    BEFORE INSERT ON 表名
    FOR EACH ROW 
    WHEN (new.主键名 is null) --只有在tid为空时，启动该触发器生成tid号 
 begin 
    select 序列名.nextval into :new.主键名 from sys.dual; 
 end; 

怎样重置自增：

1、删除当前序列，然后重新创建

2、通过执行自增来调整当前序列值（操作比较麻烦）

## 4、Oracle分析函数，以及常用场景：

在Oracle8.16后，提供一种分析函数语法，用于计算基于一组数据为结果的聚合函数（计算参数为字段）：

1、非聚合函数返回的是一组数据；而聚合函数返回的是一行数据

2、分析函数需要搭配over（）开窗函数一起使用，over（）开窗函数：

其中包括三个分析子句：**分组(partition by), 排序(order by), 窗口(rows)**

​		**开窗函数是对于分析函数来说，就是对于除分析函数外的sql语句查询进行进一步数据处理，分组、排序、窗口；sql 语句中存在order by时例外，当sql语句中的orderby和窗口函数中的orderby相同时，可以看作为sql语句数据先进行了处理，然后分析函数在对数据进行处理；当两者不相同时，sql语句中的排序，会最后执行**

**窗口代表了分析函数处理当前数据的选择范围，默认为全部，根据是否使用partition by 语句，还有所不同：**

​	1、当窗口函数中出现分组子句（partition by ）时，属性对应于分组后，每组数据中的第一行、当前行、最后行

​	2、当窗口函数中没有分组子句时，属性对应于查询表中数据的第一行、当前行、最后行	

**窗口子句语法为：rows between xxx  and xxx**

常用属性有：

第一行是 unbounded preceding，

前一行：1 preceding

当前行是 current row，

最后一行是 unbounded following,

 后一行：1following

**窗口子句不能单独出现，必须有order by子句时才能出现；当没有窗口子句，但有order by 时，默认为当组中的第一行到当前行，因此需要使用全组数据时，要么省略order by，要么在order by基础上指定窗口**

3、常用分析函数有：

1、计算：sum（column）、avg（column）、max（column）、min（column）##它们本身也是聚合函数

2、排名：row_number()   、rank（）、dense_rank（）

row_number() 在出现相同数据时，按照数据出现前后排名，不产生重复值

rank（），在出现相同数据时，排名会相同，但之后出现不同数据的排名不再连续，而是根据前面数据总数进行排名

dense_rank(),在出现相同数据时，排名会相同,之后出现不同数据的排名会紧跟其后，连接

**排名默认需要使用全部数据，因此无法指定窗口函数中的窗口子句**

3、取值：first_value(column)、last_value()、lag（）、lead（）

**first_value(column)、last_value(column)：**

​		取开窗函数处理后的第一行/最后一行数据对应的字段值，当开窗函数进行分组时，会取每组中的第一个值；**一般情况下，我们取的第一个值都是通过排序得到的，使用了order by,并且我们需要保证排序的数据总是为当前全部数据，因此需要添加窗口子句（防止只有order by时，窗口默认为当组第一行到当前行数据）**

**lag（）、lead（）：**

一次查询中取出同一字段的前n行的数据和后n行的值；语法从理论上是省略一部分数据，从而获取需要的数据

lag（column，n，‘null’）：获取column字段，省略全部数据中的后n行数据，并且可以指定省略数据的代替值为null（当然也可以不指定，从而为null）

lead（column，n，‘null’）：获取column字段，省略全部数据中的前n行数据，并且指定省略数据代替值为null（当然也可以不指定，从而为null）

**该分析函数的处理前体是数据有一定的排序，因此必须使用order by子句，由于需要全部数据，因此不能指定rows窗口子句**

4、常用使用场景：

1、first_value(column)、last_value(column)，可以用于当前场景：

- 查询消费表中，所有人员消费的最大值(最小值)，追加在所有数据后：

````sql
select t.*,FIRST_value(name) over(partition by name order by value rows between unbounded preceding and unbounded following) max_value 
from Test t
````

- 查询当前月份的前一个月和后一个月的销售额

````java
select t.*,
first_value(password) over(order by id rows between 1 preceding and current row),
last_value(password) over (order by id rows between current row and 1 following)
from test t 
````

2、row_number()   、rank（）、dense_rank（），可以用于场景：查询消费表中，所有人员各最大消费的那条记录（rank（）、dense_rank(),会由于最大消费数有相同，而出现多条）

````sql
select * from 
(select t.* ,row_number() over(partition by name  order by password) mm from test t) where mm=1
````

3、sum（column）、avg（column）、max（column）、min（column），可以用于场景：

- 每月销售额记录表中，查询每月数据和今年一月到当前月的总销售额、平均销售额、最大销售额、最小销售额

````sql
select t.*,sum(month_value) over(order by month) from test t where time =trunc(SYSDATE)
````

- 每月销售额记录表中，今年所有月份的总销售额、平均销售额、最大销售额、最小销售额

````sql
select t.*,sum(month_value) over() from test t where time =trunc(SYSDATE) order by month
````

4、lag（）、lead（），可以用于场景：

- 查询当前月份的前一个月和后一个月的销售额（相对于first_value(),不会由于第一行和最后一行，而产生错误数据，而是用其他数据填充）

````sql
select t.*,
lag(value,1) over (order by month ),
lead(value,1) over (order by month)
from test t 
````

**lag(value ,1)与t.* 结合展示时，相当于是将value本来的排序数据，往下一行开始进行展示，并缺少最后一条数据**

## 4、oracle进行单条sql批量insert操作：

INSERT ALL
   INTO MY_TABLE (ID, CREATE_TIME, MODIFY_TIME, NAME) VALUES ('1', SYSDATE, SYSDATE, '我的表 1')
   INTO MY_TABLE (ID, CREATE_TIME, MODIFY_TIME, NAME) VALUES ('2', SYSDATE, SYSDATE, '我的表 2')
   INTO YOUR_TABLE (ID, CREATE_TIME, MODIFY_TIME, NAME) VALUES ('1', SYSDATE, SYSDATE, '你的表 1')
 SELECT * FROM dual

**dual为一个空表**，oracle中经常使用 SELECT * FROM dual该语句，作为一些特殊sql的补充；防止oracle报缺失关键字select的错误

**1、INSERT ALL必须搭配一个子查询使用，无论是否需要该子查询的表数据**

**2、既可以插入同一个表，又可以插入不同表**

## 5、oracle按时间每小时分组：

````sql
select to_char(DATAUPDATETIME-4, 'yyyymmddhh24'),max(value) from zyz where DATAUPDATETIME>trunc(sysdate-4) and DATAUPDATETIME<trunc(sysdate-3)
group by to_char(DATAUPDATETIME-4, 'yyyymmddhh24') order by to_char(DATAUPDATETIME-4, 'yyyymmddhh24') 
````

## 6、Oracle DBlink

**用于在本地数据库，远程访问另外一个数据库中的数据，并进行增删改操作**

在实际开发过程中，会直接创建两个数据库的连接，然后进行操作访问；但有了DBlink后，就可以把两个不同数据库的表数据，交给oracl进行sql处理（数据库处理效率和便利性，远大于JAVA）

## 7、通过过程函数，来筛选varchar字段中，无法转换为number的数据

```sql
CREATE OR REPLACE
FUNCTION isnumeric (str IN VARCHAR2)  RETURN NUMBER IS
 v_str  FLOAT; --定义临时变量
BEGIN
 IF str IS NULL --如果输入的字符串是空
   THEN
      RETURN 0;--返回0，结束方法
   ELSE
     BEGIN
       SELECT TO_NUMBER (str) INTO v_str  FROM DUAL; --将字符串转number格式
       EXCEPTION --异常
       WHEN INVALID_NUMBER  --出现异常【INVALID_NUMBER：Oracle定义的异常类型】
       THEN
         RETURN 0; --返回0，结束方法
       END;
```

## 8、sql控制流函数/语句

```
select decode(numberA，0,1,-1) from table   //字段A为0，则返回1，反之返回-1

select decode（SIGN（numberA-300），1，好，0，差，-1，差）//判断A字段大于、等于、小于300；依次返回好、中、差

nvl（column，10） 字段colum为null是，自动替换为10

CASE   WHEN 布尔表达式（条件，一般与表字段相关） THEN 结果  {WHEN（条件，一般与表字段相关） THEN 结果}.....
ELSE 结果  END   //多条件中可以为任意字段
```

## 9、Oracle索引：

- 索引分类：B树索引（普通索引）、位图索引、唯一索引、函数索引

- B数索引结构：

  每个节点保留索引关键值和执行下个子节点的指针；而最后一行则保存索引key值以及rowID（实际数据的物理存储地址）；并且最后一行时连续的，而索引本身有序，可以实现索引范围查找

- 唯一索引：

  和B数索引一样，只是所有节点的关键字没有重复值，因此速度更快（默认将主键使用唯一索引）

- 位图索引：

  根据数据对于每一个rowID提供一个位图值(BIT 值)，对应当前一个或多个字段的索引key，当进行条件查询时，就对位图进行位运算，取出bit值符合实际条件结构的rowID

  优点：存储空间小，执行速度快

  缺点：在插入、更新时需要计算位图值，因此不适合频繁更新的字段；列的特征值过多，导致位图向量太多，从而bit值种类太多，索引太多得不偿失

## 10、Oracle优化：

**使用客户端工具中的解释，能够清楚知道sql执行过程**

