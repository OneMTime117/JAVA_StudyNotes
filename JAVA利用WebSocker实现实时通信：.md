# JAVA利用WebSocket实现实时通信：

实时通信：长连接的双向通信，不再依靠 请求-响应的Http单向通信模式

## **1、springboot整合webSocket,搭建推送服务**

​    1、建立war包的springboot项目（用于使用外部tomcat），导入spring-boot-starter-websocket依赖

​	2、添加供内部tomcat使用的websocket配置类	

当在websocket类中使用mapper、service注入时，不能通过@Autowired进行注解注入；原因：

spring默认管理的都是单例:如mapper类、service类，因此websocket在项目启动时，会初始化创建一个Websocket，并进行依赖注入；当客户端进行websocket连接时，由于websocket是多对象创建（每一个客户端对应一个websocket对象），因此再次创建时，spring就不会进行依赖注入；此时必须通过手动方式进行依赖注入：

springboot中，由于进行了自动配置，因此提供了获取spring容器的ApplicationContext对象方式，即使用配置类：

````java
//当使用外部Selvet容器（tomcat）时，则不需要进行配置
//springboot会自动将所有webSocket服务（被@Component、@ServerEndpoint修饰的类交给外部容器管理
@Configuration 
public class WebSocketConfig {
	@Bean
	public ServerEndpointExporter getServerEndpointExporter() {
		//管理webSocket服务的组件
		return new ServerEndpointExporter();
	}	
}
````

​	3、编写websocket服务类

````java
@Slf4j
@Component//被spring管理（@Component虽然是单例模式，但还是会创建多个对象）
@ServerEndpoint("/websocket")//服务端口名（类似于http的url）
public class WebSocketServer {

	//统计连接推送的客户端数
	private static int onlineCount=0;
	//线程安全的set，来保存所有客户端对于的webSocket服务对象
	private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();
	//客户端与对于webSocketServer的Session对象，webSocketServer通过它来向客户端发送数据
	private Session session;
	
	@OnOpen//websocket连接建立成功，执行的方法
	public void onOpen(Session session) {
		//获取webSocket连接建立的session
		this.session=session;
		//将该webSocket放入set集合
		webSocketSet.add(this);
		//webSocket统计+1
		onlineCount++;
		
		try {
			//服务器主动推送消息
			session.getBasicRemote().sendText("连接成功");
		} catch (IOException e) {
			e.printStackTrace();
			log.debug("webSocket IO异常");
		}
	}
	
	@OnClose//当websocket连接关闭时，执行的方法
	public void onClose() {
		webSocketSet.remove(this);
		onlineCount--;
		log.info("webSocket连接关闭，当前服务连接数为"+onlineCount);
	}
	
	@OnMessage//当webSocket收到客户端消息后，执行的方法（可以作为启动消息推送的入口）
	public void onMessage(String message, Session session) {
		log.info(message);
		ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
		Runnable task = new Runnable() {
			public void run() {
				for (WebSocketServer webSocketServer : webSocketSet) {
					try {                       
						webSocketServer.session.getBasicRemote().sendText("当前时间："+System.currentTimeMillis());
					} catch (IOException e) {
						e.printStackTrace();
						log.debug("webSocket IO异常");
					}
				}
			}
		};
		//每5s所有websocket服务发送一次消息
		executorService.scheduleAtFixedRate(task, 0, 5, TimeUnit.SECONDS);
	}
	
	@OnError//websocket服务发生错误时，执行方法
	public void onError(Session session, Throwable error) {
		log.error("发生错误");
		error.printStackTrace();
	}
}
````

​		4、websocket客户端（前端测试代码）

````js
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
Welcome<br/>
<input id="text" type="text" /><button onclick="send()">Send</button>    <button onclick="closeWebSocket()">Close</button>
<div id="message">
</div>

</body>
<script type="text/javascript">
    var websocket = null;

    //判断当前浏览器是否支持WebSocket
    if('WebSocket' in window){
        websocket = new WebSocket("ws://10.8.96.228:8080/test/websocket");
    }
    else{
        alert('Not support websocket')
    }

    //连接发生错误的回调方法
    websocket.onerror = function(){
        setMessageInnerHTML("error");
    };

    //连接成功建立的回调方法
    websocket.onopen = function(event){
        setMessageInnerHTML("open");
    }

    //接收到消息的回调方法
    websocket.onmessage = function(event){
        setMessageInnerHTML(event.data);
    }

    //连接关闭的回调方法
    websocket.onclose = function(){
        setMessageInnerHTML("close");
    }

    //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
    window.onbeforeunload = function(){
        websocket.close();
    }

    //将消息显示在网页上
    function setMessageInnerHTML(innerHTML){
        document.getElementById('message').innerHTML += innerHTML + '<br/>';
    }

    //关闭连接
    function closeWebSocket(){
        websocket.close();
    }

    //发送消息
    function send(){
        var message = document.getElementById('text').value;
        websocket.send(message);
    }
</script>

</html>
````

- 问题：

1、当在websocket类中使用mapper、service注入时，不能通过@Autowired进行注解注入；原因：

spring默认容器管理的都是单例模式:如mapper类、service类，因此websocket在项目启动时，只会初始化创建一个Websocket，并进行依赖注入；当客户端进行websocket连接时，由于websocket是多对象创建（每一个客户端对应一个websocket对象），因此再次创建，此时spring就不会再进行依赖注入；所有必须通过手动方式进行依赖注入：

springboot中，由于进行了自动配置，因此并不是通过application.xml 文件来创建spring容器上下文，而是使用配置类，来获取spring容器的ApplicationContext对象：

```java
@Component
public class SpringApplicationContextFactory implements ApplicationContextAware{

	private static ApplicationContext applicationContext;
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext=applicationContext;
	}
	
	public static ApplicationContext getApplicationContext(){
		return applicationContext;
	}
}
```

2、使用websocket进行连接参数传递

​	websocket通讯时，使用ws协议接口，可以直接通过url来进行传参，和http协议一样，既可以将参数放在url最后面，通过键值对获取；又可以使用url路径占位传参：

前者使用session.getPathParameters()直接获取url？后面所有的键值对

后者在方法参数列表中使用@PathParam获取url路径站位参数（springMVC提供）

## 2、java原生使用websocket

相比于springboot，java原生使用websocket时，直接使用Tomcat服务器中提供的websocket-api.jar依赖包

并不需要配置websocketConfig配置类。

## 3、基于Netty框架实现websocket的消息推送

