# Zookeeper

## 1、什么是zookeeper：

zookeeper是一个分布式、高性能、google开源的分布式系统的协调服务，为分布式应用提供数据一致性服务。

- 设计模式角度：

​		是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接收观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式

- 本质核心角度：

  是一个拥有文件修改 **通知机制** 的 **文件管理系统**

## 2、Zookeeper功能：

1、命名服务：

​		zookeeper文件系统，提供了一个类似于linux系统的文件目录，并以path为节点名，非常简单的解决了创建zookeeper资源访问的唯一ID

2、统一配置维护（数据发布/订阅）：

​		通过zookeeper来存放配置信息，给多台客户端服务器使用，保持一致性。当需要改变配置时，只有变动zookeeper中的信息，则客户端配置的监听器就会自动回调提示客户端修改相应配置

3、集群管理（提供选举机制）：

​		通过集群客户端创建临时目录节点。可以很好的获取客户端的在线状态（离线，则临时节点消失），从而使其他客户端快速获取该变化，然后通过临时节点的编号顺序（该顺序可以通过一个选举机制进行控制），选择第一个作为客户端集群的master（如activeMQ的集群管理）

4、分布式消息同步和协调机制：

​		支持对Dubbo RPC注册：实现RPC的注册中心，感知服务实例的上下线

​		zookeeper分布式锁：当需要并发访问某个分布式节点资源时，通过在zookeeper上创建该资源对于的znode节点，来从而获取该资源的锁，进行操作；而操作完成后进行删除该节点，从而释放资源的锁

​		zookeeper分布式队列：通过zookeeper的节点自动编号特性，来实现简单的**FIFO队列**，优先获取第一个节点或最后一个节点；或者**同步队列**，当保存一定数量的节点后，才能对数据进行处理

5、负载均衡：

​		通过记录每个节点的访问次数，来控制该访问哪个相同资源的集群节点服务器（控制信息不全面，因此功能不够强大）

## 3、Zookeeper安装和配置：

Windows系统中，需要创建一个conf/zoo.cfg 的配置文件，默认有一个zoo_sample.cfg文件 最为模板使用（Linux系统中可以使用docker容器搭建，并设置对应镜像启动参数）

- zoo.cfg配置文件中主要的几个属性设置：

dataDir  数据缓存目录（默认为/tmp/zookeeper） 

clientPort  客户端监听端口（默认为2181） 

tickTime  zookeeper和观察者/zookeeper集群 之间保存通信的心跳间隔时间，默认2000ms

intitlimit  zookeeper集群建立的最大初始化时间，默认10 个心跳间隔

syncLimit  zookeeper集群保持通信的最大间隔时间，默认 5个 心跳间隔，如果该间隔内，zookeeper没有通信响应，则认为节点已经挂掉，将其踢出集群

添加**admin.serverPort=xxxx**属性，替换8080默认值

在3.5版本后，Zookeeper提供一个HTTP服务，用于远程操作执行zookeeper  命令，并且默认端口为8080，因此为了防止和Tomcat冲突，应在zoo.cfg配置文件中添加  admin.serverPort=8888，用于修改该HTTP服务默认端口

- zookeeper启动、关闭、客户端连接、退出（Linux操作，window直接使用cmd文件）

zkServer.sh start  启动

zkServer.sh stop  关闭

zkServer.sh  status  查看zookeeper运行状态

zkCli.sh  客户端连接

quit    客户端退出

## 4、Zookeeper文件管理系统基本操作：

zookeeper文件管理系统，和linux基本一致，使用树型节点结构，但每个节点并不是表示的文件夹，而类似于是一个文件。使用路径名作为该文件在Zookeeper中的唯一标识，并可以通过K-V方式，在节点中存储数据，类似于windows中的注册表

因此zookeeper的文件管理操作，基本就围绕着znode节点：

1、查询连接的zookeeper中，指定节点下的子节点：根路径（/）下 默认有一个zookeeper  子节点			

​		ls  Path   							

​		ls2  path    除了查询子节点外，还会查询当前节点的stat结构体数据							

2、添加节点：create  Path

3、删除节点（必须保证该节点没有子节点）：delete Path

4、设置节点中的数据：set  Path  msg

5、获取节点中的信息：get  Path  msg

6、获取节点的stat结构体：stat  Path

7、递归删除当前节点下的所有节点（包括当前节点）：rmr Path   （remove recursive）

## 5、Zookeeper文件系统结构特征：

1、zookeeper中每一个节点，叫做znode，默认能够存储1M的数据，它们的路径为其唯一标识

2、zookeeper维护了一个stat结构体，stat包含了zookeeper所有znode的基础信息和数据变化信息：

cZxid：引起该节点被创建的操作id，或创建该节点事务id

ctime：znode创建时间 （ms）

mzxid：znode最近更新的操作id

mtime：znode最近更新的时间

pZxid：znode最近更新的子节点的操作id

cversion：znode子节点版本号，每修改一次+1

numchildren：子节点数

dataLength：znode数据字符长度

3、znode有两种存在类型：持久化/临时

在使用create 命令时，可以添加 -e、-s参数，分别为；对于序列号会从0000001开始增加，保证了为znode标识符的唯一

| 参数    | 意义                                               |
| ------- | -------------------------------------------------- |
| 无参数  | 创建持久化znode                                    |
| -e      | 创建临时znode（在客户段连接断开时，临时znode丢失） |
| -s      | 创建持久化znode，并且在znode标识符上添加序列号后缀 |
| -e   -s | 创建临时znode，并在znode标识符上添加序列号后缀     |

## 6、Zookeeper的java客户端操作：

Zookeeper目前有两个客户端工具：zkclient和curator

由于Doubbo2.7.x版本后，只支持curator，因此对于zookeeper的java客户端操作，推荐使用curator包进行连接操作

- Pom.xml

```xml
		<!-- 导入zookeeper依赖 -->
		<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.2.0</version>
		</dependency>
```

- Zookeeper的HelloWorld

```java
	//zkURL、session超时时间
	private final static String ZK_URL = "127.0.0.1:2181";
	private final static int SESSION_TIMEOUT = 30 * 1000;

	//创建zookeeper连接实例
	public ZooKeeper startZookeeper() throws IOException {
     //当watcher参数，传递null时，会出现一个NPE，但没有关系；当然也可以传入一个new Watcher()内部类
		return new ZooKeeper(ZK_URL, SESSION_TIMEOUT,null);
        //该watcher用于监控zookeeper连接成功/失败的事件
	}

	//关闭zookeeper连接
	public void stopZookeeper(ZooKeeper zk) throws InterruptedException {
		if (zk != null) {
			zk.close();
		}
	}
	
	//创建znode
	public void createZnode(ZooKeeper zk,String nodePath,String nodeValue) throws KeeperException, InterruptedException {
		//Ids.OPEN_ACL_UNSAFE 关闭节点访问权限（ACL为zookeeper权限）
		//CreateMod.PERSISTENT 创建持久化的节点
		zk.create(nodePath, nodeValue.getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
	}
	
	//get znode数据
	public String getZnode(ZooKeeper zk,String nodePath) throws KeeperException, InterruptedException {
        //必须传递一个空Stat
		byte[] data = zk.getData(nodePath, null, new Stat());
		String result = new String(data);
		return result;
	} 
	
	//main方法，进行基本zookeeper API的客户端操作
	public static void main(String[] args) {
		HelloZK helloZK = new HelloZK();
		try {
			ZooKeeper zoo = helloZK.startZookeeper();
			//查询是否有节点
			if (zoo.exists("/zooTest", false)==null) {
				helloZK.createZnode(zoo, "/zooTest", "testValue");
				String znode = helloZK.getZnode(zoo, "/zooTest");
				System.out.println(znode);
			}else {
				System.out.println("该节点已经被创建");
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

## 7、zookeeper的watcher通知机制（监控器）：

​	用于实现zookeeper的观察者功能，通过**异步回调的触发机制**来实现；zookeeper客户端可以在zookeeper服务器的每个znode上设置一个观察，当znode节点发生变更时，watch触发，此时客户端就会回收到一个通知包，被告知所观察znode的变化

- watcher（监控器）特点：


1、一次触发，当zookeeper的znode数据发送变更后，watcher触发，向客户端发送要给watch通知包，此时客户端就不再继续监听该znode了。如果想要进行永久监控，就需要在处理watch通知包的回调方法中，新设置一个watcher

2、监控数据响应的一致性，Watcher是异步发送通知包的，但zookeeper提供了一个顺序保证：客户端在没有收到通知包之前，无法获取watcher事件触发后，该znode变化后的数据，保证了客户端访问zookeeper数据的一致性

3、提供对znode数据的三种观察列表：

​	节点的创建或删除：exists（）设置对当前path的znode创建和删除的watcher

​	节点数据观察：getData（）设置对znode数据的watcher

​	子节点观察：getChildren（）设置对znode子节点的列表变化（当子节点数据发送变化时，并不会触发watch；只对子节点的增删操作进行watch）**###子节点指的时下一级节点，并不是多级子节点**

- OneWatcher（一次性监听）

```java
	//获取znode数据，并添加watcher(一次性触发)
	public String getZnodeAndWatch( String nodePath) throws KeeperException, InterruptedException {
		byte[] data = zk.getData(nodePath, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				// 当前znode节点发生变化时，获取最新的值
				try {
					String trigerValue = trigerValue(nodePath);
					System.out.println(trigerValue);
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, new Stat());
		String result = new String(data);
		return result;
	}

	//监听器回调执行方法
	private String trigerValue(String nodePath) throws KeeperException, InterruptedException {
		String result = "";
		byte[] byteArray = zk.getData(nodePath, false, new Stat());
		System.out.println("监听到"+nodePath+"znode发生改变，重新获取");
		return new String(byteArray);
	}


	public static void main(String[] args) {
		WatcherZK watcherZK = new WatcherZK();
		try {
			watcherZK.setZk(watcherZK.startZookeeper());
			if (watcherZK.getZk().exists("/testZnodeWatch",false)==null) {
				watcherZK.createZnode("/testZnodeWatch", "testValue");
			}
			String znodeAndWatch = watcherZK.getZnodeAndWatch("/testZnodeWatch");
			System.out.println(znodeAndWatch);
			
			System.in.read();
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

- ContinuousWatcher持续监听：通过对监听器回调的执行方法进行递归操作，实现触发监听的同时，添加监听

```java
	private String trigerValue(String nodePath) throws KeeperException, InterruptedException {
		String result = "";
        // 当前znode节点发生变化时，获取最新的值的同时，添加一个同样的监听器，递归执行trigerValue方法
		byte[] byteArray = zk.getData(nodePath, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				try {
					trigerValue(nodePath);
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, new Stat());
		System.out.println("监听到" + nodePath + "znode发生改变，重新获取");
		return new String(byteArray);
	}
```

- 子节点持续监听:

```java
// 获取znode子节点列表，并添加watcher
	public List<String> getChildrenAndWatch(String nodePath) throws KeeperException, InterruptedException {
		List<String> data = zk.getChildren(nodePath, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				// 当前znode节点发生变化时，获取最新的值
				try {
					List<String> trigerValue = trigerChlidren(nodePath);
					trigerValue.forEach(value -> System.out.println(value));
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, new Stat());
		return data;
	}

	private List<String> trigerChlidren(String nodePath) throws KeeperException, InterruptedException {
		String result = "";
		List<String> list = zk.getChildren(nodePath, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				try {
					trigerChlidren(nodePath);
				} catch (KeeperException | InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, new Stat());
		System.out.println("监听到" + nodePath + "子节点发生改变，重新获取子节点列表");
		return list;
	}
```

- 全局事件监听：对于整个zookeeper进行监听，然后根据不同event类型，进行不同处理：

  当exists、getData和getChildren方法，使用（String path, boolean watch）参数列表时，则默认使用客户端zookeeper实例创建时的Watch进行一次性监控：

```java
	public static void main(String[] args) {
		GlobalEventWatcherZK zk = new GlobalEventWatcherZK();
		try {
			ZooKeeper zooKeeper = zk.startZookeeper();
			try {
				zooKeeper.exists("/test", true);
				zooKeeper.getData("/test", true, new Stat());
				zooKeeper.getChildren("/test", true);
			} catch (KeeperException | InterruptedException e) {
				e.printStackTrace();
			}
			System.in.read();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	public ZooKeeper startZookeeper() throws IOException {
		return new ZooKeeper(ZK_URL, SESSION_TIMEOUT, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				System.out.println(event.getType());
				if (EventType.None==event.getType()) {
					System.out.println("客户端连接成功");
				};                
                
//				&&event.getPath().equals("/zookeeper")
				// 监听节点变化(可以针对于单个节点/或者全部节点)
		
				if (EventType.NodeCreated == event.getType()) {
					System.out.println(event.getPath() + "节点被创建");
				}
				// 监听节点数据变化
//				&&event.getPath().equals("/zookeeper")
				if (EventType.NodeDataChanged == event.getType()) {
					System.out.println(event.getPath() + "节点数据发送变化");
				}
			}
		});
	}
```

**watcher的eventType：**

None   连接成功  、NodeCreated 节点创建、 NodeDataChanged  节点数据变化、 NodeChildrenChanged  节点的子节点变化 、DataWatchRemoved 数据监控删除 、childWatchRemoved 子节点监控删除

## 8、zookeeper集群：

### 1、zookeeper服务器集群配置：

集群配置信息模板：server.N=IP:A:B

N:集群服务器编号  A：当前服务器与Leader服务器进行信息交互的通信端口    B：当Leader挂掉时，进行集群选举的端口

1、在所有zookeeper服务器zoo.conf文件中添加集群配置信息：

```
server.1=127.0.0.1:2181:3991
server.2=127.0.0.2:2181:3991
server.3=127.0.0.3:2181:3991
```

并且设置各自的clientPort端口都为2181

2、选择一个服务器作为Leader

### 2、zookeeper集群特点：

#### 1、集群角色划分：

Leader（领导者）：负责所有写请求，可以处理读请求，实时监控所有其他节点运行情况，并通知其他节点自身运行情况

follower（跟随者）：只能处理读请求，写请求会自动转发给Leader进行处理，并且可以参与选举投票

observer（观察者）：不参与选举投票，其他和follower一致，主要时处理读请求（单纯提供集群的查询能力）

#### 2、集群的选举机制：

- zookeeper会由于两种请求，而进行Leader选举：

1、Leader会和所有其他集群节点进行心跳连接，当Leader宕机时，心跳包延迟发送时间达到最大值时，所有其他节点则认为主节点挂了，进入Lokking节点状态，进行Leader选举：

2、当Leader出现失去半数以上其他节点时，则会进入崩溃恢复阶段（默认当前Leader发生了网络分区问题），重新进行Leader选举：

- 选举投票包核心数据：

  1、logicalClock：当前节点服务器的第几次投票，每参与选举则加1

  2、self_id:当前节点id

  3、self_zxid:当前节点最大zxid，越大说明处理的写操作越多（同步速度越快，实际最终同步时，所有节点的最大zxid的一致的）

  4、vote_id:当前节点所投票支持的节点id

  5、vote_zxid:当前节点所投票支持的节点的最大zxid

- 选举投票过程：

  1、到follower节点发起选举后，会发送和收到其他follower的投票包（所有初始投票包中的vote_id、vote_zxid都为self_id和self_zxid，即默认投给自己）

  2、follower收到第一个外部投票包时，开始和自己的投票包比较

  3、首先比较logicalClock，当自身logicalClock小时，说明参与的选举次数少，直接落选，次数清空自己的投票箱，不再和其他节点投票包进行比较，并将自身logicalClock+1,并通过广播更新自己的投票包；反之，则继续进行其他参数比较

  4、比较vote_zxid,当自身vote_zxid小时，说明自己同步速度低，则落选，并修改自己的投票对象（vote_id，vote_zxid），并通过广播进行更新

  5、所有follower会按照以上步骤，依次对投票箱中的投票包进行处理时，当其中一个follower发现自己的投票箱票数一半是投给自己时，则会将自己的Lokking节点状态转变为Leading，而其他follower则会更新状态为following

  6、之后，成为Leader的节点会将自己commit的数据同步给其他所有follower节点

  7、其他follower节点完成同步后，则进入uptoDate状态，进行客户端消息处理

  **所有follower节点的投票箱只会保存每个其他节点的最近更新发送的投票包**

#### 3、集群消息同步策略：

1、当Leader收到写操作请求后，将其保存到本地

2、然后将这条写操作内容（propose）广播给所有follower，当一半以上的follower响应确认后，Leader就会执行写操作，修改节点中的数据，并进行持久化

3、Leader执行写操作的同时也会向所有follower广播一条Commit消息，让所有其他节点一起执行该写操作，并进行持久化

4、通过队列来维护Leader对每个集群的请求，保证propose和commit的响应（propose响应过半，才能进行commit；commit没有响应数限制）

**zookeeper原子广播协议（Zab协议）**，实现了集群的数据同步和Leader选举，保证了集群数据的最终一致性

#### 4、消息同步的有序性：

​		由于Zookeeper可能同时收到多个写操作请求，并且处理写操作请求的过程时异步非阻塞的。如果不做任何处理，这样就可能出现后一个写操作请求优先于前一个执行完整个同步。因此zookeeper需要保证数据同步的有序性：

​		Zookeeper通过要给全局有序消息Id和FIFO队列来实现：

- 全局有序Id：

  ​		Leader节点在处理每个写请求时，会自动分配一个全局有序的消息id，即zxid。zxid是一个64位的数字，其中前32位为选举轮次(epoch)，后32位为当前选举后的该节点执行事务次数（每经过一次Leader选举后，epoch则会递增，而处理事务次数就会清零），从而保证了每一个写操作唯一性

- FIFO队列：

  ​		当Leader将写操作内容广播给所有其他节点时，会每一个节点维护一个FIFO队列，来保存发送给它们的写操作顺序队列，然后依次进行ack处理，从而保证了写操作针对于每一个节点的有序性

​	虽然不能保证集群数据的实时一致性，但是可以保证节点消息同步的有序性，避免了写操作同步时，由于网络原因带来的加塞问题（后一个写操作优先于前一个写操作被处理），导致集群数据不能保存最终一致性

#### 	5、集群容错策略：session

​		当一个客户端与zookeeper服务器进行连接时，会创建一个session，并且有一个sessionId作为连接的唯一标识。当连接期间服务器出现离线时，客户端就会失去连接，并在sessionTimeout时间内一直进行重连，此时集群Leader感知到了该follower的离线，因此重新分配一个服务器创建于该客户端的连接（sessionID不变）

## 9、疑问：

### 1、watch是只针对于一个客户端，还是针对于zookeeper当前节点

通过实现证明：

1、使用一个java程序进行一次性watch监听，并且进行触发

2、然后使用 一个java程序进行持续watch监听，此时进行触发

3、观察前一个java程序是否再次触发后一次操作的监听

结果：后一个java程序设置的持续监听，并不会影响到前一个java程序，它们的监听器都是客户端 对 服务器的；两者没有联系

事实证明，对于客户端对设置的Watch，只是针对于自身对zookeeper的节点监控，而不是针对于zookeeper的节点；因此当节点改变时，之后只会通知所有客户端中，任然具有该节点watcher的终端

### 2、集群怎样通过广播原子协议来保证集群数据的最终一致性

广播原子协议进行数据同步时，会有两种状态：

- 广播状态：完成写操作的同步
- 恢复状态：完成选举后，follower和Leader之间的数据同步

广播原子协议通过额外处理机制，来解决特殊场景下的同步问题：

**1、当Leader发送写操作时，宕机**：此时Leader并没有commit的，因此并没有影响；选举后，并不会继续进行该操作，客户端会返回该操作没有被执行

**2、当Leader进行commit后，宕机：**此时Leader还没有提醒其他节点进行commit，但自己却进行了交付，并回应客户端成功执行，此时在进行Leader选举后，Leader将会重新发送该写操作（propose），然后过半响应后，进行commit。**保证所有未完成的事务commit，之后再退出恢复模式**

**3、当Leader发送请求时，Follower由于网络问题没有被接收到：**，该请求会被Leader进行维护，在请求超时后继续发送，直到认为Follower宕机；之后Follower重新并入集群时，进行强制同步处理（数据更新到Leader最大同步点）

### 3、zookeeper集群节点为什么要设置为奇数，且 >=3（不包括observer观察者）

​		1、由于zookeeper集群选举机制中，选票必须大于所有集群节点数的一半，即存活节点要大于必须保证n/2;当最多只能挂掉2台节点时，集群服务器为偶数，则需要6台服务器；当集群服务器为奇数时，则只需要5台服务器；因此节点为奇数时，可以保证服务器最大资源利用

​		2、同时当只有2台服务器时，一台挂掉后，选票只可能为1，因此无法满足选票必须大于所有follower节点数的一半；因此集群服务器至少3台

### 4、zookeeper怎样给其他分布式服务集群提供选举机制

自己搭建一个zookeepr+ActiveMQ（levleDB）的高可用集群，观察zookeeper的目录结构，看ActiveMQ是怎样使用zookeeper来进行选举的

###**zookeeper给其他分布式服务集群提供选举机制时，实际上利用了自身集群的选举机制，因此对分布式服务集群的要求和自身集群要求相同**