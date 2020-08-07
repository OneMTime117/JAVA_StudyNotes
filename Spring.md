Spring

## 1、spring概念

​		spring是一个轻量级的开源JAVAEE框架，用于解决企业应用开发的复杂性。有两个核心部分：

- IOC：控制反转，将创建对象的过程交给spring进行管理
- AOP：面向切面编程，通过动态代理，将一些模块化代码分离，不写入源代码

### spring特点：

- 方便解耦，简化开发
- 面向切面编程，使开发者专注于业务逻辑代码的编写
- 支持对各种框架的集成
- 支持各种JAVAEE规范，并提供更加简化的API

### spring核心容器模块：

spring是一个轻量化框架，通过多个模块来给开发者提供自己应用程序所需要的功能，实现JAVAEE平台的所有规范。其中核心容器模块就是spring框架的基础，用于整个spring容器的搭建和使用，也是整合所有其他模块的基础，它包括：

- Beans：访问配置文件、管理创建bean、IOC容器的依赖注入
- Core：spring基本核心工具类，其他spring组件的基本核心
- Context：springIOC功能上的扩展服务：邮件服务、任务调度、JNDI定位、缓存、远程访问、视图层框架的封装等
- SpEL：spring表达式语言，spring3.0中提出，使用#{}作为定界符，用于配置Bean对象的属性注入

对应四个spring依赖包：spring-beans、spring-core、spring-context、spring-expression

## 2、IOC容器

​		IOC容器，即spring容器，实现spring的依赖注入，控制反转，整个容器的创建和使用依赖于spring的核心容器模块，因此只需要导入基础的四个依赖包

### springIOC基础入门：

- 整个IOC容器的搭建，需要使用spring配置文件来完成：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
                        http://www.springframework.org/schema/context    
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd">
    
    <!-- 配置User对象 -->                    
    <bean id="user" class="com.yh.User"></bean>                                           
</beans>  
```

**根据需要使用spring模块的标签，来引入对应的XML Schema文件，定义springXML配置文件的文档结构**

- 获取IOC容器上下文，然后依赖注入获取IOC容器中创建好的实例

```java
	public static void main(String[] args) {
		//获取IOC容器上下文
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
		User user = context.getBean("user", User.class);
	}
```

### IOC容器的底层原理：

IOC容器设计和目的：

​	将对象创建和对象调用的过程，交给spring进行管理；从而降低对象之间的耦合

IOC容器的底层技术：

​	xml解析、工厂模式、反射

IOC容器底层原理：

- 在spring的xml配置文件中，配置IOC容器需要创建管理的对象	

- spring通过对xml解析，获取需要创建对象的信息、实例化方式和某些特定行为

- 通过反射技术，使用xml解析得到的信息创建该实例；

- 最后使用工厂模式，来提供获取该实例的方法

  因此实际上，IOC容器底层就是一个对象工厂，使用java反射技术，读取xml配置文件，提供实例化对象的方法

手动实现IOC容器的实例化对象过程：

```java
public class IocFactory {

	private static HashMap<String, Object> singleBeanMap=new HashMap<String, Object>(10);
	
    //省略了xml解析获取bean全类名的过程
	public static void setBean(String beanName,String className) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
		Class clazz = Class.forName(className);
		boolean contains = singleBeanMap.keySet().contains(beanName);
		//保证bean对象的单例创建
		if (!contains) {
			synchronized (IocFactory.class) {
				if (!contains) {
					Object newInstance = clazz.newInstance();
					singleBeanMap.put(beanName, newInstance);
				}
			}
		}
	}
	
	public static <T> T getBean(String beanName,Class<T> clazz) {
		if (!singleBeanMap.keySet().contains(beanName)) {
			System.out.println("IOC容器中不存在该"+beanName+"Bean");
		}
		T bean = (T)singleBeanMap.get(beanName);
		return bean;
	}
}
```

### IOC容器的实现接口：

spring提供两个接口ApplictionContext、BeanFactory，来分别实现IOC容器的功能，两者有一定的区别：

- ApplictionContext是BeanFactory的子接口，在实现BeanFactory的基础上，增加了：

  - 集成SpringAOP功能

  - 消息资源处理

  - 活动发布

  - 获取应用层特定上下文，如webApplicationContext

- BeanFactory进行Bean的实例化过程，默认是懒加载的，即加载配置文件时并不会创建对象，而是在进行依赖注入时，才会创建；ApplicationContext则默认是非懒加载，即加载配置文件时就会创建对象（可以通过Bean.xml设置是否延迟加载，即懒加载）

**一般情况下，BeanFactory用于spring在内部实现IOC容器基本功能，而对外推荐使用ApplictionContext，并且对于创建消耗资源较大的对象，使用ApplicationContext进行创建**

BeanFactory和ApplicationContext接口实现类：

ClassPathXmlAppliationContext和FileSystemXmlApplicationContext，前者读取项目class文件根目录路径下的xml文件；后缀读取操作系统文件路径下的xml文件

### IOC容器的Bean管理：

IOC容器的Bean管理，就是对spring的xml配置文件Bean标签的定义（也可以使用java注解），主要包含如下操作：

- Bean的命名：作为每个Bean在IOC容器的为唯一标识
- Bean的全类名：定义bean的实现类
- Bean的实例化：定义使用Bean的哪种构造方法，以及对应参数
- Bean的行为配置：声明Bean在IOC容器中的行为：作用域、生命周期的回调
- Bean实例化对象的属性注入：对IOC容器中创建好的实例对象，进行额外的属性注入

### XML方式的容器配置：

#### 1、Bean的配置

**springIOC容器，提供两种类型Bean的创建，普通Bean和工厂Bean**

##### 普通Bean：

特点：通过构造函数来实例化

```xml
<bean id="user" class="com.yh.User" name="userA;userB"></bean>
```

**bean标签的基本属性：**

| 属性  | 描述                                                         |
| ----- | ------------------------------------------------------------ |
| id    | bean的唯一标识符，用于获取IOC容器中指定的Bean；              |
| name  | bean的别名，也可以用于获取IOC容器中指定的Bean，可以通过；指定多个别名 |
| class | 指定Bean的实现类                                             |

**id和name的区别和作用情况：**

1、当id和name都不指定时，spring默认使用使用该Bean实现类的全类名作为id；出现多个相同类型的Bean没有指定时，则使用#number后缀来区分

2、当只指定name时，spring会将name的第一个值作为id使用

3、当出现多个Bean的name相同时，spring5版本会报错（之前版本不会，取最后创建的Bean，原因是：spring本质进行Bean注册时，对于出现相同id，会自动覆盖选取最后一个作为该id对应的bean；在spring5后，spring.xml会自动对重复name进行检测报错）

4、id和name可以同时指定，都可以进行Bean的获取

**一般情况下，推荐使用id进行Bean的命名，并且使用驼峰命名规则**

**bean标签的子标签：**

- property子标签：用于指定Bean实现类的属性注入，**需要保证Bean实现类提供该属性的set方法**


```xml
<bean id="user" class="com.yh.User">
	<property name="name" vlaue"yh"/>
    <property name="gread" ref="gread">
</bean>
```

- constructor-arg子标签：用于有参构造器创建，默认bean的实例化是使用无参构造器

```xml
    <bean id="user" class="com.yh.User"  name="userA;userB" >
        <constructor-arg name="name" value="yh"></constructor-arg>
        <constructor-arg index="1" value="123456"></constructor-arg>
    </bean> 
```

​		**constructor-arg有两种定义有参构造器的方式**：一、通过参数名选择，二、通过参数索引选择；spring通过对应的参数名或参数个数来选择满足条件的有参构造方法；但这样并不能确定唯一构造方法，因此可以选用type进行参数类型的指定

- property、constructor-arg属性注入：

  - value属性：用于注入基本数据类型属性

  - ref属性：用于注入引用数据类型属性，使用IOC容器中的实例对象

  - idref标签代替ref属性：可以在xml中有效验证进行属性注入Bean的id是否在IOC容器中存在
  
    ```xml
    <bean id="gread" class="com.yh.Gread" />
    
    <bean id="user" class="com.yh.User">
    	<property name="gread">
           <idref bean="gread"/>
    	</property>
  </bean>  
    ```

  - 内部bean代替ref属性：**其bean的作用域只在该对象实例属性中，因此不能够在全局IOC容器获取该Bean的实例对象，因此一般不进行bean命名**

    ```xml
    <property name="gread" >
        <bean  class="com.yh.Gread">
    		 <property name="id" value="1" />
    		 <property name="math" value="98" />
    		 <property name="english" value="100" />
        </bean>
    </property>
    ```
  - property的联级属性注入：
  
    需要保证嵌套的引用类型对象提前注入，否则会提示gread属性未null，无法执行set方法
  
    ```xml
    	<bean id="user" class="com.yh.User" name="userA;userB">
    		<property name="gread" ref="gread" />
    		<property name="gread.math" value="99" />
    	</bean>
    	<bean id="gread" class="com.yh.Gread" >
        </bean>
    ```
  
  - 特殊值注入（null、特殊符号）：null使用null标签；特殊符号使用xml转义字符完成
  
    ```xml
    <property name="name" vlaue"yh">
    	<null/>
    </property>
    ```
  
  - 容器对象属性注入（数组、list、map、properties），根据容器的泛型，既可以注入基本数据类型，又可以注入IOC容器中的引用数据类型
  
    ```xml
    <property name="arrayId">
    	<array>
    		<value>111</value>
    	</array>
    </property>
    
    <property name="listId">
    	<list>
    		<value>11</value>
    	</list>
    </property>
    
    <property name="listId">
     	<map> 
     		<entry key="1"><value>abc</value></entry>
    	</map>
    </property>  
    
    <property name="listId" >
     	<map> 
     		<entry key="1"><ref bean="user"/></entry>
    	</map>
    </property>  
    
    <property name="adminEmails">
        <props>
             <prop key="administrator">administrator@example.org</prop>
             <prop key="support">support@example.org</prop>
             <prop key="development">development@example.org</prop>
        </props>
    </property>
    ```

##### 工厂Bean：

特点：通过进行工厂方法实例化Bean

- 工厂静态方法实例化

  必须保证factory-method对应方法为静态方法

```xml
<bean id="user" class="com.yh.UserFactory" factory-method="createInstance"/>
```

```java
public class UserFactory {
	private static User user = new User();

	public static User createInstance() {
		return user;
	};
}
```

- 工厂方法实例化：

  使用IOC容器中的工厂Bean，调用其实例对象的工厂方法

```xml
<bean id="userFactory" class="com.yh.UserFactory"/>

<!-- 使用factory-bean时，需要保证class属性为空，即当前bean的类型，交给工厂Bean决定 -->
<bean id="user"
    factory-bean="userFactory"
    factory-method="createInstance"/>
```

```java
public class userFactory {
    private static User User = new User();

    public  User createInstance() {
        return clientService;
    }
}
```

**由于工厂类可以有多个工厂方法，因此一个工厂Bean可以配置创建多个产品Bean**

- 工厂Bean实例化产品对象时的参数注入：

  spring提供constructor-arg子标签，来提供工厂方法的参数

```xml
<bean id="user" class="com.yh.UserFactory" factory-method="createInstance">
    <constructor-arg name="name" value="yh111"></constructor-arg>
</bean
```

```java
public class UserFactory {
	private static User user = new User();

	public static User createInstance(String name) {
        user.setName(name);
		return user;
	};
}
```

##### Xml方式属性注入方式的简化:

- **引入P命名空间，简化xml中property标签的使用，实现属性注入时根据实现类的set方法，进行xml属性自动提示**
- **引入util命名空间，用于在IOC容器中配置集合Bean，来代替ListFactoryBean、MapFactoryBean、SetFactoryBean、PropertiesFactoryBean对象方式创建集合Bean**
- **引入C命名空间，简化constructor-arg标签的使用，实现在Bean标签中，使用内联属性进行构造函数参数的声明（spring3.1引入）**

#### 2、Bean的行为配置：

##### Bean的作用域：

spring提供Bean的六个作用域，通过Bean标签的属性Scope设置

IOC容器中所有的Bean默认是单例的

| 作用域                    | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| singleton（单实例）       | 默认值，针对于IOC容器Bean的实例化，只创建一个对象；和设计模式中的单例模式有所区分，其作用域是在spring容器中；而单例模式是针对于整个应用 |
| prototype（原型，多实例） | 针对于IOC容器Bean的实例化，每次实例化都会创建一个新的对象    |
| request                   | 作用域在当前web应用的request请求中，即每个request请求都会创建一个Bean的新实例 |
| session                   | 作用域在当前web应用的session中，即每个session都会创建一个Bean的新实例 |
| application               | 作用域在当前web应用中，整个web应用启动后，只会创建一个Bean的实例对象 |
| websocket                 | 作用域在当前Websocket连接中，即每个Websocket连接都会创建一个新实例 |

request、session、application和websocket并不能在spring常规IOC容器中定义，否则会抛出IllegalStateException异常，未知bean作用域；它们只用于springMVC中的IOC容器

- singleton和protoype作用域的使用场景和注意事项：

  singleton：适合于无状态Bean，即Bean实例化对象不保存当前线程使用的数据，线程安全，如DAO层（数据方法对象）、service层（业务逻辑对象）

  protoype：适合于有状态Bean，每个Bean实例化对象都保存了当前线程的数据信息，伴随调用者的消亡而消亡；并且在被IOC容器创建后，就不会被IOC容器管理，因此其Bean定义的销毁方法也不会被执行，需要开发者手动进行原型Bean中的资源释放

- 具有原型Bean依赖关系的单例Bean：

  此时IOC容器只会创建一个原型Bean实例，提供给单例Bean使用，导致原型Bean的作用域失效；

  解决方式：可以通过手动方法注入的方式，实现原型Bean实例的获取，放弃IOC容器的依赖注入

  ```java
  
  public class CommandManager implements ApplicationContextAware {
  
      private ApplicationContext applicationContext;
  
      public Object process(Map commandState) {
          Command command = createCommand();
          command.setState(commandState);
          return command.execute();
      }
  
      protected Command createCommand() {
          return this.applicationContext.getBean("command", Command.class);
      }
  
      public void setApplicationContext(
              ApplicationContext applicationContext) throws BeansException {
          this.applicationContext = applicationContext;
      }
  }
  ```

##### Bean的生命周期：

IOC容器能够实现Bean生命周期的方法回调：

- 初始化回调：在Bean实例化后（**属性注入之后**），实现无参初始化方法的回调

  ```xml
  <bean id="user" class="com.yh.User" init-method="init" />
  ```

  ```java
  	public void init() {
  		System.out.println("User对象被创建");
  	}
  ```

- 销毁回调：在Bean被销毁之前，实现无参销毁方法的回调

  ```xml
  <bean id="user" class="com.yh.User" init-method="init" />
  ```

  ```java
  	public void destroy() {
  		System.out.println("User对象被销毁");
  	}
  ```

- 默认全局初始化和销毁方法：在Beans根标签中设置，bean的自身属性会进行覆盖

  ```xml
  <beans default-init-method="init" default-destroy-method="destroy">
  ```

- IOC容器的关闭：当spring整合web应用时，会基于web应用的ApplicationContext来实现web应用关闭时，正常关闭springIOC容器；而对于客户端应用，则需要手动调用ConfigurableApplicationContext的registerShutdownHook方法，进行IOC容器的关闭

##### Bean之间的依赖关系：

​		spring在创建IOC容器时，会验证每一个Bean的配置，并根据bean的加载方式，选择性直接加载所有的非懒加载的Bean（单例Bean默认非懒加载），然后根据Bean直接的依赖关系，来确定所有Bean的实例化顺序；如bean A依赖于bean B时，IOC容器则会在调用bean A的**setter方法或构造方法**之前，完成bean B的配置

- 循环依赖：

  ​		当Bean使用构造方法进行参数注入时，可能会出现循环依赖问题：类A构造方法参数需要类B实例，类B构造方法参数需要类A的实例；此时spring就无法进行Bean的创建，抛出BeanCurrentlyInCreationException异常，因此对于存在循环依赖的Bean，需要使用setter方法进行注入

- 延迟Bean的初始化：**延迟并不会影响bean的注册**

  默认情况下，applicationContext会积极创建和配置单例Bean，从而检测Bean配置的正确性

  - 对于单例作用域的Bean，默认使用非懒加载的的方式创建对象

  - 对于多例作用域的Bean，默认使用懒加载方式创建对象
  - 对于web作用域的Bean，会根据web作用域对象的创建而加载

  通过Bean标签的lazy-init属性，可以控制是否进行延迟加载（使用时再创建Bean实例）；但是由于Bean的依赖关系，当非延迟单例Bean初始化创建依赖延迟单例Bean时，则延迟的单例Bean会被迫实例化创建

  ```xml
  <bean id="lazy" class="com.yh.User" lazy-init="true"/>
  ```

- 强制指定Bean的初始化顺序：

  一般情况下，我们可以依赖于spring来完成Bean初始化顺序的确定，但是对于spring还是无法解决一种特殊情况，当Bean依赖关系不太直接时，spring就无法严格要求它们的初始化顺序，如类A需要使用到类B的某个属性，但类B的属性有效值，需要通过类C初始化方法进行重新赋值；此时类A的正确性就需要类C优先被初始化来保证，但IOC容器中无法明确它们的依赖关系；

  通过Bean标签的depends-on属性，可以显式强制实现进行当前Bean的实例化前，spring要保证指定的其他Bean要完成实例化

  ```xml
  <bean id="user" class="com.yh.User" depends-on="gread"/>
  <bean id="gread" class="com.yh.Gread" />
  ```

#### 3、自动装配:

1、基于setter方法的自动装配（在不指定有参构造方法的情况下，默认使用默认构造方法，当没有默认构造方法就会报错）

IOC容器提供自动装配的方式，进行Bean的属性依赖注入，即自动注入当前Bean中所有提供setter方法属性，然后通过属性名或者属性类型自动注入IOC容器中相对应的Bean实例

- 通过属性名自动装配：寻找IOC容器中bean id和属性名相同的Bean实例，进行注入

- 通过属性类型自动装配，寻找IOC容器中bean class和属性类型相同的Bean实例，进行注入

```xml
<Bean id="user" class="com.yh.User" autowire="byNmae"/>
<Bean id="user" class="com.yh.User" autowire="byType"/>
    
<Bean id="gread" class="com.yh.Gread"/>    
```

当IOC容器中，没有与之匹配的Bean时，并不会报错；对于byName，由于Bean id的唯一性，因此只可能匹配到一个Bean；对于byType，可能有多个相同类型的Bean，此时就会报错

2、基于构造方法的自动装配

```xml
<Bean id="user" class="com.yh.User" autowire="constructor"/>
    
<Bean id="gread" class="com.yh.Gread"/>    
```

和byType类型，通过寻找IOC容器中与之匹配的Bean实例，作为构造方法形参注入；当存在多个构造方法，spring会对进行一定策略的选择

3、基于构造方法的自动装配时，构造方法的选择：

- spring首先会获取当前Bean的所有构造方法，然后对它们进行排序放在集合中：参数多的在前面，权限修饰符广的在前面（public在前面）

- 对于参数个数和权限修饰符相同的两个构造方法，则会比较构造方法参数类型的差异性（一般会减少这种情况的发生）

- 然后对该集合进行遍历，找到第一个所有参数都满足IOC容器依赖注入的构造方法，此时就是用该构造方法进行自动装配

  因此当存在多个构造方法并且可以实现所有参数依赖注入时，spring优先选择参数个数多的、权限大的构造方法

4、自动装配的缺点：

- 自动装配无法完成简单数据类型属性的注入

- 使用property、constructor-arg手动装配，会使自动装配失效，因此无法实现两钟方式同时使用

- 存在多个匹配项时，自动装配会报错，当然可以使用Bean标签的primary,来确定以类型匹配的主键Bean

  ```xml
  <Bean id="gread" class="com.yh.Gread" primary="true"/> 
  ```

#### 4、Context命名空间功能：

##### 开启IOC容器基本注解

```xml
    <context:annotation-config />
```

此时IOC容器中，会自动创建如下Bean：

| Bean类型                                 | 作用                                        |
| ---------------------------------------- | ------------------------------------------- |
| internalConfigurationAnnotationProcessor | 注册解析**@Configuration注解**的后置处理器  |
| internalAutowiredAnnotationProcessor     | 处理带有**@Autowired注解**Bean的后置处理器  |
| internalCommonAnnotationProcessor        | 处理带有**JSR-250规范注解**Bean的后置处理器 |
| internalEventListenerProcessor           | 处理监听带有**方法注解**Bean的后置处理器    |
| internalEventListenerFactory             | 事件监听工厂                                |

##### @Component注解扫描开启

```xml
	<context:component-scan base-package="com.yh" />
```

使指定包下的@Component及其子注解@Service、@Reposity、@Controller生效，从而使用注解的方式注册Bean；同时也会隐式的开启 <context:annotation-config />配置，即开启IOC容器基本注解

并且spring提供`<context:include-filter />`或 `<context:exclude-filter />`子标签，来对自定义扫描路径下的类进行过滤，其标签属性为：type、expression；一般只需要对MVC进行类型过滤，因此使用type=annotation,expression为需要过滤注解的全类名

- 白名单：spring提供一个默认过滤器，它会自动对所有含义@Component注解及其子注解的类进行Bean注册，这样就起不到白名单的作用，因此需要使用use-default-filters="false"进行关闭

```xml
	<context:component-scan base-package="com.yh"  use-default-filters="false">
	   <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
	</context:component-scan>
```

- 黑名单：

```xml
	<context:component-scan base-package="com.yh"  >
	   <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
```

##### 外部properties文件导入

需要使用context命名空间,来引入class根目录下的jdbc.properties文件，之后使用${} 标识符，来进行properties文件的属性注入，如JDBC连接四要素

```xml
    <context:property-placeholder location="classpath:jdbc.properties">
```

同时IOC容器中，会自动创建一个PropertySourcesPlaceholderConfigurer类型Bean，用于@Value外部属性注入, 此时不允许对于peroperties文件不存在该值

### java注解方式的容器配置：

在spring3.0后，就提供了java注解的方式代替xml配置

#### 1、Bean的注册：

##### @Component及其MVC模型分类的子注解

spring提供@Component注解，来简化Bean的注册

 ```java
@Component
public class User{
    
}
 ```

@Component是一个元注解，元注解可以提供给其他注解定义时使用，从而实现组合注解；spring通过MVC模型设计，提供了额外特定组件类的Bean注册的注解：

​		@Repository、@Service和@Controller，spring为它们提供了额外处理，如用于识别AOP的切入点等；因此在MVC模型的类Bean注册时，推荐使用这三个注解；而@Component则用于常规类的Bean注册

- @Component及其MVC模型分类的子注解默认使用类名的驼峰命名方式，作为Bean注册的id；可以通过其value属性修改

##### @Bean

spring使用@Component、@Bean来实现工厂bean的注册，可以定义在类的实例方法和类方法中：

 ```java
@Component
public class UserFactory {
	
	@Bean
	public static User getUser() {
		User user = new User();
		user.setName("yh123");
		return user;
	}

	@Bean
	public  User createInstance(@Value("${name}")String name,@Qualifier("user")User user) {
		System.out.println("zheshiwode1"+user.getName());
		user.setName(name);
		return user;
	};
}
 ```

- @Bean注解的方法，其参数会默认使用IOC容器进行依赖注入（类似于方法上添加了@Autowired注解，在当前类进行Bean注册后，执行该方法创建工厂Bean），因此可以使用@Qualifier来指定唯一的Bean
- @Bean默认使用方法名作为Bean注册的name；可以通过其name属性修改（value属性为name属性的别名，用于作为name属性的默认值，因此不能和name属性同时使用）；**并且会选择name中的第一个元素，作为Bean的id**
- @Bean注解提供initMethod、destroyMethod属性，指定该Bean生命周期的回调方法

##### 注解方式下对于id重复的处理：

​		对于@Component及其子注解，spring会自动对其value值（bean注册的id）进行唯一性检测；而对于@Bean，spring允许出现多个相同id，但只会选取一个进行注册

#### 2、Bean的属性注入：

**spring 提供@Autowired、@Qualifier、@Value来完成Bean的属性注入**

##### @Autowired（@Qualifier、@Primary）

可以注解在构造方法、普通方法和成员变量上，来完成Bean的自动装配

```java
@Component
public class User {

    @Autowired
    private Gread gread;
    
    @Autowired
    public User(Gread gread){
     	this.gread=gread;   
    }
    
    @Autowired
    public void setGread(Gread gread) {
        this.gread=gread; 
    }
}
```

- 通过三种方式来实现IOC容器的自动装配：

1、构造方法：当User类被IOC容器创建时，则会使用User构造方法，将IOC容器中的Gread实例作为参数属性注入

2、普通方法：当User类被IOC容器创建后，会继续执行被@Autowired的普通方法，将IOC容器中的Gread实例作为参数，将进行属性注入

3、成员变量：@Autowired注解在成员变量上时，**不需要为该类添加当前成员变量的setter**；当User类被IOC容器创建后，会将IOC容器中的Gread实例，**使用反射的方式进行属性注入**

**@Autowired不能注解在静态成员变量和静态方法上，否则spring进行错误提示，无法起作用；而对于静态成员变量的属性注入，只能通过手动编写注入方法（普通方法、构造方法）并添加@Autowired注解来完成**

- @Autowired有一个唯一属性required，默认为true；当自动装配在IOC容器中没有查询到可以匹配的bean时，会抛出异常；而当设置required=false时，就会忽略异常不进行当前的自动装配

- @Autowired自动装配默认使用byType，当IOC容器中存在多个bean时，**则默认使用参数名/变量名指定注入Bean的唯一id，如果没有匹配则报错存在多个同类型Bean**；我们也可以通过**@Qualifier**注解来，手动指定注入Bean的唯一ID；或者在多个相同类型Bean中，选择一个Bean作为默认值，添加**@Primary**注解

  ```JAVA
     	@Qualifier("gread")
  	@Autowired
      private Gread gread;
      
      @Autowired
      public User(@Qualifier("gread") Gread gread){
       	this.gread=gread;   
      } 
  ```

- 自动装配时，构造方法的选择：
  1. 当不使用@Autowired不指定构造方法时：
     - 只有一个构造方法：则直接使用该构造方法
     - 有多个构造方法：则使用默认构造方法（无参构造方法），没有就会报错
  2. 当使用@Autowired指定构造方法时：
     - 只指定一个构造方法：则使用该构造方法；当设置required为false时，该构造方法参数无法通过IOC容器依赖注入时，则会使用默认构造方法
     - 指定了多个构造方法：必须保证所有@Autwired属性required为false，即允许当前构造方法匹配不到Bean，这样spring就在所有注解的构造方法中，选择一个构造方法来实例化Bean（**选择策略和xml基于构造方法自动装配的一致**）

- @Autowired的额外功能：获取容器中所有当前类型的Bean实例，使用数组、集合进行保存

  ```java
  @Component
  public class MovieRecommender {
  
      @Autowired
      private MovieCatalog[] movieCatalogs;
      
      @Autowired
      private Set<MovieCatalog> movieCatalogs;
      
      @Autowired
      private Map<String, MovieCatalog> movieCatalogs;
      
      // ...
  }
  ```

  对于Map存储，key为当前Bean的id、value为当前Bean实例对象；所有集合元素的排序，遵循它们的注册顺序，但是@Priority注解的Bean会默认排序第一位；而@Order注解，可以修改注册优先级，但并不会影响Bean的实例化顺序（注意：@Order不能注解在方法中，因此无法改变工厂bean的注册优先级）

- spring可以通过@Autowired注解，直接进行springIOC容器API接口的实例化创建，如`BeanFactory`，`ApplicationContext`，`Environment`，`ResourceLoader`， `ApplicationEventPublisher`，和`MessageSource`。这些接口及其扩展接口（例如`ConfigurableApplicationContext`或`ResourcePatternResolver`）

  ```JAVA
  	@Autowired
  	public  ApplicationContext context;
  ```

##### @Value

@Value可以注解在带有@Autowired注解的方法中、成员变量中，用于字面量的属性注入；**在注解在成员变量上时，不需要set方法，会使用反射的方式进行属性注入**

- ${}方式：

  ```java
  @Value("${name}")
  private String name;
  ```

  1. 首先引入外部properties文件
  2. 在指定方法参数或成员变量上，添加@Value注解
  3. spring会自动通过注解值，在所有外部properties文件中，选择该key；找到则进行自动注入，找不到则使用注键值的字面量（即name）

  如果需要严格控制不存在的值时，则需要声明一个**PropertySourcesPlaceholderConfigurer**bean，此时如果找不到，则会在spring初始化时报错

  ```java
  @Configuration
  public class SpringConfig {
  
       @Bean
       public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
             return new PropertySourcesPlaceholderConfigurer();
       }
  }
  ```

- #{}方式，使用SpEL表达式，进行动态属性注入

  通过SpEL表达式，可以是实现复杂数据结构对象的属性注入

  ```java
  @Value("#{{'math':100,'english':99}}")  
  private  Map<String, Integer> countOfMoviesPerCatalog;
  ```

@Value和@Autowired一样，也无法注解在静态变量和静态方法上，因此静态变量的属性注入，可以通过set方法来完成

#### 3、JSR-250规范注解

##### @Resource

作用和@Autowired+@Qualifier一致，用于依赖注入Bean，指定Bean的id；使用成员变量类型名、方法参数名作为默认值

```java
public class SimpleMovieLister {

    @Resource
    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

##### @PostConstruct和@PreDestroy

定义Bean生命周期的回调方法：在IOC容器进行Bean生命周期回调处理时，会在指定时间点，自动执行带这些注解的方法，**即该注解可以和xml中的Bean生命周期回调同时存在并执行**

```java
public class CachingMovieLister {

    @PostConstruct
    public void init() {
        
    }

    @PreDestroy
    public void destroy() {
       
    }
}
```

- 它们和xml中Bean生命周期回调方法配置一样， @PostConstruct在Bean完成初始化（包括属性注入）后执行

#### 4、其他IOC容器注解（不常用）：

##### @Required、@Scope、@Lazy、@DependsOn

| 注解       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| @Required  | 和@Autowired的required属性作用类似；默认为true，表示当前属性注入一定要成功完成；但只用于注解在setter方法上，搭配xml配置方式使用，表示进行该类的xml方式的Bean注册时，必须进行当前setter方法属性的注入 |
| @Scope     | 和@Component、@Bean一起使用，定义Bean注册时的作用域，默认为singleton |
| @Lazy      | 和@Component、@Bean一起使用，使bean的初始化延迟，getBean时才进行加载，默认为true |
| @DependsOn | 和@Component、@Bean一起使用，定义该bena的初始化，要在指定bean完成之后进行 |

##### JSR-330注解

| 注解                  | 等效spring注解      |
| --------------------- | ------------------- |
| @Inject               | @Autowired          |
| @Named / @ManagedBean | @Component          |
| @Singleton            | @Scope("singleton") |
| @Qualifier / @Named   | @Qualifier          |

#### 5、基于全注解的spring配置

##### @Configuration

**spring提供@Configuration注解，来声明配置类，完全代替spring.xml，并开启spring注解功能；**

@Configuration也是@Component的子注解，可以配合@Bean进行bean注册；

**applicationContext的获取方式此时也就发生了改变：**

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);  
}
```

使用AnnotationConfigApplicationContext类，来实现applicationContext接口，并且可以通过非参构造方法，读取多个配置类；**或者使用@Import注解，在一个配置类上加载其他配置类（即当前为主配置类）**

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class);
    ctx.register(OtherConfig.class,AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);  
}
```

此时xml中的一些常规配置就需要交给配置类完成，spring提供如下注解，来实现对于功能：

##### @ComponentScan

用于定义spring@Component注解的扫描路径

常用属性有：

| 属性名            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| basePackages      | 定义扫描路径，为string[]，可以定义多个路径                   |
| value             | basePackages属性别名，作为默认值                             |
| excludeFilters    | 黑名单过滤器                                                 |
| includeFilters    | 白名单过滤器                                                 |
| useDefaultFilters | 是否使用默认过滤器；默认为true，注册路径中所有带有@Component及其子注解的类 |

```java
//如：排除controller注解的类
@ComponentScan(basePackages= {"com.yh"},excludeFilters= {
		@Filter(type=FilterType.ANNOTATION,classes= {Controller.class})})
```

##### @PropertySource

用于加载外部properties文件，进行外部属性注入

常用属性：

| 属性名                 | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| name                   | 资源命名（一般不需要）                                       |
| value                  | 外部资源路径                                                 |
| encoding               | 编码格式，默认使用项目编码格式                               |
| ignoreResourceNotFound | 是否忽略找不到指定值；默认false，不忽略                      |
| factory                | 定义PropertySources的生成规则，默认PropertySourceFactory.class |

**ignoreResourceNotFound，一般使用默认值，降低异常发生时的排除难度，但还需要搭配PropertySourcesPlaceholderConfigurer一起使用，才能够正确的抛出异常**

### IOC容器源码解析

- 初始化spring环境（IOC容器）

  ```java
  AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(springTest.class);
  ```

  - 调用其子类**GenericApplicationContext**默认构造方法,初始化一个**beanFactory**

    ```java
    this.beanFactory = new DefaultListableBeanFactory();
    ```

    beanFacotry中初始化了多个重要属性，用于IOC容器的实现；如IOC容器的Bean定义池：**beanDefinitionMap**

  - AnnotationConfigApplicationContext自身默认构造方法，初始化**reader**、**scanner**成员变量

    ```java
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
    ```

    - AnnotatedBeanDefinitionReader初始化,创建**注解模式下的Bean定义注册器**

      ```java
      //保存ApplicationContext的引用	
      this.registry = registry;
      //创建@Conditionl注解处理器，用于条件判断Bean是否被注册（spring4添加的新注解）
      this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
      //注册spring自身的注解配置处理器	AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
      ```

    - ClassPathBeanDefinitionScanner初始化,创建**类路径扫描的Bean定义注册器**

      ```java
      	//保存ApplicationContext的引用	
      	this.registry = registry;
      	//注册默认规则的扫描器
      	if (useDefaultFilters) {
      		registerDefaultFilters();
      	}
      	//设置外部环境对象
      	setEnvironment(environment);
      	//设置外部资源加载器
      	setResourceLoader(resourceLoader);
      ```

      默认扫描器的注册：

      ```java
      //添加@Component注解的扫描器		
      this.includeFilters.add(new AnnotationTypeFilter(Component.class));
      
      //添加JSR-250中@ManageBean注解扫描器和JSR-330中@Named注解扫描器
      ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
      try {
      	this.includeFilters.add(new AnnotationTypeFilter(
      					((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
      			logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
      		}
      		catch (ClassNotFoundException ex) {
      			// JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
      		}
      		try {
      			this.includeFilters.add(new AnnotationTypeFilter(
      					((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
      			logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
      		}
      		catch (ClassNotFoundException ex) {
      			// JSR-330 API not available - simply skip.
      		}
      ```

  - 调用有参构造方法，注册配置类，并刷新ApplicationContext

    ```java
    	register(componentClasses);
    	refresh();
    ```

    - 使用AnnotatedBeanDefinitionReader进行配置类的Bean注册

      ```java
      private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
      			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
      			@Nullable BeanDefinitionCustomizer[] customizers) {
      
      		//创建IOC容器用于存储bean信息的重要组件BeanDefinition    
      		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
          	//是否存在@Conditionl注解，并满足条件不进行Bean注册
      		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
      			return;
      		}
      
          	//设置BeanDefinition对象中的一系列属性（作用域、懒加载、beanName、Primary、qualifier）  
      		abd.setInstanceSupplier(supplier);
      		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
      		abd.setScope(scopeMetadata.getScopeName());
      		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));
      
      		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
      		if (qualifiers != null) {
      			for (Class<? extends Annotation> qualifier : qualifiers) {
      				if (Primary.class == qualifier) {
      					abd.setPrimary(true);
      				}
      				else if (Lazy.class == qualifier) {
      					abd.setLazyInit(true);
      				}
      				else {
      					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
      				}
      			}
      		}
      		if (customizers != null) {
      			for (BeanDefinitionCustomizer customizer : customizers) {
      				customizer.customize(abd);
      			}
      		}
      
          	//将BeanDefinition对象put到factroyBean的beanDefinitionMap中
      		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
      		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
      		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
      	}
      ```

    - 在注册所有配置类后，刷新spring上下文环境

      



@Component、@Conguration和@Bean的区别

好像配置信息注解，放在@Component类上，也可以生效

bean的后置处理器 BeanPostProcessor：定义初始化方法前后的执行代码

springIOC容器的Aware

ApplicationContextAPI的使用

IoC容器，事件，资源，i18n，验证，数据绑定，类型转换，SpEL，AOP

## spring核心组件：

### 资源加载：

- spring提供了Resource接口，重新定义了java中各种资源加载的API，功能更加强大，且使用方便；其实现类包括：

  | 类名                   | 作用                                                         |
  | ---------------------- | ------------------------------------------------------------ |
  | UrlResource            | 对java.net.URL进行了包装，通过不同前缀，来进行不同类型的资源访问 |
  | ClassPathResource      | 从当前类路径中获取资源                                       |
  | FileSystemResource     | 从文件系统中获取资源                                         |
  | ServletContextResource | 从web应用根目录中获取资源                                    |
  | InputStreamResource    | 通过一个输入流来创建资源对象                                 |
  | ByteArrayResource      | 通过一个字节数组来创建资源对象                               |

- UrlResource，前缀和访问资源类型的匹配策略：

  | 前缀           | 访问资源类型          |
  | -------------- | --------------------- |
  | classpath：    | 类路径                |
  | file：         | 文件系统              |
  | https：/http： | URL                   |
  | 无             | 根据ReourceLoader决定 |

- ReourceLoader接口资源加载器

  ApplicationContext就实现了这个接口，根据不同了Reource实现类，加载特定的Resource资源

- spring提供ResourceLoaderAware接口，用于获取ApplicationContext中的ReourceLoader,然后重新设置ReourceLoader

### 

## 3、AOP

### AOP概念

- AOP的定义：

  ​		面向切面编程，提供了另一种编写程序结构的方式，是对面向对象编程（OOP）的补充

- AOP的作用：

  ​		AOP可以对业务逻辑的各个部分进行隔离，使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，提升开发效率；因此也就能实现，在不改变原来业务代码的基础上，添加新功能，并且可以通过配置随时删除

- AOP的实际应用：

  ​		安全控制、事务处理、异常处理、日志打印、性能统计等

- AOP在spring中的实现应用：
  1. 提供声明式的企业服务，如spring的声明式事务管理
  2. 为用户提供切面编程开发手段，即spring aop

### AOP底层原理

- AOP底层原理技术：动态代理

- 两种动态代理：JDK动态代理、CGLIB动态代理

  - JDK动态代理：创建接口实现类的代理对象，从而增强实现类的方法

    代码实现过程如下：

    通过JDK提供的java.lang.reflect.Proxy对象，来创建singerL类的代理对象，参数需要singerL的类加载器、singerL类的接口和java.lang.reflect.InvocationHandler接口的实现类，在实现InvocationHandler接口中，编写方法增强代码;在执行Method.invoke方法时，还需要singerL实例对象

    ```java
    		//动态代理
    		Singer singerProxy1 = (Singer)Proxy.newProxyInstance(singerL.getClass().getClassLoader(),
    				new InvocationHandler() {
    					@Override
    					public Object invoke(Object proxy, Method method, Object[] args){
    						System.out.println("进行动态代理1");
    						Object invoke = method.invoke(singerL, args);
    						System.out.println("进行动态代理2");
                            //invoke为singerL实现类的当前方法的返回值；当return null时，代理类执行返回值就为null
    						return invoke;
    					}
    				});
    		singerProxy1.sing();
    ```

    InvocationHandler接口的invoke方法参数：proxy 生成的代理对象；method 目标类方法对象； agrs 目标类方法参数

    method.invoke(singerL, args) 直接执行目标类对象方法

  - CGLIB动态代理：创建当前类子类的代理对象，从而增强当前类的方法

    代码实现过程如下：

    - 首先需要导入额外依赖包：cglib（在spring-core中已经包含了该工具包）

    创建一个enhancer对象，用于设置动态代理的目标类、方法拦截器；最后在生成以目标类作为父类继承的子类代理类

    ```java
    		Enhancer enhancer = new Enhancer();
    		enhancer.setSuperclass(SingerL.class);
    		enhancer.setCallback(new MethodInterceptor() {
    			@Override
    			public Object intercept(Object Proxy, Method method, Object[] arg, MethodProxy methodProxy) throws Throwable {
    				System.out.println("进行动态代理1");
    				Object invokeSuper = methodProxy.invokeSuper(Proxy, args);
    				System.out.println("进行动态代理2");
    				return invokeSuper;
    			}
    		});
    		
    		SingerL singerLProxy = (SingerL)enhancer.create();
    		singerLProxy.sing();
    ```

    MethodInterceptor接口的intercept方法参数：Proxy 生成代理对象、method 目标类的方法对象、arg 方法参数、methodProxy 代理类的方法对象

     methodProxy.invokeSuper(Proxy, args) ：使用代理对象来执行其父类方法（即目标类的方法）

- spring动态代理方式的选择：

  spring默认使用JDK动态代理，由于JDK动态代理需要基于目标类的接口来实现，因此要确保目标类有对应的接口；当没有接口时，spring则使用CGlib动态代理的方式实现

- springAOP在使用动态代理实现切面编程中，是基于IOC容器的依赖注入，实现动态代理类的创建和使用的；**因此在使用AOP时，需要保证被增强类和增强类都被IOC容器管理**

### AOP术语

- 连接点：程序运行中，被AOP代理执行的所有方法，即可以被增强的方法

- 切入点：表示实际被真正增强的方法

-  增强/通知：用于方法增强的代码；通知可以分为多种类型：前置通知、后置通知、环绕通知、异常通知、最终通知

  | 通知类型 | 代码执行时间点                                               |
  | -------- | ------------------------------------------------------------ |
  | 前置通知 | 在原方法执行前                                               |
  | 后置通知 | 在原方法执行后                                               |
  | 环绕通知 | 在原方法执行前后                                             |
  | 异常通知 | 当原方法出现异常时                                           |
  | 最终通知 | 类似于JAVA异常中的finally，无论方法是否发生异常，都会在最后被执行 |

- 切面：将通知配置到切入点上的过程

### AOP的使用

#### AOP相关依赖

- springIOC容器和AOP是虽然是两个不同的模块，但是springAOP的使用依赖于IOC容器，因此需要spring核心容器的相关依赖：spring-beans、spring-core、spring-context、spring-expression

- 同时springAOP容器使用还需要：

  spring-aop（spring-context会将其依赖导入）：提供spring对AOP的实现

  spring-aspects（会依赖导入aspectjweaver包）：提供spring对AspectsJ框架配置驱动的整合；aspectjweaver则就是支持aop相关注解和切入点表达式

#### AOP切入点表达式

作用：用于定义需要对哪个类的哪个方法进行增强

execution（[权限修饰符]    [返回类型]    [全类名].[方法名] ([参数列表])）  

可以通过*   表示匹配任意值，例子：

```java
execution(public void yh.com.User.setUser(String id))   
```

#### springAOP的声明方式：

##### springAOP提供三种声明方式：

1. xml
2. java注解
3. AOP api

- 对于XML和java注解，spring是利用AspectJ框架中配置引擎来完成的，但是在实现上并不依赖于AspectJ编译器，而是使用动态代理完成；而AOP api 是提供了spring在实现AOP的一系列接口，从而以代码编程的方式来完成，一般不推荐使用

##### AspectJ和springAOP的区别：

- AspectJ是一个AOP框架，它扩展了java语言，定义了AOP语法。它的底层实现是需要使用一个专门的编译器，解读AOP语法，然后在目标类中进行切面代码的织入，生成class文件；称之为静态织入
  - 缺点：需要专门的编译器ajc；新增增强代码时，需要重新编译被增强目标类
  - 优点：从class文件上，就进行了AOP的实现，性能更好；支持多个所有级别的切入点，AOP能力更加强大
- springAOP利用动态代理，来在运行期实现目标类的增强；在ioc容器初始化Bean时，创建动态代理类进行依赖注入；因此称之为动态织入
  - 缺点：依赖于springIOC容器，由于是动态代理实现方法增强，因此性能不如AspectJ；只支持方法级别的切入点
  - 优点：纯java代码实现，无需额外编译，使用更加简单

#### 基于AspectJ注解的AOP

- 在配置类中添加@EnableAspectJAutoProxy

  - @EnableAspectJAutoProx：启用@AspectJ的所有注解

- 编写一个增强类，用于提供切面编程的逻辑代码，并添加@Aspect、@Component

  - @Aspect：声明该类为一个切面类，提供切面编程的逻辑代码
  - @Component：springAOP注解的作用，是基于ICO容器Bean进行管理的，因此@Aspect注解需要被IOC容器管理，才能够生效

- 使用**通知注解** 注解在增强类的方法上，并配置切入点

  | 通知注解        | 对应通知类型 |
  | --------------- | ------------ |
  | @Before         | 前置通知     |
  | @AfterReturning | 后置通知     |
  | @Around         | 环绕通知     |
  | @AfterThrowing  | 异常通知     |
  | @After          | 最终通知     |

```java
@Component
@Aspect
public class SingerProxy {
	
	@Before("execution(public String com.yh.aop.SingerL.sing()) ")
	public void before() {
		System.out.println("sing前");
	};
    
    @AfterReturning("execution(public String com.yh.aop.SingerL.sing()) ")
	public void afterReturning() {
		System.out.println("sing后");
	};
}
```

- 通过@Pointcut可以实现**公共切入点的声明**，实现切入点声明代码的复用，直接使用@Pointcut注解的**方法签名**作为**通知注解的value属性值**

```java
@Component
@Aspect
public class SingerProxy {
    
    @Pointcut("execution(public String com.yh.aop.SingerL.sing())")
    public void sing(){};
	
	@Before("sing()")   
	public void before() {
		System.out.println("sing前");
	};
    
    @AfterReturning("sing()")
	public void afterReturning() {
		System.out.println("sing后");
	};
}
```

- 通过@Order可以设置增强类在IOC容器中注册的优先级，从而也就定义了目标动态代理类中相同切入点，增强代码的优先级

  原则：优先级越高，增强代码越靠近目标源代码，原理为**动态代理的迭代**

  - 原代码

  ```java
  	public String sing() {
  		System.out.println("singerL实现类");
  		return "sing";
  	}
  ```

  - 最高优先级的增强类方法，进行切入点的动态代理

  ```java
  	public String sing() {
          system.out.println("sing前A")
  		System.out.println("singerL实现类");
          system.out.println("sing后A")
  		return "sing";
  	}
  ```

  - 最高优先级的增强类方法，进行切入点的动态代理

  ```java
  	public String sing() {
          system.out.println("sing前B")
          system.out.println("sing前A")
  		System.out.println("singerL实现类");
          system.out.println("sing后A")
          system.out.println("sing前B")
  		return "sing";
  	}
  sys
  ```

  因此可以得知：**优先级越高，Before越后执行，after越先执行**

  **在同一个增强类中，相同切入点，会根据方法名来定义其优先级，无法使用@Order进行控制**

## 4、springMVC

## 4、jdbcTemplate

## 5、事务管理器

## 6、spring5的新特性