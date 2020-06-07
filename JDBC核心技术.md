# JDBC核心技术

## 1、数据库四要素：

username：用户名

password：密码

url：JDBC连接地址，包含数据库地址、端口、实例名（服务名）、数据库名，然后一些连接属性：是否使用Unicode字符集，Unicode字符集的编码格式、连接使用的时区（useUnicode=true、characterEncoding=utf8、serverTimezone=GMT）

driver：JDBC数据库连接驱动，是JDBC原生api的实现层，用于操作指定数据库

### **常用数据库的url和driver：**

**1、mysql：**

​      url：jdbc:mysql://localhost:3306/hibernatedemo

​	  driver: com.mysql.cj.jdbc.Driver（mysql 6.x及以上）/com.mysql.jdbc.Driver(mysql 5.x及以下)

**2、Oracle：**

​      url：jdbc:oracle:thin:@localhost:1521/XE（服务名）

​				jdbc:oracle:thin:@localhost:1521:XE （实例名）

​		        jdbc:oracle:thin:@ (DESCRIPTION =

​				(ADDRESS = (PROTOCOL = TCP)(HOST = DESKTOP-B9FUSHL)(PORT = 1521))   
​				(CONNECT_DATA =               
​			    (SERVER = DEDICATED)          
​				(SERVICE_NAME = XE)         )  )   （TNS连接）

​     driver： oracle.jdbc.driver.OracleDriver

**3、sqlServer：**

​		url：jdbc:sqlserver://localhost:2433; DatabaseName=test

​		driver: com.microsoft.sqlserver.jdbc.SQLServerDriver

**4、Postgresql：**

​		url： jdbc:postgresql://10.19.0.11:5432/authorization

​		driver：org.postgresql.Driver

## 2、JDBC三对象：

### 1、JDBC使用过程

````java
        //通过配置文件读取四要素
		String url= PropertiesUtil.getPropertiesValue("DBConnection", "url");
		String username= PropertiesUtil.getPropertiesValue("DBConnection", "username");
		String password= PropertiesUtil.getPropertiesValue("DBConnection", "password");
		String driver= PropertiesUtil.getPropertiesValue("DBConnection", "driver");
		
		//创建JDBC三对象
		Connection conn=null;
		PreparedStatement pre = null;
		ResultSet res = null;
		try {
			Class.forName(driver);//驱动加载后，自动注册驱动管理器对象
			conn = DriverManager.getConnection(url, username, password);
			String sql = "xxxxx";
			pre = conn.prepareStatement(sql);
			res = pre.executeQuery();
			
		} catch (SQLException e) {
			e.printStackTrace();
		}finally {
			ConnectionClose.close(conn, pre, res);
		}
````

### 2、JDBC三对象作用

Connection：数据库连接对象，用于创建可以执行sql语句的Statement对象

Statement：用于执行sql语句，获得查询结果，分为三种实现类

​		1、Statement   正常执行sql语句，无法预编译，因此不能进行#{}参数传递；只能直接拼接sql（使用${}）从而可能导致sql注入

​		2、prepatedStatement    先对sql语句进行预编译，能进行#{}参数传递（来代替拼接sql、${},放在sql注入）；由于预编译，可以重复高效执行一条sql语句（批量插入）；可以操作流（Stream）等其他数据类型插入到sql中（Blob类型数据读取、插入）

​		3、CallableStatement   用于执行存储过程语句

ResultSet： 用于保存Statement执行sql的查询结果

## 3、数据库事务：

**事务：**一组逻辑操作单元，使数据从一种状态转化为另一种状态

**事务处理原则：**一个事务中执行多个操作时，即使出现故障，也要保证这个事务操作全部执行或者都不执行；保证数据库数据的一致性。

### **1、JDBC事务基本操作：**

1、默认情况下，JDBC执行sql后，会直接提交事务，因此为保证事务处理原则，应该关闭自动提交

````java
conn.setAutoCommit(false);
````

2、当执行完事务的所有操作后，手动提交事务

````java
conn.commit();
````

3、对于这些操作进行try-catch，当发生异常后，在catch进行事务回滚

````java
conn.rollback();
````

****当使用数据库连接池时，连接不会关闭，而是之后在继续使用。因此对于进行事务操作的conn对象，应在使用完后，恢复到默认设置，保证不影响再次使用。（将conn的自动提交事务打开，然后归还conn对象）

### **2、事务的ACID属性：**

1、原子性：事务时不可分割的工作单位，所有操作要么都执行，要么都不执行

2、一致性：事务提交会保证数据库从一个状态转化为另一个状态；不会出现其他的中间状态

3、隔离性：并发执行多个事务时，不能被相互干扰

4、持久性：事务一旦被提交，其数据的修改就是永久性的

### **3、事务并发问题：**

当同时执行多个事务，并且多个事务访问相同数据时，若不采取一定的隔离机制，会导致一些并发问题：

1、脏读：T1事务读取的T2事务更新的字段数据，但此时T2事务又发送了回滚，导致T1事务读取的数据是临时且无效的

2、不可重复读：T1读取一个字段数据，T2又更新该字段数据，并提交，之后T1在未提交前又读取该字段数据，导致事务前后两次相同操作的结果不同（破坏了事务本身的一致性）

3、幻读：T1读取一个字段的数据，T2又插入了多行该字段数据，并提交，之后T1在未提交前又读取字段数据时，导致多出多个查询数据（破坏了事务本身的一致性，但一般允许该问题产生）

### **4、四种事务隔离级别：**

针对于解决事务并发问题，数据库设置了四种隔离级别，来保证不同需求下，数据库事务的隔离性

1、Read Uncommited（可读未提交的事务）：会导致三种并发问题出现，因此一般不会使用

2、Read Commited（可读已提交事务）：只允许事务读取被提交的变更数据，解决了脏读问题

3、Repeatable Read （可重复读）：在事务执行期间，禁止其他事务对同行数据进行修改，解决脏读、不可重复读问题 

4、Serializable（串行化）：禁止多个事务同时修改（增删改）同一个表数据，解决所有并发问题，但是这样会大大降低并发性能，不推荐使用

Oracle支持2种事务隔离级别：Read Commited、Serializable，默认为 Read Commited

Mysql支持4种事务隔离级别，默认为Repeatable Read

**JDBC隔离级别设置：**

````java
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
````

**并不会改变数据库默认的隔离级别，只是修改当前连接操作的隔离级别**

## 4、数据库连接池：

### **1、普通JDBC数据库连接缺点：**

1、JDBC使用DriverManager来进行来Connection连接对象创建，每次连接数据库都需要将 conn 加载到内存中，然后验证用户名密码，执行完sql后断开连接。每次使用数据库都需要这个过程，导致高并发时，占用大量系统资源，导致服务器崩溃

2、每次连接Conn创建都需要关闭，否则会发送数据库系统内存泄漏，最终导致数据崩溃重启

3、开发过程中不能控制连接Conn对象创建数据，这样会导致系统资源毫无顾虑的分配。当连接过多时，导致内存溢出，服务器崩溃

### **2、数据库连接池技术原理：**

1、为解决传统开发中数据库连接问题，可以采用数据库连接池技术

2、**数据库连接池基本思想和java线程池类似**。保证连接池中放入一定数量的连接Conn，当需要使用数据库时，则冲连接池中取出，使用完后再放回去。

3、数据库连接池负责分配、管理、释放数据库连接，它允许应用程序重复使用一个现有的数据库连接。

4、数据库连接池可以设置最小连接数数和最大连接数。

### **3、数据库连接池技术优点：**

1、数据库连接资源重用

2、数据库连接池在项目启动时，会进行初始化，同时创建最小数的连接对象，备用用于数据库连接操作，并且使用完后不用释放。避免了数据库来连接初始化和释放的过程，减少了系统的响应时间

3、可以管理数据库连接资源的分配

4、有完善的数据库连接创建、释放机制，避免了数据库连接操作可能出现的资源泄漏

### 4、连接池对于资源的关闭：

当使用连接池后，还是需要对于JDBC三对象进行关闭，减少资源占用。

对于conn对象的close，连接池对于该方法进行了重写，close并不会释放数据库连接，而是改变该连接在连接池中的状态，有占用（活跃）转变为队列（休眠）状态，从而让连接池去判断是否释放不必要的连接。

### **5、Druid数据库连接池的实现：**

JDBC的数据库连接池统一使用了JDK javax.sql.DataSource接口包。根据不同的开源组件来实现，目前一般使用Druid（德鲁伊）连接池。

#### JDBC 对于Druid连接池配置

````java
username=root
password=123456
url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT&Unicode=true&characterEncoding=utf8
driver=com.mysql.cj.jdbc.Driver

#数据库连接池常用配置属性
#最大连接数、初始化连接数、获取连接最大等待时间、最小连接数
maxActive:20  
initialSize:1  
maxWait:60000  
minIdle:10  
 
#检查连接活动状态线程执行间隔时间、连接最小休眠时间（当检查连接活动线程启动，查询该连接休眠时间大于该值时，释放连接（保证连接数大于最小值））  
timeBetweenEvictionRunsMillis:60000  
minEvictableIdleTimeMillis:300000  

#检查sql是否有效，mysql为  SELECT 'x' ；oracle为select 1 from dual
validationQuery:SELECT 'x'

#申请连接时，检查连接是否有效、归还连接时，检查连接是否有效  
testOnBorrow:false 
testOnReturn:false

#周期性检查连接是否有效（休眠时间大于timeBetweenEvictionRunsMillis的连接）
testWhileIdle:true   

#对PreparedStatements对象进行对象池缓存、PreparedStatements缓冲池最大数
poolPreparedStatements:true  
maxOpenPreparedStatements:20  
````

#### **JDBC 对于Druid连接池使用**

````java
//		String url= PropertiesUtil.getPropertiesValue("DBConnection.properties", "url");
//		String username= PropertiesUtil.getPropertiesValue("DBConnection.properties", "username");
//		String password= PropertiesUtil.getPropertiesValue("DBConnection.properties", "password");
//		String driver= PropertiesUtil.getPropertiesValue("DBConnection.properties", "driver");
//		
//		//创建对应连接池的数据源对象
//		DruidDataSource druidDataSource = new DruidDataSource();
//		druidDataSource.setUrl(url);
//		druidDataSource.setUsername(username);
//		druidDataSource.setPassword(password);
//		druidDataSource.setDriverClassName(driver);
		
//		//配置连接池数据源其他属性
//		druidDataSource.setInitialSize(initialSize);//初始化连接数,默认0
//		druidDataSource.setMaxActive(maxActive);//最大连接数，默认8
//		druidDataSource.setMinIdle(value);//最小连接数，默认0
//		druidDataSource.setMaxWait(maxWaitMillis);//获取连接最大等待时间
		
		//也可以直接通过配置文件来创建Druid数据源
		InputStream resourceAsStream = DruidConnetionUtil.class.getClassLoader().getResourceAsStream("DruidDataSource.properties");
		Properties properties = new Properties();
		Connection conn = null;
		try {
			properties.load(resourceAsStream);
			DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
			conn = dataSource.getConnection();
			
		} catch (Exception e) {
			e.printStackTrace();
		}
		return conn;
````



## 5、spring事务注解

### 1、spring事务注解常见问题

1、当使用spring事务注解时，一般放在serviceImpl层，其工作方式是：当该层方法发生异常时，就进行事务回滚。因此对于该方法的异常不能进行捕捉，应向上抛出，在controller层进行try-catch，否则会导致事务回滚无法进行

2、spring @Transactional事务注解默认只对运行时异常进行处理，当需要该方法产生非运行异常时，进行回滚。则应进行属性配置：**@Transactional(rollbackFor = Exception.class)//发送任何异常都进行回滚操作**

3、在多数据源环境下，@Transactional应指定value值，否则会使用主键数据库事务管理器，导致事务没有使用对应的管理器，事务无法正常执行

4、@Transactional 只能被应用到public方法上, 对于其它非public的方法,如果标记了@Transactional也不会报错,但方法没有事务功能

### **2、spring事务注解基本属性：**

1、value   事务管理器

2、Propagation  事务传播行为

3、timeout 事务超时时间

4、isolation 事务隔离级别

5、rollbackFor 能触发事务回滚的异常

## 6、springboot Druid连接池使用

1、导入Druid连接池依赖

````java
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.20</version>
		</dependency>
````

2、配置被Druid连接池管理的数据源及其它组件

````java
@EnableTransactionManagement//该注解放在主配置类或者自定义配置类中，都能全局生效(是针对于所有数据源的事务注解启动)
@Configuration
//指定mapper接口注入容器，并指定这些mapper接口操作sql的sqlSessionTemplate对象(mapperScan也可以放在其他配置类定义，全局生效)
@MapperScan(basePackages="com.example.demo.test1.mapper" ,sqlSessionTemplateRef="test1SqlSessionTemplate")
public class DataSourceConfig {

	@Bean("test1DataSource")
	@Primary//默认数据源，当匹配不到相应数据源组件时，使用该数据源
	@ConfigurationProperties(prefix="spring.datasource.test1")
	public DataSource dataSource(){
		return new DruidDataSource();    //使用Druid线程池
//		return DataSourceBuilder.create().build();  //该为使用spring boot默认线程池（1.5为tomcat-jdbc、2.x为HikariCP）
	}
	
	@Bean("test1SqlSessionFactory")
	public SqlSessionFactory sqlSessionFactory(@Qualifier("test1DataSource") DataSource dataSource) throws Exception{
		SqlSessionFactoryBean sSFB = new SqlSessionFactoryBean();
		sSFB.setDataSource(dataSource);
		return sSFB.getObject();
	}
	
	@Bean("test1SqlSessionTemplate")
	public SqlSessionTemplate sqlSessionTemplate(@Qualifier("test1SqlSessionFactory") SqlSessionFactory sqlSessionFactory){
		return new SqlSessionTemplate(sqlSessionFactory);
	}
	
	@Bean("test1TransactionManager")
	public DataSourceTransactionManager dataSourceTransactionManager(@Qualifier("test1DataSource") DataSource dataSource){
		return new DataSourceTransactionManager(dataSource);
	}
}
````



3、配置Druid数据源属性

````java
#Druid线程池使用 url
spring.datasource.test1.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT&Unicode=true&characterEncoding=utf8
spring.datasource.test1.username=root
spring.datasource.test1.password=123456
spring.datasource.test1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.test1.type=com.alibaba.druid.pool.DruidDataSource

spring.datasource.test1.filters=stat,wall
spring.datasource.test1.maxActive=20
spring.datasource.test1.initialSize=1
spring.datasource.test1.maxWait=60000
spring.datasource.test1.minIdle=10
spring.datasource.test1.timeBetweenEvictionRunsMillis=60000
spring.datasource.test1.minEvictableIdleTimeMillis=300000
spring.datasource.test1.validationQuery=SELECT 'X'
spring.datasource.test1.testOnBorrow=false
spring.datasource.test1.testOnReturn=false
spring.datasource.test1.testWhileIdle=true
spring.datasource.test1.poolPreparedStatements=true
spring.datasource.test1.maxOpenPreparedStatements=20
````



4、配置Druid对应用和sql的监控配置

````JAVA
/**
     * 配置监控服务器
     * @return 返回监控注册的servlet对象
     * @author SimpleWu
     */
	//druid监控网站地址： ip：端口/项目名/druid
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        // 添加IP白名单
        servletRegistrationBean.addInitParameter("allow", "127.0.0.1");
        // 添加IP黑名单，当白名单和黑名单重复时，黑名单优先级更高
        servletRegistrationBean.addInitParameter("deny", "127.0.0.1");
        // 添加控制台管理用户
        servletRegistrationBean.addInitParameter("loginUsername", "yh");
        servletRegistrationBean.addInitParameter("loginPassword", "123456");
        // 是否能够重置数据
        servletRegistrationBean.addInitParameter("resetEnable", "false");
        return servletRegistrationBean;
    }

    /**
     * 配置服务过滤器(不需要监控的接口)
     *
     * @return 返回过滤器配置对象
     */
    @Bean
    public FilterRegistrationBean statFilter() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        // 添加过滤规则
        filterRegistrationBean.addUrlPatterns("/*");
        // 忽略过滤格式
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*,");
        return filterRegistrationBean;
    }
````

## 7、JDBC使用prepatedStatement进行高效批量插入、更新    

1、JDBC原生

````java
			String sql = "INSERT INTO user(id,name,password) values (?,?,?)";
			pre = conn.prepareStatement(sql);
			for (int i = 0; i < 10000; i++) {
				pre.setObject(1, "id_"+i);
				pre.setObject(2, "name_"+i);
				pre.setObject(3, "password_"+i);
				
//				通过pre的Batch对象，来实现批量执行sql，减少sql执行与数据库的交互次数（大大提升性能）
//				将该sql保存到Batch中，但不执行
				pre.addBatch();
				if ((i!=0 && i%500==0) || i=10000-1) {
					//将Batch中所有的sql一起执行
					pre.executeBatch();
					//清空Batch，用于后面保存sql
					pre.clearBatch();
				}
			}
````

2、springboot集成mybatis注解版

mapper层

````java
@Insert("<script> "+
	"INSERT INTO user(id,name,password) VALUES " +
     //传入的集合参数、迭代时的数据变量、迭代时的集合下标变量、迭代时的分割符
	"<foreach collection='accountList' item='item' index='index' separator=','>"+ 
	" (#{item.id},#{item.name},#{item.password}) "+
	"</foreach> "+
	"</script>")
void addListAccount(@Param("accountList")List<Account> accountList);
````

serviceImpl层

````java
List<Account> list = new  ArrayList<Account>();
		for (int i = 0; i <10000; i++) {
			Account account = new Account();
			account.setId(i);
			account.setName("yh"+i);
			account.setPassword("123456"+i);
			list.add(account);
		}
		as.addListAccount(list);
````

## 8、JDBC操作blob（二进制）数据

1、JDBC原生

由于Blob数据可能很大，因此需要配置数据库插入的最大数据量（mysql默认为1M）

````java
		Connection conn=null;
		PreparedStatement pre =null;
		conn = DruidConnetionUtil.getConnection();
		
插入blob、clob（text）数据
//		String sql="insert into phtot(id,`blob`,`clob`) values(1,?,?)";
//		pre = conn.prepareStatement(sql);
////		将文件转化为字节流
//		FileInputStream fileInputStream = new FileInputStream(new File("C:\\Users\\OneMTime\\Desktop\\Typora图片\\1566201801526.png"));
////		将文件转化为字符流
//		FileReader fileReader = new FileReader(new File("C:\\Users\\OneMTime\\Desktop\\Mysql1.txt"));
//		
//		pre.setBlob(1, fileInputStream);
////		pre.setClob(2, fileReader);//Oracl 大文本为clob类型
//		pre.setCharacterStream(2, fileReader); //mysql 大文本为text类型
//		pre.executeUpdate();
		
	
读取blob、clob（text）数据
		String sql = "select * from phtot where id =1";
		pre=conn.prepareStatement(sql);
		ResultSet executeQuery = pre.executeQuery();
		while (executeQuery.next()) {
			Blob blob = executeQuery.getBlob("blob");
//			Clob clob = executeQuery.getClob("clob");
//			//将blob转化为字节流
//			InputStream binaryStream = blob.getBinaryStream();
//			将clob转化为字符流
//			Reader characterStream = clob.getCharacterStream();
			
			//txet转化为字符流
			Reader characterStream2 = executeQuery.getCharacterStream("clob");
			if (characterStream2!=null) {
				BufferedReader bufferedReader = new BufferedReader(characterStream2);
				
				String s="";
				while((s=bufferedReader.readLine())!=null) {
					System.out.println(s);
				}
			}

		}
````

2、springboot集成mybatis注解版

**对于clob、text字段，可以直接通过String来进行字段数据接收和存放**

对于blob字段，存放可以直接通过#{}占位符注入byte[ ]来实现

而接收必须建立相应表的实体类，进行byte[ ]类型属性与字段名的数据映射

````java
	@Select("select * from phtot where id=#{id} ")
	Phtot getPhtot(String id);//必须使用实体类接收，byte[]（java常用数据类型）无法触发类型转化处理器
````



````java
		FileOutputStream fileOutputStream = null;
		try {
//			FileInputStream fileInputStream = new FileInputStream(new File("C:\\Users\\OneMTime\\Desktop\\11.txt"));
////			字节输入流转二进制数组
//			byte[] copyToByteArray = FileCopyUtils.copyToByteArray(fileInputStream);
//			as.addPhoto(copyToByteArray);
//			BlobTypeHandler
			
			
			Phtot phtot = aM.getPhtot("18");
			fileOutputStream = new FileOutputStream(new File("C:\\Users\\OneMTime\\Desktop\\hh.PNG"));
			fileOutputStream.write(phtot.getBlob1());			
		} catch (Exception e) {
			e.printStackTrace();
		}finally {
			try {
				if (fileOutputStream!=null) {
					fileOutputStream.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
````

## 9、mybatis自定义类型转化处理器（用于处理数据库数据与java数据的类型转化）

​	1、自定义typeHandle类

````java
@MappedJdbcTypes(value=JdbcType.BLOB)  //定义该类型处理器 数据库数据类型	
@MappedTypes(String.class) //定义该类型处理器  java数据类型
public class BlobTypeHandler extends BaseTypeHandler<String> {
	    public void setNonNullParameter(PreparedStatement ps, int i,  
	            String parameter, JdbcType jdbcType) throws SQLException {  
	    	System.out.println("使用处理器1");
	        //声明一个输入流对象
	        ByteArrayInputStream bis;  
	        try {  
	            //把字符串转为字节流
	            bis = new ByteArrayInputStream(parameter.getBytes("utf-8"));  
	        } catch (UnsupportedEncodingException e) {  
	            throw new RuntimeException("Blob Encoding Error!");  
	        }     
	        ps.setBinaryStream(i, bis, parameter.length());  
	    }  

	    @Override  
	    public String getNullableResult(ResultSet rs, String columnName)  
	            throws SQLException {  
	        Blob blob = (Blob) rs.getBlob(columnName);  
	        byte[] returnValue = null;  
	        if (null != blob) {  
	            returnValue = blob.getBytes(1, (int) blob.length());  
	        }  
	        try {  
	            //将取出的流对象转为utf-8的字符串对象
	        	System.out.println("使用处理器2");
	            return new String(returnValue, "utf-8");  
	        } catch (UnsupportedEncodingException e) {  
	            throw new RuntimeException("Blob Encoding Error!");  
	        }  
	    }

		@Override
		public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
			System.out.println("使用处理器3");
			return null;
		}

		@Override
		public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
			System.out.println("使用处理器4");
			return null;
		}  
	    }
````

2、指定mapper映射字段中，数据转化的类型处理器

````java
	@Select("select * from phtot where id=#{id} ")
	@Results({			         		
              @Result(column="blob1",property="blob1",
              typeHandler=com.example.demo.typehandler.BlobTypeHandler.class)   
	})//通过@Results、@Result来设置pojo中部分字段映射规则（字段名、pojo属性名、数据类型处理器）
	Phtot getPhtot(String id);
````

