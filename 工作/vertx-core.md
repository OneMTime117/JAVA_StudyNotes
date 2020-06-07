# vertx-core

##### Vert.x:

每一个实例都有自己的线程池（及Vertx框架默认是多线程的），从而该Vertx对象会有多个Event loop线程池和worker线程池来执行该Vertx部署的所有Verticle模块
所有的Verticle是通过事件驱动执行的（异步执行），这样就很好的解决了单线程阻塞问题；这样使用Vertx开发，就直接保证了代码的线程安全，并且默认使用多线程开发

就每一个Event Loop线程可以执行所有Verticle模块，不会阻塞

worker线程用来执行需要阻塞的代码（如转账），防止用Event loop线程执行时，发生阻塞

## 1、创建Vertx对象

**Vertx vertx = Vertx.vertx();**

也可以修改vertx对象的默认值（如池大小、高可用性）：

Vertx vertx = Vertx.vertx(new **VertxOptions().**setWorkerPoolSize(40));

## 2、vertx进行事件驱动

### *****服务器请求事件：

**server.requestHandler(request -> {**
**request.response().end("hello world!");
});**

### *****计时器事件：

**vertx.setPeriodic(1000, id -> {**
**System.out.println("timer fired!");**
**});**

**取消计时器**

**DeploymentOptions options = new DeploymentOptions().setConfig(config);**

## 3、vertx不会阻塞线程

**Vert.x API不会阻止线程，这意味着可以使用Vert.x通过少量线程来处理大量并发**

**一个Vertx实例默认维护N个event loop（事件循环）线程**（N为cpu核心数*2）

## 4、event bus（事件总线）

**event bus 是Vert.x的神经；负责Vert.x各Verticle模块之间的消息的传送**

**每一个Vertx实例，都有一个event bus实例**

**EventBus提供发布订阅功能和点对点的消息服务**

**EventBus是使用TCP协议进行通信，因此，在任何的应用中，只要能够创建TCP连接，都可以通过EventBus连接到Vert.x实例**

##### 1、通过Vert.x获得EventBus实例

**EventBus eb=vertx.eventBus（）**

​		

##### 2、发布消息  

#####          所有人都可以通过address获得该消息

​         **eb.publish(address, message)**

#####          通过点对点的方式发布消息（其他人接收不到）

​	 **eb.publish(address, message, options)**

##### 3、订阅消息(获得消息，并进行处理)

​	  eb.consumer(address, handler)

## 5、Verticle模型

Verticle是由Vert.x部署运行的代码块

每个Verticle模块都可以通过EventBus来进行相互调用

#### 1、verticle模块编写

```
public class MyVerticle extends AbstractVerticle {

 public void start() {
 }

 public void stop() {
 }

}
```

#### 2、返回异步结果（即有返回值的）Verticle的start和stop

```
public class MyVerticle extends AbstractVerticle {

 private HttpServer server;

 public void start(Future<Void> startFuture) {
   server = vertx.createHttpServer().requestHandler(req -> {
     req.response()
       .putHeader("content-type", "text/plain")
       .end("Hello from Vert.x!");
     });
     
   // Now bind the server:
   server.listen(8080, res -> {
     if (res.succeeded()) {
       startFuture.complete();
     } else {
       startFuture.fail(res.cause());
     }
   });
 }
}
```

```
public class MyVerticle extends AbstractVerticle { 

 public void start（）{ 
   // Do something 
 } 

 public void stop（Future <Void> stopFuture）{ 
   obj.doSomethingThatTakesTime（res  - > { 
     if（res.succeeded（））{ 
       stopFuture.complete（） ; 
     } else { 
       stopFuture.fail（）; 
     } 
   }）; 
 } 
}
```

不需要在Verticle的stop方法中手动停止有Verticle启动的http服务器，当Verticle取消部署时，Vertx自动停止其模块中运行的服务器



#### 3、Verticle三种类型

**Standard Verticle（标准）**

它们总是使用事件循环线程池线程执行

**Worker Verticle（工人）**

它们使用Worker线程池中的线程运行。实例永远不会由多个线程并发执行

**Multi-threaded worker verticle（非线程工人）**

它们使用Worker线程池中的线程运行。实例可以由多个线程并发执行

##### **1、标准Verticle**

标准Verticle在创建时被赋予事件循环线程，并且使用该事件循环调用start方法。当您从事件循环中调用任何其他从核心API获取处理程序的方法时，Vert.x将保证这些处理程序在被调用时将在同一个事件循环上执行。

这意味着我们可以保证Verticle实例中的所有代码总是在同一个事件循环上执行（只要你不创建自己的线程并调用它！）。

这意味着您可以将应用程序中的所有代码编写为单线程，让Vert.x担心线程和扩展。不再担心同步和易失性，并且在进行手动“传统”多线程应用程序开发时，您还可以避免许多其他竞争条件和死锁情况。

##### **2、工人Verticle**

**工作者Verticle就像标准Verticle，但它是使用Vert.x工作线程池中的线程执行的，而不是使用事件循环。**

Worker Verticle设计用于调用阻塞代码，防止使用事件循环线程时发生阻塞

##### 3、多线程工人Verticle

多线程工作者Verticle就像普通的worker Verticle一样，但它**可以**由不同的线程同时执行

多线程工作者Verticle是一项高级功能，大多数应用程序都不需要它们

#### 4、部署Verticle

**vertx.deployVerticle( new myVerticle());**

**也可以修改默认值，进行部署**

如：修改部署数量。。

DeploymentOptions options = new DeploymentOptions().setInstances(16);

vertx.deployVerticle(verticle, options);

修改默认值可以使用josn配置文件





DeploymentOptions options = new DeploymentOptions().setConfig(config);

config为JSON对象

## 6、支持JSON

通过JSONObject、JSONArray两个类来处理json数据

#### 1、创建json对象

可以通过String、map创建json对象

通过put()添加json数据

#### 2、处理json对象

通过getxxx(key)方法获得Json对象中的value

通过mapTo（jopo.class）方法将json对象映射成java实体对象

tostring转换为string类型变量（用于数据传递）

#### 3、json数组

通过add方法来添加数据

getxxx（index）获得json数组值



## 7、Buffer对象

#### 1、创建buffer对象

Buffer buff = Buffer.buffer();

还可以设置编码格式、内容（只能通过byte【】、string）、初始值

#### 2、buffer追加数据

`appendXXX`方法 buffer数据后面写入

setXXX方法  buffer通过索引（index）插入到buffer数据

#### 3、buffer读取数据

getXXX方法，可以将buffer中的数据转换为几种基础类型数据

## 8、编写非阻塞式TCP服务器和客户端

### 1、创建TCP服务器

#### 1、创建服务器

//	    创建tcp服务器 
		NetServer netServer = vertx.createNetServer();
//	     设置TCP服务器连接
	     netServer.connectHandler(scoket ->{
//	    	 设置scoket连接对请求的处理
	    	scoket.handler(buffer ->{
//	    		获得scoket buffer中的数据（请求数据）
	    		System.out.println(buffer.toString());
	    		
//	    		在scoket buffer中写入数据（用于响应到客户端）
	    		scoket.write(Buffer.buffer("helloworld!!!!!"));
	    	});
	     }); 
	     	     	     	     
	     
//	           当监听器启动时，服务器上线
	     netServer.listen(8887, Result ->{
				System.out.println("服务器上线");
			

**tcp服务器通过NetSocket对象调用处理程序；*

**NetSocket是vertx提供的封装好的Socket接口，用于读、写Socket bufer中的数据**

**java中是通过TCP连接来完成网络通信，Socket接口就是使用的TCP连接**

#### 2、Socket连接关闭时，处理请求方法

socket.closeHandler(v -> {
System.out.println("The socket has been closed");
});

#### 3、Socket连接发生异常时，处理请求方法

socket.eceptionHandler(v->{

System.out.println("The socket have error");

})

#### 4、通过Socket发送类路径下的资源

socket.sendFile("myfile.dat");

#### 5、关闭TCP服务器

**关闭服务器将关闭所有打开的连接并释放所有服务器资源**

server.close(res -> {
if (res.succeeded()) {
System.out.println("Server is now closed");
} else {
System.out.println("close failed");
}
});

**也可以直接取消部署对应的Verticle，来关闭其所以服务器、客户端**

#### 6、部署多个服务器

**可以通过Verticle中循环代码实现**

**也可以通过部署多个Verticle来实现**



### 2、创建TCP客户端

#### 1、创建客户端

//       创建tcp客户端
		NetClient netClient = vertx.createNetClient();
//		设置客户端连接
		netClient.connect(8887, "localhost", result ->{
//			if连接成功
			if (result.succeeded()) {
//			获得被封装的socket
				NetSocket socket = result.result();
//			将请求数据放入socket buffer中
				socket.write(Buffer.buffer("nihao"));
				
//			设置socket响应数据的处理	
				socket.handler(buffer ->{
					System.out.println(buffer.toString());
				});
			}else {
//				如果连接失败
				System.out.println("服务器连接异常");
			}

### 2、客户端使用代理进行服务器连接（实现sockes5代理）

NetClientOptions options = new NetClientOptions()
.setProxyOptions(new ProxyOptions().setType(ProxyType.SOCKS5)
.setHost("localhost").setPort(1080)
.setUsername("username").setPassword("secret"));
NetClient client = vertx.createNetClient(options);

## 9、编写非阻塞式HTTP服务器和客户端

#### 1、创建Http服务器

HttpServer server = vertx.createHttpServer();

#### 2、配置Http服务器

HttpServerOptions options = new HttpServerOptions().setMaxWebsocketFrameSize(1000000);

HttpServer server = vertx.createHttpServer(options);

#### 3、监听服务器

HttpServer server = vertx.createHttpServer();
server.listen(8080);

#### 4、服务器请求处理

server.requestHandler(request -> {
request.response().end("Hello world");
}).listen(8080)

#### 5、处理请求中的数据

request.handler(buffer -> {
System.out.println("I have received a chunk of the body of length " + buffer.length());
});

**已经处理完请求中的所以数据后，需要执行的处理**

request.endHandler(v -> {
System.out.println("Full body received, length = " + totalBuffer.length());
});

#### 6、处理表单提交数据

**处理表单参数：**

server.requestHandler(request -> {

需要先设置其ExpectMultipart为true，才能对form数据进行处理

request.setExpectMultipart(true);

一般在请求体被全部读取后，再检索表单参数，因此在endHandler处理中进行

request.endHandler(v -> {
*// The body has now been fully read, so retrieve the form attributes*
MultiMap formAttributes = request.formAttributes();
});
});

#### **7、处理表单上传文件**

server.requestHandler(request -> {

需要先设置其ExpectMultipart为true，才能对form数据进行处理request.setExpectMultipart(true);
request.uploadHandler(upload -> {

通过upload对象处理上次文件

System.out.println("Got a file upload " + upload.name());
});
});

大部分文件上次不是在单个buffer中整个上传，而是以块的方式

request.uploadHandler(upload -> {
upload.handler(chunk -> {
System.out.println("Received a chunk of the upload of length " + chunk.length());
});
});

#### 8、回复消息，编写HTTP响应

request.response()
                .putHeader("Context-type", "text/html;chareset=utf-8")  设置响应头  

​                .setChunked(true)   使用HTTP分块模式（无需预先设置`Content-Length`请求头）
​                .write(buffer)    写入数据（通过buffer对象）
​                .end("hello");    结束响应处理

8、WebSocket

[WebSockets](http://en.wikipedia.org/wiki/WebSocket)是一种Web技术，允许HTTP服务器和HTTP客户端（通常是浏览器）之间进行socket连接，从而进行双向通信（改变了之前的HTTP短连接模式）

Vertx就支持此技术，区别于Socket,WebSockets针对于HTTP连接；而Socket针对于TCP连接



## 10、Vertx中使用共享数据

#### 1、本地共享map

`Local shared maps` 允许您在同一Vert.x实例中的不同事件循环（例如，不同的Verticle）之间安全地共享数据。

并且map中的数据类型是不可变的

##### **1、在某个Verticle中创建LocalMap**

SharedData sd = vertx.sharedData();
LocalMap<String, String> map1 = sd.getLocalMap("mymap1");
map1.put("foo", "bar"); 
LocalMap<String, Buffer> map2 = sd.getLocalMap("mymap2");
map2.put("eek", Buffer.buffer().appendInt(123));

**2、在另一个Verticle中通过EKY获得localMap**

map1 = sd.getLocalMap("mymap1");

String val = map1.get("foo");

map2 = sd.getLocalMap("mymap2");

Buffer buff = map2.get("eek");

##### 3、异步获得loaclMap

SharedData sd = vertx.sharedData();

sd.<String, String>getAsyncMap("mymap", res -> {

if (res.succeeded()) {
AsyncMap<String, String> map = res.result();
} else {
*// Something went wrong!*
}
});}
});

##### 4、异步添加数据

map.put("foo", "bar", resPut -> {
if (resPut.succeeded()) {
*// Successfully put the value*
} else {
*// Something went wrong!*
}
});

##### 5、异步获得数据

map.get("foo", resGet -> {
if (resGet.succeeded()) {
*// Successfully got the value*
Object val = resGet.result();
} else {
*// Something went wrong!*
}
});

#### 2、异步获得lock（同步锁）、Counter（计数器）

**获得lock**

sd.getLock("mylock", res -> {
if (res.succeeded()) {
*// Got the lock!*
Lock lock = res.result();

*// 5 seconds later we release the lock so someone else can get it*

vertx.setTimer(5000, tid -> lock.release());

} else {
*// Something went wrong*
}
});

**获得Counter**

sd.getCounter("mycounter", res -> {
if (res.succeeded()) {
Counter counter = res.result();
} else {
*// Something went wrong!*
}
});