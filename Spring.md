# Spring Framework

## 1、spring概念

​		spring是一个轻量级的开源JAVAEE框架，用于解决企业应用开发的复杂性。有两个核心部分：

- IOC：控制反转，将创建对象的过程交给spring进行管理
- AOP：面向切面编程，通过动态代理，将一些模块化代码分离，不写入源代码

### spring特点：

- 方便解耦；自动依赖注入，从而简化开发
- 面向切面编程，使开发者专注于业务逻辑代码的编写
- 支持对各种框架的集成
- 支持各种JAVAEE规范，并提供更加简化的API

### spring模块划分：

spring是一个轻量化框架，通过多个模块来给开发者提供自己应用程序所需要的功能，实现JAVAEE平台的所有规范。spring主要分成六个模块：

- spring核心容器
- 面向切面编程
- Instrumentation和Messaging（使用比较少）
- 数据访问于集成：简化数据访问操作，提供封装好操作的相应模板；同时提供整合orm框架的能力，并支持事务管理
- Web和远程调用：支持Web开发和其他远程调用交互方案
- spring测试：提供单元测试模块

![](C:\Users\OneMTime\Desktop\Typora图片\spring模块架构.png)

### spring核心容器模块：

核心容器模块就是spring框架的基础，用于整个spring容器的搭建和使用，也是整合所有其他模块的基础，它包括：

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

IOC容器的底层原理：

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

#### IOC容器的实现接口：

spring提供两个接口ApplictionContext、BeanFactory，来分别实现IOC容器的功能，两者有一定的区别：

- BeanFactory接口方法主要为获取Bean实例和别名、判断Bean的一些属性（作用域、类型）

- ApplictionContext是BeanFactory的子接口，在实现BeanFactory的基础上，增加了：

  - 集成SpringAOP功能

  - 消息资源处理

  - 活动发布

  - 获取应用层特定上下文，如webApplicationContext

- BeanFactory进行Bean的实例化过程，默认是懒加载的，即加载配置文件时并不会创建对象，而是在进行依赖注入时，才会创建；ApplicationContext则默认是非懒加载，即加载配置文件时就会创建对象（可以通过Bean.xml设置是否延迟加载，即懒加载）

**一般情况下，BeanFactory用于spring在内部实现IOC容器基本功能，而对外推荐使用ApplictionContext，并且对于创建消耗资源较大的对象，使用ApplicationContext进行创建**

BeanFactory和ApplicationContext接口实现类：

ClassPathXmlAppliationContext和FileSystemXmlApplicationContext，前者读取项目class文件根目录路径下的xml文件；后缀读取操作系统文件路径下的xml文件

#### IOC容器的Bean管理：

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

特点：通过构造函数来实例化，只进行Bean的声明，实例化过程交给IOC容器

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
  
    需要保证嵌套的引用类型对象提前注入，否则会提示gread属性为null，无法执行set方法
  
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

特点：通过进行工厂方法实例化Bean,将Bean的实例化过程显式声明出来

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

request、session、application和websocket作用域并不能在spring常规IOC容器中定义，否则会抛出IllegalStateException异常，未知bean作用域；它们只用于springMVC中的IOC容器

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

- 具有单例Bean依赖关系的原型Bean

  在每次实例化创建时，都会使用同一个单例Bean对象，进行依赖注入

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

  - 对于多例作用域的Bean，默认使用懒加载方式创建对象（**非懒加载对多例bean没有意义**）
  - 对于web作用域的Bean，会根据web作用域对象的创建而加载

  通过Bean标签的lazy-init属性，可以控制是否进行延迟加载（使用时再创建Bean实例）；但是由于Bean的依赖关系，当非延迟单例Bean初始化创建依赖延迟单例Bean时，则延迟的单例Bean会被迫实例化创建

  ```xml
  <bean id="lazy" class="com.yh.User" lazy-init="true"/>
  ```

- 强制指定Bean的初始化顺序：

  一般情况下，我们可以依赖于spring来完成Bean初始化顺序的确定，但是对于spring还是无法解决一种特殊情况，当Bean依赖关系不太直接时，spring就无法严格要求它们的初始化顺序，如类A需要使用到类B的某个属性，但类B的属性有效值，需要通过类C初始化方法进行重新赋值；此时类A的正确性就需要类C优先被初始化来保证，但IOC容器中无法明确它们的依赖关系，只能保证B类在C类前面，B类在A类前面；

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

| Bean类型                                 | 作用                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| internalConfigurationAnnotationProcessor | 处理用于注册Bean的注解（**@Component、@ComponentScan、@Bean**。。） |
| internalAutowiredAnnotationProcessor     | 处理**@Autowired注解**                                       |
| internalCommonAnnotationProcessor        | 处理带有**JSR-250规范注解**                                  |
| internalEventListenerProcessor           |                                                              |
| internalEventListenerFactory             | 事件监听工厂                                                 |

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

spring使用@Configuration、@Bean来实现bean的注册，并显式定义bean的实例化过程，可以定义在类的实例方法和类方法中：

 ```java
@Configuration
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
- 同一个类中的所有@Bean方法，会按照其声明顺序执行；当@Bean方法之间出现依赖注入时，则按照依赖关系执行（但Bean的定义顺序还是其声明顺序，只是初始化顺序会改变）

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

3、成员变量：@Autowired注解在成员变量上时，**不需要为该类添加当前成员变量的setter方法**；当User类被IOC容器创建后，会将IOC容器中的Gread实例，**使用反射的方式进行属性注入**

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

  对于Map存储，key为当前Bean的id、value为当前Bean实例对象；所有集合元素的排序，遵循它们的注册顺序，但是@Priority注解的Bean会默认排序第一位；而@Order注解，可以修改注册优先级，但并不会影响Bean的实例化顺序（**注意：@Order不能注解在方法中，因此无法改变@bean方法的注册优先**级）

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

用于加载外部**properties**文件，进行外部属性注入

常用属性：

| 属性名                 | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| name                   | 资源命名（一般不需要）                                       |
| value                  | 外部资源路径                                                 |
| encoding               | 编码格式，默认使用项目编码格式                               |
| ignoreResourceNotFound | 是否忽略找不到指定值；默认false，不忽略                      |
| factory                | 定义PropertySources的生成规则，默认PropertySourceFactory.class |

**ignoreResourceNotFound，一般使用默认值，降低异常发生时的排除难度，但还需要搭配PropertySourcesPlaceholderConfigurer一起使用，才能够正确的抛出异常**

##### @Import

用于手动注册Bean时，依赖自动注册其他Bean

是一种区别于@Component、@Bean的另一种注册Bean的注解，一般用于直接导入第三方类，注册Bean，相对于@Bean更加简单，但只能使用默认构造方法进行初始化（没有则会报错）

### spring注解使用注意事项：

#### 1、@Bean和@Component的使用：

​		两者都可以定义bean：@Component只能声明式的定义Bean，Bean的创建过程完全交给IOC容器，通过构造方法完成；而@Bean可以有效的显式控制Bean的创建过程，并进行Bean实例对象的配置；

​		对于我们自己定义的类，一般完全就可以使用@Component来定义Bean；但对于第三方库包，由于不能直接修改源码，因此只能使用@Bean来定义相应类的Bean，并设置该实例对象的属性

​		**@Bean方法创建返回的实例，其生命周期会满足该类本身的生命周期配置（即实例还会执行自身属性注入、初始化方法和销毁方法）**

#### 2、@Component和@Configuration的区别：

​		在测试中，我们可以发现@Component和@Configuration都可以搭配@Bean进行Bean的注册；但实际上，两者有本质的区别：

​	@Configuration是一个全配置类，在注册到IOC容器中后，会创建一个CGLIB的动态代理类，作为其初始化实例；此时在执行@Bean方法时创建Bean实例时，会执行方法拦截器，将配置类的@Bean方法间出现的依赖调用，转化为先从IOC容器中获取，没有则直接执行依赖方法的Bean创建，然后获取其实例化对象，进行当前方法的Bean创建，有效避免出现多个对象创建

```java
	@Bean("userB")
	public  User getUserB() {
		System.out.println("b前");
		User user = new User();
		user.setName("yh123B");
		System.out.println("b后");
		return user;
	}

	@Bean("userA")
	public  User getUserA() {
		System.out.println("a前");
        User user = getUserB();
		user.setName("yh123A");
		System.out.println("a后");
		return user;
	}
```

​	@Component则会正常进行的@Bean方法，来创建Bean实例；当出现@Bean方法间的依赖调用时，就会创建多个对象，即一个Bean的创建依赖于另一个Bean，但实际IOC容器中该Bean中依赖的对象实例，和IOC容器中的存在的对象实例不相同

​	当然，对于@Component的缺点，可以使用方法参数注入的方式解决:

```java
	@Bean("userB")
	public  User getUserB() {
		System.out.println("b前");
		User user = new User();
		user.setName("yh123B");
		System.out.println("b后");
		return user;
	}

	@Bean("userA")
	public  User getUserA(@Qualifier("userB")User user) {
		System.out.println("a前");
		System.out.println(user);
		user.setName("yh123A");
		System.out.println("a后");
		return user;
	}
```

#### 3、@Bean注解普通方法和静态方法的区别：

@Bean可以注解在普通方法和静态方法，**对于静态@Bean的实例化，不会触发当前配置类的实例化**；因此在使用@Bean方式添加BeanPostProcessor、BeanFactoryPostProcessor时，该bean会在spring所有Bean初始化之前，提前实例化使用，因此当不声明为静态方法时，就会导致当前配置类提前实例化，从而无法进行spring的色号给生命周期，无法完成依赖注入

**即一般Bean直接使用普通方法；对于BeanPostProcessor、BeanFactoryPostProcessor这些需要提前实例化、不参与spring生命周期的Bean，则使用静态方法**

#### 4、@Bean和@Component出现beanName相同时：

由于IOC容器是先处理@ComponentScan注解，来注册使用@Component声明的Bean，因此其优先度高（相同优先度，出现同一BeanName时，会直接初始化报错），因此@Bean出现相同beanName时，会直接忽略不进行注册

### IOC容器的常用回调接口（仅spring）：

#### Aware接口

向bean提供容器的基础信息

##### BeanClassLoaderAware

获取spring使用的类加载器

```java
public interface BeanClassLoaderAware extends Aware {
	void setBeanClassLoader(ClassLoader var1);
}
```

##### ResourceLoaderAware

获取spring的资源加载器，来手动加载外部资源

```java
public interface ResourceLoaderAware extends Aware {
	void setResourceLoader(ResourceLoader var1);
}
```

##### BeanNameAware

获取当前Bean被IOC容器管理使用的id(name)

```java
public interface BeanNameAware extends Aware {
	void setBeanName(String name);
}
```

##### BeanFactoryAware

获取IOC容器的BeanFactory对象，并可以转化为已知子类，进行Bean的生产和管理

```java
public interface BeanFactoryAware extends Aware {
	void setBeanFactory(BeanFactory var1) throws BeansException;
}
```

##### ApplicationContextAware

和BeanFactoryAware类似，获取IOC容器的上下文（ApplicationContext），并可以转化为已知子类，在BeanFactory基础上提供额外操作

```java
public interface ApplicationContextAware extends Aware {
	void setApplicationContext(ApplicationContext var1) throws BeansException;
}
```

除此之外，还提供了直接使用ApplicationContext额外操作的接口（可以直接通过ApplicationContext实现）

- MessageSourceAware：用于国际化资源解析

- ApplicationEventPublisherAware：用于事件发布

#### IOC容器扩展接口

##### BeanPostProcessor

定义了IOC容器中所有Bean初始化方法前后所执行的逻辑

**配置方式：**BeanPostProcessor接口的实现类，可以直接像普通Bean一样，将其放入IOC容器，ApplicationContext会进行自动检测和配置；但需要注意：当使用@Bean注解来声明时，返回值类型应该为BeanPostProcessor，否则spring无法实现自动检测

两个方法，提供当前Bean的实例对象和beanName，需要将实例对象返回，在此期间可以对实例对象进行操作

```java
public interface BeanPostProcessor {
	Object postProcessBeforeInitialization(Object var1, String var2) throws BeansException;

	Object postProcessAfterInitialization(Object var1, String var2) throws BeansException;
}
```

**优先级：**可以定义多个接口实现类，通过@Order注解，来定义Bean的注册顺序，从而定义处理器执行的优先级

##### BeanFactoryPostProcessor

定义了IOC容器完成Bean注册后，所执行的逻辑

**配置方式：**同样和BeanPostProcessor一样，放在IOC容器中，被自动扫描配置

提供BeanFacotry的实例化对象，用于对Bean配置元数据（**BeanDefintion**）进行操作

```java
public interface BeanFactoryPostProcessor {
	void postProcessBeanFactory(ConfigurableListableBeanFactory var1) throws BeansException;
}
```

**优先级：**可以定义多个接口实现类，通过@Order注解，来定义Bean的注册顺序，从而定义处理器执行的优先级

##### FactoryBean

实现XML中的工厂bean，定义bean的实例化逻辑，但只能有一个产品类；**用于创建实例化逻辑比较复杂的Bean**

提供了三个方法，getObject（）用于提供产品类Bean的实例化对象；getObjectType（）用于提供产品类的类型；isSingleton（）提供产品类是否为单例（默认为单例，则不需要重写返回true）

```java
public interface FactoryBean<T> {
	T getObject() throws Exception;

	Class<?> getObjectType();

	boolean isSingleton();
}
```

对于工厂Bean，进行注册时，ID会添加&前缀；在获取工厂Bean和其产品Bean的方式也就有区别：

```java
	User user = context.getBean("userBeanFactory",User.class);

	UserBeanFactory bean = context.getBean("&userBeanFactory",UserBeanFactory.class)
```

**FactoryBean和@Bean的本质区别：**

它们相对于@Compenton注解，都可以显式的定义bean的实例化逻辑，但侧重点不同：@Bean是用于解决第三方类注册Bean的问题；而FactoryBean是用于创建实例化逻辑比较复杂的Bean，相对于@Compenton自动装配、手动注入和调用初始化方法，这种方式更加优雅

### IOC容器源码解析（注解版）

#### IOC容器内部功能实现的常用对象：

##### GennericBeanDefinition：

​	**BeanDefintion**的实现类，IOC容器进行Bean注册后的生成的对象，用于保存当前bean的类信息（类名、类calss对象。。）和bean定义信息（作用域、懒加载、自动装配Mode、是否为工厂Bean。。。）

##### DefaultListableBeanFactory：

​	**BeanFactory**的最常用实现类，IOC容器的核心类，实现Bean的生产和获取，常用成员变量有（**子类还有其他属性**）：

- beanDefinitionNames：所有bean注册名称列表，按注册顺序排序
- singletonBeanNamesByType：所有单例bean其类型和beanNames的 K-V映射
- allBeanNamesByType：所有bean其类型和beanNames的 K-V映射
- beanDefinitionMap：所有bean其beanName和BeanDefintion的K-V映射

beanName和beanNames的区别：beanName为bean在IOC容器中的唯一标识符（即ID），beanNames为bean的id和别名的集合（默认使用第一个值作为id）

##### ConfigurableListableBeanFactory ：

​	**postProcessBeanFactory**的实现类，其postProcessBeanFactory方法在**Bean注册完成后**执行，并能获取当前上下文中的BeanFactory，修改所有**BeanDefintion**的属性

##### BeanDefinitionRegistryPostProcessor:

​	为BeanFactoryPostProcessor接口的子接口，额外提供了postProcessBeanDefinitionRegistry，用于完成Bean的注册，其实现类为**ConfigurationClassPostProcessor**

##### BeanPostProcessor：

​	IOC容器的一个重要接口，提供了postProcessBeforeInitialization、postProcessAfterInitialization两个方法，参数为Bean实例化对象和BeanName，在bean初始化方法执行之前（创建Bean实例后）和之后执行；用于实现操作管理springBean的整个初始化过程（如属性注入）

##### DefaultSingletonBeanRegistry：

​	为**BeanFactory**的中间实现类，实现IOC容器单例Bean的初始化管理，其成员变量有：

- singletonObjects：存放所有单例Bean的初始化对象（初始化完成的），也称之为IOC容器的单例池

- earlySingletonObjects：存放所有单例Bean的实例化对象（没有进行属性注入的），用于解决循环依赖

  其子类FactoryBeanRegistrySupport，实现了工厂Bean产品类的初始化管理，其成员变量有：

  - factoryBeanObjectCache：存放工厂Bean中的单例产品Bean的初始化对象

#### 1、初始化spring环境（IOC容器）

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(springTest.class);
```

- 调用其父类**GenericApplicationContext**默认构造方法,初始化一个**beanFactory**

  ```java
  this.beanFactory = new DefaultListableBeanFactory();
  ```

- 调用**AnnotationConfigApplicationContext**自身默认构造方法，初始化**reader**、**scanner**成员变量

  ```java
  this.reader = new AnnotatedBeanDefinitionReader(this);
  this.scanner = new ClassPathBeanDefinitionScanner(this);
  ```

  - AnnotatedBeanDefinitionReader初始化,创建**读取器**，完成IOC内置Bean的注册和IOC容器中的一些处理器配置（这些bean和处理器都是用于处理spring注解）

    ```java
    //注册spring内置的注解处理器（会进行初始化）
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
    ```
  
    **配置处理器：**
  
    - AnnotationAwareOrderComparator   解析@Order、@Priority注解
    - ContextAnnotationAutowireCandidateResolver  提供延迟加载功能
  
    **注册内置Bean：**
  
    | 内置Bean类型                         | 接口类型                   | 作用                                         |
    | ------------------------------------ | -------------------------- | -------------------------------------------- |
    | ConfigurationAnnotationProcessor     | BeanFactoryPostProcessor   | 用于完成Bean的注册                           |
    | AutowiredAnnotationBeanPostProcessor | BeanPostProcessor          | 处理@Autowired注解，完成Bean的属性注入       |
    | CommonAnnotationBeanPostProcessor    | BeanPostProcessor          | 处理@PostConstruct和@PreDestroy还有@Resource |
    | PersistenceAnnotationProcessor       | BeanPostProcessor          | 处理JPA注解（需要导入spirng-orm包）          |
    | EventListenerMethodProcessor         | ApplicationContext额外功能 | 处理事件监听                                 |
    | DefaultEventListenerFactory          | EventListenerFactory       | 用于事件监听（）                             |
  
  - ClassPathBeanDefinitionScanner初始化,创建**类路径扫描器**，将类转化为BeanDefinition对象
  
    该扫描器并不是spring默认使用的扫描器，而是提供给开发者使用的（一般不会用到）
  
- 调用**AnnotationConfigApplicationContext**有参构造方法（调用无参构造方法时，则需要手动调用refresh（））

  ```java
  //手动注册Bean，生成相应beanDfinition对象；
  register(componentClasses);
  //初始化spring上下文
  refresh();
  ```
  

**refresh方法源码：**

```java
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
		//为初始化上下文做准备
		prepareRefresh();

        //创建BeanFactory（对于注解方式，则会使用原来的，原因：注解版需要提前创建一个beanFactory来注册配置类Bean；而xml方式，则是在此时进行beanFactory的注册）
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
            
        //配置beanFactory：
   		//配置beanFactory内部使用的类加载器、Bean表达式解析器。。
        //添加ApplicationContextAwareProcessor、ApplicationListenerDetector两个Bean后置处理器
        //添加beanFactory相关依赖的其他Bean（MessageSource、Environment）。。。   
        //忽略Aware系列接口的自动装配
		prepareBeanFactory(beanFactory);

		try {
            //spring5没有做任何操作
			postProcessBeanFactory(beanFactory);
            
			//调用所有的BeanFactoryPostProcessors（内置和自定义）
			invokeBeanFactoryPostProcessors(beanFactory);
            
			//注册所有的BeanPostProcessor（自定义）
			registerBeanPostProcessors(beanFactory);

			//初始化上下文消息源（直接注册初始化相应bean，不需要创建BD对象，然后进行Bean初始化）
			initMessageSource();

			//初始化上下文事件广播器（直接注册初始化相应bean）
			initApplicationEventMulticaster();

			//注册其他特殊bean
			onRefresh();

			//注册监听器bean
			registerListeners();

			//初始化IOC容器中的所有单例bean（同时调用所有BeanPostProcessor）	
			finishBeanFactoryInitialization(beanFactory);

			//完成其他初始化上下文工作
			finishRefresh();
			}
        。。。。
		}
	}
```

#### 2、Bean的注册过程

在Refresh方法中，执行所有的BeanFactoryPostProcessor，从而完成Bean的注册

```java
invokeBeanFactoryPostProcessors(beanFactory);
```

- getBeanFactoryPostProcessors()目前为null;只有在refresh之前，手动调用context.addBeanFactoryPostProcessor(xxx)方法传入BeanFactoryPostProcessor对象，才会有（一般情况下不使用该方式）

```
invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors())；
```

##### 1、执行所有BeanDefinitionRegistryPostProcessor类型的BeanFactoryPostProcessor

- 创建一个set，保存已经执行的BeanFactoryPostProcessor的BeanName

```java
Set<String> processedBeans = new HashSet<>();
```

- 创建两个list，保存不同类型的BeanFactoryPostProcessor

```java
//常规后置处理器
List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
//用于注册Bean的后置处理器
List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
```

- 首先执行BeanDefinitionRegistryPostProcessor类型的BeanFactoryPostProcessor对象（因为此时beanFactoryPostProcessors为null，所以没有可执行的）

```java
	for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
                    //执行后置处理器
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
                    //并添加到用于注册Bean的后置处理器集合中
					registryProcessors.add(registryProcessor);
				}
				else {
                    //否则添加到常规后置处理器集合中
					regularPostProcessors.add(postProcessor);
				}
			}
```

- 获取beanFactory中BeanDefinitionRegistryPostProcessor类型的Bean（spring内置处理器 BeanName=internalConfigurationAnnotationProcessor，class=**ConfigurationClassPostProcessor**)，**初始化并执行spring内置的BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry方法**

```java
//创建一个List，保存当前用于注册Bean的后置处理器
List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

//获取当前已注册的BeanDefinitionRegistryPostProcessor类型BeanNames
String[] postProcessorNames =				beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);

//执行getBean，初始化internalConfigurationAnnotationProcessor类型Bean
for (String ppName : postProcessorNames) {
    //实现PriorityOrdered接口的Bean
	if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
		currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
        //表示当前处理器Bean要被执行
		processedBeans.add(ppName);
		}
	}

//进行排序
sortPostProcessors(currentRegistryProcessors, beanFactory);

//添加到注册Bean的后置处理器集合中
registryProcessors.addAll(currentRegistryProcessors);

//执行internalConfigurationAnnotationProcessor，然后清空currentRegistryProcessors
invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
currentRegistryProcessors.clear();
```

-  执行internalConfigurationAnnotationProcessor，会完成所有Bean的注册，因此BeanNames中就会存在自定义的BeanFactoryPostProcessor的BeanName；此时**初始化并执行自定义的BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry方法**

```java
//获取当前BeanDefinitionRegistryPostProcessor类型的所有BeanName
postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);

//遍历，找出没有被执行、实现Ordered接口的BeanName，执行getBean初始化
for (String ppName : postProcessorNames) {
		if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
		currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
		processedBeans.add(ppName);
		}
	}

//排序
sortPostProcessors(currentRegistryProcessors, beanFactory);

//添加到注册Bean的后置处理器集合中
registryProcessors.addAll(currentRegistryProcessors);

//执行自定义BeanDefinitionRegistryPostProcessor类型Bean（一般用于扩展IOC容器的管理，识别其他注解，来注册Bean，如mybatis中的mapper）
invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
currentRegistryProcessors.clear();
```

- 再一次遍历postProcessorNames，初始化并执行**其他没有实现Ordered接口的BeanDefinitionRegistryPostProcessor的postProcessBeanDefinitionRegistry方法**

```java
//执行其他BeanDefinitionRegistryPostProcessor，可能会进行额外Bean的注册，因此需要不断循环遍历，获取所有的BeanDefinitionRegistryPostProcessor
boolean reiterate = true;
while (reiterate) {
	reiterate = false;
	postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
    //遍历，找出其他没有执行的BeanDefinitionRegistryPostProcessor，执行getBean初始化
	for (String ppName : postProcessorNames) {
		if (!processedBeans.contains(ppName)) {
			currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
			processedBeans.add(ppName);
			reiterate = true;
			}
		}
    	//排序
		sortPostProcessors(currentRegistryProcessors, beanFactory);
    	//添加到注册Bean的后置处理器集合
	    registryProcessors.addAll(currentRegistryProcessors);
    
    	//执行所有其他没有实现Ordered接口BeanDefinitionRegistryPostProcessor
		invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
		currentRegistryProcessors.clear();
	}
```

- 最后，再按顺序执行所有BeanDefinitionRegistryPostProcessor的postProcessBeanFactory方法（先执行注册Bean的后置处理，再执行常规处理器）

```java
invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
```

**作用：**

完成IOC容器中所有Bean的注册

##### 2、执行所有非BeanDefinitionRegistryPostProcessor类型的postProcessBeanFactory

按照  实现PriorityOrdered接口-------实现Ordered接口-------没有实现Ordered接口的顺序执行；

**作用：**

用于执行自定义postProcessBeanFactory接口，对已注册的Bean的BeanDefinition进行属性设置

##### 3、ConfigurationClassPostProcessor

**默认情况下，BeanDefinitionRegistryPostProcessor类型的ProcessBeanFactory就只有ConfigurationClassPostProcessor，因此整个注册过程就是由它来完成**

- **postProcessBeanDefinitionRegistry方法：**

```java
postProcessor.postProcessBeanDefinitionRegistry(registry);

processConfigBeanDefinitions(registry);
```

processConfigBeanDefinitions：

1、获取注册的自定义Bean（在refresh之前，调用register(xxx)方法注册的Bean，也就是配置类Bean）

```java
//获取所有BeanNames
List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
String[] candidateNames = registry.getBeanDefinitionNames();

//遍历BeanNames
for (String beanName : candidateNames) {
    //获取BeanDefinition对象
	BeanDefinition beanDef = registry.getBeanDefinition(beanName);
    
    //判断当前类为FUll配置类Bean（@Configuration注解）还是Lite配置类Bean（@Component注解）
    //已经check的就不在做处理（排除spring内部处理器Bean）
	if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
		if (logger.isDebugEnabled()) {
			logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
		}
		//无法判断，则check来标记其类型，并添加到一个list中（获取已注册的自定义Bean）
	  }else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
			configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
		}
	}

//没有则return，结束方法
if (configCandidates.isEmpty()) {
	return;
}

//排序
configCandidates.sort((bd1, bd2) -> {
	int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
	int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
	return Integer.compare(i1, i2);
});
```

2、解析配置类Bean

```java
//初始化配置类解析器
ConfigurationClassParser parser = new ConfigurationClassParser(
		this.metadataReaderFactory, this.problemReporter, this.environment,
		this.resourceLoader, this.componentScanBeanNameGenerator, registry);
		
//创建集合保存所有配置类的BeanDefinitionHolder和ConfigurationClass对象
Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());

do {
    //进行配置类解析，并进行校验
    //通过解析配置类中所有注解，并注册所有@Component注解声明的Bean
	parser.parse(candidates);
	parser.validate();
	Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
    
    //删除已经解析过的配置类的ConfigurationClass对象（目前为null）
	configClasses.removeAll(alreadyParsed);

	//获取spring初始化前创建的读取器
	if (this.reader == null) {
		this.reader = new ConfigurationClassBeanDefinitionReader(
			registry, this.sourceExtractor, this.resourceLoader,this.environment,this.importBeanNameGenerator, parser.getImportRegistry());
		}
    //对已经解析的注解内容（@Import、@Bean、@ImportResource）进行处理，注册Bean
	this.reader.loadBeanDefinitions(configClasses);
    //并将其设置为已解析的
	alreadyParsed.addAll(configClasses);
	candidates.clear();
  
}
```

parser.parse(candidates)：

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
    //遍历配置类的BeanDefinition
	for (BeanDefinitionHolder holder : configCandidates) {
		BeanDefinition bd = holder.getBeanDefinition();
		try {
            //是否为注解声明注册的Bean（直接进入）
			if (bd instanceof AnnotatedBeanDefinition) { 
			parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
			}
            xxxx
		}
```

processConfigurationClass》》》》doProcessConfigurationClass

```java
//递归处理内部类
if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
	processMemberClasses(configClass, sourceClass, filter);
}

		//处理其@PropertySource，加载外部资源
for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
sourceClass.getMetadata(),PropertySources.class,org.springframework.context.annotation.PropertySource.class)) {
	if (this.environment instanceof ConfigurableEnvironment) {
		processPropertySource(propertySource);
	}
	else {
		logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
		"]. Reason: Environment must implement ConfigurableEnvironment");
	}
}

//处理其@ComponentScan注解
//获取@ComponentScan中的具体内容
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
	
	//如果有@ComponentScan注解，则进行处理
	if (!componentScans.isEmpty() &&
		!this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
        
        //遍历所有componentScan内容
		for (AnnotationAttributes componentScan内容 : componentScans) {
            //创建@component注解扫描器，并立即执行
			Set<BeanDefinitionHolder> scannedBeanDefinitions =
			this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
            
            //遍历通过扫描@component注解创建的BeanDefinition对象
			for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
				BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
				if (bdCand == null) {
					bdCand = holder.getBeanDefinition();
				}
				if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
                    //递归处理解析该Bean
					parse(bdCand.getBeanClassName(), holder.getBeanName());
				 }
			}
		}
	}

//处理@Import注解
processImports(configClass, sourceClass, getImports(sourceClass), filter, true);

//处理@ImportResource注解，导入spring的XML配置文件，读取配置资源
AnnotationAttributes importResource =
AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
	if (importResource != null) {
		String[] resources = importResource.getStringArray("locations");
		Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
		for (String resource : resources) {
			String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
				configClass.addImportedResource(resolvedResource, readerClass);
		}
	}

//处理当前所有Bean中的@Bean注解
Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
for (MethodMetadata methodMetadata : beanMethods) {
	configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
	}
processInterfaces(configClass, sourceClass);
```

**对于@Import、@Bean、@ImportResource，只是进行了内容解析，所有Bean的注册在this.reader.loadBeanDefinitions(configClasses)中完成**

- **postProcessBeanFactory方法**

完成配置类（@Configuration）的动态代理，实现配置类中@Bean方法间的依赖注入

#### 3、Bean初始化过程

以refresh方法中，实例化所有单例bean为例：（多例bean和懒加载Bean会在**依赖注入或getBean**时初始化）

```java
finishBeanFactoryInitialization(beanFactory);
```

- Bean注册后，会将封装的**BeanDefinition**对象保存到beanfactory的**BeanDefinitionMap**中

- 进行单例bean的初始化：

  ```java
  beanFactory.preInstantiateSingletons();
  ```

  ```java
  		//获取beanfactory中beanDefinitionNames，所有所有bean注册名称列表
  		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
  
  		//遍历，实例化所有非懒加载的单例bean
  		for (String beanName : beanNames) {
              //通过beanName获取BeanDefinition对象
  			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
              
              //判断当前bean是否为单例、非懒加载、非抽象
  			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                  //判断当前bean是否为工厂bean
  				if (isFactoryBean(beanName)) {
                  //如果为工厂bean，在使用带有&前缀的beanName参数执行getBean方法，初始化bean
  				Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                  xxxx
                  }
                  //非工厂bean
  				else {
                      //初始化bean
  					getBean(beanName);
  				}
  			}
  		}
  
  		// 执行所有单例池中Bean初始化后的回调方法
  		for (String beanName : beanNames) {
  			Object singletonInstance = getSingleton(beanName);
  			xxxx
  			}
  		}
  ```

  getBean》》》》doGetBean：

  ```java
  	//第一次getSingleton
      //检测该bean是否在单例池中已存在（用于getBean获取实例对象）、或在初始化中（防止循环依赖）
  	Object sharedInstance = getSingleton(beanName);
  	if (sharedInstance != null && args == null) {
          xxx
          //如果为工厂Bean，则根据name和BeanName判断返回工厂对象实例还是产品对象实例
          bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
      }
  	else{
          xxx
          //保证当前bean所依赖的bean都已初始化
           String[] dependsOn = mbd.getDependsOn();
  		if (dependsOn != null) {
  			for (String dep : dependsOn) {
  			xxxx
              }
          }
          
          //创建bean实例化对象，并执行一系列BeanPostProcessor，完成bean的初始化
          //为单例
          if (mbd.isSingleton()) {
              //第二次getSingleton（单例池中不存在时，同步进行bean的初始化，并使该beanName添加到表示处于初始化状态中的集合）	
              sharedInstance = getSingleton(beanName, () -> {	
  				return createBean(beanName, mbd, args);
              });
  			
          } 
          //为多例
          else if(mbd.isPrototype()){
              //用于多例Bean的实例获取
              xxx
          }
      }        
  ```

  createBean：

  **lookup-method和replaced-method的标签用于实现对方法返回值的替换和方法的重写**

  ```java
  //获取Bean的class对象
  Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
  
  //处理lookup-method和replaced-method的标签方法
  mbdToUse.prepareMethodOverrides();
  
  //实现Bean代理对象的初始化（不执行）
  Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
  
  //实现Bean的初始化
  Object beanInstance = doCreateBean(beanName, mbdToUse, args);
  ```

  doCreateBean：

  ```java
  //调用Bean的构造方法，创建实例对象
  instanceWrapper = createBeanInstance(beanName, mbd, args);
  
  //属性注入
  populateBean(beanName, mbd, instanceWrapper);
  
  //执行Bean生命周期的初始化回调
  exposedObject = initializeBean(beanName, exposedObject, mbd);
  ```
  
- 整个初始化过程就是通过八个BeanPostProcessors来实现：

##### 	BeanPostProcessor有5种类型接口：

- **BeanPostProcessor**

  定义Bean初始化回调前后的逻辑代码

  ```java
  public interface BeanPostProcessor {
      //Bean执行初始化回调之前执行
  	@Nullable
  	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  		return bean;
  	}
  
      //Bean执行初始化回调之后执行
  	@Nullable
  	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  		return bean;
  	}
  }
  ```

- **InstantiationAwareBeanPostProcessor**

  提供三个方法：

  - Bean实例化前，是否提供其他对象，作为其实例化对象
  - Bean实例化后，执行属性注入前，设置Bean是需要正常执行自动依赖注入
  - Bean属性注入后，实现Bean的初始化回调，检测属性注入值，可以进行重新赋值

  ```java
  public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
      //这个方法用来在对象实例化前直接返回一个对象（如代理对象）来代替通过内置的实例化流程创建对象；
      @Nullable
      default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
          return null;
      }
      
      //在对象实例化完毕执行populateBean之前 如果返回false则spring不再对对应的bean实例进行自动依赖注入。
      default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
          return true;
      }
      
      //这里是在spring处理完默认的成员属性，应用到指定的bean之前进行回调，可以用来检查和修改属性，最终返回的PropertyValues会应用到bean中
      //@Autowired、@Resource等就是根据这个回调来实现最终注入依赖的属性的
      @Nullable
      default PropertyValues postProcessPropertyValues(
              PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
          return pvs;
      }
  }
  ```

- **SmartInstantiationAwareBeanPostProcessor**

  实例化之前执行，提供给spring内部使用，来确认Bean的实例化方式

  ```java
  public interface SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor {
      //用来返回目标对象的类型（比如代理对象通过raw class获取proxy type 用于类型匹配）
      @Nullable
      default Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException {
          return null;
      }
      //这里提供一个拓展点用来解析获取用来实例化的构造器（比如未通过bean定义构造器以及参数的情况下，会根据这个回调来确定构造器）
      @Nullable
      default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
              throws BeansException {
          return null;
      }
      //获取要提前暴露的bean的引用，用来支持单例对象的循环引用（一般是bean自身，如果是代理对象则需要取用代理引用）
      default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
          return bean;
      }
  }
  ```

- **MergedBeanDefinitionPostProcessor**

  bean实例化之后执行，检测和修改当前Bean的BeanDefinition属性

  ```java
  public interface MergedBeanDefinitionPostProcessor extends BeanPostProcessor {
      //在bean实例化完毕后调用 可以用来修改merged BeanDefinition的一些properties 或者用来给后续回调中缓存一些meta信息使用
      //这个算是将merged BeanDefinition暴露出来的一个回调
      void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
  }
  ```

- **DestructionAwareBeanPostProcessor**

  bean销毁前执行，用于Bean声明周期的销毁回调

  ```java
  public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {
      //这里实现销毁对象的逻辑
      void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;
      //判断是否需要处理这个对象的销毁
      default boolean requiresDestruction(Object bean) {
          return true;
      }
  }
  ```

##### Bean初始化过程中BeanPostProcessor的执行过程：

Bean的初始化过程有8个BeanPostProcessor、实例化执行、属性注入和初始化回调共同构成

| 序号 | BeanPostProcessor接口类型和方法                              | 执行点和作用                                                 |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation | Bean实例化前（提供实例对象（如代理对象），否则交给spring进行实例化） |
| 2    | SmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors | Bean实例化前（确定构造方法）                                 |
| 3    | MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition | Bean实例化后（检测和修改当前Bean的BeanDefinition属性）       |
| 4    | InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation | Bean属性注入前（确定是否执行Bean属性注入）                   |
| 5    | InstantiationAwareBeanPostProcessor.postProcessPropertyValues | Bean属性注入后（对Bean的属性进行最终确认）                   |
| 6    | BeanPostProcessor.postProcessBeforeInitialization            | Bean初始化回调前                                             |
| 7    | BeanPostProcessor.postProcessAfterInitialization             | Bean初始化回调后                                             |
| 8    | DestructionAwareBeanPostProcessor.postProcessBeforeDestruction | 执行Bean的销毁回调                                           |

spring初始化过程对应执行的注解：

| 初始化过程       |                                                              |
| ---------------- | ------------------------------------------------------------ |
| 实例化           | 实现构造方法的属性注入（@AutoWired注解在构造方法上）         |
| 属性注入         | @AutoWired、@Value                                           |
| Bean初始化回调前 | BeanPostProcessor.postProcessBeforeInitialization、@PostConstruct |
| Bean初始化回调   | 实现InitializingBean接口                                     |
| Bean初始化回调后 | BeanPostProcessor.postProcessAfterInitialization             |
| Bean销毁回调前   | @Processor                                                   |
| Bean销毁回调     | 实现DisposableBean接口                                       |

**BeanPostProcessor接口和InitializingBean、DisposableBean接口区别：**

1、前者是通过BeanPostProcessor的执行方式，作用于所有Bean的初始化阶段

2、后者是提供针对于实现该接口的单个Bean，编写初始化和销毁方法，无法作用于所有Bean（XML中的init-method、destroy-method就是使用这种方式）

3、@PostConstruct发生在BeanPostProcessor接口方法后；@Processor发生在DisposableBean接口前

##### Aware接口的回调执行点：

1、对于Bean完成属性注入后，在执行BeanPostProcessor.postProcessBeforeInitialization方法前，会进行部分Aware接口的回调，回调顺序如下：

BeanNameAware、BeanClassLoaderAware、BeanFactoryAware

2、然后开始执行BeanPostProcessor.postProcessBeforeInitialization，其中一个为：

ApplicationContextAwareProcessor.postProcessBeforeInitialization,用于进行其他Aware接口的回调，回调顺序如下：

EnvironmentAware、EmbeddedValueResolverAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware

### ApplicationContext其他功能：

#### 资源加载：

- spring提供了Resource接口，重新定义了java中各种资源获取的API，功能更加强大，且使用方便；其实现类包括：

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

- spring提供ResourceLoaderAware接口，用于获取ApplicationContext中的ReourceLoader,手动进行外部资源加载

#### 国际化功能（i18n）

spring提供了一个**MessageSource接口**，用于提供国际化功能（i18n）。同时还提供了**HierarchicalMessageSource接口**，用于分层解析消息，**根据不同的地区，对消息进行不同的解析**

**自定义MessageSource：**

提供三种解析消息的方法，以消息编码、参数、默认消息、地区四个参数进行自定义逻辑代码解析，实现国际化

```java
public class MyMessage implements MessageSource{

	@Override
	public String getMessage(String code, Object[] args, String defaultMessage, Locale locale) {
	}

	@Override
	public String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException {
		return null;
	}

	@Override
	public String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException {
		return null;
	}
}
```

spring提供一些MessageSource实现类，按照相应配置实现国际化功能（由于一般用于JSP页面，因此不进行深入）

#### 事件监听

spring提供ApplicationEvent类和ApplicationListener接口提供事件监听处理，在spring4.2后，提供基于注解的方式发布事件：

spring提供多个默认事件:

| 事件                       | 作用                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ContextRefreshedEvent      | applicationContext刷新（只支持可以热刷新的applicationContext，如XmlWebApplicationContext） |
| ContextStartedEvent        | applicationContext调用Start方法                              |
| ContextStoppedEvent        | applicationContext调用Stop方法                               |
| ContextClosedEvent         | applicationContext调用close方法                              |
| RequestHandledEvent        | web应用中，spring的DispatcherServlet拦截到http请求           |
| ServletRequestHandledEvent | RequestHandledEvent的子类，添加了拦截http请求的servlet信息   |

通过实现ApplicationListener接口的Bean来监听这些事件，执行相应代码：

- 自定义事件（不需要放到spring容器中）

```java
public class MyEvent extends ApplicationEvent {
	
	public MyEvent(Object source) {
		super(source);
	}
}
```

- 自定义事件监听器（被spring容器管理）

```java
@Component
public class MyEventListener implements ApplicationListener<MyEvent>{

	@Override
	public void onApplicationEvent(MyEvent event) {
		System.out.println(event.getSource());
	}
}
```

- 使用ApplicationContext发布事件，监听器触发执行代码

```java
context.publishEvent(new MyEvent("sd"));
```

#### 在web应用快速创建ApplicationContext实例化

ApplicationContext提供声明式的方式来创建其实例，用于web应用中，通过监听器伴随web应用的启动，初始化spring容器：

在web项目的web.xml中进行配置：

 ```java
	<!-- 定义上下文参数，用于指定spring的xml配置文件 -->
	<context-param>
    	<param-name>contextConfigLocation</param-name>
    	<param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-	value>
	</context-param>

	<!-- 配置spring容器监听器，当项目启动时，自动初始化spring容器 -->
	<listener>
		<listener-class>
			org.springframework.web.context.ContextLoaderListener
		</listener-class>
	</listener>
 ```

当**contextConfigLocation**参数没有指定时，spring该监听器会默认扫描**/WEB-INF/applicationContext.xml**路径

### spring实现Bean的手动注册：

1、直接通过ApplicationContext，在IOC容器初始化后手动注册；该方式并不会将该Bean交给spring管理，因此**不会遵循springBean的生命周期，只是单纯用于spring将其依赖注入到其他bean中**（一般不使用）

```java
context.getBeanFactory().registerSingleton("lisr", new HelloWord());
```

2、在IOC容器初始化前，创建指定类的BeanDefinition对象，手动注册

```java
RootBeanDefinition bean = new RootBeanDefinition(HelloWord.class);
registry.registerBeanDefinition("helloWord", bean);
```

可以通过两种方式完成：

- 使用BeanDefinitionRegistryPostProcessor接口，在执行完默认注册处理器后，执行手动注册处理器

  ```java
  public class MapperBeanDefinitionRegistart implements BeanDefinitionRegistryPostProcessor{
  
  	@Override
  	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  	}
  
  	@Override
  	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
  		RootBeanDefinition bean = new RootBeanDefinition(HelloWord.class);
          registry.registerBeanDefinition("helloWord", bean);
  	}
  }
  ```

- 使用ImportBeanDefinitionRegistrar接口，创建一个动态注册Bean的实现类

  使用方式：

  - 实现ImportBeanDefinitionRegistrar接口

    registerBeanDefinitions方法提供两个参数：

    - AnnotationMetadata  使用@Import导入自身的配置类的所有注解信息
    - BeanDefinitionRegistry spring的Bean注册类（BeanFactory实现类）

    ```java
    public class MyBeanDefinitionRegistart implements ImportBeanDefinitionRegistrar{
    
    	@Override
    	public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry registry) {
            //获取当前配置类的注解信息（可以创建一个自定义注解，来获取需要注册的全类名，从而获取class对象）
    	AnnotationAttributes annoAttrs = AnnotationAttributes.fromMap(annotationMetadata.getAnnotationAttributes(ComponentScan.class.getName()));
    	String[] basePackages = annoAttrs.getStringArray("basePackages");
    		
    	//手动注册Bean（通过class对象创建）
    	RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(TestMapper.class);
    	registry.registerBeanDefinition("testMapper", rootBeanDefinition);
    	}
    }
    ```

  - 在配置类上使用@Import注解到，来导入该实现类（使用@Component注册不会生效）

    ```java
    @Configuration
    @Import(ImportBeanDefinitionRegistrarTest.class) 
    public class ConfigurationTest {
     	
    }
    ```

**也可以将@Import声明到自定义注解上，然后通过自定义注解声明到配置类上，来实现整个功能（即利用组合注解，来使自定义注解具有动态注册Bean的能力，mybatis整合spring中，@MapperScan就利用了这种方式）**

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

  - 第二优先级的增强类方法，进行切入点的动态代理

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

## 4、spring-web模块

spring-web模块包括三个部分：SpringMVC、Spring WebFlux、Websocket

### springMVC：

- springMVC的依赖包：

  - spring-webMVC包：该包会依赖导入spring-web，一般情况下两者需要同时使用
  - servlet-api.jar：springMVC是基于servlet api来实现的

  **spring-webmvc和spring-web两个依赖包的作用：**

  ​       spring-web提供了核心HTTP集成，和一些便捷的servlet过滤器，可以整合其他基础的web框架技术（Hessian，Burlap）

  ​       spring-webmvc是对springMVC功能的实现，依赖于spring-web；一般只需要显式添加spring-webmvc依赖

- springMVC基本概念：

  springMVC是基于Servlet API构建的原始Web框架，使得开发者能够简单高效地实现了MVC分层架构，并且有很强地扩展性

#### 前端控制器：

​		springMVC和其他web框架一样，围绕前端控制器模式来操作servlet API；**DispatcherServlet**就是springMVC的前端控制器；并且springMVC搭配IOC容器，完成所有对象的管理

##### springMVC启动过程：

- web容器初始化过程：

web容器启动时，会先初始化Listener，执行其contextinitialized方法；然后初始化filter，执行init方法；最后初始化servlet，执行init方法；从而完成整个web容器的初始化过程

- IOC容器初始化过程：

springMVC提供ContextLoaderListener、DispatcherServlet来伴随web容器的初始化，完成整个IOC容器的初始化

1、web容器初始化ContextLoaderListener，执行contextinitialized方法，使用ContextLoader读取ServletContext中的初始化参数（web容器的初始化参数），创建Root webApplicationContext（spring容器）；最后将创建好的webApplicationContext放入ServletContext中属性中（web容器中）

2、web容器进一步初始化DispatcherServlet，执行init方法：

获取ServletContext中的webApplicationContext,作为父容器（spring容器）

然后读取DispatcherServlet属性，创建一个新的webApplicationContext，作为子容器（springMVC容器）

之后执行DispatcherServlet其他初始化方法，在子容器中注册多个Bean，实现DispatcherServlet功能

##### spring容器和springMVC容器：

springMVC容器为子容器，可以访问父容器中的Bean；而父容器不能访问子容器中的Bean；

父容器通过ApplicationContext来自动注入获取；子容器通过webApplicationContext自动注入获取

子容器可以获取到父容器上下文：

```java
ApplicationContext parent = webApplicationContext.getParent();
```

##### dispatcherServlet配置：

- xml方式：

```xml
<!-- 配置servelt容器初始化参数 -->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:spring-mybatis.xml</param-value>
</context-param>

<!-- 配置servelt容器监听器，当项目启动时，自动初始化spring容器 -->
<listener>
	<listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>

<!-- 配置dispatcherServlet参数 -->
<servlet>
	<servlet-name>springMVC</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springMVC-config.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup><!-- 让springMVC的servlet在项目启动时就加载 -->
	<async-supported>true</async-supported><!-- 开启servlet异步线程处理 -->
</servlet>

<!-- 配置dispatcherServlet映射规则，指定需要拦截的请求url -->
<servlet-mapping>
     <servlet-name>springMVC</servlet-name>
     <url-pattern>/</url-pattern>     <!-- /,拦截所有请求 -->
</servlet-mapping>
```

- spring配置类方式

  springMVC提供**WebApplicationInitializer**接口，来通过ServletContext来初始化当前web项目的Servlet容器，配置Servlet容器组件(Servlet、Listener、Filter)（实际上就是替代了web.xml文件，将该文件的配置信息交给springMVC管理，**但Servelt容器在启动时，还是会先读取web.xml，然后交给springMVC配置**

  ），此时可以通过在pom文件中添加如下配置来消除报错：
  
  ```xml
  <properties>
      <failOnMissingWebXml>false</failOnMissingWebXml>
  </properties>
  ```
  
  ```java
  public class WebConfig implements WebApplicationInitializer{
  	@Override
  	public void onStartup(ServletContext servletContext) throws ServletException {
  		//初始化spring容器 (ContextLoaderListener只能通过读取配置文件来完成spring容器的创建)
  		servletContext.setInitParameter("contextConfigLocation", "classpath:spring.xml");
  		ContextLoaderListener contextLoaderListener = new ContextLoaderListener();
  		servletContext.addListener(contextLoaderListener);
  
  		// 创建springMVC容器（可以使用xml或者注解）
  		AnnotationConfigWebApplicationContext appContext = new AnnotationConfigWebApplicationContext();
  		appContext.register(MvcConfig.class);
  //		XmlWebApplicationContext appContext = new XmlWebApplicationContext();
  //		appContext.setConfigLocation("/WEB-INF/classes/springMVC-config.xml");
  
  		// 注册DispatcherServlet，并进行初始化配置（初始化springMVC容器，并注册springMVC需要的Bean）
		DispatcherServlet servlet = new DispatcherServlet(appContext);
  		Dynamic addServlet = servletContext.addServlet("springMVC", servlet);
		addServlet.setLoadOnStartup(1);
  		addServlet.addMapping("/");
		addServlet.setAsyncSupported(true);
  	}
}
  ```

  2、springMVC还提供**AbstractAnnotationConfigDispatcherServletInitializer**类，为**WebApplicationInitializer**接口的实现类，简化springMVC的配置：
  
  只需要提供spring容器配置类、springMVC容器配置类，就可以完成整个springMVC的配置；并且对于DispatcherServlet属性，默认添加了如下设置：（通过重写其方法来改变）
  
  addServlet.setLoadOnStartup(1);
  
  addServlet.setAsyncSupported(true);
  
  ```java
  public class WbeConfigSimple extends AbstractAnnotationConfigDispatcherServletInitializer {
  
      //spring容器配置类
  	@Override
  	protected Class<?>[] getRootConfigClasses() {
  		return new Class<?>[] { springTest.class };
  	}
  
      //springMVC容器配置类
  	@Override
  	protected Class<?>[] getServletConfigClasses() {
  		return new Class<?>[] { WebConfig.class };
  	}
  
      //dispatcherServlet映射
	@Override
  	protected String[] getServletMappings() {
		return new String[] { "/" };
  	}    
  }
  ```
  
  springMVC配置类：一般情况下，springMVC容器只需要扫描Controller注解的类
  
  ```java
  @EnableWebMvc
  @ComponentScan(basePackages = { "com.yh.mvc" }, useDefaultFilters = false, includeFilters = {
  		@Filter(type = FilterType.ANNOTATION, classes = { Controller.class }) })
  @Configuration
  public class MvcConfig {
  }	
  ```

- 配置方式选择：

**XML配置和springMVC配置的两种方式，三个只能同时存在其一，否则会导致Servelt容器启动报错：web.xml中存在多个web应用程序上下文的加载定义，推荐使用AbstractAnnotationConfigDispatcherServletInitializer，更加简单方便**

##### WebApplicationContext：

springMVC IOC容器的上下文对象，是ApplicationContext的子接口

springMVC提供一个静态，通过一个selvet请求，快速获取当前WebApplicationContext对象

```java
RequestContextUtils.findWebApplicationContext(request);
```

##### dispatcherServlet属性：

dipatcherServlet初始化参数：（用于在web.xml中配置dipatcherServlet使用）

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| contextClass                   | 用于实例化的参数class对象，默认XmlWebApplicationContext，即springMVC容器的上下文对象 |
| contextConfigLocation          | pringMVC容器的上下文对象的配置文件路径                       |
| namespace                      | XmlWebApplicationContext的命名空间，默认为**[servlet-name]-servlet**，在web.xml中使用 |
| throwExceptionIfNoHandlerFound | 在找不到请求的处理程序时是否抛出NoHandlerFoundException异常；默认为false |

dipatcherServlet常用属性：

| 属性名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| isAsyncSupported | 默认为false，让dipatcherServlet对请求进行异步多线程处理      |
| loadOnStartup    | 默认-1，让dipatcherServlet懒加载；可以使用设置为1，让其在项目启动时就加载 |

##### dispatcherServlet组件：

dispatcherServlet会依赖一系列springMVC容器中的Bean，来实现其功能：

| Bean                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| HandlerMapping                        | 将请求、拦截器与对应处理程序进行映射，用于进行请求的预处理和后置处理 |
| HandlerAdapter                        | 帮助DispatcherServlet调用请求映射的处理程序                  |
| HandlerExceptionResolver              | 用于解析异常，将其映射到对应处理程序、HTML错误视图或其他目标中 |
| ViewResolver                          | 视图解析器，用于提供视图名称和视图之间的映射                 |
| LocaleResolver、LocaleContextResolver | 提供国际化视图                                               |
| ThemeResolver                         | 提供web程序多主题布局                                        |
| MultipartResolver                     | 处理post请求中的multipart/form-data类型的表单提交，用于处理二进制数据（文件） |
| FlashMapManager                       | 管理FlashMap对象，实现在重定向时，属性从要给请求传递到另一个请求 |

##### multipart/form-data处理：

​		在Servelt中，对于multipart/form-data的表单数据，只能手动解析body获取，springMVC提供MultipartResolver组件来处理，对HttpServletRequest进行封装，生成MultipartHttpSerletRequest，方便@RequestParam、@RequestPart获取请求数据；springMVC默认没有配置MultipartResolver

​		MultipartResolver配置：

1、在Servelt3.0中，已经提供了对应的解析处理和针对于ServletRequest操作其数据的API。但是需要进行一定的配置，才能开启；springMVC提供standardServletMultipartResolver来使用Servelt3.0这个原生功能

```java
public class WbeConfigSimple extends AbstractAnnotationConfigDispatcherServletInitializer {
	xxxx
	@Override
	protected void customizeRegistration(Dynamic registration) {
        //配置Servelt提供的解析器的本地缓存路径（解析请求中的multipart/form-data数据，需要进行缓存，才能多次操作）
		registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
	}
}
```

**此时，ServletRequest就可以使用getPart，操作Part对象来处理multipart/form-data的表单数据****

```java
	@Bean("standardServletMultipartResolver")
	public MultipartResolver multipartResolverConfig() {
		StandardServletMultipartResolver standardServletMultipartResolver = new StandardServletMultipartResolver();
		return standardServletMultipartResolver;
	}
```

**目前，standardServletMultipartResolver并不需要注册为Bean，dispatcherServelt会自动获取该组件，来处理request请求**

2、使用Commons-FileUpload包实现，springMVC提供CommonsMultipartResolver类来封装使用，需要导入额外的包，在项目、Tomcat不支持Servlet3.0时使用（不进行配置介绍）

#### springMVC过滤器：

Servlet容器提供三大组件：Servelt、Filter、Listener；它们实现了整个Servlet容器对于Web请求的处理

##### 过滤器、监听器配置：

springMVC本身提供dispatcherServelt来拦截处理所有的请求，因此一般情况下，Servlet容器不会进行其他Servelt的配置；而对于Filter、Listener，在springMVC项目下，提供多种配置方式：

- XML配置：

  Servlet容器会默认读取项目下的WEB-INF/web.xml文件，来进行Servelt组件配置；对于**Filter执行顺序，会根据在web.xml中的声明顺序决定**

- 基于Servelt3.0注解配置：

  在Servelt3.0,为了减少xml配置的繁琐，提供了@WebFilter、@WebServlet、@WebListener来配置Servelt组件，对于**Filter执行顺序，会根据@WebFilter所注解的类名自然排序，因此推荐使用Filter01作为前缀来控制**，此时就可以省略web.xml

- springMVC配置：

  springMVC提供**WebApplicationInitializer**、**AbstractAnnotationConfigDispatcherServletInitializer**接口，来获取servelt容器的上下文对象ServeltContext，配置其Servelt组件，对于Filter执行顺序，会根据配置顺序决定

  ```java
  public class WbeConfigSimple extends AbstractAnnotationConfigDispatcherServletInitializer {
  	xxxx	
  	@Override
  	protected Filter[] getServletFilters() {
          //按照数组排序顺序执行
  		return new Filter[] {new MyFilter()};
  	}
  }
  ```

由于Servelt容器，会优先读取WEB-INF/web.xml文件，然后再将配置好的ServeltContext交给springMVC处理，因此其Filter优先级为：

XML配置    >  注解配置  >   springMVC配置，**推荐单独使用一种进行配置，减少不必要的处理**

springMVC配置的过滤器、Servelt会自动开启非懒加载和异步处理模式；而ServeltAPI 方式需要手动配置相应参数

##### springMVC常用过滤器：

​	springMVC提供一套基于Filter接口的过滤器类架构，springMVC在Filter基础上，提供了几个重要的类：

- GenericFilterBean：Filter的子类，提供了Filter访问web.xml文件中Filter标签下的子标签init-param值，是实现Filter的初始化
- OncePerRequestFilter：GenericFilterBean的子类，实现了对于request只进行一次过滤（解决转发时，会导致request再次被过滤器处理的问题）
- AbstractRequestLoggingFilter：OncePerRequestFilter的子类，提供额外beforeRequest、afterRequest方法，提供对过滤前后的编写相应代码（将整个过滤器代码分步化，增强过滤器全局处理请求的能力）

对于这几个类，springMVC提供了几个使用的实现类：

- HiddenHttpMethodFilter：可以将form表单的请求转化为DELETE、PUT（form表单只支持put、post）
- ShallowEtagHeaderFilter：实现对HTTP缓存中Etag请求头的处理
- CorsFilter：实现CORS处理
- CharacterEncodingFilter：实现对请求数据参数的编码处理（**非常重要**），注意get传参数据，会被servlet容器处理，并不会被CharacterEncodingFilter处理；但springMVC在进行参数封装时，会自动根据Servlet容器编码来进行转化（tomcat8.0 默认为utf-8）

#### springMVC拦截器：

​	拦截器是web框架的必备功能，增强开发者对请求全局的控制处理，相对于Servelt api提出的过滤器理念，功能更加强大

##### 拦截器使用：

​	springMVC提供HandlerInterceptor接口，对所有被HandlerMapping处理的请求进行拦截处理：

```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
     	return true;
	} 
    
    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable ModelAndView modelAndView) throws Exception {
        
	}
    
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable Exception ex) throws Exception {
        
	}
```

HandlerInterceptor提供三个方法：

- preHandle：在控制器方法（Handler）执行前执行，返回值为一个布尔类型，true则将请求交给下一拦截器或Handler处理，false则认为请求已被处理，不在进行任何后面处理。可以用于身份认证和授权
- postHandle：在控制器方法返回ModelAndView之前执行，这里并不是控制器方法的实际返回值，springMVC提供对控制器方法各种返回值的支持，但无论是否为Model、View等返回值类型，最后反射调用后都会通过Hander返回值处理器处理，返回一个ModelAndView对象，此时就会开始执行postHandle，可用于对MV对象进行额外处理(**当Handler执行出现异常时，则不会执行postHandle方法**)
- afterCompletion：执行完控制器方法并返回ModelAndView后执行，一般用于对控制层的统一异常处理、日志处理（**无论Handler执行是否发生异常，都会执行该方法**）

**拦截器配置：**

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    	//拦截器配置
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		InterceptorRegistration addInterceptor = registry.addInterceptor(new MyInterceptor());
		addInterceptor.addPathPatterns("/*");//拦截器映射匹配路径
		addInterceptor.excludePathPatterns("/testSync");//拦截器排除路径
	}
}
```

##### 拦截器和过滤器的区别：

1、拦截器基于JAVA反射机制，过滤器是基于函数回调

2、过滤器依赖于Servlet容器，拦截器不依赖

3、拦截器只会对映射了控制器方法的请求进行处理；过滤器会对所有请求进行处理

4、拦截器可以访问springMVC处理请求所提供的各对象（handle、ModelAndView、Exception）；过滤器只能获取Servelt请求的相应对象

5、在一个请求中，只会调用一次过滤器；但对于拦截器，会在控制器中进行映射方法处理器的跳转，从而调用多次拦截器（请求每映射一次处理器，都会调用拦截器）

#### springMVC控制器：

resposeon中的ContentType默认为：text/plain;charset=ISO-8859-1

springMVC提供基于注解的编程模型，来实现控制器功能，处理被前端控制器拦截映射的请求，其方法叫做请求映射处理器（**Handler**）

##### @Controller、@RestController

通过上面两个注解来注册控制器bean，@Controller使用上和spring的@Component一致，通过@ComponentScan来将被@Controller注解的类注册到springMVC容器中

@RestController是一个组合注解，@Controller+@ResponseBody，让当前类中所有的方法都继承@ResponseBody，表示所有方法的返回值都直接写入responseBody中，不进行String类型的视图解析，**对于javaBean，会通过HttpMessageConverter进行数据转化，默认使用JSON序列化的形式，将JAVAbean转化为JSON字符串，并且修改response.ContentType="application/json;charset=UTF-8"**；

##### HttpMessageConverter：

HttpMessageConverter，http消息转化器，有两个使用场景：

1、获取requestBody中的数据，将其转化为javaBean；

2、将带有@ResponseBody注解的Handler的返回值，进行类型转化，并写入到responseBody中，并指定其Content-Type响应头

**springMVC对Content-Type响应头处理：**

对于Content-Type响应头时，会受到请求头中Accept属性的影响，也叫做MediaType。而requestMapping中的produces就是用于设置请求中的Accept属性，其原理为：

​		在springMVC中，response.ContentType会被HttpMessageConverter单独处理，首先根据requestMapping中的produces确定MediaType的范围，然后根据MediaType和javaBean类型，选择合适的HttpMessageConverter，此时来最终确定response.ContentType，因此在springMVC中response.ContentType无法被手动修改

springMVC提供多个默认HttpMessageConverter，常用有两个：

- MappingJackson2HttpMessageConverter：支持javaType=Object、MediaType=“application/json；charset=UTF-8”；但需要导入Jackson的jar包（jackson-databind）
- StringHttpMessageConverter：支持javaType=String、MediaType=="text/plain;charset=ISO-8859-1"；

由于StringHttpMessageConverter的先声明，因此对于String类型的javaBean，优先被StringHttpMessageConverter处理，这样就会导致String类型无法被json序列化，并导致中文乱码，对于这种问题有两种解决方法：

1、我们可以通过@RequestMapping的Produces属性，来控制跳过StringHttpMessageConverter，选择MappingJackson2HttpMessageConverter进行处理；

2、StringHttpMessageConverter的supportedMediaTypes属性为“application/json；charset=UTF-8”

##### 映射注解

springMVC提供**@RequestMapping**注解，将请求映射到指定控制器方法中；并提供各种属性，通过URL、http方法、请求参数、Headers来进行匹配

@RequestMapping属性：

| 属性名   | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| name     | 该控制器的映射Name                                           |
| value    | path的别名                                                   |
| path     | 所映射请求的url，支持相对路径匹配、REST风格传参              |
| method   | HTTP方法，RequestMethod[]，可以匹配多个HTTP方法              |
| params   | 请求参数列表，不是固定匹配，包括所有就满足，**即请求参数中必须包含当前规定的所有参数** |
| produces | 指定处理后响应头中，Content-Type类型值，**通过该属性可以定义响应数据的编码格式** |
| headers  | 请求中必须包含指定请求头                                     |

springMVC还提供@RequestMapping多个变体注解，用于方便定义Http方法类型：

**@GetMapping、@PostMapping**、、、

注意：@RequestMapping可以注解在类上，定义所有方法的映射规则，此时还可以使用@RequestMapping注解在方法上，来进一步缩小映射匹配范围：

- 对于Path，会将类和方法上的值进行组合：

```java
@RestController
@RequestMapping(path="/data")
public class DataController {
	
    @RequestMapping(path="/test")
	public String Test() {
		return "succsee";
	}
```

**此时实际请求映射的Path值为 /data/test**

- 对于method、params、produces、headers，方法级别会覆盖类级别

##### 方法参数注解

​	springMVC提供一系列方法参数注解，将springMVC封装好的Http请求数据，注入到控制器方法形参中；在@Controller注解的类被注册到springMVC容器后，会进行自动装配；当其映射方法被执行时，会通过方法参数中的注解，将请求中的参数对应传入，实现对请求数据的自动获取

| 注解              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| @PathVariable     | 获取URL中使用REST风格的传参值                                |
| @MatrixVariable   | 获取URL中矩阵变量传参值（一般不使用）                        |
| @RequestParam     | 获取Servlet中的请求参数（url传参、form-data传参），springmvc对于文件上传，提供MultipartFile对象进行接收（File无法接收） |
| @RequestHeader    | 获取请求头参数值                                             |
| @CookieValue      | 获取Cookie中的参数值                                         |
| @RequestBody      | 获取请求Body中的内容，并通过HttpMessageConverter根据请求头中的Content-Type对内容进行解析，转化为java对象（一般用于json反序列化为java对象） |
| @RequestPart      | 将multipart/form-data中的json数据直接反序列化为java对象（避免后台手动反序列化，但是需要前端定义当前数据的content-type=application/json） |
| @ModelAttribute   | 将所有URL获取的参数和form-data获取的参数，绑定到java对象中；必须保证java对象提供要给无参构造方法，无需保证至少有一个属性注入 |
| @SessionAttribute | 获取HttpSession对象中的属性，如sessionID                     |
| @RequestAttribute | 获取HttpServeltRequest对象中的属性（用于请求转发，但在前后端分离的项目中不会使用） |

- 除@ModelAttribute外，所有方法参数注解都有一个**required**属性，默认为true，即不能够忽略：

  ​	当请求参数中不包含当前指定的参数时，springMVC就会抛出异常**响应400**，提示当前类型的参数没有被绑定，即请求中没有包含当前数据；当required属性为false时，springMVC就会忽略没有绑定的参数，默认为null

- 以@RequestParam为例，方法参数注解的使用：

  ​	@RequestParam提供name、value两个属性值，来对应获取http请求数据中的参数名，从而获取其参数值；**当不进行两个属性值声明时，默认使用方法形参名匹配**

  ​	@RequestParam单独还提供defaultValue属性，定义当参数没有被绑定时的默认值（不设置，则默认为null）；并且defaultValue只能提供String类型值，需要保证所对应形参类型可以进行有效转化

- 当映射方法参数没有使用注解时，首先会与springMVC默认定义的一些早期值进行匹配，当没有匹配值时，则通过BeanUtils.isSimpleProperty方法判断当前参数类型是否为简单数据类型；是，则自动添加一个属性required=false的@RequestParam注解；不是，则自动添加@ModelAttribute注解

- @RequestParam和@RequestParth都可以获取multipart/form-data数据，但是两者有区别：

  @RequestParam只能将text解析为String类型

  @RequestParth还可以解析如json字符串，将其转化为javaBean；但需要配合数据Content-Type值来解析（并不是请求头中的Content-Type），因此一般不使用，这样只是让后台代码更加优雅，但是增加了不必要的前后端传参定义规范

##### 方法参数早期匹配值

​		spring提供一系列对象，方便操作servelt API；这些值的注入不需要特殊注解，springMVC会在早期，根据参数类型进行匹配注入：

| 对象类型                                                    | 作用                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| WebRequest、NativeWebRequest                                | springMVC封装好的对象，用于简化servletAPI操作，如对请求参数、请求属性和会话属性的访问 |
| javax.servlet.ServletRequest、javax.servlet.ServletResponse | 获取Servlet API提供的请求和响应对象                          |
| javax.servlet.http.HttpSession                              | 获取Serlvet API提供的请求会话对象；该对象的线程不安全，因此要避免业务层缓存数据放在Session属性中 |
| HttpMethod                                                  | HttpMethod的枚举值                                           |
| java.io.InputStream、java.io.Reader                         | 获取Servlet API的 请求输入流                                 |
| java.io.OutputStream、java.io.Writer                        | 获取Servlet API的 响应输出流                                 |
| java.util.Locale                                            | 当前请求的语言环境（Http请求并不会带有当前客户端的语言环境，需要通过LocaleContextResolver来解析请求中的特定数据，来设置） |
| java.util.TimeZone、java.time.ZoneId                        | 当前请求的时区（同上）                                       |
| HttpEntity<B>                                               | 获取请求中的body和headers，使用                              |
| UriComponentsBuilder                                        | 用于构造和编码URI                                            |
| Errors、BindingResult                                       | 用于获取springMVC参数绑定的错误信息，需要保证它们在所有绑定数据的最后 |
| javax.servlet.http.PushBuilder                              | 获取Servlet 4.0 提供用于访问HTTP/2资源推送的对象             |
| java.security.Principal                                     | 当使用spring-security时，获取用于验证用户身份的Principal实现类 |

另外有关视图解析、重定向的对象，无需了解

##### 方法返回值和相关注解

映射方法返回值用于定义springMVC处理请求后，响应数据的传递方式和响应信息的设置：

| 返回值类型                                                   | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| HttpEntity<B>、ResponseEntity<B>                             | 用于定义完整的响应信息（body与Headers），返回值会通过HttpMessageConverter写入Response对象中 |
| HttpHeaders                                                  | 定义Headers信息，即无正文数据                                |
| String、View、java.util.Map、org.springframework.ui.Model、ModelAndView | 定义视图、视图模型数据（前后端分离不会使用）                 |
| void                                                         | 默认响应已经被处理，若没有则会抛出异常响应500                |

除了常规响应信息外，还提供对异步请求、HTTP流的处理，在此不进行了解

​	springMVC提供@ResponseBody注解，使用**HttpMessageConverter**将返回值转化为ResponseBody，默认使用jackson JSON来实现对象序列化，简单数据类型则转化为String

##### @ExecptionHandler

​		当请求映射处理期间发生异常时（控制层抛出异常），DispatcherServlet会将异常交给一系列HandlerExceptionResolver类型Bean，以责任链模式进行异常处理，即**请求错误响应**

springMVC提供如下HandlerExceptionResolver实现类：

| HandlerExceptionResolver实现类    | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| SimpleMappingExceptionResolver    | 异常类名称和错误视图名称之间的映射，定义响应错误页面（一般用于前后端不分离项目，使用Servlet进行页面跳转） |
| DefaultHandlerExceptionResolver   | springMVC默认提供的异常解析器，解决所有springMVC引发的异常，并映射响应对应的HTTP状态代码；它的解析结果可以被ResponseEntityExceptionHandler、REST API异常所覆盖 |
| ResponseStatusExceptionResolver   | 提供@ResponseStatus注解，来映射指定HTTP状态代码              |
| ExceptionHandlerExceptionResolver | 提供@ExceptionHandler注解，来定义指定异常的处理方法          |

我们可以自定义多个HandlerExceptionResolver类型Bean，根据order来设置它们的优先级，即在责任链中的优先级；当所有HandlerExceptionResolver都没有解决该异常时，则交给servlet容器处理

**springMVC推荐使用@ExceptionHandler来定义异常处理：**

```java
@RestController
public class TestController {
    
	@ExceptionHandler(Exception.class)
	public String exceptionHandler(Exception exception) {
		return exception.toString();
	}
    
    @RequestMapping("/test")
	public String  test() {
		int a =1/0;
        return "hh";
    }
}
```

**@ExceptionHandler注解的异常处理器只能作用当前Controller类中**

##### @ModelAttribute

​	**dispatcherServlet在处理每个映射请求时，都提供了一个Model对象，用于作为数据中间层，存储和访问数据**；**在前后端不分离的情况下，Model对象可以被视图层访问**

​	@ModelAttribute注解就是用于开发者操作Model对象数据，它可以注解在三种位置：

- 映射方法上，将当前方法的返回值作为Model中数据（前后端分离不会使用）
- 映射方法参数上，首先根据Model中的已有数据进行Name匹配注入，不存在匹配值则会将参数放入Model中，使用**WebDataBinder**进行参数绑定；（**可以直接通过org.springframework.ui.Model进行早期值参数注入，获取Model对象**）
- 控制器的非映射方法上，用于执行@RequestMapping方法前，初始化Model对象

```java
	@ModelAttribute("userA")
	public User initModel(String id,String name) {
		User user = new User();
		user.setId(id);
		user.setName(name);
		return user;
	}

	@RequestMapping("/test")
	public String  test(Model model) {//通过Model对象获取其中数据
		User user = (User)model.getAttribute("userA");
		System.out.println(user.toString());
        return "hh";
    }

	@RequestMapping("/testA")
	public String  test(@ModelAttribute("userA") User user) {//直接匹配Model对象中的userA参数镀锡，进行注入
		xxx
    }
	
	@RequestMapping("/testA")
	public String  test(@ModelAttribute User user) {//此时会通过WebDataBinder，获取servelt参数值，进行参数绑定
		xxx
    }
```

##### @InitBinder

​	@InitBinder用于初始化当前控制器的WebDataBinder，该对象有如下作用：

- 将请求参数绑定到Model对象中
- 将所有的String类型请求数据（请求参数、url、Headers、Cookie等）转化为控制器方法参数
- 将Model对象中的数据转化为String值

```java
    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }
```

通过 @InitBinder 可以定义WebDataBinder 一些基本属性，如date转化器

##### @ ControllerAdvice

​	@ControllerAdvice为增强版控制器，用于全局处理请求数据，搭配@ExceptionHandler、@ModelAttribut、@InitBinder使用；

**@ControllerAdvice并不是@Controller的子注解，并不能搭配@requestMapping使用，实现控制器功能；而是@Component子注解，会被注册到spring容器中**

- 全局异常处理：

  @ControllerAdvice和@ExecptionHandler搭配使用，全局处理控制层抛出的所有异常；当发生发生指定异常时，会被相应Handler方法处理，从而指定当前请求的响应内容

  ```java
  @ControllerAdvice
  public class CustomExceptionHandler {
  
      //@ExceptionHandler注解的方法，会自动注入Exception类型参数，即全局控制层抛出的异常对象
  	@ResponseBody
  	@ExceptionHandler(Exception.class)
  	public String exceptionHandler(Exception exception) {
  		return exception.toString();
  	}
  }
  ```

- 全局请求参数绑定

  @ControllerAdvice和@ModelAttribut搭配使用，全局处理所有请求的参数绑定；一般情况下，优先作用局部方法，不使用默认全局值，因为参数绑定的通用性不强，以免干扰过多不相干的映射处理器

- 全局初始化WebDataBinder 

  @ControllerAdvice和@InitBinder 搭配使用，全局初始化WebDataBinder对象，和@ModelAttribut一样，优先作用局部方法

@ControllerAdvice默认作用所有@Controller，但是可以通过指定basePackages，来所需作用范围

#### springMVC其他功能：

##### URI处理：

​	springMVC提供UriComponentsBuilder工具类，来构建和编码URI

##### 异步请求处理：

​	在Servlet3.0后，使服务器支持request异步请求处理，使用线程池技术，来多线程异步处理请求，而不是单线程阻塞的形式，导致服务器并发量降低

- Servelt异步请求处理方式：

  1、request.startAsync() 将当前Servelt请求设置为异步处理模式，从而将当前Servelt线程释放，但响应处于处理状态

  2、并且request.startAsync()方法会返回一个AsyncContext对象，用于缓存当前请求对于的reques、response对象，并存放在application作用域中；当请求完成响应后，则会重新将其reques、response对象交给Servelt容器

  3、Servelt会存在一个异步线程池，用于执行所有被延迟响应的异步请求；此时servelt容器就会异步线程池中的servelt线程处理该请求，完成请求响应

​	springMVC提供两种方式来封装实现Servelt异步请求处理：

- DeferredResult：

  springMVC提供DeferredResult对象，来搭配控制器实现Servelt异步请求处理；我们需要手动创建新线程，来执行异步代码，将结果放入DeferredResult对象，进行响应

  ```java
  @RequestMapping("/testSync")
  	public DeferredResult<String> syncTask(){
          xxxxx
  		//定义异步结果集，定义异步处理超时时间
  		DeferredResult<String> deferredResult = new DeferredResult<String>((long) 11000);
  		Runnable runnable = new Runnable() {
  			@Override
  			public void run() {
  				try {
  					Thread.sleep(10000);
  				} catch (InterruptedException e) {
  					e.printStackTrace();
  				}
  				deferredResult.setResult("hhhhSync");
  			}
  		};
  		Thread thread = new Thread(runnable);
  		thread.start();
  		return deferredResult;
  	}
  ```

  DeferredResult的工作过程：

  - 控制器返回DeferredResult对象
  - springMVC调用request.startAsync() ，此时当前servelt线程释放（处理当前请求的线程）
  - DeferredResult被其他线程处理，并执行setResult方法
  - 此时springMVC将请求重新交给Servelt容器管理，从而获取一个servelt线程，处理当前请求

- Callable：

  springMVC还支持JDK提供的java.util.concurrent.Callable作为处理器返回值，springMVC会使用一个线程池来处理控制器所返回的Callable任务；springMVC默认提供SimpleAsyncTaskExecutor对象作为线程池，但是它实际并不是一个线程池，每次执行任务都需要创建一个新的线程；

  因此我们需要进行手动配置springMVC的异步线程池，并且可以设置异步请求默认超时时间

  springMVC提供多个AsyncTaskExecutor实现类，推荐使用ThreadPoolTaskExecutor，它是对java.util.concurrent.ThreadPoolExecutor的封装

   ```java
  @Configuration
  public class MvcConfig implements WebMvcConfigurer{	
  	@Override
  	public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
  		ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
          threadPoolTaskExecutor.initialize();
  		configurer.setTaskExecutor(threadPoolTaskExecutor);
  		configurer.setDefaultTimeout(10000);
  	}
  }
   ```

  ```java
  @RequestMapping("/testSyncA")
  	public Callable<String> syncTaskA(){
  		xxxx
          return new Callable<String>() {
  			@Override
  			public String call() throws Exception {
  				try {
  					Thread.sleep(10000);
  				} catch (InterruptedException e) {
  					e.printStackTrace();
  				}
  				return "hhhhSyncA";
  			}
  		};
  	}
  ```

  Callable的工作过程：

  - 控制器返回Callable
  - springMVC调用request .startAsync，此时当前servelt线程释放（处理当前请求的线程），并将Callable对象交给AsyncTaskExecutor线程池执行
  - 当Callable执行完成，返回结果后，springMVC再次将请求交给Servlet容器处理，从而获取一个servelt线程，处理当前请求

- 在进行异步请求处理过程中，HTTP客户端会有一个Read超时时间（即HTTP响应时间）；而服务端也可以设置自己处理请求的超时时间，但一般情况下，该超时时间要小于客户端Read超时时间

- **注意，异步请求需要保证处理请求的Servert和Filter都设置为可异步处理：asyncSupported=true**

##### CORS：

​	springMVC的handlerMapping实现了对CORS内置支持，当请求被映射到相应处理器后，HandlerMapping会对请求进行CORS拦截处理，并且开发者可以为url-pattren或每个映射的url设置单独的CORS配置：

**全局配置：**

在**WebMvcConfigurer** 接口中，提供**addCorsMappings**回调方法，来设置指定映射url模板的CORS配置

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {	
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/*")//拦截url-pattern 
			.allowedOrigins("*")	//允许的跨域请求，* 为允许所有(浏览器对于跨域请求，会自动添加origin请求头，value为当前项目的ip+端口)
			.allowedHeaders("*") //允许的请求头
			.allowedMethods("*")  //允许的HTTP请求方法
			.allowCredentials(true) //允许请求发送cookie等安全证书
			.maxAge(3600);//预检请求有效期 单位s
	}
```

**局部配置：**

spring4.2后，提供@CrossOrigin注解，进行局部控制器的CROS处理

@CrossOrigin可以注解在类或方法上，默认使控制器支持跨域请求处理：

@CrossOrigin属性

| 属性名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| origins          | 指定允许的跨域请求地址，默认为空，允许所有                   |
| allowedHeaders   | 指定允许的请求头，默认空，但CROS规范默认允许一些请求头如：Content-type |
| exposedHeaders   | 指定要暴露的请求头，默认为空                                 |
| methods          | 指定允许的http请求方法，默认为空，为当前控制器方法允许值     |
| allowCredentials | 允许请求发送cookie等安全证书，默认不启用                     |
| maxAge           | 预检请求有效期，默认-1 永不过期                              |

**局部配置会覆盖全局配置**

**当然springMVC也支持原生CROS解决方案，使用CROS过滤器,并且springMVC提供更加简便的配置方法**

##### 网络安全：

**springMVC提供整合spring Security框架，来实现web项目的权限和网络安全**

##### HTTP缓存：

​	HTTP缓存属于客户端缓存，即浏览器缓存，它是基于HTTP协议存在的，并分为两种：强制缓存和协商缓存

HTTP缓存作用过程：

1. 浏览器发出请求，首先判断是否命中强制缓存，命中则从浏览器缓存中读取资源
2. 如果没有命中则将资源请求发送给服务器，此时服务器判断客户端的本地缓存是否失效
3. 没有失效，则服务器不返回任何信息，浏览器转而从缓存中加载资源
4. 如果失效，服务器就会返回完成资源信息，交给浏览器加载

HTTP缓存设置：

通过修改HTML meta标签，来关闭浏览器缓存；

```java
<META HTTP-EQUIV="Pragma" CONTENT="no-store">
```

- 强制缓存：

  当强制缓存命中后，请求会直接返回200，Size会显式from cache；强制缓存根据HTTP协议版本，分两个响应请求头控制：Expires、Cache-Control；它们代表了当前服务器返回资源的缓存时间

  Expires为HTTP1.0定义，Cache-Control为1HTTP1.1定义；它们向上兼容，因此Cache-Control的优先级更高

- 协商缓存

  协商缓存相对于强制缓存，是将浏览器缓存的决定权始终交给服务器完成，当第一次请求被服务器处理时，会在响应头中添加一个缓存标识，再第二请求时，服务器通过判断缓存标识，来决定是否读取缓存数据

  在HTTP1.0中,通过Last-Modified响应头来记录资源上一次访问时间，第二次请求时，就会使用If-Modified-Sinc请求头带上该时间，此时服务器就可以通过该请求头时间，进行逻辑代码处理，判断是否需要使用浏览器缓存；当使用，则直接返回304状态码；不使用，则返回新资源，并使用Last-Modified响应头记录资源访问新时间

  HTTP1.1中提供Etag来作用缓存标识，If-None-mactch作为请求是否使用缓存标识

springMVC提供CacheControl对象，来实现服务器端对Cache-Control响应头的处理，并在控制器返回值中，进行Cache-Control、Etag处理

##### FreeMarker：

​	springMVC内部集成了Apache FreeMarker模板引擎，来配置web项目视图及其数据访问

#### springMVC配置：

​		springMVC提供**WebMvcConfigurer**接口，让开发者对springMVC中的内部组件进行配置，常用包括：

```java
public interface WebMvcConfigurer {
    //url路径匹配解析
    default void configurePathMatch(PathMatchConfigurer configurer) {
	}
    
    //内容类型解析
    default void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
	}
    
    //异步请求处理配置
    default void configureAsyncSupport(AsyncSupportConfigurer configurer) {
	}
    
    //默认servetl处理，优先级最低，当没有可匹配的Servet时生效
    default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
	}
    
    //数字、日期格式转化器，通过@NumberFormat、@DateTimeFormat使用，在javaBean接收请求参数时，进行相应类型转化
    default void addFormatters(FormatterRegistry registry) {
	}
    
    //配置拦截器
    default void addInterceptors(InterceptorRegistry registry) {
	}
    //静态资源映射
    default void addResourceHandlers(ResourceHandlerRegistry registry) {
	}
    //CORS配置
    default void addCorsMappings(CorsRegistry registry) {
	}
    //视图控制器，定义指定处理器返回值对应视图
    default void addViewControllers(ViewControllerRegistry registry) {
	}
    //视图解析器，实现对视图解析和数据渲染
    default void configureViewResolvers(ViewResolverRegistry registry) {
	}
    //Handler参数解析器
    default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
	}
    //Handler返回值处理器
    default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
	}
    
    //消息转化器，用于处理request、response的Body数据
    default void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
	}
    //
    default void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
	}
    
    //异常处理器
    default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
	}
    default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
	}
    
    //用于控制器参数校验，通过@Valid、Validate字段定义使用
    @Nullable
	default Validator getValidator() {
		return null;
	}
    
    //响应码解析器，生成对应错误的响应码
    @Nullable
	default MessageCodesResolver getMessageCodesResolver() {
		return null;
	}
}
```

当然springMVC提供了一个**WebMvcConfigurer**接口的具体实现类：**DelegatingWebMvcConfiguration**，为springMVC提供默认配置，在在其基础上进行扩展、删除

#### springMVC请求处理流程：

![](C:\Users\OneMTime\Desktop\Typora图片\springMVC请求处理流程.png)

过程如下：

- Servlet容器启动，初始化Listener、Filter、Servlet，完成整个Servlet容器上下文的配置

  在初始化Servelt时，就会实例化DispatcherServelt，从而创建springIOC容器、初始化springMVC组件：

  包括对控制器中映射处理器方法的获取，保存到HandlerMapping中

- 当用户发送请求后：
  1. 首先被Servlet容器获取，进行Filter处理
  2. 在交给url映射的Servlet中（springMVC的DispatcherServelt一般拦截所有请求）
- DispatcherServelt将拦截到的请求进行处理：
  1. 通过HandlerMapping，找到当前请求的Handler
  2. 根据Handler类型，找到对应的HandlerAdapter
  3. 调用拦截器preHandler方法，对请求进行前置处理
  4. 使用HandlerAdapter对根据Handler对象，对控制器方法进行反射调用

- Handler的执行过程：
  1. 通过Handler参数解析器，完成控制器方法的参数注入（此时回对请求的request对象进行处理）
  2. 反射调用控制器方法
  3. 通过Handler返回值处理器，对控制器方法返回值进行处理（此时会对请求的response对象进行处理）
  4. 最后返回ModelAndView对象

- Handler执行后，DispatcherServlet进行整个处理结果进行进一步处理：（期间当出现异常，则调用springMVC异常处理器，来响应请求）
  1. 调用拦截器的postHandler方法，对请求ModelAndView对象进行处理
  2. 通过视图解析器，完成对MV对象的解析，找到相应视图
  3. 然后将MV对象中的Model数据，渲染到视图中
  4. 在响应请求前，调用拦截器的afterCompletion
  5. 最后响应请求

**对应前后端分离项目，一般不会进行MV对象的视图解析，即直接使用@ResponseBody，将控制器方法的返回值以json形式写入响应Body中，而Handler会返回一个null的MV对象**

#### DispatcherServelt源码解析：

DispatcherServelt的继承了一个HttpServelt，当请求被其拦截后，执行其doGet、doPost方法。从而调用DispatcherServelt核心入口方法**doDispatch**:

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
                //检查请求是否存在二进制文件表单提交，并进行缓存处理
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

                //在DispatcherServelt初始化时，会自动完成以下操作：
				//1、获取IOC容器在所有的@Controller注解类
                //2、遍历其所有方法，获取带有@RequestMapping注解的方法
                //3、创建一个Map集合，以@RequestMapping的value作为Key，其方法对应mothod对象为Value（通过HandlerMapping实现）
                	
                //根据请求rul，通过HandlerMapping从Map集合中获取对应的Handler
                //HandlerMapping有多个，对应不同的控制器定义方式：@Cotroller、Cotroller接口、HttpRequestHandler，每个HandlerMapping都有一个Map存储handler和url映射关系
				mappedHandler = getHandler(processedRequest);
                //若没有找到对应Handler，则直接响应404
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				//将获取该Handler对应的HanlerAdapter，用于调用Handler处理请求
                //同样HanlerAdapter根据控制器的定义方式，也有多个不同的实现，Handler可以是一个类，也可以是一个方法（@Controller定义的）；根据Handler的不同类型，来使用不同的调用方式
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				//如果为GET请求，则默认支持对last-modified请求头的处理，实现HTTP协商缓存
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

                //执行拦截器的preHandler方法，并是否直接拦截请求
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				//通过HanlerAdapter实际调用Hanler
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

                //判断请求是否需要进行异步处理，则先返回释放Servelt线程
				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

                //对Hanler执行结果进行默认视图解析
				applyDefaultViewName(processedRequest, mv);
                //执行拦截器的postHandler方法，对Hanler结果进行进一步处理
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
                //dispatchException会被全局异常处理器处理（@ExceptionHandler）
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            //完成最后的Disparch处理：异常处理、视图处理、拦截器afterCompletion方法执行
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				//如果进行异步请求处理，则执行异步拦截器的afterConcurrentHandlingStarted方法
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// 清空该请求的multipart缓存
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

### REST客户端

springMVC提供两种REST客户端API调用：

#### RestTemplate：

**以同步请求方式执行REST客户端服务通信，默认是对JDK中java.net包的封装，提供更加方便、高效的API**

##### RestTmplate实例化：

​	RestTmplate默认构造方法，底层使用java.net.HttpURLConnection进行HTTP请求，开发者可以通过ClientHttpRequestFactory接口参数，来通过构造方法来进行配置，springMVC提供如下HTTP客户端库的支持：

- HttpComponentsClientHttpRequestFactory ：用于配置Apache HttpComponents包
- Netty4ClientHttpRequestFactory：用于配置Netty4 包
- OkHttp3ClientHttpRequestFactory：用于配置OkHttp包
- SimpleClientHttpRequestFactory：默认使用，用于配置java.net包

##### RestTemplate常用方法：

| 方法          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| getForObject  | GET请求，获取响应数据，并使用消息转化器，转化为指定JavaBean  |
| getForEntity  | GET请求，获取响应封装对象ResponseEntity（包括响应码、Headers、Body） |
| postForObject | POST请求，获取响应数据，并使用消息转化器，转化为指定JavaBean |
| postForEntity | POST请求，获取响应封装对象ResponseEntity（包括响应码、Headers、Body） |
| exchange      | 请求通用方法，必须使用RequestEntity参数（包括HTTP方法，URL，标头和正文作为输入），获取获取响应封装对象ResponseEntity |

- 对于RestTemplate方法中的url，可以使用字符串，并提供REST风格的传参方式（也可以通过UriComponentsBuilder来创建uri，阅读性更高）

```java
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");

Map<String, String> vars = Collections.singletonMap("hotel", "42");
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

- RestTempate方法的返回值有两种，javaBean和ResponseEntity，根据方法参数中的responseType来确定它们的泛型
- springMVC提供HttpEntity对象，来定义请求中的HTTP方法，URL，标头和正文，并且使用消息转化器将javaBean转化为对应ContentType类型数据

##### RestTemplate常用场景：

- 简单Get请求：

  ```java
  String result = restTemplate.getForObject(uri, String.class);
  ```

- 自定义请求头的Get请求：

  ```java
  HttpHeaders headers = new HttpHeaders();
  httpHeaders.setOrigin("xxx");
  HttpEntity<T> httpEntity = new HttpEntity<T>(headers);
  String result = restTemplate.exchange(url, HttpMethod.GET, httpEntity, String.class)
  ```

- Body传JSON数据的Post请求：

  ```java
  HttpHeaders headers = new HttpHeaders();
  headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
  User user = new User();
  HttpEntity<User> httpEntity = new HttpEntity<User>(user);
  String result = restTemplate.postForObject(url, httpEntity, String.class);
  ```

- 表单传参的Post请求：

  ```java
  HttpHeaders httpHeaders = new HttpHeaders();
  httpHeaders.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
  MultiValueMap<String, String> map= new LinkedMultiValueMap<>();
  map.add("id", "1");
  HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(map, headers);
  String result = restTemplate.postForObject(url, httpEntity, String.class);
  ```

- 表单传二进制数据的Post请求：

  ```java
  HttpHeaders httpHeaders = new HttpHeaders();
  httpHeaders.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
  MultiValueMap<String, Object> map= new LinkedMultiValueMap<>();
  map.add("file", new File("xxx"));
  HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(map, headers);
  String result = restTemplate.postForObject(url, httpEntity, String.class);
  ```

#### WebClient：

## 5、spring数据访问模块

spring数据访问模块由如下几个包构成：spring-jdbc、spring-tx、spring-orm、spring-oxm

spring-jdbc：封装jdbc操作，简化使用jdbc对数据库的访问；（依赖spring-tx，提供其功能）

spring-tx：提供对事务的管理，并统一所有DAO层异常类结构

spring-orm：用于spring整合JPA框架，实现对资源、事务的管理和DAO访问（类似于mybatis-spring .jar）

spring-oxm：提供java对象和xml文件之间的映射，提供映射转化一致接口和异常结构，方便使用（目前没有使用需求）

### spring JDBC：

spring JDBC用于简化jdbc操作，减低开发者编写JDBC中的一些简单固定API

#### jdbc操作点的工作分配：

| JDBC操作点         | spring | 开发者 |
| ------------------ | ------ | ------ |
| 定义连接参数       |        | X      |
| 打开连接           | X      |        |
| 指定sql语句        |        | X      |
| 声明参数并提供参数 |        | X      |
| 准备和执行sql语句  | X      |        |
| 遍历处理结果集     | X      |        |
| 处理异常           | X      |        |
| 处理事务           | X      |        |
| 关闭资源           | X      |        |

因此可以看出，spring-JDBC包，可以让开发者以非常优雅的方式使用jdbc，进行数据库交互

#### Jdbc核心操作类的使用：

##### JdbcTemplate

​		JdbcTemplate是JDBC包中的核心类，主要完成如下工作：

- 处理资源的创建和释放
- 执行sql语句，并获取结果集
- 捕获JDBC异常，转换为DataAccessException

**JdbcTemplate是线程安全的，可以用于多个sql执行，在使用时，一个DataSource对应一个JdbcTemplate；JdbcTemplate，每执行依次CRUD操作，都会从连接池中获取connection，但是会优先获取当前线程中创建的connection，当线程工作完成后，则关闭该connection，并且在不使用事务管理器时，固定为自动提交**

可以看出，JdbcTemplate的线程安全和获取connection方式和myabtis中的SqlSessionTemplate一致

###### JdbcTemplate配置：

需要手动配置dataSource、JdbcTemplate，spring-JDBC包中，提供DriverManagerDataSource类，来简单创建DataSource，当然也可以使用**带有连接池功能第三方dataSource**

```java
@Component
public class DataSourceConfig {
	@Value("${db1.username}")
	String username;
	
	@Value("${db1.password}")
	String password;
	
	@Value("${db1.url}")
	String url;
	
	@Value("${driver}")
	String driverClassName;

	@Bean
	public DataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource(url, username, password);
		dataSource.setDriverClassName(driverClassName);
		return dataSource;
	}
	
	@Bean
	public JdbcTemplate jdbcTemplate(@Qualifier("dataSource") DataSource dataSource) {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		return jdbcTemplate;
	}
}
```

###### JbdcTemplate的使用：

1、查询（select）

- 使用Object和基本数据类型包装类进行接收

```java
queryForObject（sql，String.class）//查询一个数据

Map<String,Object> map = queryForMap（sql）//查询一条数据

List<Map<String,Object>> list= queryForList（sql）//查询一组数据
```

- 使用实体对象，手动映射接收

```java
//查询一条数据
User query = jdbcTemplate.query("sql...",new ResultSetExtractor<User>() {
	@Override
	public User extractData(ResultSet rs) throws SQLException, DataAccessException {
		User user = new User();
		user.setName(rs.getString(""));
		user.setPassword(rs.getString(""));
		return user;
		}
	});

//查询一组数据
List<User> query2 = jdbcTemplate.query("sql...",new RowMapper<User>() {
	@Override
	public User mapRow(ResultSet rs, int rowNum) throws SQLException {
		User user = new User();
		user.setName(rs.getString(""));
		user.setPassword(rs.getString(""));
		return user;
		}
	});
```

对于ResultSetExtractor<T>、RowMapper<T>，可以使用Lambda表达式进行简化，并在当前DAO中，jdbcTemplate多个query使用相同的ResultSetExtractor<T>、RowMapper<T>，可以将其提取到dao类的成员变量中，供多个方法使用：

```java
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
    Actor actor = new Actor();
    actor.setName(resultSet.getString("name"));
    actor.setPassword(resultSet.getString("password"));
    return actor;
};
```

**注意：**

- 当使用queryForMap、queryForList方法，将结果集封装为map时，我们使用列名进行数据取出时，spring已经对结果集中的数据进行了提取封装，因此只能使用sql中定义的大小写格式获取（**oracle在不使用双引号情况下，默认全部大写**）

- 当使用ResultSetExtractor、RowMapper进行实体对象映射时，使用resultSet.getString获取数据，不区分大小写（结果集获取数据不需要区分列名的大小写）

2、更新（insert、update、delete）

```java
int update = jdbcTemplate.update("sql...");
```

3、执行存储过程

```java
jdbcTemplate.execute("sql...");
```

3、绑定参数

**jdbcTemplate默认使用PreparedStatement**，来进行sql语句的预编码，并绑定参数;按照sql中的占位符顺序，在参数列表最后面，依次添加参数

```java
jdbcTemplate.queryForMap("selsect * from user where id=?",1);
```

4、单次批处理操作

提供一个BatchPreparedStatementSetter接口，来定义批处理的执行规则

```java
public void batchUpdate(final List<User> users) {		
		jdbcTemplate.batchUpdate("update User set name=? where id=?", new BatchPreparedStatementSetter() {
			@Override
			public void setValues(PreparedStatement ps, int i) throws SQLException {
                //遍历定义所有sql语句参数
				User user = users.get(i);
				ps.setString(1, user.getName());
				ps.setString(2, user.getId());
			}
			
			@Override
			public int getBatchSize() {
				return users.size();//批处理数
			}
		})
	}
```

或使用spring提供的SqlParameterSourceUtils来创建SqlParameterSource接口实现类数组，定义批处理对象

```java
//创建的SqlParmeterSource的实现类数组会在内部批处理绑定所有sql参数；但要保证有批处理需要的list集合
paramJdbcTemplate.batchUpdate("update User set name=? where id=?",SqlParameterSourceUtils.createBatch(users));
```

5、多次批处理

提供ParameterizedPreparedStatementSetter接口来定义

```java
jdbcTemplate.batchUpdate("update User set name=? where id=?",users,100,new ParameterizedPreparedStatementSetter<User>() {
	@Override
	public void setValues(PreparedStatement ps, User user) throws SQLException {
		ps.setString(1, user.getName());
		ps.setString(2, user.getId());
	}
});
```

##### NameParameterJdbcTemplate

​		该类对JdbcTemplate进行进一步封装，提供了一种新的绑定参数的编程方式(本质上还是使用了sql占位符)，解决了JdbcTemplateAPI使用时的弊端：

**jdbcTemplate在参数很多时，只能按顺序一一对应sql占位符，代码阅读性、维护性非常低**

1、map方式绑定参数：

```java
String sql="select * from where id=:id";
Map<String, Integer> singletonMap = Collections.singletonMap("id",1);
paramJdbcTemplate.queryForList(sql, singletonMap);
```

2、javaBean方式绑定参数：提供BeanPropertySqlParameterSource接口来实现

```java
String sql="select * from User where name=:name";
User user = new User();
user.setName("yh");
BeanPropertySqlParameterSource beanPropertySqlParameterSource = new BeanPropertySqlParameterSource(user);
List<Map<String, Object>> queryForList = paramJdbcTemplate.queryForList(sql, beanPropertySqlParameterSource);
```

##### SimpleJdbcInsert、SimpleJdbcCall（一般不使用）

​		这两个类可以通过JDBC驱动来操作数据库中的元数据，从而简化对insert操作和存储过程的调用：

1、简化insert插入表数据

2、insert时使用主键自增

3、简化调用存储过程，并处理返回值

​		使用注意：由于这两个类使用线程不安全，因此一般情况下，直接通过dataSource对象，在DAO层的初始化方法中创建，即因此DAO层进行数据访问都需要创建一个新对象

### spring tx：

​		spring tx包实现spring对事务的管理，并统一所有DAO层异常类结构

#### 正常本地事务和全局事务的局限(不使用spring事务管理)：

本地事务：单个特定资源的事务，如JDBC事务；它无法跨多个事务资源工作，并且在使用上会入侵编程代码

全局事务：javaEE规范中提供了JTA（Java Transaction API），来管理多个事务资源协调工作，如关系型数据库和消息队列。但是JTA非常繁琐，通常还需要使用JNDI从J2EE容器中获取外部数据源

spring事务提供了TransactionManager接口，来定义事务管理器，分为两种实现方式：

声明式事务管理：PlatformTransactionManager

反应式事务管理：ReactiveTransactionManager（spring5.2开始提供）

#### PlatformTransactionManager：

##### 声明式事务管理器基本概念：

```java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

TransactionDefinition接口定义了事务的基本属性，来描述当前事务：

- 事务传播行为
- 事务隔离级别
- 事务超时时间
- 是否为只读事务

TransactionStatus接口提供控制事务执行和查询事务状态的简单方法：

```java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {

    @Override
    boolean isNewTransaction();//是否为新创建的事务（事务嵌套时，判断是否使用了外层事务）

    boolean hasSavepoint();//是否存在回滚节点（用于嵌套事务）

    @Override
    void setRollbackOnly();//事务回滚

    @Override
    boolean isRollbackOnly();//是否可以是否回滚

    void flush();//事务刷新

    @Override
    boolean isCompleted();//是否完成回滚或提交
}
```

##### 声明式事务管理器的使用及其原理：

1、注册PlatformTransactionManager接口的实现类DataSourceTransactionManager，创建事务管理器

```java
@Bean
public DataSourceTransactionManager txManager(@Qualifier("dataSource") DataSource dataSource) {
	DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(dataSource);
	return dataSourceTransactionManager;
}
```

2、使用AOP来实现事务管理

- 定义指定事务管理器的AOP切面策略，进行切入点的方法增强

  ```java
  <tx:advice id="txAdvice" transaction-manager="txManager">
      <tx:attributes>
      <tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
      <tx:method name="*"/>
      </tx:attributes>
  </tx:advice>
  ```

- 定义切入点，并指定该切入点的切面策略

  ```java
  <aop:config>
      <aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
      <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
  </aop:config>
  ```

**实际上，该AOP切面过程，就是以环绕通知的方式进行切入，对整个过程进行try-catch，当前方法发生异常、或事务管理发生异常时，则进行该方法中的事务回滚;没有异常抛出，则进行事务提交**

##### @Transactional

​		spring提供注解的方式，来更加直接的实现对指定方法的事务管理器的AOP，消除AOP切入点的定义

需要在配置类上添加@EnableTransactionManagement，来使@Transactional生效；或者在spring.xml中，添加**<tx:annotation-driven transaction-manager="txManager"/>**标签；该设置只在当前IOC容器生效，因此在使用spring-MVC时，需要注意该配置应添加到spring ioc容器中

**@Transactional注解属性：**

| 属性                     | 作用描述                                                  |
| ------------------------ | --------------------------------------------------------- |
| value/transactionManager | 指定使用的事务管理器，默认使用Primary的Bean               |
| propagation              | 事务传播，默认为PROPAGATION_REQUIRED                      |
| isolation                | 事务隔离级别，默认为Isolation.DEFAULT                     |
| timeout                  | 事务超时时间，默认为TransactionDefinition.TIMEOUT_DEFAULT |
| readOnly                 | 是否为只读事务，默认为false                               |
| rollbackFor              | 引起事务回滚的异常calss对象数组                           |
| rollbackForClassName     | 引起事务回滚的异常类名数组                                |
| noRollbackFor            | 不会导致事务回滚的异常calss对象数组                       |
| noRollbackForClassName   | 不会导致事务回滚的异常类名数组                            |

所有异常必须为Throwable的子类

##### spring事务的传播行为：

**传播行为值的是当一个事务方法被另一个事务方法调用时，改事务方法改如何进行。因此其配置作用是为了改变事务方法被嵌套在其他事务方法内部时，事务生效方式**

根据实际情况有七种类型，以如下模板为例：

存在两个类分别有一个增删改操作的方法，此时方法A内部调用方法B，我们需要关注方法B的传播行为属性：

| 传播行为属性     | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| REQUIRED（默认） | 当方法A本身有事务时，方法B会使用方法A中的事务<br />当方法A没有事务时，创建一个事务（该事务只会作用于方法B） |
| SUPPORTS         | 当方法A本身有事务时，方法B会使用方法A中的事务<br />当方法A没有事务时，方法B则会非事务执行 |
| MANDATORY        | 当方法A本身有事务时，方法B会使用方法A中的事务<br />当方法A没有事务时，方法B则会抛出异常 |
| NEVER            | 总是非事务执行，当方法A存在事务时，则抛出异常                |
| NESTED           | 当方法A本身有事务时，则使用方法A事务进行嵌套事务（即方法A发生异常，会导致方法B事务回滚，当方法B异常不会影响到方法A<br />当方法A没有事务时，创建一个事务（该事务只会作用于方法B） |
| NOT_SUPPORTED    | 总是非事务执行，当方法A存在事务时，则将其挂起（两个方法不会相互影响） |
| REQUIRES_NEW     | 总是使用新事务执行，当方法A存在事务时，将其挂起（两个方法不会相互影响） |

在进行事务传播中，内部事务（方法B事务）设置参数（隔离级别，超时和只读设置），只有当创建自己事务时，才会生效；否则就继承外部事务（方法A事务）特征；**因此这些参数只有在REQUIRED、REQUIRES_NEW中才有可能生效（创建自己新事务时生效）**

**目前NESTED、NOT_SUPPORTED、REQUIRES_NEW没有测试成功？？？？**

spring多数据源事务管理：

##### spring事务手动操作：

spring提供一个静态方法，获取当前线程中的事务管理器的TransactionStatus，来控制事务执行和查询事务状态

 ```java
TransactionStatus currentTransactionStatus = TransactionAspectSupport.currentTransactionStatus();
 ```

最常用的操作，回滚事务：(**其他方法，查看TransactionStatus接口**)

```java
TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
```

##### spring事务使用注意事项：

###### 1、在同一个类中，非事务方法调用事务方法时，会导致事务失效

- 原因：

事务管理依赖于AOP，spring只会在调用事务方法时才进行AOP动态代理，在执行非事务方法时实际还是使用的原对象调用方法，因此内部调用事务方法时，会使用当前原对象，导致没有触发代理、事务失效

- 解决方式：

1、避免这种情况的方法调用，将事务方法单独写在一个类中（**推荐**）

2、在非事务方法调用事务方式时，先手动获取其类的AOP代理对象，然后调用事务方法

前提：@EnableAspectJAutoProxy(exposeProxy=true)

```java
UserServiceImpl currentProxy = (UserServiceImpl) AopContext.currentProxy();
currentProxy.testA();
```

3、在非事务方法调用事务方式时，利用一个工具类，自动获取目标类的AOP代理对象，然后调用事务方法(**和方法2基本一致**)

```java
ServiceUtil.getUserService().testA()
```

4、将非事务方法添加事务注解，使得其类方法统一进行AOP增强，从而在调用时使用同一个AOP代理对象（**但这样会增加事务管理的成本，在代码量很大的情况下，这种方式不可取**）

###### 2、只读事务和不使用事务的区别：

当只执行单条查询语句时，则不需要使用事务；当执行多条查询语句时，需要通过事务来确保整体数据的一致性，一致性的程度和当前事务的隔离级别有关；比如查询所有数据和所有数据量时，当出现并发两条sql之间数据被其他用户修改时，则导致结果不统一

###### 3、手动回滚对事务传播的影响：

当对内部方法代码进行异常捕捉手动回滚时，由于事务传播原因，当前方法使用的是外部方法事务，当手动在内部方法提交事务后，外部方法会再提交一次，因此会报错事务已被回滚

当对外部方法代码进行异常捕捉手动回滚时，在它们抛出异常时，内部方法会和外部方法一起进行手动回滚；

#### JtaTransactionManager：

JtaTransactionManager用于分布式事务管理器，对JTA进行了进一步的封装，简化了其API的使用，由于JTA只是J2EE规范接口，其实现交给第三方来完成：Atomikos是一个实现了JTA规范的框架，并且不需要在J2EE容器中使用（解决了JTA使用的弊端），用于在单个应用中，解决多个事务资源同步问题（包括多种资源的事务，如JDBC、消息队列）

##### **XA（分布式事务规范）：**

​	定义了全局事务管理器（TM）和局部资源管理器（RM）之间的接口；使得事务管理器可以和多个资源管理器相互通信，来协调不同资源的事务管理，主流的关系型数据库都实现了XA接口

​	XA使用二阶段提交方式来管理全局事务，第一阶段为准备阶段，各个资源管理器将参与事务的资源锁住，并响应回告给事务管理器；第二阶段为提交阶段，事务管理器通知资源管理器进行事务提交；并且**XA不能够进行自动提交**

​	XA的缺点：对于提交方式的准备阶段，维护成本非常高，性能与本地事务相差10倍

##### spring+mybaits+Atomikos的整合使用：

这些框架可以共同实现对多事务资源的同步管理

#### 自定义spring事务注解：

​		我们在使用@Transaction注解时，可以知道一个方法中，只能定义一个事务管理器，当方法中需要调用多个数据源的sql，当前方式就无法满足我们的使用；除了使用本地分布式事务管理器外，我们还可以使用最原始的方式，重新定义一个可以操作多个数据源事务管理器进行提交、回滚的切面，即自定义spring事务注解：

1、自定义事务管理注解类(模仿@Transaction)：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface MyTransaction {

		//事务管理器Name集合
		String[] name() default {""};

		//所有事务管理器的传播行为
		Propagation propagation() default Propagation.REQUIRED;
		
		//所有事务管理器的隔离级别
		Isolation isolation() default Isolation.DEFAULT;

		//所有事务管理器不触发回滚的异常
		Class<? extends Throwable>[] noRollbackFor() default {};
}
```

2、定义处理自定义事务注解所在方法的AOP切面类：

```java
@Component
@Aspect
public class TransactionAop {
	@Autowired
	ApplicationContext context;

	// 定义公共切入点(声明被该注解标记的方法)
	@Pointcut(value = "@annotation(com.yh.jdbc.MyTransaction)")
	public void pointCut() {
	};

	// 进行环绕通知
	@Around(value = "pointCut()")
	public Object transactionalGroupAspectArround(ProceedingJoinPoint pjp) throws Throwable {
		// 原方法返回值
		Object obj = null;
		// 是否进行了事务管理
		boolean doit = false;

		// 获取当前切入点方法对象
		MethodSignature signature = (MethodSignature) pjp.getSignature();
		Method method = signature.getMethod();

		// 获取该方法中事务注解元数据
		MyTransaction annotation = method.getAnnotation(MyTransaction.class);

		// 初始化事务管理器相关对象集合
		List<TransactionParam> tsParams = null;

		if (annotation != null) {
			Isolation isolation = annotation.isolation(); // 事务隔离级别
			Propagation propagation = annotation.propagation(); // 传播行为
			// 不触发回滚的异常类名set
			Set<String> noRollbackForNameSet = new HashSet<>();
			Class<? extends Throwable>[] noRollbackFor = annotation.noRollbackFor();
			if (noRollbackFor != null && noRollbackFor.length > 0) {
				for (Class<? extends Throwable> clazz : noRollbackFor) {
					noRollbackForNameSet.add(clazz.getSimpleName().toLowerCase());
				}
			}

			// 获取所有事务管理器Name数组
			String[] transactionNames = annotation.name();
			try {
				if (transactionNames.length > 0) {
					doit = true;// 执行当前方法的事务管理

					tsParams = new ArrayList<TransactionAop.TransactionParam>(transactionNames.length);
					for (String transactionName : transactionNames) {
						// 获取当前事务管理器,并定义其默认事务对象，获取事务状态
						TransactionParam tsParam = new TransactionParam();
						DataSourceTransactionManager tm = context.getBean(transactionName,
								DataSourceTransactionManager.class);
						DefaultTransactionDefinition def = new DefaultTransactionDefinition();
						def.setIsolationLevel(isolation.value());
						def.setPropagationBehavior(propagation.value());
						TransactionStatus transaction = tm.getTransaction(def);

						// 将所有事务管理器信息封装到TransactionParam对象中
						tsParam.def = def;
						tsParam.status = transaction;
						tsParam.tm = tm;
						tsParams.add(tsParam);
					}

					// 执行原方法代码
					obj = pjp.proceed();

					// 提交所有事务
					for (TransactionParam tsParam : tsParams) {
						tsParam.tm.commit(tsParam.status);
					}
				}
			} catch (Throwable e) {
				boolean rollback = true;// 事务回滚

				String errName = e.getClass().getName().toLowerCase();

				if ("transactionexception".equals(errName)) { // 这是sql异常，救不了了
					rollback = true;
				} else {
					if (noRollbackForNameSet.contains(errName)) { // 发生异常，不回滚
						rollback = false;
					}
				}

				if (!rollback) { // 事务不回滚
					try {
						for (TransactionParam tsParam : tsParams) {
							tsParam.tm.commit(tsParam.status);
						}
						throw e; // 直接抛出异常，end
					} catch (org.springframework.transaction.TransactionException e1) {// 如果这步还出现sql异常，只能回滚事务了
						e1.printStackTrace();
					}
				}

				// 事务回滚(在循环回滚时,第一次回滚成功后，会将事务同步锁处于非活跃状态，导致当前线程无法继续操作其他事务)
				for (TransactionParam tsParam : tsParams) {
					if (!TransactionSynchronizationManager.isSynchronizationActive()) {
						TransactionSynchronizationManager.initSynchronization();
					}
					tsParam.tm.rollback(tsParam.status);
				}

				throw e;
			}
		}
		if (!doit) {// 不需要进行方法增强时，则执行原方法代码
			obj = pjp.proceed();
		}
		return obj;
	}

	class TransactionParam {
		// 事务服务类
		DataSourceTransactionManager tm;
		// 默认事务对象
		DefaultTransactionDefinition def;
		// 事务状态
		TransactionStatus status;
	}
}
```

3、使用自定义注解，进行组合事务管理器的AOP，实现多数据源事务同步

```java
@MyTransaction(name= {"txManagerA","txManager"})
	public void testA() {
		jdbcTemplate.update("delete from user where id =?", "36");
		jdbcTemplateA.update("delete from test where id=?","13");
		System.out.println("A删除成功");
		int i =1/0;
	}
```

**该方式无法正确的使用事务的传播行为，因此要避免事务方法的嵌套调用**

#### 对DAO操作的抽象：

spring对数据方法对象（DAO）提供了一个统一的异常抽象层，来轻松了在各种dao层技术（JDBC、hibernate、JPA）中切换使用

- **统一的异常类结构框架：**

对于所有的DAO层操作错误，统一使用**DataAccessException**异常抛出，方便各种dao层技术中产生异常的捕捉；实现方式为：通过@Repository注解，来将dao层类注册到IOC容器中；对于mybatis中的@Mapper注解，和@Repository作用一致

### spring ORM：

​		spring ORM提供了对JPA的集成，用于支持它们的资源管理、dao访问和事务管理，常见实现JPA的框架有：Hibernate、spring data jpa；

​		注意：对于mybatis，并不属于JPA框架，因此不需要spring ORM；由于目前并没使用到这些ORM框架，因此不进行深入了解

## 5、spring测试

## 6、spring集成

## 7、spring5的新特性