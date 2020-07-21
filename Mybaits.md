# Mybaits

## 1、ORM框架理论：

- ORM：Object Relational Mapping，对象关系映射，即将java对象与关系数据库中的数据进行映射，从而实现将java对象持久化到数据库，或从数据库获取数据映射到java对象中

- JDBC：java data base connection，java数据库连接，即通过JDBC实现java程序与数据库的交互，进行数据库数据的读写。并定义了java与数据库交互的标准规范过程：

  **创建数据库连接----------编写sql--------预编译----------设置参数----------执行sql-------------封装结果**

- ORM与JDBC的关系：

  ​		JDBC实现了java代码和底层数据库驱动程序的解耦，作为中间层来操作数据库；但JDBC在设计上没有很好地支持java语言面向对象编程特性，ORM就解决了这个问题。对JDBC进行进一步封装，通过使用JDBC来高效地将对象与数据库字段进行映射，来完成java与数据库地交互

- JPA:java persistent Api，java持久层Api，由java研发公司sun制定地一套ORM接口规范，目前常见的ORM框架，都是根据JPA进行实现，主要有三个技术思想：

1. 定义实现ORM映射的元数据形式：JPA支持XML和注解两种元数据，来描述对象和数据库表的关系
2. API：定义操作实体对象，执行CRUD操作的接口方法，从而在后台完成JDBC操作，将开发者从繁琐的JDBC对象创建和SQL代码编写的重复工作中 解放
3. 面向对象的sql语句：制定一种面向对象的sql，减少对数据库中元数据的耦合（既然使用的对象映射数据库表，则就编写sql时，不在管表名、字段名）

## 2、常见三种ORM框架：

目前常见ORM有：Hibernate、spring data jpa、mybatis

### 1、Hibernate：

​		全自动、全映射ORM框架，直接消除了sql的编写

缺点：

1、由于全自动，也因此带来了sql无法优化、复杂sql无法实现的问题（因此提供了一个HSQL语句方式，来实现自己定制sql，但这也带来了学习新sql语句的负担）

2、全字段映射，导致表字段过大时，只能一次性映射所有字段，无法选择单个字段映射，大大增加程序运行的额外负载（或者使用HSQL来手动获取）

因此可以看出，Hibernate很全面，支持全自动和手动，但必须学习HSQL才能进行sql优化，**HSQL是由Hibernate面向对象定制的sql语言，最终还是会转化为正常sql进行JDBC操作**

### 2、spring data jpa：

​		是spring提供的一套简化JPA开发的框架，相当于是在JPA的基础上，进行接口抽象封装，通过Hibernate的JPA技术进行部分实现，既能独立完成JDBC操作，又可以整合其他遵循JPA规范的ORM框架**(在整合mybatis时，其实两种没有联系，只是各自完整自己的sql执行工作)**

​		spring data jpa通过dao层Repository接口方法的命名规范，来规定方法对应的CRUD操作，并提供分页、排序、自定义使用定制sql；可以在不改变代码的情况下，直接整合其他实现jpa规范的ORM框架（本身已经整合了hibernate的JPA模块部分，但没有hibernate性能强大，单纯的只解决了ORM，但没有实现缓存、不同数据库平台的兼容等）

### 3、mybatis：

​		半自动ORM框架，需要开发者自己编写sql，然后对其他操作进行封装。

优点：

1、由于需要自己编写sql，因此可以最大程度上进行sql优化，并且适合sql频繁修改和只进行部分字段映射的使用场景

2、相对于JPA以及其实现类的ORM框架，不需要学习额外的定制sql语句，来编写复杂sql，因此学习成本低；

3、mybatis完全实现sql语句和java代码的解耦，而其他ORM框架只能实现定制sql语句的解耦（此时也就无法消除sql编写）

4、由于mybatis使用的**数据映射器**，是将sql查询结果进行实时映射，因此不需要数据模型和对象模型精确对应，让数据模型和对象模型两者独立；而其他ORM框架是使用**元数据映射器**，直接将jopo对象和数据库表映射，导致对象必须满足和数据库表的映射，此时一个jopo对象就只能处理一个表，导致灵活性很差

5、mybatis完全支持存储过程，而其他ORM框架不能太好的兼容

### 4、spring data jdbc和JdbcTemplate：不属于ORM框架

## 3、mybatis的使用：

### 1、mybatis配置使用过程（整合spring）：

1. 在spring容器的配置文件中，配置数据源、JDBC连接池和JDBC事务管理器
2. 通过数据源来创建mybatis的sqlSessionFactory、sqlSessionTemplate对象
3. 配置mybatis mapper接口扫描器，指定获取mapperProxy对象使用的sqlSessionTemplate对象
4. 创建mapper接口对应的数据映射器，mybatis支持使用注解和XML两种方式进行配置创建映射器；**当两种同时使用时，XML方式会覆盖掉注解方式**
5. 使用spring容器依赖注入mapper接口实现类mapperProxy，进行方法调用，从而实现ORM

### 2、mybatis的数据映射器的两种配置方式(不整合spring)：

#### 1、xml方式：

- 创建mapper.xml文件，编写mapper标签(之后可以直接通过sqlSession对象读取xml文件元数据，进行ORM操作)
- 使用mybatis-config.xml配置文件中的mappers标签，将注册mapper接口（只有注册后,其对应的mapper.xml才能被sqlSession使用）

**UserMapper.xml**（sql映射文件）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">    //xml标签约束文件，定义需要可以使用的标签
        
<mapper namespace="com.yh.test.UserMapper">
    <!-- 定义一个查询sql, -->
    <select id="getUserById" parameterType="String" resultType="com.yh.entity">
        select * from student where id =#{id}
    </select>
</mapper>
```

**mybatis-config.xml**（可以使用resource/class/package方式注册）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">	
<configuration>
    。。。。。//数据库配置、事务管理和mybatis的sql操作对象
 	<mappers>
 		<mapper resource="mappers/AccountMapper.hbm.xml" /></mappers>
 		<mapper resource="mappers/CarMapper.hbm.xml" /></mappers>
	</mappers>
</configuration>	
```

**UserMapper.class**

```java
public interface UserMapper{
	//需要方法名和UserMapper.xml中crud标签的id属性对应，并且参数类型和返回类型也要一一对应
    User getUserByid( String id);
}
```

#### 2、接口注解方式：

- 创建mapper接口，编写方法，并在方法上添加需要执行的sql注解
- 使用mybatis-config.xml配置文件中的mappers标签，将注册mapper接口（只有注册后,其对应的mapper.xml才能被sqlSession使用）

**UserMapper.class**

```java
public interface UserMapper{
    @Select("select * from User where id=#{id}")
    User getUserByid(@Param("id") String id);
}
```

**mybatis-config.xml**(只能使用class/package方式注册)

```xml
<!-- 配置mybatis mapper接口 -->
<mappers>
	<mapper  class="org.mybatis.builder.UserMapper"/>  
</mappers>
```

#### 3、两种映射配置方式的选择：

对于简单的CRUD可以使用注解来完成，开发快速

对于复杂且易变的通过xml注解完成，方便修改，不需要重新编译（由于java注解的缺陷，代码动态sql编写不方便）

### 3、mybatis数据映射器的两种使用方式：

1、读取XML配置文件进行代码编程：(只能通过xml注解的方式完成配置，基本不会使用)

```java
SqlSession openSession = sqlSessionFactory.openSession();
//通过mapper.xml配置文件的对应sql的Mapper标签中的namespace属性和对应CRUD标签的id属性
//来读取需要执行sql，并进行参数传递
User user = openSession.selectOne("com.yh.test.UserMapper.getUser" ，(Object)param);
```

2、接口编程：（**推荐使用**，支持两种配置方式,并且通过 mapper接口方法来定义sql的映射传递的参数和返回值，代码更加清晰，不需要担心使用namspace.id 字面量的正确性和参数类型的强制转化错误）

```java
SqlSession openSession = sqlSessionFactory.openSession();
UserMapper userMapper = openSession.getMapper(userMapper.class);
userMapper.getUser(id);
```

### 4、mybatis的SqlSessionFactoryBuilder和sqlSeesionFactory对象

SqlSessionFactoryBuilder：用于读取mybatis配置文件，来创建sqlSessionFactory对象。为了保证xml配置文件的释放，其作用域应该为创建sqlSessionFactory对象的方法作用域，即使用后就丢弃

sqlSeesionFactory：每个数据源都需要对应一个sqlSeesionFactory，因此推荐在应用运行期间不要重复创建，保持单例模式，确保用户能访问所有数据库

**mybatis创建sqlSessionFacctory方式：**

1、通过mybatis-config.xml进行创建

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

2、通过java代码手动获取所需配置信息对象，然后创建

```java
//数据源信息
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
//事务管理器
TransactionFactory transactionFactory = new JdbcTransactionFactory();
//通过dataSource、transactionFactory创建myabtis环境对象environment
Environment environment = new Environment("development", transactionFactory, dataSource);
//创建mybatis配置对象Configuration，添加environment、mapper和一些其他配置信息
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
//最后使用SqlSessionFactoryBuilder进行SqlSessionFactory对象创建
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

## 4、mybatis的全局配置文件（mybatis-config.xml）

1、引入dtd xml约束文件，定义全局配置文件可以使用的标签和属性

```xml
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd"> 
```

2、mybatis-config.xml根标签 < configuration > 下的子标签配置：

### properties标签：

mybatis可以使用properties文件来进行外部属性配置

```xml
    <!-- 引入外部properties文件 
    	 resource:引入类路径下的资源
         url:引入网络路径或磁盘路径下的资源
    -->
    <properties resource="dbconfig.properties"></properties>
```

之后xml中，所有属性值都可以通过 ${}标识符来进行perperties文件中的KV引入

### settings标签：

如对mybatis的各项参数进行修改，如是否开启驼峰命名自动映射，默认为false：

```xml
  	<settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
```

### typeAliases标签：

mybatis别名处理器，在mybatis所有的xml文件中，给所有的java类的全类名起一个别名，简化xml文件的编写

```xml
	<typeAliases>
        <!-- 指定一个类的别名 -->
        <typeAlias type="com.yh.User" alias="user"/>
        <!-- 批量指定一个包下所有类的别名,此时为所有类名 
             (但不同子包下，会导致别名重复，此时就需要使用@Alias注解来对该类进行专门指定)
        -->
        <package name="com.yh"/>
    </typeAliases>
```

1、@Alias必须搭配批量包起别名使用

2、别名并不区分大小写

3、mybaits已经提供了java基础数据类型及其包装类的别名，和一些容器类和数据库其他数据类型的别名：

date、decimal、bigdecimal、object、map、hashmap、list、arraylist、conllection、iterato

### typeHandlers标签：

mybaits类型处理器，进行java类型和数据库数据类型的转化，从而实现数据映射

```xml
    <typeHandlers>
         <!-- handler:类型处理器类的全类名
              javaType：该类型处理器指定处理的java对象类型
              jdbcType：该类型处理器指定处理的数据库数据类型
         -->
        <typeHandler handler="" javaType="" jdbcType=""/>
    </typeHandlers>
```



**mybatis默认已经添加了各种数据库中所有的数据类型的转化处理器，实现基本的数据映射；但也可以通过自定义类型处理器，来代替对于类型的mybatis默认类型处理器，实现一些额外的操作**，如将数据库中blob字段数据，转化映射为一个图片BASE64字符串

在mybatis3.4之前版本，对于日期时间的处理，需要手动添加JSR310规范的处理器，才能实现更多样的映射规则（之后，mybatis默认自动添加）

**###JSR310：即java8提出的日期和时间的API**

### plugins标签：

mybatis插件，mybatis允许开发者手动拦截内部映射的关键方法，在其前后进行一定的代码处理，从而更底层的自定义映射规则，主要包括四个对象的执行方法：

1、Excutor（执行器），进行sql的执行

2、ParamterHandler（参数处理器），将参数注入到sql中

3、ResultSetHandler（结果集处理器），对sql执行的结果集进行数据映射

4、StatementHandler（sql语句处理器），在sql执行前，进行额外处理

### environments标签：

mybatis环境配置，配置事务管理器和数据源

```xml
   <!-- mybatis支持多环境配置，通过defult属性对应environment子标签的id
        来切换不同的mybatis环境，实现快速切换 -->
    <environments default="development">
        <environment id="development">
            
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type=""></dataSource>
        </environment>
    </environments>
```

1. 事务管理器：

   mybatis使用别名处理器，默认定义了两种事务管理器工厂：（在org.apache.ibatis.session.Configuration类中定义）

   - JDBC：jdbcTransactionFactory，使用JDBC来实现事务，并使用数据库连接中获取事务作用域

   - MANAGED：ManagedTransactionFactory，没有事务实现，需要依赖于容器（应用服务器上下文）进行管理

   **当使用spring+mybatis时，mybatis环境配置的事务管理器不会生效，因为spring会使用自己的事务管理器来覆盖以上匹配**

2. 数据源：存储数据库连接信息和连接池信息，提供一个抽象的getConnection()，根据不同数据库JDBC驱动来进行具体实现

   mybatis使用别名处理器，默认定义了三种数据源工厂：

   - UNPOOLED：UnpooledDataSourceFactory，没有连接池功能

   - POOLED：PooledDataSourceFactory，具有连接池功能

   - JNDI：JndiDataSourceFactory，实现数据源交给J2EE容器（J2EE应用服务器）进行管理，在容器启动时，通过上下文路径进行获取

   **一般情况下，我们直接使用带有连接池功能的druid数据源，并且使用spring容器对所有mybatis所需要的sql组件（数据源、事务管理器、sqlSessionFactory、sqlSsessionTemplate、mapper）进行管理，并通过依赖注入提供给mybatis使用**

**###JNDI:java命名和目录接口，为开发人员提供了查找和访问各种命名和目录服务的通用、统一的接口，J2EE容器可以通过JNDI来实现服务的对外暴露，从而使得开发者可以通过一个 唯一标识符从 J2EE容器远程获取中JNDI树的服务对象（spring的出现，导致J2EE容器的衰败，使得只用使用web容器，如Tomcat，就可以实现容器管理，并且Tomcat在4.1版本后就支持在容器中是实现数据源配置）**

### databaseIdProvider标签：

用于支持多类型数据库（如mysql、oracle、sqlServer）环境切换下，Dao接口方法可以对应多个id相同的xml标签，然后根据当前接口使用的SqlSessionConnection的数据库环境，来进行不同的xml元数据读取，实现该接口的动态代理

1、配置需要支持的数据库，并设置对于databaseId：

```xml
<databaseIdProvider type="DB_VENDOR">
     <!-- 配置属性，来添加需要支持识别的数据库厂商标识，并起一个ID  -->
     <property name="mySql" value="mysql"/>
     <property name="Oracle" value="oracle"/>    
</databaseIdProvider>
```

mybatis默认对VendorDatabaseIdProvider 进行了别名注册：DB_VENDOR，用于识别数据库厂商标识

2、在xxMapper.xml中，编写相同id的sql标签，并设置不同的数据库厂商标识对应Id（从而让mybatis根据当前数据库环境，进行xm元数据的选择加载）

```xml
	<!-- 定义一个查询sql, -->
	<select id="getUserById" databaseId="mysql" parameterType="int"
		resultType="com.yh.entity">
		select * from student where id =#{id}
	</select>

	<select id="getUserById" databaseId="Oracle" parameterType="int"
		resultType="com.yh.entity">
		select * from student_Oracle where userId =#{id}
	</select>
```

使用场景：有时由于实际因素的影响，导致项目部署会出现多个环境（同一个查询，在不同数据库中使用），因此通过databaseIdProvider标签来配置数据库厂商标识ID，来进行动态加载元数据，从而兼容多环境需求

但一般情况下，基本不使用（大大增加了代码量，因此从本质出发，解决多环境的问题）

### mappers标签：

将sql映射文件注册到mybatis的全局配置文件中，从而实现mybatis的ORM功能

```xml
<!--  mapper标签提供三种注册sql映射文件的方式：
  		  resource：通过类路径引入sql映射文件
		  url：通过url引入本地资源或网络资源
	      class：使用映射器接口的全类名
--!>
<mappers>
	<mapper  resource="org/mybatis/builder/AuthorMapper.xml"/>
	<mapper  url="file:///var/mappers/AuthorMapper.xm"/>
	<mapper  class="org.mybatis.builder.AuthorMapper"/>  

	 <!-- 直接通过package子标签，将所有包下的映射器接口注册  --!>
	<package name="">
</mappers>
```

由于mybatis的映射器有两种配置，因此需要注意：

1. 对于接口注解配置方式，由于没有使用xml文件，因此只能通过mapper-class和package标签方式，进行映射器接口注册（当然通过整合spring，可以通过@Mapper、@MapperScaner注解完成）
2.  对于xml注解配置方式，前两种引入xml文件没有任何问题，但当使用接口全类名注册时，需要保证xml文件和接口文件在同一目录下，并且名字相同

**所有标签是固定顺序的，严格按照上面顺序排序编写**

## 5、mybatis的sql映射文件（xxxMapper.xml）

引入dtd xml约束文件：

```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
```

### 1、简单CRUD标签：

```xml
<select id="getUserById"  parameterType="int" resultType="com.yh.entity">
	select * from student_Oracle where userId =#{id}
</select>	
<insert id=""></insert>
<update id=""></update>
<delete id=""></delete>
```

- CRUD标签基本属性：

  | 属性          | 描述                                                         |
  | ------------- | ------------------------------------------------------------ |
  | id            | sql语句标签的唯一标识符，对应mapper接口中的一个方法（必须）  |
  | parameterType | 指定单个参数的类型（可以省略，mybatis可以通过接口方法参数列表，自行判断） |
  | parameterMap  | 指定多个参数的类型（已经过时，推荐使用单个复制参数传参，或行内参数映射） |
  | databaseId    | 指定数据库厂商id，定义相同CRUD标签ID，根据数据库环境进行映射元数据选择 |
  | flushCache    | 为true时，执行完语句后自动清空本地缓存和二级缓存；select标签默认为false、增删改标签默认为true |
  | timeout       | sql执行等待数据结果集的最大时间；默认unset，由数据库驱动控制 |
  | statementType | sql语句执行方式：statement（正常执行）、prepared（预编译执行）、cllable（存储过程执行） |

对于增删改，执行结果会返回影响数据条数，但不需要提供所映射的javaBean全类名（即增删改标签没有resultType/resultMap属性）；**因为session.insert()方法会自己返回一个int值，只需要使用mapper接口方法返回值接收就可以**

**mapper方法返回值一般使用Integer、Long、Boolean进行接收；当影响条数为0时，boolean值为false，其他为true；也可以使用void不进行接收**

### 2、Insert标签进行自增主键值获取并插入：

使用属性：

| 属性             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| useGeneratedKeys | 适用于insert和update，使用JDBC的statement.getGenreatedKeys（）方法，获取数据库内部生成的主键，并进行隐式字段插入（只适合如mybatis提供自增字段的数据库） |
| keyProperty      | 用于将开启useGeneratedKeys属性获取的自增主键值，根据参数传递规则，放入指定参数对象中；当结果集为多个时，可以使用逗号分隔（如果想要参数中保存主键值，必须指定） |
| keyColumn        | 用于指定开启useGeneratedKeys属性获取的自增主键时，需要获取的主键的列名（当需要的主键不为结果集的第一个时，必须设置，如出现多个主键，可以使用逗号分隔） |

- mysql支持自增主键，其JDBC提供statement.getGenreatedKeys（）方法，来获取当前表主键的自增值。因此mybatis也是利用该方法，在insert标签中，实现自动使用当前表主键的自增值来进行 主键字段的插入参数填充，**所以在编写sql时，不需要对主键字段进行显式的insert：**

```xml
<insert id="insertUser" paramterType=com.yh.User useGeneratedKeys="true" keyColumn="id">
    insert into User(name,sex,age) values (#{name},#{sex},#{age})
</insert>	
```

​		**通过useGeneratedKeys="true"和keyColumn（单主键可以省略），就可以完成指定主键的获取和insert插入；然后通过keyProperty可以将主键值映射到参数对象属性中，从而不必要多一次sql查询，来获取insert后的主键值**

- Oracle不支持自增主键，需要使用序列号和触发器实现；利用selectKey标签通过JDBC查询oracle对于该表序列号的下一个值：

  ```SQL
  SELECT user_seq.nextval from dual
  ```

  selectKey标签属性：

  | 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
  | keyProperty   | selectKey语句结果映射到指定参数对象中（根据参数传递规则）    |
  | keyCoulmn     | 获取selectKey语句结果中需要的主键字段，单主键可以省略；多主键可以通过逗号分隔一一指定 |
  | resultType    | 指定主键值映射的java类型，mybatis可以通过数据库类型进行自行推断 |
  | order         | 有效值为BEFORE、AFTER；BEFORE则语句在insert语句之前执行；AFTER则语句在insert语句之后执行 |
  | statementType | sql语句执行方式：STATEMENT（正常执行）、PREPARED（预编译执行）、CALLABLE（存储过程执行）；默认为PREPARED |
  
  **在insert语句执行前，通过selectKey先查询当前表主键序列化的下一个值，并将结果保存到insert对应参数中，然后再进行insert操作，因此必须显示进行主键字段insert，并指定keyProperty**
  
  ```xml
  <insert id="insertUser" parameterType="com.yh.User">
  	<selectKey keyProperty="id" order="BEFORE" resultType="Integer">
  	    select User_Seq.nextval from dual
  	</selectKey>
  	insert into User(id,name,sex,age) values (#{id},#{name},#{sex},#{age})
  </insert>
  ```

keyProperty和keyCoulmn的区别：

keyProperty只是为了将useGeneratedKeys或selectKey查询的主键值保存到指定参数对象中；

keyCoulmn是指定主键的字段来获取主键值，当主键字段在结果集第一列时，可以省略

### 3、select标签：

select额外属性：

| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| resultType    | 结果集自动映射，指定javaBean全类名                           |
| resultMap     | 结果集手动映射，指定对应resultMap标签id                      |
| useCache      | 使用二级缓存，默认为true                                     |
| fetchSize     | 建议数据库驱动查询返回的结果集行数，默认unset，由数据库驱动决定 |
| resultSetType | 多结果集的返回类型，默认unset，由数据库驱动决定              |
| resultSets    | 用于多结果集查询时，定义每个结果集名称，按顺序逗号分隔       |
| resultOrdered | 在嵌套结果select查询时，嵌套映射的实体类中是否包含主结果集的实体类引用；默认为false；为true时可以减少内存占用 |

### 4、sql标签：

定义可重用的 SQL 代码片段，提供类所有CRUD标签使用，如：

```xml
<sql id="tableName"> from User Where </sql>

<select id="getUserByid">
    select *  <include refid="tableName" /> id=#{id}
</select>
```

sql标签中的sql可以使用${}占位符，然后使用include标签引入sql片段时，使用其子标签property 进行参数值注入(**不能使用#{}占位符)**

```xml
<sql id="tableName"> from ${alia} Where </sql>

<select id="getUserByid">
    select *  
    <include refid="tableName" >
    <property name="alia" value="user">
    </include>    
    id=#{id}
</select>
```

sql标签还可以进行嵌套使用，大大增加其sql复用的灵活性

### 5、resultMap标签：

**参考mybatis的结果集映射**

### 6、cache和cache-ref标签：

**参考mybatis的二级缓存**

## 6、mybatis的参数传递：

### 1、mybatis的sql参数传递规则：

**mybatis对简单类型参数，和复杂对象参数有不同的处理：**

1、简单类型参数：date/time、基本数据类型和其包装类

2、复杂对象参数：jopo、map、TO（数据传输对象，如经常使用 stratTime、endTime、type、Number进行综合查询时，这又不是业务数据模型，此时也可以创建一个实体对象，进行查询数据的参数封装）

- 对于简单数据类型单个参数：mybatis默认不会做任何处理，直接获取方法参数值，传递到sql语句中；当使用@Prama注解时，则使用使用map进行参数封装，并以@param注解值为key

```java
@Select("select * from user where id=#{userId}")
User getUser(String id);  //单个参数不需要方法形参名和占位符参数名一致
```

- 对于多个简单数据类型参数：mybatis会将多个参数使用map进行封装，key值默认使用param1、param2. . .paramN的占位符参数名形式，value值按方法参数列表顺序进行传递**（这样sql可读性不高，不推荐使用）**

```java
@Select("select * from user where id=#{param1} and type=#{param2}")
User getUser(String id,String type);//为了保证参数传递的正确性，要保证形参顺序和sql中需要的参数顺序一一对应
```

因此mybatis提供对mapper接口方法的**形参命名注解@Param**，使得对于多参数封装的map，使用@Param注解值作为key值

```java
@Select("select * from user where id=#{id} and type=#{type}")
User getUser(@Param("id") String id,@Param("type") String type); //此时sql中的占位符参数名和所有形参的@param注解值对应
```

- 对于单个复杂对象参数：

mybatis默认不会进行参数的map封装，在sql中可以直接使用 属性名 的形式进行参数传递；当使用@Prama注解时，则使用使用map进行参数封装，并以@param注解值为key；然后其属性值，可以使用key.属性名获取

```java
@Select("select * from user where id=#{id} and type=#{type}")
User getUser(User user);
```

- 对于多个复杂对象参数：

1、mybatis在参数map封装时，默认会使用param1、param2作为key值，使用key.属性名获取属性值；从而来区分不同复杂对象中属性

```java
@Select("select * from user where id=#{param1.id} and type=#{param1.type} and sex=param2.sex")
User getUser(User user,Map<String,String> map);
```

2、当然我们可以使用@Param注解，将paramN代替为注解值；但此时还是可以使用paramN方式

```java
@Select("select * from user where id=#{user.id} and type=#{user.type} and sex=map.sex")
User getUser(@Param("user")User user,@Param("map")Map<String,String> map);
```

**由于mybati允许@param注解、复杂对象和通用参数名三种形式在同一个方法参数列表中出现，因此我们需要注意：**

1、尽量避免这种情况的发生，使用一个map直接封装所有字段，或者创建一个更全的jopo

2、对于多参数，应对所有参数进行@Param注解，避免paramN形式的传输传递

3、对于单个参数，可以不使用@Param注解；**对于实体对象，使用@Param注解后，就需要添加注解值作为属性名前缀；不使用@param注解，则直接使用其属性名作为参数传递**

4、通用参数名除了使用的param1、param2外，mybatis还提供使用方法参数索引值的方式，arg0、arg1（但一般不使用）

**###对于Collection参数类型或数组，mybatis也会将参数对象封装在map对象中，配合foreach动态标签使用，在进行参数传递时，mybatis会根据数据类型，将其参数名转化为list/array/map；因此不能够对该类对象进行@Param注解；否则mybatis无法完成内部转化**

### 2、mybatis参数map封装过程（以及获取mapper接口实现类过程）：

- 通过sqlSession来获取Mapper接口的实例

  ```java
  SqlSession openSession = sqlSessionFactory.openSession();
  UserMapper mapper = openSession.getMapper(UserMapper.class);
  ```

- 获取当前mapper接口的MapperProxy代理对象

  ```java
  //通过该sqlSession对应的Configuration（全局配置对象）中的mapperRegistry（mapper注册器对象），获取UserMapper接口对应的mapperProxy
    public <T> T getMapper(Class<T> type) {
      return configuration.getMapper(type, this);
    }
  
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
      return mapperRegistry.getMapper(type, sqlSession);
    }
  
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
       //在Mapper被注册时，根据其元数据和sqlSession，创建一个对应的mapperProxyFactory，保存在mapperRegistry中
      final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
      if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
      }
      try {
        return mapperProxyFactory.newInstance(sqlSession);
      } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
      }
    }
  
  
  //通过JDK动态代理，实现MapperProxy作为Mapper接口的动态代理类
    public T newInstance(SqlSession sqlSession) {
      final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
      return newInstance(mapperProxy);
    }
  
    @SuppressWarnings("unchecked")
    protected T newInstance(MapperProxy<T> mapperProxy) {
      return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
    }
  ```

- mapperProxy对象实例化过程

  ```java
  public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, 
  MapperMethod> methodCache) {
      //内部保存了sqlSession、mapper接口的方法列表，和mapper接口的class对象
      this.sqlSession = sqlSession;
      this.mapperInterface = mapperInterface;
      this.methodCache = methodCache;
    }
  ```

- mapperProxy实现Mapper接口方法的动态代理

  ​		mapperProxy实现了InvocationHandler接口，重写了invoke方法，实现对应mapper接口的动态代理；当mapperProxy执行其所代理接口的方法时，自动调用invoke方法：传入代理对象、方法对象、方法参数

  ```java
  //方法调用的动态代理
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      try {
        //由于任何类都继承了Object方法，因此mapperProxy需要排除对object类方法的动态代理
        if (Object.class.equals(method.getDeclaringClass())) {
          return method.invoke(this, args);//直接放行，执行自身方法
        } else if (isDefaultMethod(method)) {
         	//如果为mapper接口的default方法（JDK8特性），则进行JDK定义的default方法代理执行方式（default方法不能直接invoke，需要做额外处理，但并没有添加额外操作）
          return invokeDefaultMethod(proxy, method, args);
        }
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
      //如果为mapper接口规定的抽象方法，则根据mapper接口的方法列表数据，创建一个MapperMethod对象，进行方法动态代理
      final MapperMethod mapperMethod = cachedMapperMethod(method);
      return mapperMethod.execute(sqlSession, args);
    }
  ```

- mapperMethod对象的方法代理执行，进行参数map封装

  pperMethod通过对   当前方法对应的映射器元数据和方法信息，使用sqlSession对应的方法进行sql执行，并将方法参数进行map封装，将方法参数和sql参数一一对应，用于sql传参
  
  ```java
   public Object execute(SqlSession sqlSession, Object[] args) {
      Object result;
      switch (command.getType()) {//判断映射器元数据中sql类型（CRUD）
        case INSERT: {
            //将方法参数进行map封装，用于sql传参
          Object param = method.convertArgsToSqlCommandParam(args);
          result = rowCountResult(sqlSession.insert(command.getName(), param));
          break;
        }
         //######其他代码省略
      return result;
    } 
   
    //获取所有参数，对应sql中参数名
    public ParamNameResolver(Configuration config, Method method) {
      final Class<?>[] paramTypes = method.getParameterTypes();
      final Annotation[][] paramAnnotations = method.getParameterAnnotations();
      final SortedMap<Integer, String> map = new TreeMap<>();
      int paramCount = paramAnnotations.length;
       
      //遍历参数
      for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
        if (isSpecialParameter(paramTypes[paramIndex])) {
          // 跳过特殊参数（RowBounds、ResultHandler 行数限制、结果处理器）
          continue;
        }
        String name = null;
        //获取当前参数的注解，将注解值作为sql占位符中参数名
        for (Annotation annotation : paramAnnotations[paramIndex]) {
          if (annotation instanceof Param) {
            hasParamAnnotation = true;
            name = ((Param) annotation).value();
            break;
          }
        }
        if (name == null) {
          //当开启userActualParamName全局设置时，没有使用@Param，则使用该方法参数名作为sql占位符中的参数名
          // @Param was not specified.
          if (config.isUseActualParamName()) {
            name = getActualParamName(method, paramIndex);
          }
          if (name == null) {
          //直接使用方法参数索引，作为sql占位符中的参数名；从0 开始
            name = String.valueOf(map.size());
          }
        }
          //key为方法参数索引，value为sql占位符中的参数名
        map.put(paramIndex, name);
      }
      names = Collections.unmodifiableSortedMap(map);
    }
       
      
       
    public Object getNamedParams(Object[] args) {
      //names为所有参数集合，key为方法参数索引，value为sql占位符中的参数名，如
      [0:id,2:name,3:3]
      final int paramCount = names.size();
      if (args == null || paramCount == 0) {//判断是否参数
        return null;
      } else if (!hasParamAnnotation && paramCount == 1) {//一个参数且没有@Param注解时，则不需要map封装，直接取方法参数中的第一个参数
        return args[names.firstKey()];
      } else {//多个参数或者使用了@Param注解时，则进行map封装
        final Map<String, Object> param = new ParamMap<>();
        int i = 0;
        for (Map.Entry<Integer, String> entry : names.entrySet()) {
          //首先将sql占位符中的参数名作为key、方法参数值（以方法参数列表索引进行查询）作为value
          param.put(entry.getValue(), args[entry.getKey()]);
          //获取当前方法参数的通用参数名称（param1、param2 ,...），从1开始
          final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
          //在map中额外存放一份通用参数名称对应方法参数值（从而在使用注解、userActualParamName和方法参数索引值时，同样可以使用通用参数名称进行参数传递）          
  * <li>aMethod(@Param("M") int a, @Param("N") int b) -&gt; {{0, "M"}, {1, "N"}}</li>
  * <li>aMethod(int a, int b) -&gt; {{0, "0"}, {1, "1"}}</li>
  * <li>aMethod(int a, RowBounds rb, int b) -&gt; {{0, "0"}, {2, "1"}}</li>
  * <li>aMethod(int a, RowBounds rb, int b) -&gt; {{0, "param1"}, {2, "param2"}}</li>
  //开启 userActualParamName   
  * <li>aMethod(int a, RowBounds rb, int b) -&gt; {{0, "a"}, {2, "b"}}</li>    
          if (!names.containsValue(genericParamName)) {
            param.put(genericParamName, args[entry.getKey()]);
          }
          i++;
        }
        return param;
      }
    } 
  ```

**mybatis后来版本将使用方法参数索引值的方式（0，1），修改为arg0、arg1（目前不知道在哪做了额外处理）**

**此时，mybatis已经将复杂类型参数和简单类型参数进行了sql传参名的绑定：**

- 简单类型参数，直接通过参数名进行获取；
- 而复杂类型参数则通过 参数名.属性名获取；并且对于**只有单个参数且没有使用@Param**时，则直接使用其属性名

### 3、mybatis参数传递的两种方式：

- #{}传参：在JDBC预编译时，将内容作为字符串，加上引号替代？占位符，传入sql语句中；

- ${}传参：传入内容会直接拼接到sql语句中，不会加上引号


使用场景：

#{}，可以有效防止sql注入，因此正常情况下的参数传递都使用该方式

${}，有sql注入的风险，对应原生jdbc不支持占位符（即预编译）的地方，则选择使用该方式进行传参，如表名、字段名、关键字

### 4、mybatis行内参数映射：

在使用#{}进行传参时，可以设置一些属性来使用sql预编译的高级功能（${}没有）：

- jdbctype：（基本只额外使用该属性）

JDK的java.sql.JDBCType类对所有数据库数据类型提供了一个枚举值，用于表示数据库数据类型；myabtis的类型处理器就使用jdbctype来确定所操作数据库的数据类型,一般情况下，mybatis会根据数据库数据类型，来自行选择默认值，但有特殊情况需要注意：

​	mybatis默认会将null映射为原生jdbc类型OTHER；而oracle的JDBC驱动不支持该jdbc类型，因此会报错；mysql支持该jdbc类型，不会报错

**因此在使用oracle环境进行null字段的参数传递时，需要修改其映射的jdbcType，防止报错，如：#{name，jdbcType=NULL}，将jdbcType修改为NULL，则oracle就能识别进行正常映射**

当然也可以修改mybatis的全局配置，将null的映射的jdbcType为NULL：

```xml
<setting name="jdbcTypeForNull" value="NULL"/>
```

- javaType：

  一般不使用，mybatis可以自动根据mapper方法中的参数类型，来自动设置；除非传入的参数对象为HashMap<String,Object>,此时就需要显式指定每个value的类型，确保mybatis能使用正确的类型处理器

- typeHandle

  用于覆盖全局typeHandle，mybatis全局默认对所有数据库类型和java数据类型的数据映射，提供了类型处理器，但我们可以通过typeHandle来重新指定（比如自定义的typeHandle）

## 7、mybatis的结果集映射：

mybatis结果集映射，有两种选择：使用java.util.HashMap进行接收，key为结果集字段名、value为字段值；使用javaBean进行接收，结果集字段和javaBean属性一一对应

### 7.1、resultType自动映射：

- mybatis提供resultType属性来进行实体类属性的自动映射，来接收数据库结果集中的字段数据；通过**autoMappingBehavior** setting标签属性进行全局设置，默认PARTIAL（开启局部自动映射），当设置为NONE时，则会关闭自动映射

- **autoMappingBehavior**属性值中，PARTIAL（局部自动映射）和FULL（全局自动映射）的区别：

  ​		PARTIAL：当所映射的实体类中，出现其他的实体类属性，并需要和结果集字段对应，因此就会出现嵌套映射；**在PARTIAL下，mybatis就不会对带有嵌套映射的结果集进行自动映射**

  ​		FULL：当发生嵌套映射时，会对所有层实体类进行自动映射

- myabtis提供 setting标签属性**mapUnderscoreToCamelCase**来全局设置，在自动映射开启的前提下，开启自动驼峰命名规则映射；默认为false

- **resultType自动映射优缺点：**

  无法自定义映射，因此灵活性差，并且依赖于**autoMappingBehavior** 属性值的开启

  ```xml
  <select id="getUserById" resultType="com.yh.entity.User" >
     select * from test.user where id =#{id}
  </select>
  ```

  **resultType本质上，还是使用的resultMap规则进行映射，只是使用默认隐式的方式，隐藏了默认配置的resultMap标签，让Mapper.xml更加简洁**

### 7.2、resultMap自定义映射：

- mybaits提供resultMap标签和resultMap属性，来自定义映射，并且可以建立在自动映射的基础上

  ```xml
  	<resultMap id="user" type="com.yh.entity.User" autoMapping="true">
          <result column="id" property="id"/>
  	</resultMap>
  
  	<select id="getUserById" resultMap="user">
  		select * from test.user where id =#{id}
  	</select>
  ```

  **resultMap标签属性：**

  | 属性        | 描述                                                         |
  | ----------- | ------------------------------------------------------------ |
  | id          | 唯一标识符，用于给CRUD标签中的resultMap属性赋值，从而指定该resultMap进行结果集映射（必须） |
  | type        | 所映射javabean的全类名（必须）                               |
  | autoMapping | 开启局部自动映射，默认为unset，在全局配置autoMappingBehavior=PARTIAL时，javaBean存在需要嵌套映射时，无法进行自动映射；如果设置true/false，则会覆盖全局属性autoMappingBehavior，可以进行自动映射（非必须） |

**resultMap提供了一些子标签，来实现更高级的映射规则：**

#### 7.2.1、constructor子标签：

​	用于将结果集字段数据，映射到指定java对象的构造方法参数中

提供两个子标签进行手动映射（无法自动映射）：

- idArg：映射到java对象构造方法参数中，并且将字段值作为该对象的唯一标识符，**实现嵌套映射和缓存时的底层优化**：
- arg：正常的构造方法参数映射

**对于多参构造方法，通过子标签的顺序来一一匹配（3.4.3版本之后，可以使用@Param注解到mapper对应实体类的构造方法参数(或开启`useActualParamName`全局配置)，然后使用name属性来匹配）；并且该标签中使用映射的字段，不会再自动映射到java对象属性中，需要手动添加映射**

```java
	public User(@Param("id") String id) {
		System.out.println(id);
	}
```

```xml
	<resultMap id="user" type="com.yh.entity.User"
		autoMapping="true">
		<constructor>
			<arg column="id"  name="id"/>
		</constructor>
		<id column="id" property="id" />
		<result column="name" property="name"/>
	</resultMap>
```

**idArg和arg标签属性：**

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| column      | 结果集中所映射的字段名或字段别名（必须）                     |
| javaType    | 映射的java数据类型；根据映射的形参类型时，mybatis可以自行判断；（非必须） |
| jdbcType    | mybati会根据数据库字段类型进行设置，一般用于对null值字段进行指定（非必须） |
| typeHandler | 类型转化器，mybatis提供一系列默认值（非必须）                |
| name        | 匹配构造方法形参中的@Param注解值（非必须）                   |

#### 7.2.2、id和result子标签：

​	它们都是用于自定义结果集的字段映射，但是id标签可以将字段值当前映射生成对象的唯一标识符（**实现嵌套映射和缓存时的底层优化**）；

- **id、idArg标签的底层优化过程：**

  ​		mybatis进行正常的结果集映射时，会将结果集中字段完全相同的数据合并为一条数据，映射为一个对象；因此当不指定id时，mybatis就需要对比所有字段，合并字段完成相同的数据（对比执行次数为M（结果集条数）*N（结果集字段数）；而指定id后，mybatis只需要对比id字段，来合并数据（对比执行次数为M（结果集条数）））。因此id的指定，在mybatis底层映射中，有很大的优化作用

- **id和result标签属性：**

| 属性        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| property    | 结果集映射到javaBean中的属性名（必须）                       |
| column      | 结果集中所映射的字段名或字段别名（必须）                     |
| javaType    | 映射的java数据类型；当需要映射到javabean时，mybatis可以通过其属性类型来自行判断；当使用Map<String,Object>接收时，需要指定，便于Object进行数据转化（非必须） |
| jdbcType    | mybati会根据数据库字段类型进行设置，一般用于对null值字段进行指定（非必须） |
| typeHandler | 类型转化器，mybatis提供一系列默认值（非必须）                |

#### 7.2.3、association（关联）子标签：

​	**实现两个表数据的一对一关联**

​	将连接查询的结果集映射到实体类对象属性中（**嵌套结果映射**）；也可以单独执行主表sql查询后，使用关联字段再执行副表sql查询，来进行关联映射（**嵌套select查询**）

- 关联的嵌套结果映射

  在开启局部自动映射的情况下（autoMappingBehavior=PARTIAL），resultMap标签中存在嵌套映射，因此并不会进行自动映射，需要对所有字段进行手动映射，或者使用 autoMapping="true"开启resultMap标签中的局部自动映射

  对于association标签，不会参考全局autoMappingBehavior属性，因此必须手动映射或使用autoMapping="true"开启当前标签的局部自动映射

```xml
	<resultMap id="userA" type="com.yh.entity.User">
		<id column="id" property="id" />
		<result column="name" property="name" />
        <result column="password" property="password">
           
        <!-- 必须指定所关联属性名、属性对象全类名 -->
		<association  property="gread" javaType="com.yh.entity.Gread">
		     <id property="id"  column="gread_id"/>
		     <result property="math" column="math"/>
		     <result property="english" column="english"/>
		</association>
	</resultMap>

    <select id="listUserById" resultMap="userA">
        select u.*,g.id as gread_id,math,english from user u left join  gread g on g.user_id=u.id where u.id=#{id}
    </select>
```

除了直接在ssociation中，规定映射规则外，还可以使用resultMap属性来复用其他的resultMap标签定义的映射规则,这样做也有如下优点：

1、减少重复映射规则的定义，实现resultMap标签的复用

2、resultMap提供对结果集的缓存

```xml
	<resultMap id="gread" type="com.yh.entity.Gread">
		<id property="id" column="gread_id" />
		<result property="math" column="math" />
		<result property="english" column="english" />
	</resultMap>
    
	<resultMap id="userA" type="com.yh.entity.User">
		<id column="id" property="id" />
		<result column="name" property="name" />
        <result column="password" property="password">
           
   <!-- 指定所关联属性名、由于resultMap标签中有指定映射的对象全类名，因此不需要声明javaType属性-->
		<association  property="gread" resultMap="gread" />
	</resultMap>
```

**mybatis内部还提供联级属性，来简化association标签的使用，但缺点是无法复用resultMap：**

```xml
	<resultMap id="userA" type="com.yh.entity.User">
		<id column="id" property="id" />
		<result column="name" property="name" />
        <result column="password" property="password" />
        <!-- 必须指定所关联属性名、属性对象全类名 -->
		<id property="gread.id"  column="gread_id"/>
		<result property="gread.math" column="math"/>
		<result property="gread.english" column="english"/>	
	</resultMap>
```

- 关联的嵌套select查询（**不推荐使用，这样会执行N+1条sql，大大增加数据库查询负担**）

```xml
	<resultMap id="user" type="com.yh.entity.User" autoMapping="true">
	   <id column="id" property="id" />
       <!-- 指定所关联属性名,由于select标签中有指定映射的对象全类名，因此不需要声明javaType属性 -->
	   <association property="gread"  column="id"  select="getGreadById">
	   </association>
	</resultMap>

	 <!-- ResultMap所关联的查询 -->
	<select id="getGreadById" resultType="com.yh.entity.Gread">
		select * from test.gread where user_id =#{id}
	</select>

	<select id="getUserById" resultMap="user">
		select * from test.user where id =#{id}
	</select>	
```

**mybatis对其进行了进一步优化，可以设置懒加载**：

1、全局属性设置lazyLoadingEnabled，默认为false，不开启懒加载

2、`fetchType` 属性，默认unset使用全局设置；有效值lazy可以开启懒加载

3、使用懒加载的前提全局属性aggressiveLazyLoading，默认为false，true时，会自动加载所有javaBean属性，因此使用懒加载时，需要设置为false

- 关联的多结果集：（**不推荐使用，需要使用存储过程保存到数据库，不被程序代码所控制**）

  在3.2.3版本开始，mybatis提供另一种解决N+1问题的方式，但需要使用的存储过程：

  通过存储过程查询该每个id对应的所有user数据和gread数据，分别保存到各自的resultMap中，通过resultSets属性进行匹配；然后在使用关联映射时，通过resultSet属性来将resultMap中的结果集数据引入，进行重新映射；

  优点：通过存储过程，有效避免了JDBC进行多条sql的执行；降低数据库查询负担

```xml
	<resultMap id="gread" type="com.yh.entity.Gread">
		<id property="id" column="gid" />
		<result property="math" column="math" />
		<result property="english" column="english" />
	</resultMap>

	<resultMap id="UserB" type="com.yh.entity.User">
        <id column="id" property="id" />
        <result column="name" property="name" />
    	<result column="password" property="password">
    <!--使用Gread结果集来实现gread对象属性的关联映射 --> 
  	 <association property="gread"  resultSet="Gread"  column="id" foreignColumn="id" />
</resultMap>

	<select id="selectUser" resultSets="User,Gread" resultMap="UserB" statementType="CALLABLE">
  		<!--getBlogsAndAuthors是保存在数据库中的存储过程，用于分别查询用户信息和成绩信息 -->
  		{call getBlogsAndAuthors(#{id,jdbcType=INTEGER,mode=IN})}
	</select>
```

- association标签属性（不包含多结果集使用的属性）：

| 属性          | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| property      | 结果集映射到javaBean中的属性名（必须）                       |
| javaType      | 映射的java数据类型;当手动映射时，必须指定；当使用resultMap、select映射时，不需要指定 |
| jdbcType      | mybati会根据数据库字段类型进行设置，一般用于对null值字段进行指定（非必须） |
| typeHandler   | 类型转化器，mybatis提供一系列默认值（非必须）                |
| column        | 嵌套select查询时，用于指定关联传入select查询的参数字段;**单个参数时，不需要字段名和对应select标签中的占位符参数名对应；多个参数时，则使用column="{prop1=col1,prop2=col2}"格式接收字段，并匹配elect标签中的占位符参数名** |
| select        | 嵌套select查询时，用于引入关联的select标签ID                 |
| fetchType     | 嵌套select查询时，懒加载（默认unset，有效值为lazy和eager，设置后会忽略全局配置参数lazyLoadingEnabled） |
| resultMap     | 嵌套结果映射时，复用已定义的resultMap映射规则                |
| columPrefix   | 嵌套结果映射时，对复用resultMap中的所有column添加前缀        |
| notNullColumn | 嵌套结果映射时，默认所有字段都为null时，则不会创建子对象（即不进行映射）；该属性可以指定哪些字段为不为null时，才创建子对象（通过逗号分隔，指定多个字段） |

fetchType对嵌套select查询的影响：

#### 7.2.4、collection（集合）子标签：

​		**实现一对多的关联**

​		和association使用非常相似，但collection是用于嵌套映射到集合对象属性中，指定java集合类型同时，指定集合的泛型；同样分为：**嵌套select查询**和**嵌套结果映射**（多结果集省略，一般不会使用）

- 嵌套select查询

```xml
		<!-- 由于collection必须使用java集合映射，因此javaType可以省略；select查询中指定了所映射的javaBean，因此ofType也可以省略 -->
		<collection property="gread" column="id" javaType="ArrayList"
			select="getGreadById" ofType="com.yh.entity.Gread">
		</collection>
```

```java
private List<Gread> gread;
```

- 嵌套结果映射

```xml
	<resultMap id="gread" type="com.yh.entity.Gread">
		<id property="id" column="gread_id" />
		<result property="math" column="math" />
		<result property="english" column="english" />
	</resultMap>

	<resultMap id="userC" type="com.yh.entity.User"
		autoMapping="true">
		<id column="id" property="id" />
		<collection property="gread"  resultMap="gread" />
	</resultMap>
	
	<!-- javaType、ofType可以省略 -->
	<select id="listUserById" resultMap="userC">
		select u.*,g.id as
		gread_id,math,english from user u left join gread g on
		g.user_id=u.id where u.id=#{id}
	</select>
```

**ofType：是collection标签和association唯一不同的属性，用于指定集合的泛型**

#### 7.2.5、discriminator（鉴别器）子标签：

​		类似于java语言中的switch语句；根据结果集中的某个字段值，来选择结果集中需要映射的部分字段

```xml
<resultMap id="vehicleResult" type="Vehicle">
  <id property="id" column="id" />
  <result property="vin" column="vin"/>
  <result property="year" column="year"/>
  <result property="make" column="make"/>
  <result property="model" column="model"/>
  <result property="color" column="color"/>
  <discriminator javaType="int" column="vehicle_type">
     <!-- 根据vehicle_type字段值，选择部分字段的resultMap映射规则 --> 
    <case value="1" resultMap="carResult"/>   
    <case value="2" resultMap="truckResult"/>
    <case value="3" resultMap="vanResult"/>
    <case value="4" resultMap="suvResult"/>
  </discriminator>
</resultMap>
```

discriminator标签属性包含：column（必须）、javaType（必须，用于匹配case子标签的value属性值）、jdbcType（非必须）、typeHandler（非必须）

case子标签属性包含：value（必须）、resultMap/resultType

### 7.3、mybatis结果集映射常用的javaBean：

1. javaBean映射，如pojo、String和基础数据类型包装类，对一条数据进行映射

2. List< javaBean> 映射，对多条数据进行映射

3. Map<key,javaBean>映射，对多条数据进行映射，同时以每组数据的一个字段作为key（可以用于查询、遍历和去重）

   **mybatis提供@mapKey注解（在mapper接口方法上使用），可以将结果集映射的javaBean；通过某个字段作为key，来生成一个map;整个映射配置和List< javaBean> 一致**

   ```java
   	@MapKey("id")
   	@Select("select * from user where id =#{id}")
   	Map<Integer,User> liseUserById(String id);
   ```

4. Map<String,Object>映射，对一条数据进行映射，使用map保存所有字段

5. List<Map<String,Object>>映射，对多条数据进行映射，map保存所有字段

## 8、mybatis动态sql：

- mybatis通过OGNL表达式来实现，在xml中的逻辑处理，动态拼接sql

- OGNL表达式会自动进行字符串和数字的转化，因此”0“==0成立

- 在xml标签中，出现需要使用特殊字符时（如&、|），则需要使用转义字符替换

### 1、if标签：

```xml
<select>
	select * from User where  id=#{id}
	<if test='name!=null'>
       and name=#{name} 
    </if>
</select>
```

if标签的test属性，就是使用了OGNL表达式，来获取一个boolean值

**只能进行条件判断，没有else语句**

### 2、chooes、when、otherwise标签：

通过这三个标签的组合使用，可以实现java中的if-else、switch语句

```xml
	<select id="getUserByName" resultType="com.yh.entity.User">
		select * from user where id=#{id}
		<choose>
			<when test='name!=null'>
				and name=#{name}
			</when>
			<otherwise>
				and name='yh'
			</otherwise>
		</choose>
	</select>
```

### 3、where、set、trim标签：

当我们编写动态sql时，会遇到如下情况：

```xml
<select>
	select * from User where 
    <if test='id!=null'>
    	id=#{id}
    </if>
	<if test='name!=null'>
       	and name=#{name} 
    </if>
</select>
```

当id、name都为null时，sql语句为 select * from User  where

当id为null时，sql语句为 select * from User  where and name='yh'

**因此动态sql会由于条件的不稳定，导致拼接sql时，出现语法异常，where、and的多余**

mybatis就在动态sql中提供了三个标签，来解决这类问题：

- where标签：

  ```xml
  <select>
  	select * from User 
      <where>
      	<if test='id!=null'>
      		id=#{id}
      	</if>
  		<if test='name!=null'>
         		and name=#{name} 
      	</if>
      </where>
  </select>
  ```

  **where标签作用：**

  ​		只有当子标签返回任何sql拼接的内容时，才会添加where关键字；并且当where子句开头有and、or关键字时，会自动删除

- set标签：

  ```xml
  	<update id="insertUserA">
  		update User
  		<set>
  			<if test="name != null">name=#{name},</if>
  			<if test="password != null">password=#{password}</if>
  		</set>
  		where id=#{id}
  	</update>
  ```

  **set标签作用：**

  ​		实现表所有字段的动态update，有效减少同一个表的update标签个数；set会自动在子句首行添加SET关键字，并且删除set标签下内容中的额外逗号

  **注意：set标签中必须保证有至少一个set子句，否则会导致sql语句错误；可以在代码中判断set的条件是否都为null，成立则不进行mapper方法的执行**

- trim标签：

  trim标签可高度自定义动态sql，提供如下属性：

  | 属性            | 描述         |
  | --------------- | ------------ |
  | prefix          | 添加前缀     |
  | prefixOverrides | 删除指定前缀 |
  | suffix          | 添加后缀     |
  | suffixOverrides | 删除指定后缀 |

  通过trim标签，可以实现where、set标签的功能，但是对于sql的语法格式更加严格：

  - trim等价where标签：**但要保证一定需要有where子句**

    ```xml
    <trim prefix="WHERE" prefixOverrides="AND |OR ">
      ...
    </trim>
    ```

  - trim等价set标签：**要保证一定要有set子句**

    ```xml
    <trim prefix="SET" suffixOverrides=",">
      ...
    </trim>
    ```

  在实际工作中，trim一般用于insert语句的动态sql编写：

  ```xml
  <insert >
  insert into user
      <trim prefix="("  suffix=")" suffixOverrides=",">
        <if test="name != null">
          name,
        </if>
        <if test="password != null">
          password,
        </if>
      </trim>
      
     	<trim prefix="values(" suffix=")" suffixOverrides=",">
          <if test="name != null">
          	#{name},
        	</if>
        	<if test="password != null">
          	#{password},
       	</if>
      </trim>
  </insert>
  ```

### 4、foreach标签：

​		foreach标签一般用于in 语句和批量insert中，对集合进行遍历，来获取集合中的所有值

- in语句

```xml
<select>
  SELECT * FROM USER  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

- 批量insert：

  不同数据库，支持的批量insert语句有所不同：

  - mysql：

    ```xml
    <insert >
    insert into user(name,password) values
     	  <foreach item="item" index="index" collection="list" separator=",">
            (#{item.name},#{item.password})
     	 </foreach>
    </insert>
    ```

  - Oracle：

    ```xml
    <insert >
    insert all
     	  <foreach item="item" index="index" collection="list">
           into user(name,password) vlaues(#{item.name},#{item.password})
     	 </foreach>
    select * from dual
    </insert>
    ```

​	foreach属性有：

| 属性       | 描述                                           |
| ---------- | ---------------------------------------------- |
| collection | 有效值为list、array、map                       |
| item       | 遍历时元素的变量名                             |
| index      | 遍历时索引的变量名（当不使用索引时，可以省略） |
| open       | 前缀                                           |
| close      | 后缀                                           |
| separator  | 遍历内容之间的分隔符                           |

##在进行item、index变量命名时，不要与mapper方法传参名重复

当遍历对象为集合或数组时，index为当前遍历的索引值；当遍历对象为map时，index为Key，item为value

### 5、script标签：

​		mybatis在接口注解方式配置中，也支持动态sql的编写，但需要使用script标签作为父元素

```java
    @Update({"<script>",
      "update User",
      "  <set>",
      "    <if test='name != null'>name=#{name},</if>",
      "    <if test='password != null'>password=#{password},</if>",
      "  </set>",
      "where id=#{id}",
      "</script>"})
    void updateAuthorValues(User user);
```

- 注解方式的符号问题：

  **由于注解编写时，是以字符串、字符串数组的形式进行表示，因此在使用标签属性时，推荐使用单引号，否则就需要使用转义字符来代替双引号的编写（xml中标签属性是支持单引号和双引号，但一般情况下推荐使用双引号，除非需要在属性中，进行字符串的表达，此时就外面使用双引号，里面字符串使用单引号；而对于注解方式，就只能外面使用单引号，里面使用双引号的转义字符）**

### 6、其他标签和操作：

- bind标签：用于在OGNL表达式外创建变量，并且可以使用执行sql的传递参数，使用场景：对于参数进行进一步处理，然后进行预编译参数传递；从而实现精确匹配和模糊匹配的动态查询

  ```xml
  	<select id="selectUserlike" resultType="com.yh.entity.User">
  	   <bind name="likeName" value="name+'%'"/>
  	   select * from user where 
  	   <if test="name!='yh'">
  	   name like #{likeName}
  	   </if>
  	   <if test="name=='yh'">
         name=#{name}
         </if>
  	</select>
  ```

- mybatis内置参数: 
  - _databaseId：代表当前数据库环境，即数据库厂商id；
  - _parameter：单个参数传递时，直接代表该参数；多个参数传递时，则代码封装了多个参数的map

CRUD标签的lang属性：通过lang属性，可以动态指定当前标签下，编写动态sql所用的脚本语言，默认为OGNL表达式

## 9、mybatis缓存机制：

mybatis提供两级缓存：

- 一级缓存：本地缓存，属于sqlSesson级别缓存，与数据库同一次会话中查询的数据，会放在本地缓存中，如果需要获取相同数据，则直接从缓存中获取；一级缓存是mybatis默认开启的，不能够手动关闭

  实际场景：使用一个sqlsession来调用相同mapper接口的select查询方法时，默认情况下只会执行一次sql

  **默认情况：即一级缓存不失效的情况下**

  - 一级缓存的失效情况：

    1. sqlsession执行增删改操作（即使当前操作对之前查询的数据没有影响，如调用操作其他表的mapper方法），此时sqlsession会自动清除一级缓存

    2. sqlsession执行commit操作时，也会自动清除一级缓存

    3. sqlsession手动清除一级缓存

       ```java
       openSession.clearCache();
       ```

    4. sqlsession生命周期结束（即sqlsession关闭），一级缓存随之消失

- 二级缓存：全局缓存，属于namespace级别缓存，每个一个mapper接口，对应一个二级缓存；

  实际场景：两个sqlsession执行相同查询，由于一级缓存为sqlSession级别，因此不能够共享；此时就需要使用二级缓存

  - 二级缓存机制：

    当会话关闭时，mybatis会将一级缓存中的数据保存到二级缓存中，当新的会话进行相同查询时，可以使用二级缓存；

    **注意：当sqlsession关闭或commit时，一级缓存失效，才会将数据转移到二级缓存中；当二级缓存和一级缓存同时存在，优先使用二级缓存（一个sqlsession查询后关闭，生成该查询的二级缓存，另一个sqlsession在二级缓存未生成前，进行相同查询生成一级缓存；此时就会存在二级缓存和一级缓存共存的情况）**

  - 二级缓存失效：

    当任意一个sqlsession调用该mapper接口的增删改方法时，会自动清空二级缓存

  - 二级缓存配置：

    ```xml
    <setting name="cacheEnabled" value="true" /> 默认true，开启二级缓存
    ```

    - 使用cache、cache-ref标签，进行查询sql的二级缓存配置

      cache标签属性：

      | 属性          | 描述                                        |
      | ------------- | ------------------------------------------- |
      | eviction      | 缓存清除算法，默认LRU（最近最少使用的缓存） |
      | flushInterval | 缓存刷新间隔；默认不定时刷新                |
      | size          | 缓存数据对象引用个数，默认1024个            |
      | readOnly      | 缓存数据是否只读，默认为false（非只读）     |
      | type          | mybatis二级缓存接口实现类的全类名，有默认值 |

      - eviction：提供四种缓存回收策略

        LRU：最近最少使用：优先删除最长时间不被使用的缓存对象

        FIFO：先进先出：按照缓存创建顺序删除，优先删除存活时间最早的

        SOFT：软引用：基于垃圾回收器状态和软引用规则移除缓存对象

        WEAK：弱引用：基于垃圾回收器状态和弱引用规则移除缓存对象

      - readOnly：

        true，只读；表示只读缓存会使用一个实例，让所有调用者共享，认为调用者不会修改实例，这样性能更快，但存在被修改导致所有调用者获取错误数据的安全风险

        false，非只读：即可读写，可读写缓存会使用JDK序列化的方式，来进行缓存对象数据的拷贝；因此需要所有实体类对象实现Serializable序列化接口，并且性能会慢一点，但是安全

      - type:

        用于自定义缓存，通过实现org.apache.ibatis.cache.Cache 接口，来自定义二级缓存的实现

  - 二级缓存注解：

    xml的Cache标签和mapper接口的@CacheNamespace缓存注解，它们都可以开启二级缓存；但是它们只作用于自己（**即二级缓存只会被当前方式定义的查询sql使用**），并且不能够同时开启；解决方式：

    使用xml的Cache标签的同时，可以通过@CacheNamespaceRef来引用xml中的缓存，便于使用注解定义的查询sql使用当前mapper的二级缓存

- mybatis二级缓存整合第三方缓存框架

  mybatis提供一个统一的缓存接口org.apache.ibatis.cache.Cache ，只要重写该接口的实现类，就可以自定义mybatis的二级缓存机制

  mybatis有支持许多第三方的缓存框架如：ehcache-cache、redis-cache。。。

- mybatis二级缓存的缺点和使用注意事项

  缺点：由于二级缓存的作用域在每个namespace中，因此当mapper接口方法中，出现了其他表的RUD操作时，就会导致该表对于的mapper接口的二级缓存出现脏数据，影响数据的一致性

  使用注意事项：

  1、保证每一个mapper接口的二级缓存不会相互影响（mapper接口方法不要出现对其他表的RUD操作）

  2、对于有多表CRUD操作的mapper接口，不要使用二级缓存；或者通过CacheNamespaceRef标签，来实现它们使用同一个全局二级缓存

## 10、sqlSession对象的深入理解：

sqlSession:用于获取数据库连接，执行sql，其线程不安全，因此要保证每个线程都有自己的sqlSession实例。因此推荐其作用域在请求作用域或方法作用域中，对于一个http请求，进行数据库操作后，直接关闭sqlSession：

```java
//使用JDK7的try-with-resources
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

**sqlSession特征（构造方法的属性设置）：**

- 事务处理：对于大多数JDBC驱动，默认是事务手动提交，事务隔离级别由数据库驱动决定

- 数据库连接：默认mybatis通过数据源获取数据库Connection对象（可以由带有线程池的数据源进行管理）

- 语句执行：选择是否支持 复用预处理语句和批量更新处理；默认为不复用，也不会批量更新

  sqlSessionFactory提供如下方法配置sqlSession：

   ```java
  SqlSession openSession()
  SqlSession openSession(boolean autoCommit)
  SqlSession openSession(Connection connection)
  SqlSession openSession(TransactionIsolationLevel level)
  SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level)
  SqlSession openSession(ExecutorType execType)
  SqlSession openSession(ExecutorType execType, boolean autoCommit)
  SqlSession openSession(ExecutorType execType, Connection connection)
   ```

  ExecutorType有三个枚举类型，对应mybatis底层三种不同类型的执行器：

  - `ExecutorType.SIMPLE`：该类型的执行器没有特别的行为。它为每个语句的执行创建一个新的预处理语句。
  - `ExecutorType.REUSE`：该类型的执行器会复用预处理语句。
  - `ExecutorType.BATCH`：该类型的执行器会批量执行所有更新语句

**sqlSession方法：**

- 语句执行方法

  statement：提供要执行的sql语句，如果为PreparedStatement实现类，还可以实现sql预编译

  parameter：提供封装好的所有参数

  ```java
  <T> T selectOne(String statement, Object parameter) //查询单条数据
  <E> List<E> selectList(String statement, Object parameter)//查询多条数据
  <T> Cursor<T> selectCursor(String statement, Object parameter)//通过游标使用迭代器进行懒加载
  <K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)//使用mapkey封装查询的多条数据
  int insert(String statement, Object parameter)//增
  int update(String statement, Object parameter)//改
  int delete(String statement, Object parameter)//删
  ```

  - 还提供另外几个额外参数的select方法

    RowBounds：用于现在结果集返回的行数

    ResultHandler：用于自定义结果映射

  ```java
  <E> List<E> selectList (String statement, Object parameter, RowBounds rowBounds)
  <T> Cursor<T> selectCursor(String statement, Object parameter, RowBounds rowBounds)
  <K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowbounds)
  void select (String statement, Object parameter, ResultHandler<T> handler)
  void select (String statement, Object parameter, RowBounds rowBounds, ResultHandler<T> handler)
  ```

- 事务控制方法

  ```xml
  void commit()
  void commit(boolean force)
  void rollback()
  void rollback(boolean force)
  ```

- 批量更新方法

  ```java
  List<BatchResult> flushStatements()
  ```

- 本地缓存方法

  ```java
  void clearCache()
  ```

- 使用映射器接口，执行语句（接口编程）

  ```java
  <T> T getMapper(Class<T> type)
  ```

- 其他方法

  ```java
  Configuration getConfiguration() //获取当前SqlSession的配置类
  void close(） //关闭当前sqlSession
  ```

**sqlSession是一个接口，mybatis默认使用DefaultSqlSession作为接口实现类，其创建过程如下：**

```java
	//通过sqlSessionFactory获取mybatis的sqlSsesion对象
	SqlSession openSession = sqlSessionFactory.openSession();
	//通过sqlSsesion获取指定mapper接口的JDK代理对象
	//代理对象内部对mapper中的所有方法根据注解和xml的元数据信息，
	//进行了JDBC操作和结果数据映射的实现
	DataMapper mapper = openSession.getMapper(DataMapper.class);
	List<Object> zdwjInfo = mapper.getZdwjInfo();
	
	//对于增删改操作，需要手动提交（或者创建自动提交的openSession）
	openSession.commit()
	//查询之后，将SqlSession资源关闭
	openSession.close();
```


## 11、mybatis映射器配置注解：

​		在mybatis3中，提供java注解的方式，来配置数据映射器；

常用注解：

- sql语句定义：@Insert、@Update、@Delete、@Select   属性 value为 sql语句

- 多条数据map封装：@MapKey   属性value为作为key值的映射对象属性名

- 参数传递的命名：@Param    属性value 用于对应sql语句中的占位符参数名

- 使用xml中自定义的映射resultMap：@ResultMap  属性value为xml对应resultMap标签的id

- 用于指定select查询返回void值：@ResultType

- 指定CRUD标签中的属性： @Options  一致提供了CRUD所有标签的属性

- 使用自动映射的同时，额外进行自定义映射 ：@Results   属性value为Result数组

  **@Results会自动打开局部自动映射**

- 自定义单条字段映射规则： @Result ，必须搭配@Results使用

  其one、many属性，搭配@One、@Many注解，实现关联和集合的嵌套select查询

```java
	@Select("select * from gread where user_id=#{id}")
	List<Gread> listGread(String id);
	
	
	@Select("select *from user where id=#{id}")
	@Results({
		@Result(column="id",property="id" ,id=true),
		@Result(column="id",property="gread",many=@Many(select="com.yh.mapper.UserMapper.listGread"))
	})
	List<User> getUserByid(String id);
```

​	通过property属性，使用联级属性的方式，实现关联嵌套结果映射

```java
	@Select("select u.*,g.id as gread_id,math,english from user u left join  gread g on g.user_id=u.id where u.id=#{id}")
	@Results({
		@Result(column="gread_id",property="gread.id"),
		@Result(column="math",property="gread.math"),
		@Result(column="english",property="gread.english")
	})
	List<User> getUserByidA(String id);
```

**注解版无法实现集合的嵌套结果映射**

## 12、mybatis整合spring使用：

- 首先通过spring配置文件，进行实现数据源、事务管理器的配置

```xml
<!-- 配置druid连接池管理的数据源 -->
	<bean id="dataSource"
		class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
		<property name="driverClassName" value="${driver}" />
		<property name="url" value="${db1.url}" />
		<property name="username" value="${db1.username}" />
		<property name="password" value="${db1.password}" />
		<!-- 初始化连接大小 -->
		<property name="initialSize" value="${initialSize}"></property>
		<!-- 连接池最大数量 -->
		<property name="maxActive" value="${maxActive}"></property>
		<!-- 连接池最小空闲 -->
		<property name="minIdle" value="${minIdle}"></property>
		<!-- 获取连接最大等待时间 -->
		<property name="maxWait" value="${maxWait}"></property>
	</bean>
<!-- 创建事务管理，并开启事务注解 -->
<bean id="transactionManager"
	class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>
<tx:annotation-driven />
```
- 通过SqlSessionFactoryBean来创建一个SqlSessionFactory实例

  ```xml
  <!-- 配置sqlSessionFactory -->
  <bean id="sqlSessionFactory"
  	class="org.mybatis.spring.SqlSessionFactoryBean">
  	<property name="dataSource" ref="dataSource" />
  </bean>
  ```

  sqlSessionFactory创建的内部过程：

  ```java
  @Bean
  public SqlSessionFactory sqlSessionFactory() {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    //factroyBean提供setConfiguration方法，来设置其Configuration对象，从而添加mybatis配置
      org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
  		//开启驼峰式命名法对应ENTITY类 userName 对应数据库 user_name
  		configuration.setMapUnderscoreToCamelCase(true);
  		bean.setConfiguration(configuration);
    return factoryBean.getObject();
  }
  ```

- 通过SqlSessionTemplate来创建对应的sqlSession实例

  **sqlSessionTemplate是sqlSession的实现，并且线程安全，可以被多个映射器使用；并且实现sqlsession和spring事务管理器绑定，控制mybatis的事务操作，将所有mybatis异常转换为DataAccessExceptions**

  ```xml
  <!-- 配置SqlSessionTemplate -->
  <bean id="sqlSessionTemplate"
  	class="org.mybatis.spring.SqlSessionTemplate">
  	<constructor-arg name="sqlSessionFactory"
  		ref="sqlSessionFactory" />
  </bean>
  ```

  sqlsession创建的内部过程：可以根据不同的构造方法，来设置sqlSession的特征（事务等级、执行器类型），**但不能设置事务是否自动提交，因为mybatis-spring是将事务提交交给事务管理器执行，不能自己设置自动提交，或者手动提交**

  ```java
  	public SqlSessionTemplate setSqlSessionTemplate( SqlSessionFactory sqlSessionFactory) throws Exception {
  		return new SqlSessionTemplate(sqlSessionFactory);
  	}
  ```

- 然后通过MapperFactoryBean来配置数据映射器，或者使用MapperScannerConfigurer进行包扫描批量配置数据映射器

  ```xml
  <!-- 逐一配置mybatis mapper接口 ；对应@Mapper注解-->
  <bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    	<property name="mapperInterface" value="org.mybatis.spring.sample.mapper.UserMapper" />
    	<property name="sqlSessionFactory" ref="sqlSessionFactory" />
  </bean>	
  
  <!-- 批量配置mybatis mapper接口； 对应@MapperScanner注解 -->
  	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  		<property name="basePackage" value="ssm.com.mapper" />
  		<property name="sqlSessionTemplateBeanName"
  			value="sqlSessionTemplate" />
  	</bean>
  ```

  @Mapper注解作用域和`<context:component-scan/>` 有关；对于springboot，默认< context:component-scan/>的扫描路径为 主启动类同包下，因此需要额外注意；

  **MapperFactoryBean可以使用 sqlSessionFactory，或sqlSessionTemplate构造参数，来获取mapper接口的动态代理类proxyMapper；两者都指定时，优先使用sqlSessionTemplate（注意，使用sqlSessionFactory进行映射器注入时，会导致创建多个sqlSessionTemplate，不推荐使用）**

  MapperFactoryBean创建proxyMapper内部过程:
  
  ```java
  @Bean
  public UserMapper userMapper(SqlSessionFactory sqlSessionFactory) throws Exception {
    //通过sqlsessionTemplate来获取mapper接口的实现类，并注册到spring容器中
    SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory）;
    return sqlSessionTemplate.getMapper(UserMapper.class);
  }
  ```

- 也可以使用`SqlSessionTemplate` 自己手动创建注入某个mapper接口的代理类；或者使用`SqlSessionDaoSupport` 来创建注入单个mapper接口的代理类

**综上所述：**

mybatis-spring包，帮助mybatis代码无缝整合到spring中，它将允许 MyBatis 参与到 Spring 的事务管理之中，创建映射器 mapper 和 `SqlSession` 并注入到 bean 中，以及将 Mybatis 的异常转换为 Spring 的 `DataAccessException`

## 13、mybatis的底层架构：

mybatis是一个ORM框架，是在JDBC的基础上，作为中间层来解决数据库字段和java对象属性之间的数据映射，从而更好的以面向对象角度交互数据库

**mybatis底层架构主要分为三个模块：**

1、动态代理，实现mybatis数据映射器的功能：参数传递构建param、读取映射器元数据构建Mapperstatemnt

**整个动态代理和功能实现过程如下：**

![](C:\Users\OneMTime\Desktop\Typora图片\mybatis动态代理执行映射器接口方法.png)

- sqlsession通过MapperRegistry获取指定mapper接口的代理实现类MapperProxy
- MapperProxy中创建MapperMethod，来是实现mapper接口方法的动态代理
- 在MapperMethod中，通过创建sqlCommand对象，来读取映射器元数据，生成对应的MappedStatement对象
- 在实现mapper接口方法的动态代理时，使用sqlSession的CRUDapi进行sql执行：并将MapperMethod创建处理好的MappedStatement、param作为参数传入

2、门面模式（外观模式），使用sqlSession，实现对外的所有CRUD api和辅助api（提交关闭会话等）

**参考sqlSession对象的深入理解**

3、Executor，执行器对象，实现mybatis所有内部CRUD操作和缓存操作

sqlsession中所有的CRUD操作，都是通过Executor执行器来完成的,mybatis提供多种执行器来完成一些额外的操作，这些执行器都继承通过一个抽象类BaseExecutor：

- SimpleExecutor执行器，作为mybatis底层CRUD的基础操作：

  - 以查询方法为例，需要MappedStatement（映射器元数据）、parameter（传递的参数）、RowBounds（结果集边界）、ResultHandler（额外的结果集映射处理器）、BoundSql（sql语句）5个参数，来执行doQuery

  - 通过confinguration对象创建SimpleStatmentHandler，从而进一步创建JDBC所需要的PrepareStatement对象；

  - 最后使用JDBC完成对应sql操作，再将结果交给SimpleStatementHandler.ResultSetHandler对象处理，返回一个mapper接口方法返回值类型的对象

  ```java
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
      Statement stmt = null;
      try {
        Configuration configuration = ms.getConfiguration();
        StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
        stmt = prepareStatement(handler, ms.getStatementLog());
        return handler.<E>query(stmt, resultHandler);
      } finally {
        closeStatement(stmt);
      }
    }
  ```

- ReuseExecutor执行器，额外实现预编译sql语句的重用
- BatchExecutor执行器，额外实现update的批量操作（实际就是提供一个批处理刷新的方法，并让修改方法不会直接执行）

- BaseExecutor提供了Query方法，而实现mybatis的一级缓存，实际执行sql还是使用子实现类的doQuery方法

  ```java
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
      ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
      if (closed) {
        throw new ExecutorException("Executor was closed.");
      }
      if (queryStack == 0 && ms.isFlushCacheRequired()) {//是否第一次查询、是否需要查询前清空本地缓存
        clearLocalCache();//清空本地缓存
      }
      List<E> list;
      try {
        queryStack++;//查询次数+1，并通过key获取本地缓存
        list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
        if (list != null) {
          //处理本地缓存的输出参数（如果为存储过程）
          handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
        } else {
          //如果本地缓存没有，则使用子类实现方法执行sql查询
          list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        }
      } finally {
        queryStack--;
      }
      if (queryStack == 0) {
        for (DeferredLoad deferredLoad : deferredLoads) {
          deferredLoad.load();
        }
        // issue #601
        deferredLoads.clear();
        if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
          // issue #482
          clearLocalCache();
        }
      }
      return list;
    }
  ```

- CachingExecutor执行器，使用装饰模式，在BaseExecutor的基础上，实现了二级缓存功能：

  ```java
   @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
        throws SQLException {
      Cache cache = ms.getCache();
      if (cache != null) {//二级缓存是否存在
        flushCacheIfRequired(ms);//是否需要刷新二级缓存
        if (ms.isUseCache() && resultHandler == null) {//是否允许使用二级缓存，并保证不会使用自定义结果集处理器（因此使用缓存，结果集就不会执行额外映射处理）
          ensureNoOutParams(ms, boundSql);//确保没有输出参数，不是执行存储过程的sql
          @SuppressWarnings("unchecked")
          List<E> list = (List<E>) tcm.getObject(cache, key);
          if (list == null) {
            //调用所修饰的BaseExecutor实现类，来执行sql查询
            list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
            //将结果保存到二级缓存
            tcm.putObject(cache, key, list); 
          }
          return list;
        }
      }
      return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }
  ```

![](C:\Users\OneMTime\Desktop\Typora图片\mybatis执行器原理.jpg)

### 手动使用SimpleExecutor，实现mybatis底层的查询操作：

```java
	private Configuration configuration;
	private JdbcTransaction jdbcTransaction;

	@Before
	public void init() {
        //初始化mybatis的 sqlsession对象和configuration对象；并创建一个jdbc事务
		SqlSessionFactory sqlSessionFactory = MybatisSqlSessionFactory.getSqlSessionFactory();
		SqlSession sqlSession = sqlSessionFactory.openSession();
		configuration = sqlSession.getConfiguration();
		jdbcTransaction = new JdbcTransaction(sqlSession.getConnection());
	}

	@Test
	public void Test() throws SQLException {
		SimpleExecutor simpleExecutor = new SimpleExecutor(configuration, jdbcTransaction);

        //获取指定id的映射器元数据
		MappedStatement ms = configuration.getMappedStatement("com.yh.mapper.UserMapper.liseUserByIdA");
		HashMap<String, String> parameter = new HashMap<String, String>();
		parameter.put("id", "11");

        //执行查询
		List<Object> list = simpleExecutor.doQuery(ms, parameter, RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER,ms.getBoundSql(parameter));
	}
```

