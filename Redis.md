# Redis

## nosql，非关系型数据库，not only sql：

### 1、nosql的产生：

对于网站访问量不大时，单个数据库可以很好的完成网站页面的数据交互；但是随着互联网的发展，单个关系型数据库开始面临以下问题：

数据量太大、内存无法放下大数据索引、数据库无法满足高并发的访问量（数据读写）

**互联网需求面临3V+3高：海量Volume、多样Variety、实时Velocity；高并发、高可扩、高性能**

因此为了解决以上问题，数据库架构开始发展：

1、Memcached（缓存）+mysql+垂直拆分：

通过Memcached（缓存）减轻数据库访问压力（将短时间不变的查询结果放入内存）；使用多个数据库实例来分担数据量（使用垂直拆分：按照业务分表，将不同业务的表数据放在不同数据库实例中）

2、mysql主从复制，读写分离：

数据库分为一个主库和多个从库，主库和从库存储相同数据，当主库数据更新，从库也立即更新。并其主库只用于更新数，从库进行读取数据，实现数据库读写分离。**在保证数据库具有容灾备份能力的同时，增加数据库的并发能力**

3、分库分表+水平拆分+mysql集群：

当垂直拆分也无法解决单表数据量大的问题是，则使用水平拆分单表数据拆分成几份，放在不同的数据库中，这些数据库组成集群，为同一个业务提供服务

通过以上的发展，一个项目需要通过多个的数据库来共同完成业务的数据访问。但关系型数据库的固定模式还是无法满足社交网络的复杂庞大的数据（**一个关系型数据表只能扩充几百个字段，在大数据量下，扩容字段的代价非常大**）

### 2、nosql的优点：

1、易扩展，不受关系型数据库的固定模式的束缚

2、处理大数据量，具有非常高的读写性能

3、多样灵活的数据模型

### 3、常用nosql四大分类：

按照数据模型进行分类：

1、KV键值数据库：Redis、Memcache

2、文档型（使用BSON格式）数据库：Mongdb   （BSON  储存json文档的二进制编码）

3、列存储数据库：HBase、Cassandra

4、图关系数据库：Neo4J、InfoGrid

### 4、分布式数据库CAP理论+BASE理论：

**CAP理论：在分布式系统中，对于数据读写操作，只能保证强一致性、可用性、分区容错性其中的两种，必须有一种被牺牲**

Consistency（强一致性）：用户当前读到的数据为数据库写入的最新数据

Availability（可用性）：每个分布式数据库节点都能保证在合理时间进行合理响应（系统不能崩）

Partition tolerance（分区容错性）：当分布式数据库任意一节点出现故障时，也能对外保证数据的可用性、一致性。

对于分布式数据库，我们必须保证分区容错性，因此需要对可用性和强一致性进行取舍。

BASE理论：基本可用（Basically Available）、软状态（Soft state）、最终一致性（Eventually consistent）

**通过让系统放松对某一时刻对数据一致性的要求，来换取系统整体的性能使用，但最终还是保证系统数据的强一致性**

## Redis：

### 1、什么是redis：

- **C语言编写、开源、遵守BSD协议、高性能的KV Nosql数据库，基于内存运行，并支持数据持久化**

- **Redis相比于其他kv数据库（Memcache）的优点：**

  1、Redis支持数据的持久化：可以将内存中的数据保存到硬盘中，在下次重启时加载

  2、Redis不仅仅支持简单的key-value数据类型，还提供list、set、zset、hash等

  3、Redis支持数据备份，即master-slave模式的数据备份（主从复制）

### 2、redis基础知识：

- 在硬件允许情况下的最高性能：读11万次/s、写8万次/s
- 单线程多路I/O复用处理读写请求
- 默认16个数据库，下标从0开始；当不指定数据库下标时，默认使用0号库
- redis默认没有密码（因为推荐在linux环境下使用，已经具有很高的安全性）；也可以统一管理所有库的密码
- 默认端口为6379
- redis基本常用命令：

| 命令行     | 作用                               |
| ---------- | ---------------------------------- |
| select   0 | 切换数据库                         |
| dbsize     | 获取当前数据库的key的总数          |
| keys   *   | 获取当前数据库的所有key的列表      |
| keys   k?  | 使用Ant匹配风格来查找满足条件的key |
| flushDB    | 清空当前库                         |
| flushAll   | 清空所有库                         |

### 3、reids数据类型：

String（字符串类型）：二进制安全（可以包含任何数据类型，如图片、java序列号对象）、最大值为512M

List（列表）：底层为一个字符串链表，类似于java的linkedList<String>;

Hash（哈希）：为一个键值对集合，类似于java的map<String,String> ,是一个String类型的field和value的映射表

Set（无序不重复集合）：String类型的无序集合，通过hashTable实现

Zset(有序不重复集合)：Zset相对于Set，会将所有元素关联一个double类型的分数；Zset的所有元素就根据这些分数来进行排序，

**redis所有数据类型都不能插入null**

5、1+5（key+五种数据类型）常用命令：

- key的常用命令：

| 命令行      | 作用                                   |
| ----------- | -------------------------------------- |
| exists key  | 该key是否存在                          |
| move key db | 将该key转移到另一个库                  |
| expire key  | 设置该key的过期时间                    |
| ttl key     | 查看还有多久过期；-1永不过期、-2已过期 |
| type key    | key指定value的类型                     |

- String的常用命令：

| 命令行                   | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| set  key  value          | 添加String类型的值（key存在时，进行覆盖）                    |
| Incr/decr   key          | 对于数字类型的值进行递增或递减                               |
| get key                  | 获取key指定的value                                           |
| getrange  key  0   3     | 获取该key的string值从0到3的截取（0 -1 相当于get key 获取整个String值） |
| setrange   key   0   xxx | 从下标为0开始覆盖xxx字符串                                   |
| setex    key   10  value | 添加String类型的值，同时设置过期时间                         |
| setnx  key   value       | 当key存在时，添加失败返回0，成功返回1（key存在时，不覆盖）   |
| mset/mget/msetnx         | 进行多步大量操作：set、get、setnx（其中一个添加失败，则全部添加失败） |

- Hash的常用命令：

| 命令行                              | 作用                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| hset/hsetnx   hash  key  value      | 添加hash键值对(hsetnx指该key存在时，不进行覆盖)              |
| hget  hash  key                     | 获得hash key映射的值                                         |
| hmset hash key1 value1  key2 value2 | 大量添加hash键值对                                           |
| hmget hash key1  key2 key3          | 大量获取指定key映射的值                                      |
| hgetall hash                        | 获取所有的key-value（按照key1-value1-key2-value2的顺序排列） |
| hkeys hash                          | 获得hash所有键值对中的key                                    |
| hvals  hash                         | 获得hash所有键值对中的vlaue                                  |
| hdel  hash key                      | 删除指定key的键值对                                          |
| hlen   hash                         | 获取hash的键值对个数                                         |
| hexists hash  key                   | 判断hash中是否有key的键值对                                  |
| hincrby hash  key  2                | 对于hash中key映射的值（值必须为数字）进行+2                  |
| hincrbyfloat hash key  2.111        | 对于hash中key映射的值（值必须为数字）进行+2.111              |

- List的常用命令：

| 命令行                                          | 作用                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| lpush list  1 2 3 4 5                           | 添加一个list类型的kv，数据从最左端插入（则数据顺序为 54321） |
| rpush list 1 2 3 4 5                            | 添加一个list类型的kv，数据从最左端插入（则数据顺序为 12345） |
| lrange list  0 2                                | 获取list下标为0到2的所有数据（1，2，3）<br />只能使用 0 -1来获取整个list中的数据，<br />没有get list的命令行，来获取整个数据 |
| lpop  list                                      | 获取list最左端的数据(并且list中没有该数据)                   |
| rpop  list                                      | 获取list最右端的数据(并且list中没有该数据)                   |
| lindex list 2                                   | 获取list下标为2的数据                                        |
| llen  list                                      | 获取list长度（多少个数据）                                   |
| ltrim list  0  2                                | 截取list中下标为0-2的数据，然后再放入到原来list中            |
| lset  list   1    value                         | 给list中下标为1的位置赋值（只能覆盖原有的值，若当前下标不存在，则添加失败，超出下标界限） |
| linsert  list before/after<br /> value1  value2 | 在list值为value1的前面/后面添加value2（存在多个value1是，只在第一个value前后进行添加） |
| lrem key  2  value                              | 删除list中2个为vlaue的数据（从左往右依次删除，没有这么多value的数据时，不会报错） |
| rpop（lpop）lpush（rpush） list1 list2          | 将pop、push中和使用，从list1中pop的数据psuh到list中          |

**如果list中的值全部移除，这该list对于的key也会消失（不允许空list存在）；**
**由于是链表结构，头尾操作效率高，中间操作效率低**
**当list存在时，则在该list的基础上进行操作；不存在，则创建一个空的list**

- Set的常用命令：

| 命令行                | 作用                               |
| --------------------- | ---------------------------------- |
| sadd set 1 2 3        | 添加数据，重复数据不插入           |
| smembers  set         | 获取set中的所有数据                |
| sismember set  value  | set中是否存在value                 |
| scard  set            | 获取set中的元素个数                |
| srem  set  value      | 删除set中的value值                 |
| srandmember set  3    | 从set中随机拿出三个值              |
| spop set              | 从set中随机出栈一个值              |
| smove set1 set2 value | 将set1中的value移动的set2中        |
| sdiff set1  set2      | 获取set1中不属于set2中的元素数据   |
| sinter  set1  set2    | 获取set1 中属于set2中的元素数据    |
| sunion set1 set2      | 获取set1和set2中的并集所有元素数据 |

- Zset的常用命令：

| 命令行                                                     | 作用                                                         |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| zadd   zset   value1  11   value2  22                      | 添加zset数据，在每个值后面绑定一个分数（整数和小数）         |
| zrange zset  0   2                                         | 获取zset排序下，下标从0到2的值（1到  -1就是全部数据）        |
| zrange zset  0  -1  withScores                             | 获取zset的所有元素数据，并显示其对应的socres值               |
| zrangebyscore  zset   （10 20   （withScores）(limit  0 2) | 获取zset元素数据中，socres在10-20（包含20）直接的值（根据”（“不包含符号来判断）,还可以使用limit对值进行截取 |
| zrem zset value                                            | 删除zest中值为value数据                                      |
| zcard   zset                                               | 获得zest中的元素个数                                         |
| zcount zset  10   20                                       | 获得zset中元素绑定分数在10-20（都不包含）直接个数            |
| zscore  zset  value                                        | 获得zest中value绑定的分数值                                  |
| zrank  zset value                                          | 获取zset中指定value的下标值                                  |
| Zrevrank zset value                                        | 获取zest中指定value下标的反向值                              |
| zrevrange zset 0 -1                                        | 对于获取的数据进行反向排序                                   |
| zrervangebyscore zset  20  10                              | 将score范围查询前后颠倒，查询满足score条件的反向数据排序     |

### 4、redis5种数据类型的实际使用场景：

String：最常规的数据存储，并可以进行一些简单的数字计算（计算器）

hash：用于存放结构化对象，来方便操作对象中的某个字段（也可以使用String将整个对象序列化保存，然后使用时反序列化取出操作对象中的字段，但这样大大增加增加了读取负担）；如更好的模拟session工作

list：做简单的消息队列，也可以做数据分页

set：用于数据去重、取数据交集、并集、差集的运算（不使用java自身set，减少在分布式系统中，设置全局变量复杂性）

zset：通过排序参数score来对数据集合进行排序，常用于排行榜

### 5、redis.conf配置文件常用配置：

**windows版，默认安装目录下的redis-server.exe不读取配置文件，因此需要手动设置执行命令（1、将配置文件拖到执行文件下打开2、当前目录下打开cmd执行redis -server   redis.windows.conf 3、或者创建windows服务，添加执行该配置文件的命令），使自定义配置文件生效**

1、redis只支持byte级别以上的大小单位（即不支持bit），并且对单位大小写不敏感

2、可以通过include  conf文件路径，可以整合redis的其他配置文件

3、通用标准配置：

daemonize   yes    默认为no，只能在linux系统中使用，将redis作为后台进程（linux也叫守护进程），不会由于终端控制器的退出而关闭，一直会随着服务器在后台运行（类似于window的服务，需要手动kill进程）

pidfile  “”                设置保存redis进程pid号信息的文件路径（Linux下才生效）

**###Windwos下，将redis作为后台进程：**

​		**在redis安装目录下打开dos窗口，输入：redis-server  --service-install redis.windows.conf  --loglevel verbose**

port  6379              设置redis的服务端口，默认为6379

bind   ip                  设置redis服务端可以监听的ip（允许连接ip），默认允许全部ip

tcp-backlog  511    tcp连接队列数，在高并发下通过连接队列防止慢连接的阻塞

tcp-keepalive 0      tcp连接生存时间，0代表一直连接，没有过期时间；建议设置为60，减少维护空闲tcp连接的消耗

loglevel  notice      redis日志级别（redis四个级别，debug、verbose、notice、warning级别越来越高，一般默认为notice，开发时推荐使用debug，获得更多日志信息）

logfile  “”               redis日志文件路径，默认为控制台

database  16          redis默认16个库

4、redis security（安全）配置，默认全部注解不使用：

通过config  get  requirepass   获取当前redis的密码（默认为“”空）

config set  requirepass  “1234”  设置redis密码

此时需要先使用 auth  1234 来给与reids客户端权限，否则所有操作都会报错no auth

5、redis限制配置

maxclients：最大连接数，默认10000，一般设置为128

maxmemory：最大使用内存数（byte），默认无限制

maxmemory-policy：设置ridis内存占用达到最大值后的处理策略：

​		noevication：无法写新数据，直接报错（不要使用）

​		allkeys-lru：使用lru算法计算出使用最少的key，将其删除（推荐使用）

​		allkeys-random：随机删除key（不要使用）

​		volatile-lru：对于已过期的key,使用使用lru算法计算出使用最少，将其删除

​		volatile-random：对于已过期的key，随机删除

​		volatile-ttl：对于已过期的key，删除更早过期的key

后三种，前提条件都为已过期的key，当redis并不是当作缓存，存储的都是不过期的key时，这样无法释放redis内存

**##redis的过期策略：定期删除+惰性删除**

并不会根据ttl使用定时过期策略，这样会消耗大量的cpu资源

**定期删除：redis默认100ms检查是否有过期key，但不会检查所有key，只会随机抽查几个设置过期的key进行判断，因此这样会导致还有许多过期key没有删除**

**惰性删除：每当redis请求get key时，会自动检查该过期时间，过期就删除，返回key不存在**

### 6、redis的持久化：

#### 1、RDB（Redis DataBase）：

- **原理：**

在指定的时间间隔内，将内存中的数据集快照写入磁盘；并可以根据保存的持久化文件，在redis重启时，将备份数据加载到redis内存中

**fork方式备份：**先将当前数据fork（复制分支）一份（fork时会redis会发生短暂阻塞，时间很短），然后创建一个后台子进程来将fork的数据集快照，写入到磁盘中生成dump.rdb，期间源数据在redis主进程成中继续进行redis命令操作

RDB保存的快照数据（数据持久化文件）默认为dump.rdb

- **RDB常用配置：**

1、RDB默认执行间隔：1分钟内改1万次、5分钟内改10次、15分钟改1次

通过save  100  1  来设置100s改1次时，触发RDB持久化

通过 save ”“   来设置不自动触发RDB持久化

**可以在redis中手动命令行  save/bgsave 来触发RDB持久化、使用flushdb时也可以触发（没有意义）**

**save是阻塞式备份（直接将当前数据进行持久化，因此不能进行redis请求响应）；bgsave是通过fork方式来完成，（所有自动触发的rdb都是bgsave，推荐使用bgsave手动保存）**

2、其他配置（推荐使用默认）；

stop-writes-on-bgsave-error yes：RDB保存出错时，redis还是继续允许写入

rdbcompression yes：RDB进行持久化快照保存时，使用LZF压缩算法

rdbchecksum yes：RDB进行持久化快照保存后，使用CRC64算法进行数据校验

dbfilename  dump.rdb   RDB保存的持久化文件名（也是redis启动自动加载持久化备份文件名）

dir ./    RBD保存的持久化文件路径（也是redis启动自动加载持久化备份文件路径）

- **RDB的优缺点：**

优点：适合大规模的数据恢复，且对于数据的完整性和一致性要求不高

缺点：1、由于在一定间隔时间做一个备份，因此当redis意外down是，会导致最后一次快照后的所有修改丢失

​            2、Fork时，会复制一份内存中的全部数据，导致内存直接2倍膨胀

#### 2、AOF（Append（追加）  Only File（只读文件））：

- **原理：**

以日志的形式来记录所有的写操作，当redis重启时，将从前到后执行所有写的操作，来还原redis之前的数据

AOF保存的追加文件名默认为 appendonly.aof

- **AOF常用配置：**

appendonly  no   默认AOF持久化不打开

appendfilename   “appendonly.aof”  AOF持久化保存/读取的默认文件名

appendfsync   everysec    使用fsync()函数，将缓冲区记录的写操作日志记录写入到系统磁盘中：

​		everysec（默认）：异步操作，每一秒，但可能在宕机时，发生前1s内的数据丢失（折中处理，推荐使用）

​		always：同步持久化，每次数据写入都立即同步持久化到磁盘，这样严重影响redis的并发性能，但能保证数据的完整性

​		no：将AOF记录数据从缓冲区写入到磁盘的时间由操作系统自己决定（这样能大大减少AOF持久化带来的性能损失，但也增加了AOF备份数据的丢失量：可能缓冲区保存了大量数据，没有写入到磁盘中，系统宕机导致需要持久化的数据丢失）

- **AOF异常恢复：**

AOF文件会由于网络异常、物理异常原因，导致文件写入损坏。可以通过redis-check-aof来进行文件恢复，然后重启redis，正常进行aof文件加载。

- **AOF的Rewrite机制：**

由于AOF使用的文件追加方式，当出现大量的写入操作时，会使appendonly.aof越来越大。此时就需要将aof备份文件进行重写简化。**减少AOF持久化文件的大小，同时减少AOF还原数据的时间**

AOF重写原理：AOF不是通过操作原来的appendonly.aof文件，而是重新扫描redis数据库，保存所有当前数据写入的记录（不是数据实际写入的记录，而是数据最终形成的写入记录，如本来是记录分步插入了list的元素，此时直接记录list同时插入所有元素的写入记录）

AOF重写触发机制：redis会记录上一次重写的AOF大小，默认配置当AOF文件大小为上次一倍，且大于64M时触发

AOF重写默认配置修改：

​		no-appendfsync-on-rewrite  no，不使用fsync进行同步，默认为no，即使用fsync同步，保证数据安全性

​		auto-aof-rewrite-min-size：重写触发机制的最小文件大小，默认64M

​		auto-aof-rewrite-percentage：重写触发机制重写文件相对于上一次重写文件的追加百分比，默认100，即两倍大小

- **AOF的优缺点：**

优点：在默认使用fsync同步，每秒或即时同步时，更好的保证了数据的完整性

缺点：1、在相同数据集持久化时，aof文件远大于rdb文件，并且恢复速度慢

​			2、使用fsync同步时，AOF持久化运行效率慢于RDB，不使用同步时和RDB一样

#### 3、RDB和AOF总结

1、当只使用Redis作为缓存时，即不需要对内存中的数据进行持久化，那么不需要使用任何持久化方式

2、**AOF和RDB共同存在时，Redis启动后，会优先使用AOF的appendonly.aof文件进行数据恢复（其保存的数据更加完整）**

3、RDB更适合于备份数据库，速度效率更高，可以更好的使redis服务快速重启

4、AOF能更好的作为redis的数据恢复，保证数据的完整性

因此一般使用RDB作为数据备份的最终保证（冷备份，有时AOF会出现bug导致恢复异常），使用AOF来容灾备份，及时恢复完整数据（热备份，作为数据恢复的第一选择）

##可以使用主从复制来代替AOF的作用，减少AOF带来的频繁IO（将写操作记录日志写入磁盘）和rewrite带来的性能波动（重写期间也可以进行redis请求，但是会降低性能）

### 7、redis的事务

- **redis事务原理：**

可以一次执行多个命令，按顺序串行化执行，而不被其他命令插入（只有这些命令完成后，才能响应其他命令；因为redis是单线程的）

**在一个队列中，一次性、顺序性、排他性的执行一系列命令，相对于RDB（关系型数据库）事务ACID的区别：**

1、不满足原子性（要么全部成功，要么全部失败）：

​		1、当事务块中的命令出现检查性错误时（如命令拼写、使用错误），会直接在添加到事务命令行队列时报错，当exec执行时，就会导致事务所有命令全部失败

​		2、当事务块中的命令出现运行性错误时（对于数据的操作不合理），exec执行事务，只会导致该命令行失败

2、没有隔离级别，因为事务提交前不会由任何命令被实际执行，只要成功执行就不会被其他客户端发送的请求打断，强制按顺序执行所有事务命令（保证了事务的隔离性）

3、redis不支持回滚，并且会出现部分事务命令成功，因此不能保证事务的一致性

4、redis持久化，在AOF的always的SYNC同步模式下，可以保证事务的持久化性

- **redis事务命令：**

| 命令行             | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| discard            | 取消事务                                                     |
| exec               | 执行事务（会自动执行unwatch，取消所有key的监控）             |
| multi              | 标记事务快命令行的开始点                                     |
| watch  key1   key2 | 监视一个或多个key                                            |
| unwatch            | 取消所有key的监视（当取消事务时，手动unwatch取消所有key的监控） |

- **redis事务执行过程：**

  1、标记事务开始点   multi

  2、事务块命令行入队    命令行

  3、执行事务，一次性（不会被其他客户端请求中断）、顺序性执行事务命令      exec

- **redis事务解决并发问题：**

当使用redis事务时，我们希望其他并发操作不去影响此时事务操作的数据。而redis是通过乐观锁来解决该问题的（使用watch命令来完成）：

​		乐观认为并发时，redis事务不会发生数据被其他客户端修改的情况。我们可以手动watch（监听）事务中的key，当事务执行前，监听的key值发生改变时，就会导致事务执行失败（事务队列命令不执行，返回nil事务执行失败，并且不管事务是否成功，都会自动取消所有key的监视）。

​		失败，则需要再次监听需要事务中的key，继续定义执行该事务（重复前面操作，直到事务成功）。

### 8、redis的发布订阅（消息服务）

**redis订阅发布命令：**

| 命令行                     | 作用                                                      |
| -------------------------- | --------------------------------------------------------- |
| subscribe c1 c2            | 订阅 c1、c2通道的消息推送                                 |
| psubscribe    c*           | 订阅所有c开头的通道的消息推送（多一个模式属性psubscribe） |
| publish   c2    hello word | 发布c2 通道消息 hello word                                |
| pubsub channels            | 获取当前redis服务器上所有订阅的通道                       |
| pubsub  numsub c2          | 获取c2通道上的订阅人数                                    |

任何客户端都可以进行订阅、发布；

**一般不使用redis作为消息中间件（无论是队列还是订阅），原因：**

1、没有应答机制，保证消息的消费，需要手动进行代码处理

2、redis只能默认全部持久化

3、redis的出队入队性能比mq差  

4、redis不提供queue消息的均衡分发消费

5、redis不提供整个消息中间件的运行监控

6、虽然有主从复制模式，来保证集群高可用；但是不能保证消息持久化的可靠性（对于主从复制进行数据同步时，不能保证消息的一致性）            

​       

​                     

​				



### 9、redis的主从复制（Master/Slave）

**1、原理：**

​		主机（master）redis数据更新后，根据配置和策略，自动同步到备用机（Slave）的redis中。并且Master以写为主，Slave以读为主。**保证数据的读写分离、容灾恢复**

​		在主从复制过程中，主库、从库进行数据同步时，并不会影响redis主线程的命令操作，他们还是继续可以操作源数据。（但在从库加载新数据（RDB文件、同步过程中的剩余写操作、命令传播，后两这数据量少，基本不会发送阻塞）时，会发生阻塞）

2、redis主从复制搭建：

- 配置：主机端口默认，只需要配置从机的conf文件（保证关键配置与主机的配置不同）：

​		1、开启daemonize yes、不同的pid文件路径（Liunx下，windows不需要）

​		2、设置不同端口、日志文件路径、RDB持久化的dump文件保存目录和文件名

- 常用操作命令：

  info  replication    查看当前redis节点主从复制角色信息（默认为master）

  slaveof    ip  端口  设置当前redis节点为指定ip、端口下redis的从库；

  - **此时从库redis将无法进行写操作，只能读数据**
  - **当主库宕机时，从库不会发生任何；主库重启后，主库也不会发生变化**
  - **当从库重启时，会恢复为master角色，需要重新执行 slaveof命令来配置主从复制（也可以在conf配置文件中，默认添加需要主从复制的主库地址 slaveof < masterip> < masterport>，当重启时自动执行slaveof命令）**
  
  slaveof   no   one   将当前slave（从机）身份改为master（主机）；停止与上一个主库的数据同步

- 复制原理：

  **Redis复制功能分为：同步（sync）和命令传播（command propagate）**

  当从库节点执行slaveof命令时，开始进行同步操作：

  ​	1、从库向指定的主库发送SYNC命令

  ​    2、收到SYNC命令的主库执行bgsave命令，进行RDB持久化生成一个RDB文件

  ​    3、在执行bgsave命令的同时，redis可以继续执行写操作，此时使用一个缓冲区记录从bgsave执行开始的所有写操作

  ​    4、当bgsave命令执行完后，主库将生成的RDB文件发送给从库。从库将RDB文件进行加载更新数据

  ​    5、当从库加载完RDB文件后，主库再将缓冲区保存的写命令发送给从库，从库按顺序执行该命令,此时由于缓冲区数据很少，因此从库完成此操作的速度很快，基本能保证主库和从库数据保证一致性

  ​    由于从库在加载RDB文件和缓冲区写操作记录后，完成了全表同步。现在当主库继续有写操作时，直接会将命令传播到从库进行执行，完成完整的主从复制

  ###当完成主从复制后的从库出现断线重连时，从库需要执行sync来同步断线时间内的写操作，此时有需要进行一次全表同步，严重影响了redis的性能和内存占用。因此在redis2.8版本后，使用PSYNC命令来代替SYNC，它具有原有完全同步和部分重同步功能。

  部分重同步：当salveof指定的主库不是之前主从复制的节点ID时，则执行全表同步；当是之前主从复制的节点ID，则执行部分重同步。首先对比主库和从库的复制偏移量，根据差值来查找主库复制积压缓冲区的最新数据，进行同步；

  在进行主从复制时，从库和主库会共同维护一个复制偏移量，每当主库进行写操作时，其复制偏移量+1；从库复制同步执行写操作时，其复制偏移量+1，保证两种相同；并且在主库将写操作传播到从库时，也会保存到复制积压缓冲区（默认1M），用于从库断开重连时，进行部分重同步

**2、主从复制模型：**

​	1、一主二仆（多仆）

一个主库，多个从库；主库只写数据，从库同步数据并只提供读操作

​	2、薪火相传（串联式的一主一仆）

一个主库对应一个从库，该从库又对应是下一个从库的主库。让所有redis节点成为一个串联的关系，但写数据的永远是第一个主库。

 ##相对于一主二仆，有效减轻了主库同步更新和命令传播的压力（只需要更新一个从库）。但是会导致其他子从库的更新延迟更高

​	3、反客为主（当主库挂掉时，可以使其他从库选一个作为主库）

首先其中一个从库执行salveof no one（取消主从复制的从库身份），然后将该节点的地址、端口修改为其他从库savleof 指定主从复制的主库地址、端口

##这样有效避免了主库挂断，系统无法继续正常运行的危害，但需要手动转化，人力无法做到及时处理

​	4、哨兵模式（自动版反客为主）

能够及时监控Master主库的运行状态，如果出现故障，则投票选出指定slave从库来转为主库；而其他从库以新的主库作为对象**（包括挂掉得老master，会转变为salve）**

**配置步骤：**

1、在Master对应redis.conf同目录下新建sentinel.conf文件

2、配置sentinel.conf文件：

​		port 该监视器的服务端口

​		sentinel monitor     ip   端口   票数（当从机得票数大于该值时，上位为主机）

3、启动sentinel来监控master主库

windows版：redis-server sentinel.conf  --sentinel

##一个sentinel服务可以监控多个master，但是sentinel自己也可能发生问题挂掉，会导致哨兵模式无法正常生效。（因此常常使用多个sentinel来集群监控）

**3、redis主从复制优缺点：**

由于主从复制在同步中,会有一定的延迟；在高并发的写操作或从库节点过多下，主库同步数据到从库的延迟更加严重；因此就会导致从库读取的数据和主库不一致。

# redis在java中的应用

1、原生可以使用 Jedis jar包来连接操作redis，并提供redis连接池技术

````java
public class JedisPoolUtil {
	private static volatile JedisPool jedisPool = null;

	private JedisPoolUtil() {

	}

	public static JedisPool getJedisPool() {
		if (jedisPool == null) {
			synchronized (JedisPoolUtil.class) {
				// redis客户端连接池配置
				JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
				jedisPoolConfig.setMaxTotal(100);// 最大连接数
				jedisPoolConfig.setMaxIdle(32);// 最大空闲连接数
				jedisPoolConfig.setMaxWaitMillis(100 * 1000);// 最长连接等待时间
				jedisPoolConfig.setTestOnBorrow(true);// 创建客户端实例时，检查是否可连接（pong）
				jedisPool = new JedisPool(jedisPoolConfig, "127.0.0.1", 6379);
			}
		}
		return jedisPool;
	}

}
````

2、使用redis客户端来执行命令

````java
public class TestMain {
	public static void main(String[] args) {

//		Jedis jedis = new Jedis("127.0.0.1",6379);
        //jedis一些常用API的方法名和redis命令一致
//		jedis.select(0);
//		jedis.set("yh", "12313");
//		jedis.flushDB();
		JedisPool jedisPool = JedisPoolUtil.getJedisPool();
		Jedis jedis = jedisPool.getResource();
		//归还Jedis客户端连接资源
		jedis.close();
	}
}
````

3、使用Jedis来操作redis事务(实例：使用redis事务的乐观机制，保证事务执行时，其操作数据不被其他客户端修改)

````java
	public static void main(String[] args) {
		Jedis jedis = new Jedis("127.0.0.1", 6379);
		jedis.select(0);
		boolean status = true;
		while (status) {
			jedis.watch("k1");
			try {
				Thread.sleep(7000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

			Transaction transaction = jedis.multi();
			transaction.set("k1", "v1");
			transaction.set("k2", "v2");
			List<Object> exec = transaction.exec();
			if (exec == null) {
				System.out.println("事务执行失败");
			} else {
				System.out.println("事务执行成功");
				status = false;
			}
		}
	}
````

# redis应用常见问题和解决方案

### 1、redis和数据库双写一致性问题：

在高并发读写下，缓存数据并不能和数据库保证强一致性，因此对于强一致性数据，不能放入缓存；

为了更高的一致性，缓存的更新策略为：

**先更新数据库，再删除缓存（当需要该数据时，再查询数据库然后生成缓存）**

### 2、redis缓存穿透和缓存雪崩问题：

- **缓存穿透：大量请求缓存不存在的数据，导致请求全部被数据库处理，最后使数据库崩溃**

解决方案：保证缓存中的数据失效时，立即更新获取最新数据：

1、使用互斥锁，保证在缓存在失效时，第一个使用使用数据库，快速更新数据，降低数据失效的空闲期

2、使用异步更新策略，在缓存失效后继续可以查询使用，并异步更新缓存

（前两种是针对于缓存失效导致的数据不存在）

3、使用合理的拦截器，拦截不合理参数请求的接口（保证参数传来的数据key是缓存内部存在的）

（后者是解决缓存中不可能存在该key时，被恶意访问该接口）

- **缓存雪崩：缓存出现同一时间大量key失效，导致请求全部被数据库处理，最后使数据库崩溃**

解决方案：

1、给缓存的过期时间添加一个随机值，防止key集体失效

2、对于缓存更新时，使用互斥锁（这样会大大降低请求效率）

3、使用双缓存机制，每个key使用两个缓冲，其中一个没有过期时间（将其作为备用缓冲），另一个设置过期时间正常使用，当后者过期没有数据时，直接使用前者；在该key数据更新时，两个同时更新

### 3、redis的并发竞争key问题：

1、使用redis事务来保证key的正常并发使用（只能在单集redis上生效，对于实际生产环境，基本都是部署redis集群，这样会导致事务操作的多个key不再同一个redis-server下，事务无法正常完成）

2、使用分布式锁，来保证key的操作隔离性（不能保证线程的哪个操作先进行）

​	使用分布式锁的同时，将key添加一个时间戳，来判断获得锁的操作是否在key前一次操作完成的时间后面；若在此之后，那正常执行set操作；在此之前，就无法执行set操作，释放锁给其他线程（这样会导致有些线程操作无法被执行，但保证了有序性）

