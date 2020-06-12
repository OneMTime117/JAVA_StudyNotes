# java消息服务（消息中间件）

## MQ（Messgae Queue，消息队列，又叫消息中间件）：

### 1、MOM（message-oriented middleware面向消息的中间件）：

通过提供消息传递和消息排队模型，在分布式环境下提供应用解耦、弹性伸缩、冗余存储、流量削峰、异步通信、数据同步等功能，大致过程如下：

​		发送者发送消息给消息服务器，服务器将消息存放在若干个队列/主题中，在合适时候取出发送给接收者。此时消息发送、接收是异步的，即发送无需等待；并且发送者和接收者没有必然关系

### 2、MQ产品种类：

1、ActiveMQ ：中庸MQ产品，完全实现了jms规范

2、RocketMQ ：阿里以kafka为模板研发产品，性能仅次于kafaka（单机吞吐量万级，消息延迟毫秒级别）

3、RabbitMQ：性能很好（单机吞吐量万级，消息延迟微秒级别，消息重复可控制，较低消息丢失），但由于erlang语言开发，不易底层修改进行动态扩展

4、Kafka：性能最好（单机吞吐量十万级，消息延迟毫秒级别，理论不会出现消息丢失，可能重复），分布式可以随意扩展

### 3、所有MQ产品都需要保证如下特性：

1、消息发送和接收的api

2、MQ的高可用性（通过集群来实现备灾容错）

3、MQ的集群和容错配置

4、MQ中数据的持久化（数据不仅仅保存在内存中，需要持久化磁盘备份）

5、MQ数据支持延时发送和定时投递

6、MQ数据发送成功后的签收机制（告诉发送者消息发送成功）

7、MQ与spring的整合

### 4、MQ的作用及使用场景：

作用：

​	1、解耦：请求发送时不再直接调用消息接收者的服务，而是将需要处理的请求，已特定格式保存到MQ中，然后接收者按照队列顺序进行请求处理；当新增其他接收者模块时，不需要改动请求发送者接口。

​	2、削峰：当出现突发的高并发请求时，为了满足应用处理需求，会提高系统对于该请求最大并发处理量。当服务器以这个标准进行该请求处理时，在正常情况下，则会让服务器保持在低占用状态，这样大大降低服务器利用率。通过实现削峰，来让服务器运行更加平稳，节省服务器资源

​	3、异步：实现异步处理机制，防止请求处理阻塞

使用场景（当没有MQ时）：

​	1、高并发下，模块处理量不足，导致请求阻塞，造成等待同步存在的性能耗损；更严重导致服务器宕机

​	2、系统之间服务直接请求调用，接口耦合严重；导致当下游接口新增时，上游接口需要同步改动；并且可能存在下游接口请求形式不同，导致上游接口更加复杂（如订单系统进行订单生成后，会调用物流、结算和库存系统）

## ActiveMQ：

### 1、ActiveMQ特点：

- 底层编程语言JAVA

- Apcha开源
- 完全支持JMS1.1 和J2EE1.4规范（JMS java messaging service java消息服务规范、J2EE java 2 企业级应用开发规范）

### 2、ActiveMQ安装：

​	1、Windows版：在apache-activemq...../win64/activemq.bat进行启动；此时activemq服务（JMS服务）默认使用61616端口；并提供一个本地web控制台http://localhost:8161/admin，使用端口8161提供监控服务；账号：admin；密码admin

​	Linux版：将apache-activemq...-bin.tar.gz进行解压，然后再解压目录/bin 下 执行 ./activemq start;此时activemq启动占用61616端口；activemq的web管理页面监控使用8161端口

2、通过 ./activemq start  xbeam：file:/activeMQ/conf/activemq02.xml  来指定ActiveMQ实例配置文件；通过不同的配置文件，能启动多个实例，类似于redis

### 3、JMS规范：

- JMS：java message service java消息服务规范，J2EE十三规范之一

![](C:\Users\OneMTime\Desktop\Typora图片\JMS编程模型.png)

​		通过connectionFactory来创建客户端（消息生产者、消费者）和消息中间件的Connection，在一个Session会话中，创建MessageProducer/MessageConsumer对象，并指定 消息目的端口（消息队列/主题）destination对象（queue、topic） ，进行消息的发送或接收；并指定消息的固定格式

- JMS定义了两种消息模型：

Point-to-Point Messaging Domain（点对点通信模型）：每个消息都有一个特定的队列，然后只被一个消费者消费

Publish/Subscribe Messaging Domain（发布/订阅通信模型）：对于订阅相同主题的消费者，可以同时对该主题的消息进行消费

- JMS定义了两种消息接收方式

  同步：消费者主动请求消息中间件，是否有消息可以接收，没有则一直阻塞

  异步：通过事件监听器，当生产者发送消息后，消费者通过监听器来主动调用消息接收方法

### 4、ActiveMQ队列/主题的消息传递Demo：

- **消息生产者进行数据发送**

```java
public static final String ACTIVEMQ_URL_STRING="tcp://127.0.0.1:61616";

	public static void main(String[] args) throws JMSException {
		//创建连接工厂
		ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL_STRING);
		//创建连接
		Connection connection = activeMQConnectionFactory.createConnection();
		connection.start();//进行连接
		//创建会话(指定事务是否开启、会话创建确认方式,如AUTO_ACKNOWLEDGE自动确认)
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		//创建会话目的地（队列/主题）
		Queue queue = session.createQueue("queueName01");
//		Topic topic = session.createTopic("topicName01");
		MessageProducer producer = session.createProducer(queue);//根据目的地来创建消息生产者（即创建消息来源对象）
		
		//创建文本消息
		TextMessage message = session.createTextMessage("msg");
		//使用消息生产者，向目的地（队列）发送消息
		producer.send(message);
		
		//关闭消息中间件连接资源
		producer.close();
		session.close();
		connection.close();
		System.out.println("发送消息到ActiveMQ操作完成");
	}
```

- **消息消费者进行数据接收(同步接收方式)**

```java
	public static final String ACTIVEMQ_URL_STRING = "tcp://127.0.0.1:61616";

	public static void main(String[] args) throws JMSException {
		ActiveMQConnectionFactory activeMqfactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL_STRING);
		Connection connection = activeMqfactory.createConnection();
		connection.start();
		// 创建session
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		// 创建目的地
		Queue queue = session.createQueue("queueName01");
		// 创建消息消费者
		MessageConsumer consumer = session.createConsumer(queue);

		while (true) {
			// 获取队列消息
//			TextMessage receive = (TextMessage)consumer.receive();//receive方法，无参构造器：一直等待直到获取消息
			TextMessage receive = (TextMessage) consumer.receive(2000); // 等待一段时间后，获取不到消息，则返回null
//			consumer.receiveNoWait()	//直接不等待，没有消息则返回null

			if (null != receive) {
				System.out.println("获取消息" + receive.getText());
			} else {
				break;// 不再获取消息
			}

			try {
				Thread.sleep(3000);// 等待3s，再获取队列消息
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

		consumer.close();
		session.close();
		connection.close();
	}
```

- **消息消费者进行数据接收(异步接收方式)**

通过消息监听器，来实现消费者进行异步的消息获取；由于监听事件是队列中是否有消息存在，因此第一次进行连接时，会直接获取队列中的消息

```java
		//异步方式接收(设置消费者当前队列的消息监听器)
		consumer.setMessageListener(new MessageListener() {
            //当messageListener监听事件触发时（队列存在消息时）；回调执行onMessage方法
			@Override
			public void onMessage(Message message) {
				//判断队列中新传入的message是否为null，且是否为TextMessage类型或其子类
				if (null!=message&&message instanceof TextMessage) {
					TextMessage textMessage=(TextMessage)message;
					try {
						System.out.println(textMessage.getText());
					} catch (JMSException e) {
						e.printStackTrace();
					}
				}
			}
		});
		System.in.read();//读取控制台输入，此时代码进入阻塞状态；当控制台使用enter时，才会继续执行
		//作用:可以让当前进程一直处于活动状态，用于等待异步事件触发
		
		consumer.close();
		session.close();
		connection.close();

```

**对于主题的消费者接收消息，只能使用异步的事件监听器来实现（或者手动死循环进行定时接收）**

### 5、ActiveMQ消费特点：

- 对于队列(queue)：

  1、每个消息只有一个消费者接收；

  2、当出现多个消费者时，会以负载均衡（默认轮询）的方式进行消息的接收

  3、消息会先保存到队列中，并进行持久化数据存储

- 对于主题(topic)：

  1、主题中的每个消息被多个消费者接收；

  2、生产者发送的消息与消费者订阅时间有相关性，订阅某个主题的消费者只能接收自它订阅以后发布的消息

  3、主题不会保存消息，消息是无状态的，当生产者发送后，主题会直接转发给所有消费者；因此一般情况下，需要先启动消费者然后启动生产者

  4、当前没有订阅者时，生产者发送的信息将会直接被丢弃

### 6、ActiveMQ（JMS规范下）的消息组成：

- **消息头：**用于定义消息发送时的某些属性

  ```java
  		message.setJMSDestination(topic);//指定消息的目的地
  		message.setJMSDeliveryMode(DeliveryMode.PERSISTENT);// 指定消息是否为持久化/非持久化,默认持久化
  		message.setJMSExpiration(30*1000);//过期时间，默认为0 永不过期
  		message.setJMSPriority(5);//消息发送优先级，默认为4；JMS不严格要求按0-9的优先级执行，但要保证 4以上的消息优于4及一些的消息，在Activemq中，默认该属性设置不生效，需要添加额外配置
  		message.setJMSMessageID("");//消息唯一标识,可以由MQ自动生成
  
  		//通过send重载方法，直接快速设置message消息头
  		producer.send(destination, message, deliveryMode, priority, timeToLive);
  
  ```

- **消息体：**用于封装具体的消息数据

  Message分为五种消息体格式：对应5种Message接口的实现类：

  1、TextMessage ：String字符串

  2、MapMessage	Map集合，并且Key为String类型，Value为java基本数据类型+String

  3、ByteMessage	byte[ ] 二进制数组，并使用 字节流 进行数据传递

  4、StreamMessage	所有java数据类型（基础+String+Object），使用 基础数据流 进行数据传递

  5、ObjectMessage	可序列化的java对象

  **必须保证发送消息和接收消息的对象类型相同**

```java
		//创建文本消息
		TextMessage message = session.createTextMessage("msg");
		
		//创建MAP集合消息
		MapMessage message2 = session.createMapMessage();
		message2.setBoolean("isNull",false);

		//创建Stream流消息对象
		StreamMessage message3 = session.createStreamMessage();
		message3.writeBoolean(false);
		message3.writeObject("sdfa");

		//接收时，使用get/readXxx方法获取
```

- **消息属性：**自定义键值对（key为String，Value为java基本数据类型+String），用于对当前数据进行额外描述（如告诉接收者，消息是否需要特殊识别、去重和是否为重点数据。。。）

```java
//提供者设置消息属性
message.setStringProperty("type","vip");
message.setBooleaProperty("isRemoveDuplicate",false);

//消费者获取消息属性
String type = message.getStringProperty("type");
```

### 7、ActiveMQ的可靠性：

**MQ消息传输的可靠性通过3个角度进行保证：持久化（Persistent）、事务（Transaction）、签收（Acknowledge）**

#### 1、持久化：

```java
messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);//该提供者所有消息发送默认持久化
message.setJMSDeliveryMode(DeliveryMode.PERSISTENT);//仅该消息的发送进行持久化
```

通过设置提供者或消息头的持久化属性来实现；队列中的消息会持久化保存在硬盘中，当MQ服务器宕机后，消息不会消失

1、队列消息默认是持久化的，保证消息能成功进行有且一次的消费

2、主题消息订阅，默认是非持久化的，当订阅者离线时，默认取消订阅，不再发送消息

3、对于持久化主题，当订阅者离线时，不会取消订阅，而是持久化关联保存没有发布的消息和需要接收消息的订阅者的ClientID；当订阅者再次连接时，会把之前消息依次推送给订阅者;并且当此时MQ服务器宕机时，这些消息会在服务器重启后发送；即持久化主题，保证了每个消息都能成功发送给它的订阅者；**持久化主题Demo**：

**producer：主题消息默认为非持久化的，因此需要进行消息持久化属性设置**

```java
		ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
		Connection connection = factory.createConnection();
		//此时连接启动
		connection.start();
		
		//创建指定主题发送者
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		Topic topic = session.createTopic("topic01");
		MessageProducer producer = session.createProducer(topic);
		
		//设置当前发送者主题消息持久化(topic消息默认non_persistent非持久化)
		producer.setDeliveryMode(DeliveryMode.PERSISTENT);
		
		//创建message
		TextMessage message = session.createTextMessage();
		message.setText("topic-------"+"msg01");
		producer.send(message);
```

**consumer：对于持久化主题消费者，必须指定当前连接的ClientID**

```java
		ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
		Connection connection = factory.createConnection();
		connection.setClientID("subscriber01");//设置当前连接的客户端ID（持久化订阅必须要设置）
		connection.start();//连接启动

		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		Topic topic = session.createTopic("topic01");
		//创建主题持久化订阅者(指定当前topic01主题订阅者的name，配合ClientID获取持久化订阅者唯一标识)
		TopicSubscriber subscriber = session.createDurableSubscriber(topic, "topic01");
		
		TextMessage message = (TextMessage)subscriber.receive();
		while (null!=message) {
			System.out.println("接收topic01 持久化订阅消息"+message.getText()); 
			message = (TextMessage)subscriber.receive();//获取下一个订阅消息
		}
	}
```

4、对于持久化消息，当MQ发现没有订阅者时，会直接丢弃；因此需要先启动消费者，进行持久化消息订阅，然后在启动消费者；之后保证所有消息都能发送到消费者

#### 2、事务：

​		用于保证MQ消息发送和接收的一致性、原子性；当一次会话，进行多个消息发送和接收时，要么全部完成，要么全部失败，并不会存在中间状态。

​		事务和签收机制时冲突的，因此事务为true开启时，签收属性会无效

```java
通过createSession方法的第一个参数，来进行事务的关闭或开启（必须显式表示）
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

Session session = connection.createSession(true, Session.SESSION_TRANSACTED);
```

- 对于生产者：当事务为true时，需要在session关闭时，进行事务提交（否则消息无法提交）。并在发送异常时，进行事务回滚; 为fasle时，自动提交，无法进行事务回滚

```java
try{
   	...//数据发送
	session.commit(); 
}catch(Exception e){
    e.printStackTrace();
    session.rollback();
}finally{
    ...//资源关闭
}
```

- 对于消费者：当事务为true时，当事务为true时，需要在session关闭时，进行事务提交（否则消息会进行重复消费）；只有事务被提交后，才能确认消息被消费，从而防止消息重复；当手动回滚时，当前所有消息就不会被消费

#### 3、签收：

​		消费者对接收的消息进行签收，从而告诉MQ该消息已被消费；因此签收机制只针对于消费者，生产者的签收属性无实际意义

1、在非事务模式下，有三种签收方式：

- AUTO_ACKNOWLEDGE（自动签收）:

当消费者成功返回Message对象时，自动签收；否则判定当前session下，消息没有完成消费

- CLIENT_ACKNOWLEDGE（客户端签收，即手动签收）：

```java
message.acknowledge();//单个消息手动进行签收
```

- DUPS_OK_ACKNOWLEDGE（批量自动签收）：

允许当前会话延迟向服务器发送消息签收信号，继续进行会话中其他消息的消费，最后再一起批量自动签收，发送给服务器；最大程度上减少当前会话中客户端与服务器之间的交互频次，从而降低会话开销；但是着也会存在当整个会话过程中出现问题时，导致所有签收全部驳回的情况，从而多个消息会被重复消费；**该模式，只针对于Topic**

2、**在事务模式下，签收参数失效，则为提交事务就全部签收，回滚事务/不提交事务 就全部不签收**

3、三种应答使用常见问题：

​	1、当使用自动签收时，消费者收到消息，则自动签收；当进行数据处理时，突然服务器崩溃，这样会导致消息丢失，因此对于重要消息，应该使用手动签收或事务签收

​	2、若在资源关闭前，没有进行签收，则默认消息没有被消费

​	3、消息重复消费，但不签收时；此时会判断当前消息消费失败，当尝试发送超过6次时，该条消息会进入DLQ（death letter queue 死信队列）

### 8、ActiveMQ的Broker对象：

由于ActiveMQ是由JAVA编写的，因此支持java嵌入式的迷你ActiveMQ服务端，由ActiveMQ的依赖提供；而Broker对象就代表的MQ服务器的实例：

```java
		BrokerService brokerService = new BrokerService();
		brokerService.setUseJmx(true);//使用JMX
		try {
			brokerService.addConnector("tcp://localhost:61616");//设置服务器连接地址
			brokerService.start();//启动mq服务器
		} catch (Exception e) {
			e.printStackTrace();
		}
```

通过上面代码，就可以创建一个简单的ActiveMQ；**但是不提供监控页面，一般不推荐使用**

### 9、ActiveMQ整合Spring：

- pom.xml：

```xml
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>5.1.8.RELEASE</version>
		</dependency>		

<!-- activeMQ所有依赖：包含：activemq-client（mq连接客户端）、activemq-spring(完成MQ和spring的整合)、activemq-broker（嵌入式mq服务器）、activemq-pool(mq连接池)等等 -->
		<dependency>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-all</artifactId>
			<version>5.15.8</version>
		</dependency>
```

-  spring的ApplicationContext配置：


```xml
<!-- 首先导入spring所需组件的xml命名空间，xmlns和xmlns:xsi必须存在；
除外还需要 amq、jms（MQ）、context（spring）
-->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:amq="http://activemq.apache.org/schema/core"<!-- 打开spring自动扫描 -->
	xmlns:jms="http://www.springframework.org/schema/jms"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd
         http://activemq.apache.org/schema/core http://activemq.apache.org/schema/core/activemq-core.xsd>

	<!-- 打开spring自动扫描 -->                        
	<context:component-scan base-package="com.yh"></context:component-scan>

	<!-- 配置mq连接工厂,用于获取connection对象，可以配置多个不同地址的连接工厂 -->
	<bean id="connectionFactory"
		class="org.apache.activemq.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://127.0.0.1:61616" />
	</bean>    

	<!-- 配置MQ目的地：队列或主题,可以配置多个 -->
	<bean id="destinationQueue"
		class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg index="0" value="spring-active-queue" />
	</bean>

	<bean id="destinationTopic"
		class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg index="0" value="spring-active-topic" />
	</bean>

	<!-- 配置创建spring提供的JMS工具类Bean，用于操作MQ消息的发送接收 属性包括：MQ连接工厂、默认MQ目的地、消息转化器,可以配置多个，用于操作不同连接的不同目的地数据 -->
	<bean id="jmsTemplate"
		class="org.springframework.jms.core.JmsTemplate">
		<property name="connectionFactory" ref="connectionFactory" />
		<property name="defaultDestination" ref="destinationTopic" />
		<property name="messageConverter">
			<bean>
				class="org.springframework.jms.support.converter.SimpleMessageConverter">			 </bean>
		</property>
        
	<!-- 配置监听器,指定监听的连接、目的地、使用的监听器；可以配置多个，用于监听不同连接的不同目的地数据 -->
	<bean id="jmsContainer"
		class="org.springframework.jms.listener.DefaultMessageListenerContainer">
		<property name="connectionFactory" ref="connectionFactory" />
		<property name="destination" ref="destinationTopic" />
		<property name="messageListener" ref="myMessageListener" />
	</bean>

```

- 对MQ消息的发送和接收


### 10、ActiveMQ整合Springboot：

- pom.xml:

```xml
		<!-- 导入activemq整合springboot依赖的启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-activemq</artifactId>
		</dependency>
```

- application.properties:**通过springboot配置文件，只能全局设置一种MQ类型（queue或topic）**

```properties
server.port=7777

#activemq连接地址
spring.activemq.broker-url=tcp://127.0.0.1:61616
#activemq目的地类型:是否为发布订阅模型（topic），默认fasle即为queue
spring.jms.pub-sub-domain=false

#自定义目的地名，可以设置多个对于不同目的地
myqueueName01=queque01  
#mytopicName=topic01
```

- activeMQ的springboot配置类：

```java
@Component
@EnableJms//开启JSM注解生效
public class ConfigBeanMQ {

	@Value("${myqueueName01}") //导入配置文件中的自定义属性，实现在配置文件中修改目的地名
	private String myQueue;
	
	//创建队列Bean
	@Bean
	public ActiveMQQueue queue() {
		return new ActiveMQQueue(myQueue);
	}
    
//   @Value("${mytopicName}")
//	private String myTopic;
//	
//	//创建主题Bean
//	@Bean
//	public Topic topic() {
//		return new ActiveMQTopic(myTopic);
//	}
}
```

- MQ的生产发送：（topic和queue一致）

```java
	@Autowired //springboot自动配置好的JMS工具类（有两种JmsMessagingTemplate和JmsTemplate）
	private JmsMessagingTemplate jmsTemplate;
	
	@Autowired  //spring依赖注入queue对象
	private Queue queue;
	
	//手动调用发送
	public void produceMsg() {
		//convertAndSend可以根据消息类型，进行合适的Message对象生成
		jmsTemplate.convertAndSend(queue, "生产者msg");
	}
	
	//并且需要在组件类（被@Configuration、@Compant、@Service等（注册为spring组件的类），但由于一般MVC层注解作用域不同，因此一般使用在@Configuration，@Compant中，如配置类）上，添加@EnableScheduling注解，让springBoot任务注解生效
	//间隔时间定时发送（即利用springboot定时任务模块，控制方法的执行）
	@Scheduled(fixedDelay=3000)
	public void produceMsgScheduled() {
		jmsTemplate.convertAndSend(queue,"生产者发送的消息"+UUID.randomUUID());
		System.out.println("发送成功");
	}
```

- MQ的消费接收：（topic和queue一致）

```java
	@Autowired
	JmsMessagingTemplate jmsTemplate;
	
	@Autowired
	private Queue queue;

	//手动调用接收
	public void receive() {
		//使用receiveAndConvert将消息体转化为指定类型		
		String msg = jmsTemplate.receiveAndConvert(queue, String.class);
		System.out.println("消费者消费msg"+msg);
	}
		
	//自动监听接收(@JmsListener根据指定的conncetionFactory、destination，来创建启动监听器)
	@JmsListener(destination="${myqueueName01}")
	public void receive(TextMessage textMessage) throws JMSException {
		System.out.println("消费者消费msg"+textMessage.getText());
	}
```

### 11、ActiveMQ数据传输协议：

ActiveMQ5.13后，支持TCP、SSL、NIO、NIO SSL、STOMP，AMQP、MQTT 和VM（用于客户端和内部虚拟机相互连接，无需网路通信）

生产中，一般使用NIO：它与TCP传输相同，但使用了NIO（非阻塞i/O）程序包代替常用I/O，从而提高更好的传输性能

#### 1、ActiveMQ传输协议和端口的修改：

通过修改conf/activemq.xml实现：

```xml
<!--默认提供tcp、amqp、stomp、mqtt、ws，不同的传输协议暴露的端口不同 
 	name:openwire 代表tcp协议
!-->      
<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <!--新增NIO协议，端口号为61618 -->
            <transportConnector name="nio" uri="nio://0.0.0.0:61618?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
        </transportConnectors>
```

#### 2、TCP协议传输特点：

1、TCP通过wire protocol（ActiveMQ将其叫为OperWire）将数据序列化成字节流

2、TCP连接URI形式为 tcp：//hostname：port？key=vlaue&key=value.....;通过后面参数来设置协议的性能属性

3、TCP协议传输稳定可靠；字节流方式传输更加高效；是基础的网络传输协议，支持任何平台

#### 3、NIO协议传输特点：

1、NIO和TCP协议类似，但底层使用了NIO程序包，来进行非阻塞的I/O创建，从而是实现更多client调用同一资源和服务端有更高的负载，减少对操作系统的线程占用

2、NIO连接URI形式为 nio：//hostname：port？key=vlaue&key=value.....;通过后面参数来设置协议的性能属性

#### 4、NIO加强版：

由于NIO只是对TCP协议数据底层传输进行了优化，理论上还可以实现多种其他协议的连接；而ActiveMQ就通过AUTO关键字 来实现使用NIO模型对任何传输协议进行自动检测，适配获取数据进行IO操作，即在使用NIO的基础上，让该端口支持多种数据传输协议：

1、**activeMQ的AUTO实现了对OpenWire（TCP），STOMP，AMQP和MQTT四种协议使用同一个端口，统一用AUTO代替，并结合NIO\SSL使用**

2、通过Auto+nio 来实现多种协议的NIO，默认允许四种协议和对应的数据格式

3、添加Auto+nio 协议和端口

```xml
<transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61608?maximumConnections=1000&amp;
                wireFormat.maxFrameSize=104857600&amp;
                org.apache.activemq.transport.nio.SelectorManager.corePoolSize=20&amp;
                org.apache.activemq.transport.nio.SelectorManager.maximumPoolSize=50" />
```

**不同的协议，会对于不同的客户端操作代码；一般以TCP/NIO协议为例编写**

### 12、ActiveMQ消息持久化机制：

ActiveMQ支持多种持久化机制，但无论那种持久化方式，消息的存储逻辑都是一致的：

**生产者发送消息后，MQ会将消息存储到本地磁盘/内存数据库/远程数据库中，当消息被成功消费后删除；失败则继续尝试发送;在服务器宕机时，可以有效保证消息不丢失；MQ重启时，再从这些存储介质中恢复消息**

- #### ActiveMQ支持的持久化机制（常用）：


#### 1、AMQ Message Store ：

（5.4之前版本的默认）使用本地文件存储，但是AMQ会对每一个Destination创建一个索引，当系统需要使用大量Queue时，索引文件大小和数量会很多，导致MQ崩溃重启时，重建索引非常慢。已经被淘汰

#### 2、JDBC：

（4.x版本推荐使用）使用数据库实现存储，从而实现远程异地持久化；**可以保证在mq本地服务器磁盘损坏的情况下，安全的保证数据的可持久化**

1、在/lib目录下，添加指定数据库对应JDBC驱动包

2、在/conf/activemq.xml 中修改持久化方式,并进行JDBC配置（以mysql为例），**ActiveMQ在启动时如果报错I llegalStateException时，可能使用计算机名带有“_”,可以更改计算机名解决**

```xml
		<!--createTablesOnStartup 自动创建MQ消息数据库表，只在MQ第一启动时开启，之后就可以注释了（不注释也没有影响） 
			ActiveMQ默认使用DBCP数据库连接池，需要使用其他连接池时，需要先在/lib中导入对应jar包，然后修改 mysql-ds组件对应的class名 
		
		对于Bean标签，应该放在broker标签外，并在import标签前
-->
		<persistenceAdapter>
			<jdbcPersistenceAdapter dataSoure="#mysql-ds" createTablesOnStartup="true"/>
		</persistenceAdapter>

		<bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-			method="close">
  			<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
  			<property name="url" value="jdbc:mysql://localhost:3306/amq_test?relaxAutoCommit=true&amp;serverTimezone=GMT"/>
  			<property name="username" value="root"/>
  			<property name="password" value="123456"/>
  			<property name="poolPreparedStatements" value="true"/>
		</bean>
```

3、在启动ActiveMQ后，数据库由于createTablesOnStartup=“true”的设置，会自动创建三张表：

- activemq_msgs，消息表，字段包括：

  id（自增主键）；

  contatiner（消息的DestinationName）；

  msgid_prod（消息发送者的主键）；

  msg_seq(相同客户端消息顺序号，代指这条消息是某个客户端发送的第几条数据，从而和消息发送者ID组合成MessageID唯一值)；

  expiration：消息过期时间，存储为从1970年到现在的毫秒数

  msg：消息对象序列化数据，blob二进制数据

  priority：优先级

- activemq_acks，用于存储持久化Topic订阅关系，字段包括：

  contatiner：消息的DestinationName

  sub_dest:显示当前mq所在集群的系统信息

  client_id:客户端唯一ID

  sub_name:TopicName

  selector：选择器，用于对满足当前客户端订阅的消息进行条件筛选，然后消费

  last_acked_id:记录当前用户对于TOPIC主题消费过的消息ID

- activemq_acks，用于记录当前集群中，MasterBroker的信息（集群中只有一台MQ服务进行消息接收和转发，即主实例），字段包括：

  id    自增主键

  time 

  broker_name  当前工作的MQ实例名

4、JDBC持久化强化：

由于JDBC持久化方式的速度比较慢，要对数据库进行多次读写；因此AcativeMQ在将消息持久化到数据库过程中，添加了一个高性能日志机制（ ActiveMQ Journal）：

当消费者消费速度能够及时跟上生产者速度时，journal日志文件就可以大大减少写入到DB中的消息

- 过程：首先MQ接收到生产者消息后，会先缓存到journal文件中，然后一段时间再一起同步到数据库；当消费者速度跟的上生产者时，journal需要同步到DB的消息就会大大减少；当消费者速度很慢时，则会将消息定时批量写入数据库，有效降低数据库的写入次数
- 配置：persistenceFactory 标签替换掉persistenceAdapter

```xml
		<persistenceFactory>
				<journalPersistenceAdapterFactory 
					journalLogFiles="4"
					journalLogFileSize="32768"
					useJournal="true"
					useQuickJournal="true"
					dataSource="#mysql-ds"
					dataDirectory="${activemq-date}/journal"/>	
		</persistenceFactory>
```

journal日志文件存放在 activemq.bat所在目录下的/activemq-date(通过dataDirectory自定义)

- 缺点：由于journal日志文件作为缓存中间层，因此不能保证数据及时持久化到DB中，因此当此时磁盘损坏时，会导致部分数据伴随journal文件一起丢失

#### 3、KahaDB：

（5.4之后版本的默认，目前最常用）和AMQ一样，使用本地文件存储，通过事务日志和一个索引文件来存储所有消息，默认保存在/data/kahadb目录下，文件包括5个：db-1.log、db.data、db.free、db.redo 和lock

- db-<number>.log:事务日志文件，默认大小32M，当文件满后创建下一个数字后缀的log文件，文件中的日志对应消息都被消费时，文件将会被删除或归档
- db.data： 索引文件，包含了所有持久化消息对应的B-Tree索引，执行db-<number>.log 日志文件存储的消息
- db.free:  保存da.bata文件中空闲页面的ID，保证后续消息持久化索引的连续性
- db.redo :用于MQ非正常关闭时，btree索引的重建
- lock ：文件锁，用于获取当前Kabadb读写权限的broker（MQ实例），保证多客户端并发读写消息时的安全性

**只需要一个索引文件（B-Trre结构），大大改善了AMQ索引过大和索引恢复慢的缺点，从而适合多Destinationd的场景，可以完全取代AMQ**

在/conf/activemq.xml 修改持久化方式，默认配置了kahadb：

```xml
        <persistenceAdapter>
            <kahaDB directory="${activemq.data}/kahadb"/>
        </persistenceAdapter>
```

#### 4、levelDB：

（5.8以后引入的）和KahaDB相似，是基于文件的本地数据库存储，但是提供比KahaDB更快的持久性，并且不使用B-Tree索引，而是使用自定义的LevelDB索引（使用不普及，但是是未来发展趋势）

- 设置producer.setDeliveryMode(DeliveryMode.PERSISTENT); 时，则消息进行持久化，保存到broker相应文件，或数据库中。非持久化时，消息保存到内存中

- 对于queue，数据被消费后，直接从broker删除（即持久化数据也会被删除）
- 对于topoc，数据被目前所有订阅者消费后，持久化数据并不会删除（对于topic，mq并不会在内存中保存）

ActiveMQ，保存到本地文件中，但是本地器硬盘损坏时，会导致消息丢失，因此需要结合**异地的持久化方式**来保证MQ持久化的可靠性（Redis只是作为缓存时（存入内存的数据，就已经默认可接受丢失风险），不考虑硬盘损坏导致本地持久化失效的情况，并且Redis一般会进行主从复制来保证数据的异地备份）

### 13、ActiveMQ的高可用（集群）：

#### 1、三种集群方式：

在使用ActiveMQ集群时，需要选择合适的持久化方式，以下是三种常用持久化的集群的优缺点（JDBC加强版，由于DB不能及时获取持久化数据，因此无法实现持久化同步）：

| 持久化方式 | 要求                                                         | 优点                               | 缺点                       |
| ---------- | ------------------------------------------------------------ | ---------------------------------- | -------------------------- |
| kahaDB     | 需要有分布式文件共享系统，如NFS（用于共享持久化缓存文件，实现持久化同步） | 实现主从复制容错模型               | 需要共享文件系统           |
| JDBC       | 其他集群共享同一个DB（实现持久化同步）                       | 实现主从复制容错模型               | 需要共享数据库，并且性能慢 |
| levelDB    | 需要zookeeper服务器                                          | 实现主从复制容错模型，并且速度更快 | 需要zookeeper服务器        |

#### 2、LevelDB持久化方式的集群实现：

5.9以后，官方推荐使用 zookeeper+可复制的levelDB来搭建ActiveMQ主从复制集群：

1、通过Zookeeper来注册所有activeMQ服务器，并使用zookeeper自身集群的选举策略，来选择ActiveMQ集群其中一个节点作为Master提供服务，其他作为Slave只进行持久化同步

2、当zookeeper检测到Master不能提供服务时，就会从slave中选举一个作为Master；而被切换下来的节点在恢复后，会被作为slave

3、slave进行消息同步时，Zookeeper选举策略要求，需要保证与新Master完成同步的slave个数要大于n/2，才能返回SUCCESS给Master，从而完成同步，Master继续提供服务，slave继续持久化同步

4、由于与新Master完成同步的slave个数要大于n/2，因此必须保证ActiveMQ集群的服务器数大于等于3（当服务器为2时，出现一个宕机后，另一个作为Master，就没有可以同步数据的slave了），并且奇数个可以有效节省服务器资源（当服务器为5，6时，它们的最大允许宕机数都为2），

#### 3、集群搭建过程：

​	1、配置zookeeper集群（zookeeper用于服务的分发，因此要保证高可用）

​	2、搭建三台不同地址（本地则使用不同端口，学习测试用）的ActiveMQ服务器，并进行配置：

- 通过jetty.xml文件，来修改控制台端口；

- 通过activemq.xml文件修改：

  1、修改端口号，

  2、borker标签中的 borkerName属性值保持一致，说明它们为同一个集群；最后设置三个节点的持久化配置：

  3、使用replicated +levelDB持久化,即可复制的levleDBC：

  directory：持久化日志文件保存目录

  replicas：集群节点数

  bind：当前节点tcp地址

  zkAddress：zookeeper集群地址

  zkPassord：zookeeper密码

  zkPath：持久化复制时，文件保存在zookeeper的路径地址

  hostname：当前主机ip/域名

  ```xml
  <persistenceAdapter>
      <replicatedLevelDB
        directory="${activemq-data}/leveldb"
        replicas="3"
        bind="tcp://0.0.0.0:61616"
        zkAddress="zoo1.example.org:2181,zoo2.example.org:2181,zoo3.example.org:2181"
        zkPassword="password"
        zkPath="/activemq/leveldb-stores"
        hostname="broker1.example.org"
        />
    </persistenceAdapter>
  ```

  4、修改transportConnector 中的uri的端口号，对应持久化中的bind属性

3、启动zookeeper集群和activeMQ集群

4、通过zookeeper查看注册的activemq集群，并找出Master节点

5、启动项目前，修改activeMQ的url：从以前的单地址连接，改为集群连接，failover策略（故障转移）：

```java
url=failover:(tcp://127.0.0.1:61617,tcp://127.0.0.1:61618,tcp://127.0.0.1:61619)
```

#### 4、ActiveMQ集群特点：

1、ActiveMQ集群当只有一个节点活动时，由于zookeeper不能进行Master选取，因此ActiveMQ不能正常运行

2、Zookeeper集群只有一个节点活动时，无论ActiveMQ有几个节点存活，都不能提供正常服务，即Active的高可用建立在Zookeeper的高可用上

3、ActiveMQ由于需要保证消息消费的可靠性，在满足性能需求的情况下，不能实现多节点同步消费。因此集群只对外暴露Master进行消息中间件服务，即集群不能分担Master的负载，只进行备灾容错（这个和redis集群有很大区别）

### 14、ActiveMQ高级特性：

#### 1、保证MQ的高可用性：

消息事务发送、消息消费应答、消息持久化，以及搭配zookeeper使用带复制功能的levelDB存储，来实现集群的主从复制

#### 2、异步投递：

当使用MQ进行消息发送和消费时，默认已经就是异步模式，即一个消息的发送和消费是异步进行的。

- 异步投递，只是用于描述客户端对MQ的消息发送，可以是异步或者同步。ActiveMQ默认使用异步发送的模式，只有在如下情况下，才会使用同步发送方式：

  1、显式使用同步发送方式

  2、未使用事务的持久化消息发送。当未使用事务时，消息发送后，需要保证MQ对消息进行持久化，因此就需要接收持久化的确认信息，从而就只能使用同步方式完成；

- 在允许消息发送失败或持久化失败的情况下丢失少量数据时（userAsyncSend=true），推荐使用异步方式发送，最大化的提高生产者的生产效率，当然提高效率的同时，也会增加服务器负载：

  1、会消耗较多的client内存，并提高MQ服务端接收消息的负载

  2、异步消息发送，在没有事务的情况下，不能保证消息的成功发送，需要容忍消息丢失的可能

  3、在消费者端消费能力较低时，会导致MQ的消息积压，最后出现溢出

- 异步投递的使用方法：

  1、在创建AMQ连接工厂的URL中，添加配置参数

  ```java
  ActiveMQConnectionFactroy connectionFactroy = new ActiveMQConnectionFactroy("tcp://localhost:61616?jms.useAsyncSend=turn")
  ```

  2、在AMQ连接工厂创建后，注入userAsyncSend属性为true

  ```java
  factory.setUseAsyncSend(true);
  ```

  3、AMQ连接对象创建后，注入userAsyncSend属性为true

  ```java
  ((ActiveMQConnection)connection).setUseAsyncSend(true);
  ```

  对应三种不同作用域，来设置最后连接对象创建后的userAsyncSend属性为true

- 异步投递保证消息成功发送：

异步投递想要保证消息成功发送，就需要进行send方法的回调，来接收消息发送成功的确认信息，这也是同步和异步最大的区别：

同步：消息发送后，send方法阻塞等待确认信息返回

异步：消息发送后，send方法无需阻塞等待，而是通过send方法的回调机制，回调接收确认信息，客户端自己判断消息是否发送成功

```java
		ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL_STRING);
		Connection connection = factory.createConnection();
		//开启异步发送
		((ActiveMQConnection)connection).setUseAsyncSend(true);
		//此时连接启动
		connection.start();
		
		//创建指定主题发送者
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		Topic topic = session.createTopic("topic01");
		ActiveMQMessageProducer producer = (ActiveMQMessageProducer)session.createProducer(topic);
		
		//设置当前发送者主题消息持久化(topic消息默认non_persistent非持久化)
		producer.setDeliveryMode(DeliveryMode.PERSISTENT);
		
		//创建message
		TextMessage message = session.createTextMessage();
		
		//设置当前消息头ID，并且本地保存
		message.setJMSMessageID(UUID.randomUUID().toString()+"myqueue01");
		String msgId = message.getJMSMessageID();
		
		//set消息体，并异步发送，并添加回调
		message.setText("topic-------"+"msg02");
		producer.send(message, new AsyncCallback() {
			
			@Override
			public void onException(JMSException exception) {
				//发送消息失败
				System.out.println(exception.getMessage());
			}
				
			@Override
			public void onSuccess() {
				//发送成功
				System.out.println(msgId+"发送成功");
			}
		});
```

#### 3、延迟投递和定时投递

延迟、定时投递可以通过springboot的定时任务机制来完成，但是只支持方法级别，当一个方法中存在多个消息发送，并只有其中部分消息需要延迟、定时定时投递时，此时就需要message级别的定时设置

ActiveMQ自身的延迟投递和定时投递机制，需要通过配置文件进行激活：

**在activemq.xml中的broke标签中添加schedulerSupport="true“属性**

```java
		ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(ACTIVEMQ_URL_STRING);
		Connection connection = factory.createConnection();
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		connection.start();

		Queue queue = session.createQueue("queue01");
		MessageProducer producer = session.createProducer(queue);

		long delay = 3 * 1000;//延时投递时间
		long period = 4* 1000;//定时投递间隔
		int repeat =5;//定时投递次数

		for (int i = 0; i < 3; i++) {
			TextMessage message = session.createTextMessage("msg"+i);
			message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
			message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
			message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, repeat);
			producer.send(message);
			System.out.println("发送成功"+i);
		}
```

- 使用activeMQ自身的延时、定时投递功能时，是通过ActiveMQ自己完成的，客户端只会将消息异步发送一次，MQ接收到消息后台，通过对消息属性的解析，指定该条消息定时发送策略，从而在queue或topic中自动延迟、定时生成消息

#### 4、消息消费重试机制：

- 当消息消费时没有做出应答，则消息会重新消费（CLIENT_ACKNOWLEAGE，没有手动应答时，不会触发重试机制，消息会永远重复消费，因此CLIENT_ACKNOWLEAGE模式下，必须进行手动应答），常见情况有：

1、消费端使用事务进行了rollback操作

2、消费者使用事务，但没有进行commit操作

3、消费者在CLIENT_ACKNOWLEAGE应答模式下，session调用了recover（），即执行消息重发；

**ActiveMQ默认 消息重发间隔 1s  、消息重发次数 6次**

- 有毒消息Poison ACK

  当消息重发超过默认次数，则MQ会将该消息标记为 poison ack，表示该消息有毒，从而broker不再发送该消息给消费者，而是将该条消息放入 DLQ（死信队列）

- 消息重试机制常用参数与配置：**（重试机制是根据消费者进行消息消费失败时，对当前消息设置下一次重试参数来实现的，因此只能配置消费端的ConnectionFactory，之后每消费一次，redeliveryCounter+1，当redeliveryCounter达到重试次数时，消费者告诉MQ将其放入DLQ）**

1、maxmumRedeliveries,最大重发次数，默认6次

2、initialRedeliveryDelay，初始重发间隔时间，默认1s

3、useExponentialBackOff，启用倍数递增的方式增加延迟时间，默认为false

4、maixmumRedeliveryDelay，最大重发时间间隔，默认-1 即没有最大值限制（useExponentialBackOff=true时生效）

5、backOffMultiplier，重发时间间隔递增倍数，默认为5（useExponentialBackOff=true时生效）

6、useCollisionAvoidance：防止冲突功能，默认false，避免消息在同一个时间点被延迟重发处理

7、collisionAvoidanceFactor：设置防止冲突的正负百分比

```java
		//设置connetionFactory的重发策略
		RedeliveryPolicy redeliveryPolicy = new RedeliveryPolicy();
		redeliveryPolicy.setMaximumRedeliveries(3);//重发次数
		redeliveryPolicy.setInitialRedeliveryDelay(2000L);//重发间隔时间
		activeMqfactory.setRedeliveryPolicy(redeliveryPolicy);
		
		Connection connection = activeMqfactory.createConnection();
```

同理，spring则创建RedeliveryPolicy bean，然后注入到ActiveMQConnectionFactory 组件的属性中

#### 5、DLQ（死信队列）：

当一条消息被多次重发后，会被ActiveMQ移入死信队列，用于开发人员获取哪些消息重发发送失败，然后进行一定的人工干预

一般情况下，生产中会使用MQ来设计两个队列；核心业务队列和死信队列

前者用户保存生产者发送的消息，用于消费者消费；后者就是保存多次消费失败的消息，如系统故障导致的订单多次消费失败，从而恢复处理这些失败的订单

默认情况下，DLQ被当前MQ的所有消息消费失败重试策略共享，名为ActiveMQ.DLQ

也可以针对于每个queue、Topic来创建独立的DLQ，使用activemq.xml如下配置：

 ```java
<policyEntry queque="queue01">  //这里也可以为topic
    <deadLetterStrategy>
    	<individualDeadLetterStrategy queuePrefix="DLQ" userQueueForQueueMessages="false" />  //死信队列的前缀，是否使用该DLQ来保存topic的重试消费失败的消息
    </deadLetterStrategy>
<policyEntry />
 ```

- DLQ特点：

  1、Activemq默认是不会将非持久化的有毒消息放入死信队列中（非持久化消息，就容忍了消息的丢失，因此更不会去处理消费失败的消息）

  可以通过配置文件进行设置，保存非持久化有毒消息，设置processNonPersistent=true

  2、Activemq默认会保存过期消息，如果不需要是，可以通过 processExpired=fasle  开启自动删除过期消息

#### 6、防止重复消费：

在正常使用MQ的过程中，由于网络原因，也可能会导致消息重复消费（即在消息被获取，但是消费端还没有返回应答信息时，connection断开）

1、如果消费的消息是进行数据库的插入操作，则可以通过使用消息数据中的某个唯一标识符字段作为数据库存储的主键ID，从而防止消息的重复消费；但这种方式有条件限制：需要数据存在唯一标识符、消息用于存储到数据库中

2、使用redis进行第三方的MQ消费记录，在消息消费时，给消息分配一个全局唯一ID，和消息作为K-V保存到redis中，当消费者消费消息前，先在redis中查询该消息ID是否存在，没有即可消费

#### 7、答疑：

##### 1、非持久化主题，能进行重复消费吗？

A:首先，Topic和queue的应答机制和重复消费机制相同，对于消息是否为持久化消息，并不会影响消息的重复消费机制。而只是会影响MQ服务器重启后的未消费消息，是否能重新恢复

##### **2、对于生产者，设置消息头JMSpriority属性，有什么作用？**

A：默认情况下，该属性设置不会生效（MQ保存的消息的JMSpriority=4）；需要通过配置文件设置进行改变，一般情况下不使用

**对于消费者的优先级，可以通过配置文件进行设置**

##### **3、生产者开启事务进行异步发送的过程是怎样？**

A：

​	1、首先默认情况下，非持久化消息和开启事务的消息都使用异步发送，而持久化消息使用同步发送，确保消息持久化；当显式开启异步发送模式（useAsyncSend=true）时，任何类型消息进行send方法都会进行异步调用。

​	2、当开启事务进行异步发送时，虽然send方法不会阻塞，但是commit方法还是会阻塞，用于返回消息的成功发送（对于持久化消息，则为消息的成功持久化）

​	3、对于没有开启事务的异步发送，需要添加回调机制，来处理消息是否发送成功：producer.send(message, new AsyncCallback(){XXX}）

##### **4、mq整合spring、springboot，配置连接池：**

1、activemq提供PooledConnctionFactory来实现缓存技术，用于缓存connction、session、productor，但是不会缓存consumer。适合于发送者，可以设置maxConnections（最大连接数）、maximumActiveSessionPerConnection（每个连接最大会话数）、idleTimeout（连接最大空闲时间，超出则自动回收）、expiryTimeout（连接过期时间，即最长使用时间，**该设置已过期**）

使用：activemq连接池依赖activemq-pool和commons-pool2依赖，（注意当通过activemq-all包来引入activemq-pool依赖时，并不会自动依赖commons-pool2；而单独使用activemq-pool依赖时，则会自动依赖commons-pool2，不需要显式依赖导入）

```xml
    <!-- 注册connectionFactory组件 -->
	<bean id="activeMQConnectionFactory"
		class="org.apache.activemq.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://127.0.0.1:61616" />
	</bean>

	<!-- 注册PooledConnectionFactory组件，管理connectionFactory对象 -->
	<bean id="jmsPoolFactory"
		class="org.apache.activemq.pool.PooledConnectionFactory" >
		<property name="connectionFactory" ref="activeMQConnectionFactory" />
		<!-- 设置最大连接数 -->
		<property name="maxConnections" value="10" />
	</bean>
```

2、而spring2.5.3后，提供CachingConnectionFactory实现对JMS资源缓存功能，包括messageProducer、messageConsumer和session的缓存功能，来优化JmsTemplat的使用,可以设置sessionCacheSize（session的最大缓存数）；CachingConnectionFactory是继承了SingleConnectionFactory，因此会忽略connction.close()方法，一直使用同一个connection对象进行JMS操作，当出现异常时，才会重新创建一个connection对象**（推荐使用）**

使用：由于是spring针对于JMS的缓存优化，由spring-jms提供（activemq-all包含该依赖，因此无需导入其他依赖）

```xml
    <!-- 注册connectionFactory组件 -->
	<bean id="activeMQConnectionFactory"
		class="org.apache.activemq.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://127.0.0.1:61616" />
	</bean>
	
	<bean id ="jmsPoolFactory" 		     class="org.springframework.jms.connection.CachingConnectionFactory">
        <property name="targetConnectionFactory" ref="activeMQConnectionFactory"/>  
        <property name="sessionCacheSize" value="10"/>
    </bean>

```

3、springboot默认配置支持activemq原生pool和spring对JMS的缓存；但是springboot-activemq启动器，并不包含activemq-pool依赖，因此需要自己手动导入

```properties
#spring对JMS的缓存配置
spring.jms.cache.enabled=true
spring.jms.cache.session-cache-size=10

#spring对actviemqPool配置（需要额外导入activemq-pool依赖，否则会出现JmsTemplate注入失败）
spring.activemq.pool.enabled=true
spring.activemq.pool.idle-timeout=30s
spring.activemq.pool.max-connections=100
spring.activemq.pool.max-sessions-per-connection=10
```

对于原生使用activemqPool，springboot2.0版本和以下，使用activemq-pool依赖；但springboot2.1后，使用pooled-jms依赖

- 当两种一起配置时，由于jmsCache只会一直使用同一个conncetion，因此导致activemqPool并不会起作用，一般情况下，不应该都配置，推荐使用jmsCache

##### **5、spring、springboot配置多个MQ/多个队列或主题**

1、对于spring，通过配置文件，可以直接进行不同mq的连接工厂配置，并根据connectionFactory、jmsTemplate、DefaultMessageListenerContainer配置；

2、对于springboot，默认配置只支持一个MQ的一种消息类型（队列或主题），**只支持单消息类型，不并不是只能进行单消息类型的发送和消费，简单的queue/topic消息发送都可以正常完成；**

**只是springboot默认只能自动创建一种类型的消费监控器，因此会导致另个目的地消费者无法使用默认MessageListenerContainer的@JmsListener注解**

- 需要手动创建配置类，根据对应目的地配置监听器工厂、JmsMessagingTemplate：

1、使用springboot提供的注册的ConnectionFactory

```java
	//依赖调用，使用spring自动配置注册的ConnectionFactory，id=jmsConnectionFactory
	@Autowired
	private ConnectionFactory factory;

	@Value("${clinetId01}")
	private String clientId;

	//自定义JmsListenerContainerFactory工厂,分别为queue、topic
	@Bean
	public JmsListenerContainerFactory<?> jmsListerContainerQueue() {
		DefaultJmsListenerContainerFactory lisenterFactory = new DefaultJmsListenerContainerFactory();
		lisenterFactory.setConnectionFactory(factory);
		lisenterFactory.setPubSubDomain(false);//属于队列监听器工厂
		return lisenterFactory;
	}

	@Bean
	public JmsListenerContainerFactory<?> jmsListerContainerTopic() {
		DefaultJmsListenerContainerFactory lisenterFactory = new DefaultJmsListenerContainerFactory();
		lisenterFactory.setConnectionFactory(factory);
        lisenterFactory.setPubSubDomain(true);//属于主题监听器工厂
        lisenterFactory.setClientId(clientId);//对于需要指定clientId
        lisenterFactory.setSubscriptionDurable(true);//开启持久化主题订阅		
		return lisenterFactory;
	}
```

**springboot中，消息发送默认是持久化的，而持久化主题订阅只能使用监听器完成**

2、自定义ConnectionFactory（用于配置多个MQ服务连接）

```java
//	//当有多个MQ数据源时，可以通过自定义properties文件种的k-V进行brokerURL的配置注入
	//然后使用@Qualifier将自定义ActiveMQConnectionFactory作为参数
	@Bean
	public ActiveMQConnectionFactory defaultConnectionFactory() {
		return new ActiveMQConnectionFactory(brokerURL);
	}

	//自定义JmsListenerContainerFactory工厂,分别为queue、topic
	@Bean
	public JmsListenerContainerFactory<?> jmsListerContainerQueue(@Qualifier("defaultConnectionFactory")                            ActiveMQConnectionFactory factory) {
		DefaultJmsListenerContainerFactory lisenterFactory = new DefaultJmsListenerContainerFactory();@Qualifier("defaultConnectionFactory") ActiveMQConnectionFactory factory
		lisenterFactory.setConnectionFactory(factory);
		lisenterFactory.setPubSubDomain(false);//属于队列监听器工厂
		return lisenterFactory;
	}
```

3、自定义JmsMessagingTemplate：

对于配置MQ目的地，并不依赖于ConnectionFactory，只需要对每个目的地进行唯一命名，并指定目的地类型。因此不同MQ的JmsMessagingTemplate可以使用同一个目的地

```java
	//定义主题/队列的JmsMessagingTemplate（针对于某个Connectionfactory）
	@Bean
	public JmsMessagingTemplate topicJmsMessageingTemplate(@Qualifier("defaultConnectionFactory") ActiveMQConnectionFactory factory) {
		JmsTemplate jmsTemplate = new JmsTemplate(factory);
		jmsTemplate.setPubSubDomain(true);
        //开启deliveryMode持久化、priority优先级、timeToLive消息过期时间的配置
		jmsTemplate.setExplicitQosEnabled(true);
		return new JmsMessagingTemplate(jmsTemplate);
	}
	@Bean
	public JmsMessagingTemplate queueJmsMessageingTemplate(@Qualifier("defaultConnectionFactory") ActiveMQConnectionFactory factory) {
		JmsTemplate jmsTemplate = new JmsTemplate(factory);
		jmsTemplate.setPubSubDomain(false);
        //开启deliveryMode持久化、priority优先级、timeToLive消息过期时间的配置
		jmsTemplate.setExplicitQosEnabled(true);
		return new JmsMessagingTemplate(jmsTemplate);
	}
```

- 当进行自定义JmsMessagingTemplate时，由于自动配置添加@ConditionalOnMissingBean注解（当spring容器中存在该类型Bean，则不进行注入），因此自定义时会覆盖自动配置的JmsMessagingTemplate（jmsTemplate同理）；

- JmsListenerContainerFactory,对于spring提供一个默认工厂，当使用@JmsListener没有指定containerFactory时，提供使用，并根据springboot的配置进行属性设置（如PubSubDomain，根据spring.jms.pub-sub-domain=false配置决定）

- 由于JmsListenerContainerFactory、JmsMessagingTemplate它们都区分queue和主题，通过spring.jms.pubSubDomain属性进行设置，默认为false（queue）；

- PubSubDomain设置true/false的区别：

  1、针对于JmsListenerContainerFactory，根据PubSubDomain属性，只能对一种类型消息进行@JmsListener监听处理

  2、对于生产者，true/false的JmsMessagingTemplate都能进行消息发送；对于消费者，默认持久化方式的消息、？？？？

##### 6、JmsMessagingTemplate和JmsTemplate对象，两者的AMQ参数配置和使用方法：

- JmsMessagingTemplate和JmsTemplate对象的区别:

  JmsMessagingTemplate是对JmsTemplate的进一步封装，屏蔽了JmsTemplate对消息发送和接收的一些配置方法，使用更加简洁;

  JmsMessagingTemplate的注入依赖于JmsTemplate，在使用JmsMessagingTemplate进行配置修改时，需要先getJmsTemplate，然后操作获得的JmsTemplate对象

- JmsMessagingTemplate和JmsTemplate对象的都有的常用方法：

  1、都提供了对connectionFactory、destination、MessageConvert的配置与获取

  2、提供了send、convertAndSend、recerive、receriveAndConvert方法，用于消息的发送和接收，但又有些许区别，部分方法JmsMessagingTemplate做了额外处理（对于message会进行动态转化）

  - send方法，JmsTemplate提供一个MessageCreator接口对象作为参数，来简化对message的创建；JmsMessagingTemplate需要Message对象做参数；**一般情况下，需要对message对象进行处理时，使用**
  - receriveAndConvert方法，JmsTemplate直接使用Object接收，然后进行类型转化；而JmsMessagingTemplate会已消息体类型的class对象作为参数，使用泛型进行动态类型接收
  - recerive方法，JmsTemplate直接使用父类JMS.Message接口类型变量接收，需要自己强转转化为数据约定类型；而JmsMessagingTemplate使用messaging.Message<?>，提供获取当前消息头、消息体的方法，通过对消息头的判断，然后来选择性的进行消息体类型转化

  3、对于converAndSend方法提供多种参数列表的方法，其中**JmsTemplate.convertAndSend(D destination, Object payload,  MessagePostProcessor postProcessor)方法，提供对java数据转化Message对象后的额外处理，如设置消息头和属性、延迟投递和定时投递**

- JmsTemplate的常用参数配置（JmsMessagingTemplate是通过JmsTemplate实现的，因此其参数配置就是设置它成员变量JmsTemplate的参数）

```java
		jmsTemplate.setDeliveryDelay(3*1000L);//发送延迟时间，默认-1不延迟
		jmsTemplate.setDeliveryPersistent(true);//消息是否持久化，默认持久化
		jmsTemplate.setReceiveTimeout(3*1000L);//接收超时时间，默认0
		jmsTemplate.setSessionAcknowledgeMode(Session.AUTO_ACKNOWLEDGE);//应答方式，默认AUTO_ACKNOWLEDGE
		jmsTemplate.setTimeToLive(0);//消息过期时间，默认0永不过期
		jmsTemplate.setMessageConverter(messageConverter);//自定义消息转化器
		jmsTemplate.setPriority(4);//设置消息消费的优先级，默认为4
```

**除了对JmsTemplate进行设置外，还可以通过配置发送的message来重写设置（部分设置需要通过ConnctionFactory进行设置，如重发策略）**

- JMS消息转化器				 

messageConverter，spring提供对数据对象自动封装为Message，和Message自动解析为数据对象的功能；从而不需要手动创建Message，进行属性设置；spring提供的messageConverter的具体类为SimpleMessageConverter：

主要实现两个方法：fromMessage（Message message）、toMessage(Object object, Session session)；

toMessage针对于消息体Object数据类型，来创建对应Message实现类

fromMessage针对于Message 的具体实现类类型，进行不同数据类型的Message的消息体获取

##### 7、spring整合AMQ的异步发送和事务：

- 异步发送：

和原生方式一致，默认情况在非持久化消息和事务内消息都使用异步发送；非事务的持久化消息使用同步发送，因此可以通过控制消息的持久化，来确定消息是否同步发送；当url添加**useAsyncSend=true**，此时非事务的持久化消息也使用异步发送

- 事务：

1、消费者事务：

消费者通过DefaultMessageListenerContainer（它自带本地JMS的处理），可以直接对整个方法进行事务处理，从而触发消息重试机制，因此只需要在DefaultMessageListenerContainer注入spring容器时，设置setSessionTransacted(true)，默认为false

2、生产者事务：

默认情况下，默认关闭事务，可以通过**jmsTemplate.setSessionTransacted(true)**进行设置;

但是该事务，只是处理消息发送和接收时的异常，即jmsTemplate的send方法和receive方法中的异常；无法因为代码出现处理异常，而回滚，消息重新发送，或重新消费（目前springboot并没有单独的方式处理，但可以通过分布式事务来实现）

3、分布式事务：

在DefaultMessageListenerContainer中，setTransactionManager，添加一个事务管理器，用于进行分布式事务处理，让该事务管理器协同JMS事务一起处理监听器中的代码：

通过添加jtaTransactionManager实现，如在进行MQ消息消费的的同时，需要对数据库进行insert/update操作，此时就需要使用分布式事务，实现JDBC和JMS事务的共同commit or rollback

