# Spirng Boot 与分布式（Zookeeper +dubbo ）

在分布式系统中，常使用 Zookeeper +dubbo 组合；而spring boot 推荐使用Spring Boot +spring Cloud。

**分布式系统是若干个独立计算机的集合，这些计算机对于用户来说，就像是单个相关系统**

## 1、分布式系统建立：

由于一个系统规模的不断扩大，使用一台服务器运行一整个应用程序的方式，已经无法满足基本使用性能需求；因此需要将系统进行模块划分，使用不同的机器运行各自模块。但当单个模块用户访问量过大时，就需要使用和数据库主从复制一样的集群方式。多台服务器运行相同模块应用，通过http服务器（Nginx）进行请求转发，选择后台服务器处理请求。有效的提升了模块的并发访问量

- **应用架构发展：**

1、单一应用架构：将应用放在一台服务器上运行，通过多台服务器运行同一个应用，来提高并发访问量

​	缺点：系统应用功能扩展、修改都会导致整个系统需要重新打包，重新部署；

​		   在协同开发中，都操作同一个应用工程文件，会导致开发出现错误分支

​		   当系统越来越大时，单台服务器无法满足系统正常运行所需计算性能	

2、垂直应用架构：将整个系统按功能垂直拆分为多个应用，互相独立，互不影响；并通过MVC实现前后台分离

​	优点：系统方便扩展；

​	            协同开发更加方便，各个垂直应用单独开发、维护

​		    前后端分离，保证了后端系统不会受到前端页面频繁改动的影响

​	缺点：各应用直接不可能完全独立。只通过访问相同数据库来实现他们之间的数据交互，会大大降低代码复用，增加开发量

3、分布式服务架构：将公共应用模块提取出来，单独部署在分布式服务器上，供所有应用服务器调用

​	优点:通过分布式RPC框架，实现独立应用模块直接的服务复用

​	缺点：当业务模块的公共服务拆分越多，系统RPC调用复杂度就越高；越不容易维护

4、面向服务架构（SOA）：在分布式服务架构的基础上，添加调度中心，负责集群的调度和管理。而Doubbo就是出于SOA架构的RPC框架

**RPC：Remote Procedure Call  远程过程调用；它是一种通过网络进行远程计算机程序服务调用，并不需要了解底层网络的技术思想**

**RPC为什么会替代HTTP，在网络中进行服务远程调用：**

1. RPC网络传输大多数基于TCP、也可以基于HTTP；但是在它们的基础上进行了数据报文的缩减，提高了传输效率；对于HTTP，基于HTTP1.1的协议，则请求数据报文为了通用性，会包含很多无用内容，基于HTTP2.0时，则可以简单的封装作为RPC使用

2. RPC支持多种序列化协议，相对于HTTP的json序列化，效率更高，性能消耗小

3. RPC自带负载均衡和集群容错策略；HTTP需要额外配置Nginx或HAProxy做负载均衡器

   **综上所述：RPC传输效率高、性能损耗小、服务治理更加方便**

## 2、Doubbo：

Doubbo是Alibaba开源（现在被apache管理）的分布式系统的RPC框架，实现分布式模块之间服务调用。

- **Doubbo特性：**

1、面向接口代理的高性能RPC调用，服务以接口的方式直接跨应用调用，并屏蔽封装了远程调用的底层细节

2、智能负载均衡，当使用多台服务器提供同一个服务时，能够以负载均衡的方式选择要进行RPC调用的服务器

3、支持多种注册中心框架（一般使用Zookeeper），实时感知服务实例的上下线

4、支持运行期进行流量调度，通过配置不同路由规则，来实现灰度发布（对服务集群进行分批次更新，并定向测试，让更多的请求使用已更新的服务器）等功能

5、提供可视化的服务治理与运维工具（Dubbo-ops）

- **Dubbo设计架构：**

Dubbo中存在以下几个角色：Registry（注册中心）、Consumer（服务消费者）、Provider（服务提供者）、Monitor（监控中心）,运行过程如下：

​	Dubbo初始化时，服务提供者将服务发布到注册中心，而服务消费者询问注册中心，获取自己所有使用的PRC服务对应的提供者。之后消费者需要使用PRC调用时，则根据负载均衡进行RPC调用；当服务提供者进行变更时，注册中心会异步通知服务消费者该服务提供者的相关变更。同时所有的服务消费者的远程过程调用都会被监控中心异步监控

![](C:\Users\OneMTime\Desktop\Typora图片\DubboRPC调用模型.png)

- **Dubbo协议：**

  dubbo 框架进行远程服务调用时，默认使用dubbo网络通讯协议，该协议使用单个长连接和NIO异步通信，适合于小型数据传输，但具有高并发RPC调用。一般情况下，不适合用于大量数据传输，如文件传输、视频传输

  **消费者通过dubbo协议，来调用提供者的服务;因此dubbo协议的端口号，指定的是提供者服务器的端口号**

  **dubbo提供多种RPC网络通讯协议：定义了数据的网络传输和序列化方式**

## 3、Zookeeper：

Zookeeper为一个分布式系统的协调服务框架，即分布式系统的服务注册中心。它代码是Apache开源

- **Zookeeper设计原理：**
  	以观察者模式设计的分布式服务管理框架，将需要的远程调用的服务交给Zookeeper管理，然后需要远程调用服务的应用向zookeeper申请所需服务的观察权，一旦所需调用的服务发生改变，则会马上通知观察者做出相应反应。因此zookeeper就是一个文件管理系统，并具有监控通知机制（这里的文件相当于分布式系统在zookeeper中注册的所有服务）

- **Zookeeper特性：**

  1、Zookeeper的使用一般是集群模式，有一个领导者（Leader），多个跟随着（Follower）；一半以上集群存活时，就能保证zookeeper的正常使用

  2、所有zookeeper集群服务器中的数据都保持一致

  3、对于来自同一个客户端的更新请求，会按照顺序依次执行

  4、数据的更新具有原子性，一次数据更新要么成功，要么失败

  5、Zookeeper集群服务器之间的数据同步，实时性非常高，基本能保证客户端能读取最新数据

  6、Zookeeper的数据模型结构为树模型结构，每一个树形节点默认存储1MB数据，使用路径名作为节点的唯一标识

- **搭建Zookeeper服务端：**


Windows版本中，需要创建一个conf/zoo.cfg 的配置文件，默认有一个zoo_sample.cfg文件 最为模板使用（Linux版本使用docker容器搭建，并设置对应镜像启动参数）

配置文件中主要的几个属性设置：

dataDir  数据缓存目录（默认为/tmp/zookeeper） 

clientPort  客户端监听端口（默认为2181） 

添加**admin.serverPort=xxxx**属性，替换8080默认值

在3.5版本后，Zookeeper提供一个HTTP服务，用于远程操作执行zookeeper  命令，并且默认端口为8080，因此为了防止和Tomcat冲突，应在zoo.cfg配置文件中添加  admin.serverPort=8888，用于修改该HTTP服务默认端口

- Dubbo支持zkclient和curator两种Zookeeper客户端（在Dubbo2.7.x版本后，只支持curator，移除了zkclient）


以Dubbo2.6.7版本为例，使用curator客户端4.0.1版本（curator-framework），同时还需要添加额外的curator-recipes依赖，版本为2.8.0

## 4、Dubbo-ops：

Dubbo-ops 是dubbo框架提供的一个可视化监控功能（目前分为dubbo-admin、dubbo-monitor）

- Dubbo-admin：管理控制台，主要包含:路由规则、动态配置、服务降级、访问控制、权重调整、负载均衡等管理功能

- Dubbo-monitor：用于统计服务调用次数和时间

- 由于只是本地测试，因此使用第三方开源的简单ops插件：

​	在GitHub上下载，然后使用maven进行打包，默认为jar方式，直接通过内置tomcat运行（其中，可以通过修改application.properties配置文件，来设置相关参数：
server.port  默认为8080
env 默认设置有三种开发环境：测试、预发、生产；然后可以对应设置他们zookeeper地址（ip+端口）

）

使用maven clean package进行项目打包之后，在工程目录下打开cmd，使用java -jar  xxx.jar运行

​	Dubbo-ops监控工具使用：

通过localhost：port   来进入监控页面。然后根据RPC服务接口的全类名进行已注册的服务查询：

此时就可以查询到该服务的dataID、提供者应用地址和应用名、消费者列表、当前服务注册的配置文件和测试服务方法

## 5、Dubbo使用Demo（整合springboot）：

**以springboot整合Dubbo为例：**

1、创建服务提供者、消费者两个项目，并添加Dubbo依赖、Zookeeper客户端（curator）依赖

```xml
		
		<!-- 添加dubbo依赖(2.7版本后，支持springboot启动器，并整合springboot实现自动配置) -->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>2.7.6</version>
		</dependency>

		<!-- 需要替换dubbospringboot启动器中的 netty-all版本，避免报错-->
		<dependency>
			<groupId>io.netty</groupId>
			<artifactId>netty-all</artifactId>
			<version>4.1.32.Final</version>
		</dependency>

		<!-- 添加zookeeper客戶端工具,curator依赖 -->
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.0.1</version>
		</dependency>
```

2、配置两个项目的注册中心地址、项目被dubbo管理所使用的项目名、指定扫描哪个包下dubbo注册服务的注解

```properties
#dubbo配置（当前应用name、对应注册中心地址、指定需要dubbo暴露的服务包名（保证dubbo的@service注解生效））
dubbo.application.name=provider-ticket
dubbo.registry.address=zookeeper://192.168.31.56:2181
#该配置可以替换为在springboot配置类中添加注解@EnableDubbo(scanBasePackages={"com.yh.service"})
dubbo.scan.base-packages=com.yh.service

#dubbo进行远程服务调用时，默认使用的协议：dubbo协议  默认使用的协议端口为 20880；要保证每个服务器中所有提供者的端口号要不同
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
```

3、编写服务提供者、服务消费者业务代码

服务提供者，编写注册服务

```java
@Component  //将service交给spring容器管理
@Service  //将服务交个dubbo进行发布到指定zookeeper注册中心
public class TickerSerivceImpl implements TickerService {

	@Override
	public Ticker getTicker() {
		Ticker ticker = new Ticker();
		ticker.setName("我和我的祖国");
		ticker.setMoney("25");
		return ticker;
	}
}
```

服务消费者，远程调用服务业务

````java
//zookeeperRPC服务消费者
@Service  //为spring注解，用于该服务被spring容器管理
public class UserService {
	
	@Reference //通过注册中心，获取对应接口全类名的RPC服务地址
	TickerService tickerService;
		
	public void hello(){
		Ticker ticker = tickerService.getTicker();
		System.out.println(ticker);
	}
}
````

4、对于两项目的接口，dubbo都是通过它的全类名（包名+类名）进行注册和远程调用的。为了减少该服务接口在所有服务提供者和消费者中重复创建，dubbo官方推荐使用专门一个MAVEN项目对这些服务接口和服务所需的bean进行统一管理，然后作为依赖给其他应用调用，实现代码复用。因此所有消费者和提供者都需要添加该公共服务接口依赖

**同时dubbo进行远程调用时，返回值如果为java实体对象，这需要通过网络传输将结果对象传递给消费者，因此需要对该实体对象进行序列化，Dubbo只支持jdk原生序列化方式（即实体类Bean implements Serializable）**

````xm
		<!-- 添加dubbo统一管理的所有服务接口和服务所需bean的本地依赖 -->
		<dependency>
			<groupId>com.yh</groupId>
			<artifactId>dubboService_Interface</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
````

**这种MAVEN本地项目创建依赖过程，必须先对所依赖的项目进行maven安装，将项目打包到本地仓库中，之后其他使用该依赖的项目才能正常打包**

5、保证Zookeeper服务正常启动，然后依次启动服务提供者、消费者和dubbo-ops监控工具（由于他们都是springboot应用，使用内置Tomcat，默认端口都为8080，因此需要设置各自端口号，避免重复）

## 6、Dubbo常用配置（详细可参考dubbo用户模板demos模块和模式配置参考模块）：

- **dubbo配置方式优先级：**

  启动参数配置>>外部化配置>>API编程接口配置>>本地配置文件配置

  **本地配置文件方式（springboot都支持）：**springboot注解配置、XML配置、dubbo.properties配置（当然也可以直接写在springboot的主配置文件application.properties中;并且属性冲突是，主配置文件的优先级高，其他属性互补）

- **dubbo 服务配置覆盖优先级：**

**方法级优先（reference），接口级次之，全局配置最后**

**如果级别一样，则消费者优先，提供者次之**

- **dubbo推荐在provider(提供者)上进行尽量多的服务配置，原因是：**

1、作为服务提供者，更清楚服务在调用时的一些性能参数

2、即使服务提供端配置不合理，也可以被消费者端配置覆盖；防止当服务消费端忘记配置时，服务使用消费端的全局配置；导致服务不可控。（即把服务的默认配置，给予服务方管理）

- **springboot配置注意：**

在springboot中，properties文件不支持对reference（接口级）和reference.method（方法级）节点的配置；因此是通过对服务接口的@reference和@Service注解，来添加服务消费者和提供者对当前服务接口的属性配置**（但可以使用XML进行配置，从而支持服务提供者和消费者的方法级配置）**

### **1、启动时检查(check)**

对于消费者，在启动时默认，dubbo会自动检查相关服务是否可以用（即检查需要RPC调用的服务是否在注册中心注册）；若发现不可用，则阻止项目启动，并抛出对应异常；

对于消费者和提供者，启动时默认，dubbo会自动检查配置的注册中心是否在线，若没有则会报错，启动失败

springboot通过以下配置来关闭check：

```properties
##dubbo.reference.com.yh.service.UserService.check=false  //关闭指定服务的启动check
##dubbo.reference.check=false  //强制关闭所有服务的启动check（当单个服务check为true时，还是会关闭）
前两个配置以失效，只能通过@reference（check=false）来实现指定接口的启动check
duboo.comsumer.check=false  //关闭所有服务的启动check（当单个服务check为true时，该服务会进启动check）
dubbo.registry.check=false  //关闭注册中心的启动check
```

### 2、RPC调用超时(timeout)

声明远程服务接口调用的超时时间，超过则调用失败；有效防止网络超时引起多个RPC请求阻塞（超时默认值为1000ms）

```properties
dubbo.comsumer.timeout=3000   //全局设置消费者对该服务调用的超时时间
dubbo.provider.timeout=5000   //全局设置提供者对该服务提供调用的超时时间

#单个服务使用注解
@Reference(timeout=3000)    //设置消费者对单个服务接口调用的超时时间
@Service(timeout=5000)		//设置提供者对单个服务接口提供调用的超时时间重试次数（retries）**
```

当PRC调用失败是，消费者会默认继续尝试调用2次（2次指的是第一次失败后，继续尝试调用的最大次数）；通过设置retries=0，来设置服务调用失败是，不进行重试

**使用和timeout一致，但要注意：**

1、注意在设置服务调用的重试次数时，要保证服务调用具有幂等性，即多次操作的结果保持一致；防止在调用超时时，数据已经进行的操作，只是由于网络问题结果没有回传，从而导致多次操作数据；对于新增操作，应该将retries=0

2、当调用的服务有多个提供者，服务出现调用失败，在进行下一次重试时，消费者会根据默认的集群容错模式 Failover Cluster，依次尝试其他具有相同服务的提供者

### 3、服务版本（version）

当服务接口更新时，保证多版本能够同时使用；即创建多个该服务接口的实例，在注册时声明每个的version；然后消费者调用该接口时，指定实例的version:

默认version=”0.0.0“；当消费者指定调用服务的version=“*”时，表示随意调用任意版本的服务，实现多版本的灰度发布

### 4、本地存根（stub）

在进行RPC是，消费者仅仅是调用提供者去执行该服务，然后获取结果数据。有时需要对该服务调用前后做处理，即让消费者对应服务进行进一步封装，如执行ThreadLocal缓存、验证参数、在调用失败时返回模拟数据等。Dubbo提供本地存根的方式，来实现该过程：

![](C:\Users\OneMTime\Desktop\Typora图片\Dubbo本地存根原理图.jpg)

对指定消费者服务配置本地存根，然后调用该本地存根服务。此时会自动创建该服务的代理，用于远程调用该服务的实现。从而在本地存根服务内部，实现rpc的同时，添加额外的处理代码

配置如下：

1、在消费者订阅服务时，打开本地存根设置:  添加Reference（stub=“com.yh.service.UserServiceStub”）注解属性，指定消费者使用本地存根时的实现类名

2、在消费者端，创建该服务的实现（本地存根）：

```java
public class TickerServiceStub implements TickerService{
	
	private final TickerService tickerService;
	
	public TickerServiceStub(TickerService tickerService) {
		this.tickerService=tickerService;
	}
	
	@Override
	public Ticker getTicker() {
		Ticker ticker = getTicker();
		
		//对服务方法，通过代理方式进行额外处理
		if (ticker==null) {
			ticker=new Ticker();
			ticker.setName("空");
		}
		return ticker;
	}
}
```

**要保证该本地存根，有对应服务的传参构造器，用于Dubbo提供远程调用，实现该服务的代理**

### 5、服务协议（protocol）

可以对每个服务设置单独的RPC调用协议，从而覆盖全局协议。并且单个协议支持

### 6、仅订阅/仅注册（register）

有时可能一个项目既是服务提供者又是消费者；但是当所提供的服务还在开发时，就只使用dubbo的订阅功能，来进行RPC调用,则添加该配置：

```properties
dubbo.registry.register=false;
```

当该项目只作为服务提供者，在当前zookeeper进行服务的注册。而自己本身依赖的服务在另一个zookeeper环境中远程调用，而不去调用当前作为服务提供者的zookeeper中的注册服务。此时就需要对指定zookeeper，进行仅注册配置：

```properties
dubbo.registry.subscribe=false
```

### 7、服务直连

当消费者进行服务调用时，会根据注册中心的注册信息，来获取该服务可调用服务提供者url，然后按照负载均衡策略进行服务调用；但是也可以自定义指定服务提供者的url，来跳过注册中心，实现服务直连调用。

```java
@Reference(url="127.0.0.1:20882")
```

- 异步调用
- 异步执行

## 7、Dubbo高可用配置（通过设计，减少系统不能提供服务的时间）：

### 1、zookeeper宕机

一般情况下，zookeeper使用集群方式在生产环境下运行；当其中一台zookeeper服务器宕掉时，会自动切换到另一台服务器最为Leader作为dubbo的注册中心使用；当zookeeper集群全部失效时，服务提供者和服务消费者也能通过本地缓存的服务注册表，来进行它们之间的RPC调用，只是此时服务提供者的上下线无法被监控，无法注册新服务，或在服务下线时，无法通知对应消费者。

只有当所有服务者都宕机时，服务消费者才不能正常进行PRC调用

### 2、集群容错模式（cluster）

dubbo在进行集群提供者的服务调用时，会根据集群容错模式进行失败调用或其他调用；模式有：

cluster=failover		失败自动切换；出现失败时，重试其他服务器 ，为默认值

cluster=failfast		快速失败；出现失败后立即报错，不进行重试；一般用于具有非幂等性的写操作

cluster=failsafe		失败安全；都失败后，出现异常时直接忽略

cluster=failback		失败自动恢复；失败后后台记录，定时重发；用于消息通知操作，保证操作一定要成功

cluster=forking		并行调用集群多个服务器；只需成功返回一个即可；用于实时性较高操作，但浪费资源；可以通过 forks=2 来设置并行调用的服务器数

cluster=broadcast		通知所有集群服务器，逐个调用，任意一台出现报错，则报错。用于通知所有提供者进行日志或缓存更新等操作

- **在实际开发中，还可以导入Hystrix集群容错解决方案；它是springCould框架中默认集群容错组件；通过控制远程系统、服务和第三方库的节点，对延迟和故障提供更强大的容错能力：具备回退机制和断路器功能**

回退机制：当服务调用发生异常时，可以返回调用一个默认响应方法

短路器功能：当该服务在一定时间（默认10s）的调用失败次数超过X次（默认20次），并且失败率超出x%(默认50%)时；服务的断路器打开；此时所有服务请求都直接走fallback（回退机制）；一段时间后（休眠窗口期），断路器半打开，允许处理一个请求，若失败则断路器继续打开，等待下一个休眠窗口，成功则关闭短路器

- **Hystrix实现原理：**

Hystrix提供一种特殊的线程池模型，为每个服务提供一个线程池，从而将每个服务隔离开来；如果某个服务，出现阻塞，这会慢慢耗尽自己的线程池，最后进入调用失败状态，然后进行服务隔离；这样有效防止了单个服务阻塞导致耗尽整个Tomcat的线程池，导致服务器挂掉

- **Hystrix使用方法：**

1、导入依赖，由于Hystrix属于springcloud组件，因此直接使用springboo-starter

2、在配置类中添加注解 @EnableHystrix  开启Hystrix断路器注解

3、 在服务提供者中，在服务类的方法上添加@HystrixCommand注解；此时该方法的调用被Hystrix管理，当对应服务方法调用出现异常时，则被Hystrix获取到执行失败，并记录执行容错策略

4、在服务消费者中，对进行远程调用服务的方法上添加@HystrixCommand注解，此时该方法的调用被Hystrix管理，当对应服务方法调用出现异常时，则被Hystrix获取到执行失败，并记录执行容错策略

5、在服务方或者提供方方法的@HystrixCommand中添加容错策略属性：

fallbackMethod=“xxx”    当方法执行发生异常时，重新调用当前类中的xxx方法

根据对HystrixCommand的其他配置，可以设置HystrixCommand更多短路器的配置

6、Hystrix组件同时还提供Hystrix-Dashboard来对所有被Hystrix管理的服务进行监控

### 3、集群负载均衡（loadbalance）

在进行提供者集群负载均衡时，提供如下均衡策略：

loadbalance=random		按权重设置随机概率（默认值）；由于通过概率随机，因此访问量越大越有效

loadbalance=roundrobin		按权重，设置轮询比率；当出现某台服务器变慢时，请求会堆积，无法实现负载均衡

loadbalance=leastactive		最少活跃数调用；即请求处理速度越快，处理的请求数越多；这样会导致速度慢的提供者会收到更少的请求

loadbalance=consistentHash 		请求会根据一个hash值进行提供者绑定；当该hash绑定的提供者下线时，则会平摊到其他提供者上；保证了提供者数量突然减少时，请求会均分到其他服务器上处理

### 4、服务降级（mock）

有时突然要面临高并发场景时（秒杀抢购），需要减少一些非关键服务的资源占用，让服务器在保证正常工作的前提下更好的面对高并发。因此需要对这些服务在特点时间段进行服务降级

通过API的方式，动态修改对应注册中心的服务配置（一般通过dubbo-admin管理台直接进行操作）：

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

其中服务降级规则有两种：

mock=force：return+null   对于该服务的远程调用都直接返回null，不进行实际远程调用

mock=fail：return+null	对于该服务当远程调用失败后，直接返回null；不进行调用重试，防止浪费资源

- 路由规则和配置规则的动态更改

## 8、Dubbo注意问题：

- 延迟发布

- dubbo默认支持springboot日志的适配

通过dubbo.application.logger=log4j  启动对应日志框架的默认日志策略

通过dubbo.protocol.accesslog=true 记录当前应用的dubbo服务访问日志

通过@Service(accesslog="true") 注解 记录当前服务的访问日志

## 9、Dubbo原理：

### 1、RPC原理：

- RPC，远程过程调用，它是一种通过网络进行远程计算机程序服务调用，并不需要了解底层网络的技术思想。

- RPC架构：

一个完整的RPC架构中包含四个核心组件，分别为Client、Client stub，Server和Server stub。stub可以理解为存根

Client：客户端，服务调用方

Client Stub: 客户端存根，存放服务端地址，并可以将客户端服务调用的请求参数打包为网络消息，通过网络远程发送给服务方；同时可以解析服务端发过来的服务方法调用结果

Server：服务端，服务提供方

Server Stub：服务端存根，用于接收客户端发送的消息，解析然后作为参数进行本地方法调用；并记录客服端地址，将方法调用结果打包为网络消息发送给客户端

- RPC调用过程:

![](C:\Users\OneMTime\Desktop\Typora图片\RPC调用模型.jpg)

1、客户端以本地调用方式调用服务（调用服务接口方法）

2、客户端存根接收到调用后，将调用的方法、参数打包成网络传输的消息体（需要对对象进行序列化）

3、然后客户端通过sockets 将消息发送到服务端

4、服务端存根收到消息，进行解码（将消息体反序列化为对象）

5、服务端存根格局解码结果，让服务端进行本地方法调用

6、然后将方法调用结果交给服务端存根

7、服务端存根对结果打包

8、服务端通过sockets将消息发送到客户端

9、客户端存根 对收到的消息进行解码

10、此时客户端获的最终结果

**而RPC技术思想的目的就是，将2-9的过程都封装起来，让过程细节透明化(即对用户不可见，方便用户直接使用）**

**一个完整的RPC框架是包含，服务发现、负载、容错、网络传输、序列化等组件。而RPC的核心还是在于如何实现数据的序列化和网络传输**

### 2、Netty：

Netty是一个异步事件驱动的网络应用程序框架，用于快速开发具有可维护和高性能的网络服务器和客户端。极大简化基于TCP和UDP的socket服务开发。

- BIO  阻塞式IO，没一个I/O请求都对应一个线程执行，当数据还没有处于可以进行底层I/O操作状态时，会导致整个IO线程阻塞，浪费计算机性能。

- NIO 非阻塞式IO，将IO请求作为多个通道传输，并统一交给通道选择器管理监听；当通道的读写状态准备好时，则创建一个线程进行最底层的数据IO操作；当数据未做好准备，不会提前创建线程执行I/O操作；这样大大减少了I/O请求对线程的占用，有效提高I/O请求处理速度，避免I/O请求的阻塞。这种I/O处理方式也叫做NIO的多路复用

Netty就是基于NIO的多路复用模型，来实现的网络I/O传输。大多数RPC框架的底层都是使用的Netty(如Dubbo)，来实现网络传输

### 3、Dubbo框架设计原理：

![](C:\Users\OneMTime\Desktop\Typora图片\dubbo工作原理图.jpg)

整个Dubbo，由浅到深可分为三个部分，业务、RPC组件和远程调用底层实现

1、业务部分，就是Dubbo面向用户的service层。通过接口的方式，来进行远程调用

2、RPC组件部分，由配置层、服务代理层、注册层、路由层、监控层、协议层

​	配置层，用于封装配置文件中解析的信息，包括服务消费者和提供者两端的配置

​	服务代理层，通过配置信息，生成服务代理工厂；用于在消费者端创建服务远程代理对象，实现远程方法调用

​	注册层，用于封装服务的注册和发现

​	路由层，实现服务调用的负载均衡

​	监控层，对RPC进行监控

​	协议层，实现对RPC的封装

3、远程调用底层实现部分，由交换层，传输层

​	交换层，创建数据的网络传输通道，实现客户端与服务端之间的网路连接

​	传输层，封装数据的传输过程，底层使用Netty实现

​	序列化层，定义数据传输中的序列化方式

### 4、Dubbo配置加载过程：

1、Dubbo容器启动，通过DubboNamespaceHandler（dubbo配置文件的命名空间处理器），在初始化时，就对Dubbo配置文件的命名空间解析，根据不同标签，来初始化创建所有标签对应的DubboBeanDefinitionParser对象（dubbo标签解析器）

2、所有DubboBeanDefinitionParser会根据配置文件，依次解析自己对应标签的配置属性，并添加到dubbo默认规定好的各个配置组件对象中（没有，则添加默认值；组件对象有：ApplicationConfig、ProviderConfig、ConsumerConfig、ServiceBean、ReferenceBean等）

3、对于ServiceBean，它实现了InitializingBean接口，会当所有dubbo配置组件完成自己的属性配置后，执行afterPropertiesSet方法，将部分组件的信息保存到当前ServiceBean中（**实现服务的全局配置**，当服务设置了相关信息时，则不进行全局配置）

还ApplicationListener<ContextRefreshedEvent>事件监听器接口，当dubbo容器初始化完成后，会回调一个onApplicationEvent方法：执行 export（） **实现服务暴露**

### 5、服务暴露过程：

1、在ServiceConfig类中执行doExport（）-----》dodoExportUrls（），在其中就加载注册中心信息，并指定服务暴露协议（默认dubbo协议）

2、通过服务代理工厂，包装服务的注册信息（服务对象、注册中心url，暴露协议）

3、通过RegistryProtocol对象进行服务暴露(将服务注册到注册中心)

4、通过dubboProtocol对象（根据服务使用的暴露协议来选择）打开服务的远程调用（运行该服务被其他计算机进行远程调用）

### 6、服务引用过程：

1、Dubbo启动后，加载配置文件，会自动创建一个ReferenceBean配置组件，该组件就是用于服务引用配置

2、然后ReferenceBean调用ReferenceConfig类中的get（）方法，来通过代理工厂创建服务代理对象

3、通过RegistryProtocol对象向注册中心订阅服务，获取服务注册信息（服务对象、注册中心url，暴露协议）

4、通过dubboProtocol对象获取服务提供者进行远程连接的客户端

5、最后将每个服务所有创建、获取的信息，都封装为一个invoker模型，保存到服务代理对象中

### 7、服务调用过程：

![](C:\Users\OneMTime\Desktop\Typora图片\dubbo服务调用接口流程.jpg)

1、通过ReferenceBean的getObject方法获取服务代理对象，由于服务多个提供者，因此就有多个invoker模型，进行选择调用

2、在代理对象执行invoke方法之前，会经过一个filter过滤器，用于实现服务缓存、服务降级（将服务调用设置为直接失败）

3、然后执行代理对象的invoke方法，在调用失败时，会根据dubbo集群容错策略进行重试

4、同时invoker的执行也会满足dubbo集群负载均衡策略，invoker模型进行选择性调用

5、在根据网络传输协议进行远程调用时，通过invoker模型中的暴露协议，底层使用netty进行数据传输

