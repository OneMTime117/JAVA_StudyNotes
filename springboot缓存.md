# springboot缓存

## 1、springboot本地缓存

### JSR107：

用于定义缓存规范，提供五个核心接口：CachingProvider、CacheManager、Cache、Entry、Expiry

CachingProvider  缓存供应，定义了创建、配置、管理和控制多个CacheManager
CacheManager  缓存管理器，定义了创建、配置、管理和控制多个唯一名Cache
Cache  缓存，是一个类似于map数据结构的对象，用于临时存储key及其对应的value
Entry 一组数据（条目），存储在Cache中的一组键值对
Expiry  用于定义Cache中的一组数据的有效期
由于JSR107是一个抽象层，而与其适配的实现层比较少，需要自己编写实现代码，使用比较少；springboot提供了自己的缓存抽象层

### springboot缓存注解：

**springboot使用了JSR-107的CacheManager、Cache接口来统一不同的缓存技术，并支持JSR-107的注解简化开发**

- @Cacheable，注解在方法上，根据方法请求参数对方法的结果进行缓存，当再需要该结果时（方法参数一致时），就不需要执行方法，而是直接使用缓存
  不进去方法。
  可以添加属性：value，指定缓存的名字（必须），可以指定多个名字，则生成多个缓存；key，指定缓存的key，也可以使用keyGenerator，来指定key的生成器；cacheManager，指定缓存管理器；
  condition指定缓存条件，条件trun则进行结果缓存；unless，否定缓存，条件为trun则不进行缓存，该条件可以指定结果
- @Cacheable运行流程：1、方法运行之前，先通过cacheName获取对应Cache组件，若没有则生成
  		     2、然后查找Cache中的对应缓存，使用key来匹配
        		KEY的按照固定的策略生成：
  默认使用SimplekeyGenerator生成key，如果没有参数，key=new simpleKey()；有一个参数，key=参数值；有多个参数，key=new simplekey(params)
  		     3、没有查到相应缓存则调用目标方法
  		     4、将方法返回结果放入到缓存	
- @CachePut   既调用方法，又更新缓存；用于更新缓存，它会先调用方法，然后再将结果放入缓存中
  由于缓存保存时，是通过key来匹配，因此理论上不会更新查询方法中的缓存，因此需要指定对应查询方法的key和查询方法的CacheName，来进行缓存的更新
  如查询时使用的id作为参数，则更新时key=“#user.id”，Value="user"
- @CacheEvict，清空对应方法的缓存；用于清除缓存，它会先调用方法（删除数据库数据），然后再将删除的数据对应的缓存删除
  因此需要指定value=cacheName（清除哪个缓存的数据）、数据对应的key，来匹配删除缓存key=“#id”
  通过allEntries=true属性，可以请求这个缓存Cache中的所有数据
  通过beforeIncocation=true，缓存清除操作在方法执行之前，用于使缓存清除，不会因为方法执行出现异常而不执行
- @Caching ,用于设置复杂缓存规则，它可以设置多个@Cacheable、@CachePut、@CacheEvict注解
  如：@Caching{
  	cacheable={
   		@Cacheable（value=“”，key=“”）
  		@Cacheable （value"“，key=""）
     	},
  	put={
  		@CachePut(value="",key="")
  	}
  }
- @CacheConfig  用于注解类，指定该类中所有cache注解的一些公共属性，如cachename=""指定了改类所有缓存注解用到的cachename
- @EnableCaching   开启缓存注解，一般注解到主配置类上

## 2、springboot整合Redis：

### Redis：

**是一个开源的，内存中的数据结构存储系统，可以作为数据库、缓存和消息中间件**

Redis一般操作常见的五大数据类型：String（字符串）、list（链表）、set（无序不重复集合）、hash（散列）、Zset（有序不重复集合）

### 1、springboot使用redis 作为nosql数据库

**springboot提供stringRedisTemplate、redisTemplate封装了操作redis中数据的方法（对应redis客户端中的命令）**

- **StringRedisTemplate 和redisTemplate的区别：**

1. 两者的关系是StringRedisTemplate继承RedisTemplate


2. 两者的数据是不共通的；也就是说StringRedisTemplate只能管理StringRedisTemplate里面的数据，RedisTemplate只能管理RedisTemplate中的数据。


3. SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。


StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。

RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

两者的使用选择：当redis数据库中本来存放、取出时就是String类型，就使用StringRedisTemplate；
当保存数据时复杂的对象类型时，取出又不想进行数据转换，那么就使用RedisTemplate

- **redisTempalate（StringRedisTemplate）常用方法：**

redisTemplate.opsForValue();//操作字符串

redisTemplate.opsForHash();//操作hash

redisTemplate.opsForList();//操作list    list是链表，使用rightpush、leftpush等进行操作

redisTemplate.opsForSet()；//操作set

redisTemplate.opsForZSet();//操作有序set

后面接set、get、add、remove等来操作键值对；delete删除整个集合，如：

````java
Strign value = redisTemplate.opsForValue().get(key)
````



###还可以使用boundValueOps（key），来直接获取该操作对象，从而进行set、get、add、remove操作，

这样减少对某个数据多次操作时，key的多次填写（常用），如：

````java
	BoundValueOperations<String, String> boundValueOps = Template.boundValueOps("key");
	boundValueOps.get();
	boundValueOps.set('fsfads');
````

### 2、springboot使用redis作为缓存：

springboot在导入redis的start后，spring容器会自动将redisCacheManager作为缓存组件（已经被springboot自动配置好了）,然后结合springboot的缓存注解进行redis缓存操作（也可以使用编码的方式，直接注入一个缓存管理器对象，获取Cache进行增删改查）

**redis作为缓存保存数据时，默认使用    缓存对象名：：作为前缀**

### 3、springboot配置使用redis：

- 在主配置文件application.properties中添加redis服务器数据源（服务器地址、端口名、数据库）

  ````java
  spring.redis.host=localhost
  spring.redis.port=6379
  spring.redis.database=3   只能为数字，对应redis中的第几个数据库
  
  #连接池最大连接数（使用负值表示没有限制）
  spring.redis.jedis.pool.max-active=50
  #连接池最大阻塞等待时间（使用负值表示没有限制）
  spring.redis.jedis.pool.max-wait=3000
  #连接池中的最大空闲连接
  spring.redis.jedis.pool.max-idle=20
  #连接池中的最小空闲连接
  spring.redis.jedis.pool.min-idle=2
  #连接超时时间（毫秒）
  spring.redis.timeout=5000
  ````

- redis保存的对象数据时默认使用的jdk序列化策略（即保存对象类需要实现Serializable接口），因此redis中保存的是字节数组,一般情况下，我们需要保存的是json数据，因此需要对redisCacheManager进行一些额外配置：

**将redisCacheManager使用Jackson2JsonRedisSerializer序列化策略**
1、在配置类中添加自定义redisTemplate组件，其中使用Jackson2JsonRedisSerializer序列化（springboot中 redisTemplate也就使用该组件操作redis no数据库，来完成对象数据的json序列化）

````java
@Bean
	public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
		//创建json序列化策略，针对于object对象（即所有对象）
		Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
		
		//自定义reidsTemplate对象(用于操作string-object类型的键值对数据)
		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
		redisTemplate.setConnectionFactory(factory);//初始化redisTemplate
		//设置其序列化策略（普通java数据（对应redis的）和转化为hash值数据（））
		redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
		redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
		redisTemplate.setKeySerializer(new StringRedisSerializer());
		redisTemplate.setHashKeySerializer(new StringRedisSerializer());
		return redisTemplate;
	}
````

自定义RedisTemplate会根据声明的RedisTemplate<>泛型进行匹配注入。如果没有指定泛型，且不指定@qualifier（组件名）时，则会出现异常；

因此在使用自定义RedisTemplate时，要么指定对应的泛型，要么直接@qualifier（组件名）对应自定义组件名。当时有默认组件时，直接声明泛型 RedisTemplate<Object,Object>

2、在配置类中添加自定义redisCacheManager组件：~~使用自定义的redisTemplate作为参数
~~之后再缓存注解时，就选择该cachemanager组件作为缓存管理器~~~~（该方法只试用于springboot1.5x,而springboot2.x版本不再提供redisTemplate为参数的redisCacheManager的构造器，**当只使用redis作为缓存时，可以直接设置redisCacheManager，不需要先自定义使用json序列化的RedisTemplate**）

springboot2.x版本：

**对应自定义的cacheManager会直接替换默认的RedisCacheManager（当springboot检测到有其他CacheManager时，将不会自动配置默认的RedisCacheManager）**

````java
@Bean
	public CacheManager cacheManager(RedisConnectionFactory factory) {
		//通过RedisCacheConfiguration对象生成一个默认config对象，并修改其默认属性
	RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig();
	cacheConfig=cacheConfig.entryTtl(Duration.ofDays(1))//设置缓存过期时间（1天）
				           .disableCachingNullValues()//不允许缓存有空值
	//设置缓存管理器key-value数据序列化策略（key为String，value为json）	
			              .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new Jackson2JsonRedisSerializer(Object.class)))
.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
		
		//通过RedisConnectionFactory、RedisCacheConfiguration来创建RedisCacheManager
		return RedisCacheManager.builder(factory).cacheDefaults(cacheConfig).build();
	}
````

**###redis缓存在使用时，虽然可以使用redisTemplate来将数据存入redis，但是缓存使用不了该数据，即数据默认不被Cache保存使用（因为缓存在使用时，需要先指定缓存名获取缓存对象，这样对应redis中的缓存目录名，因此即使key值一样，也不会去匹配非缓存目录中的数据值）**





