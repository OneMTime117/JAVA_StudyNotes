# Spring

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

- Beans
- Core
- Context
- SpEL

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

- BeanFactory进行Bean的实例化过程，是懒加载的，即加载配置文件时并不会创建对象，而是在进行依赖注入时，才会创建；ApplicationContext则是饿加载，即加载配置文件时就会创建对象

**一般情况下，BeanFactory用于spring内部实现IOC容器，而对外推荐使用ApplictionContext，并且对于创建消耗资源较大的对象，使用ApplicationContext进行创建**

BeanFactory和ApplicationContext接口实现类：

ClassPathXmlAppliationContext和FileSystemXmlApplicationContext，前者读取项目class文件根目录路径下的xml文件；后缀读取操作系统文件路径下的xml文件

### IOC容器的Bean管理：

**spring3.0后，提供两种IOC容器的Bean管理方式：xml和java注解**

IOC容器的Bean管理，就是对spring的xml配置文件Bean标签的定义（也可以使用java注解），主要包含如下操作：

- Bean的命名：作为每个Bean在IOC容器的为唯一标识
- Bean的全类名：定义bean的实现类
- Bean的实例化：定义使用Bean的哪种构造方法，以及对应参数
- Bean的行为配置：声明Bean在IOC容器中的行为：作用域、生命周期的回调
- Bean实例化对象的属性注入：对IOC容器中创建好的实例对象，进行额外的属性注入

#### XML方式：

**springIOC容器，提供两种类型Bean的创建，普通Bean和工厂Bean**

##### 1、普通Bean：

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

2、当只指定name时，spring会将name的一个值作为id使用

3、当出现多个Bean的name相同时，spring5版本会报错（之前版本不会，取最后创建的Bean）

4、id和name可以同时指定，都可以进行Bean的获取

**一般情况下，推荐使用id进行Bean的命名，并且使用驼峰命名规则**

**bean标签的子标签：**

- property子标签：用于指定Bean实现类的属性注入，需要保证Bean实现类提供该属性的set方法


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
  
  - 特殊值注入（null、特殊符号）：特殊符号使用xml转义字符完成
  
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

##### 2、工厂Bean：

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

**Xml方式属性注入方式的简化：**

- **引入P命名空间，简化xml中property标签的使用，实现属性注入时根据实现类的set方法，进行xml属性自动提示**
- **引入util命名空间，用于在IOC容器中配置集合Bean，来代替ListFactoryBean、MapFactoryBean、SetFactoryBean、PropertiesFactoryBean对象方式创建集合Bean**
- **引入C命名空间，简化constructor-arg标签的使用，实现在Bean标签中，使用内联属性进行构造函数参数的声明（spring3.1引入）**

##### 3、Bean的行为配置：

- Bean的作用域：

  spring提供Bean的六个作用域

- Bean之间的依赖关系：
  - 循环依赖
  - 延迟Bean的初始化
  - 定义多个Bean的初始化顺序
- Bean的声明周期：

确定Bean运行时类型

#### java注解方式：

## 3、AOP

## 4、jdbcTemplate

## 5、事务管理器

## 6、spring5的新特性