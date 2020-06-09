# Springboot的任务模块

springboot任务模块支持三种常用任务模块：

异步任务、定时任务、邮件任务

### 1、异步任务

- **什么是异步任务：**

**异步任务就是将一个特殊的方法多线程执行，达到方法与其调用同级代码异步执行的效果**

该方法一般是执行时间较慢，并且对于web请求结果没有直接影响的操作：如处理多个订单请求时，可以在录入订单信息时使用异步任务，而直接在controller直接响应用户“订单成功”，异步慢慢执行订单信息的录入，这样大大增加了用户的交互体验。

- **使用springboot异步任务：**

1、在配置类（如启动类）中添加 @EnableAsync注解 ：开启异步任务注解

2、创建一个类，编写需要异步执行的方法；在该类上添加@Component注解 ：类扫描到spring容器中，作为其组件进行依赖注入； 在对应的方法上添加 @Async注解：该方法为异步任务执行（也可以直接在该类上添加 @Async注解 ：该类的所有方法都为异步任务执行）

3、然后在调用类中注入该异步任务类，执行异步方法

**注意：异步方法不能直接写在调用该方法的类中，必须所在类被springboot依赖注入后，@Async才会生效（也因此@Async注解静态方法无效，因为静态方法不会随着类的实例化被springboog管理）**

- **异步任务的本质是多线程：**

异步任务默认使用一个springboot的core.task包下提供配置的一个线程池对象，继承了JDK中的Executor类

1、默认异步任务线程池

springboot默认自动配置一个TaskExecutorBuilder对象，用于创建ThreadPoolTaskExecutor异步任务线程池；默认配置了线程池的属性：

最小线程数：8

最大线程数：Interger.MAX_VALUE 即int类型最大数 21亿

线程池任务队列容量： 21亿

线程池中的线程name的前缀： task-

2、自定义异步任务线程池

````java
//@Configuration
public class AsyncThreadPoolConfig {

	//ThreadPoolTaskExecutor 实现了AsyncListenableTaskExecutor、SchedulingTaskExecutor接口，它们的超类都是JDK的concurrent包中Executor
	@Bean
	public Executor asyncTaskExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();//springboot自带的线程池对象,推荐使用
//		Executor executor1 = Executors.newSingleThreadExecutor();//也可以使用JDK的concurrent包中实现的线程对象,不需要进行其他配置
		executor.setCorePoolSize(10);//最小线程数
		executor.setMaxPoolSize(60);//最大线程数
		executor.setQueueCapacity(10);//线程池任务队列容量
		executor.setThreadNamePrefix("asyncTask");//该线程池中的线程name的前缀
		executor.initialize();//初始化自定义线程池
		return executor;
	}
}

，然后在异步方法上添加@Async 注解（组件名：即@Bean中的值或方法名）
````



当自定义异步任务线程池后，会取代默认线程池；需要多个线程池时，应添加@Primary ：指定主键（默认）组件

- **异步任务线程池的异常捕获**

由于多线程在执行时，并不能将异常进行抛出，因此其调用者无法捕获其异步任务中发生的异常（但可以在内部中捕获处理）

1、对于没有返回值的异步方法，可以通过AsyncUncaughtExceptionHandler异常处理器来统一处理任务内部未捕捉的异常

````java
@Configuration
public class AsyncThreadPoolConfig implements AsyncConfigurer{
	
	@Override
	public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
		return new MyAsyncExceptionHandler();
	}
	
    //通过定义AsyncUncaughtExceptionHandler来统一处理异步任务线程中的异常
	class MyAsyncExceptionHandler implements AsyncUncaughtExceptionHandler{
		@Override
		public void handleUncaughtException(Throwable ex, Method method, Object... params) {
			System.out.println(method.getName()+"    "+ex.getMessage()+"    "+params.toString());
		}
	}
}
````



2、对于有返回值的异步方法，和JDK中的线程池对象一样，返回一个Future<T>类型接口，需要通过创建一个AsyncResult<T>来进行实现返回需要的值。

此时就可以在异步方法内部捕捉异常，然后通过AsyncResult<T>将需要的异常结果返回给调用者，此时调用者通过future.get()方法进行调用（无参数的方法会发生阻塞，这样就违背了异步任务执行的目的）

### 2、定时任务

**定时任务可以可以看作是一个异步任务，但是它能设置任务的执行时间，并能间隔执行；一般用于在服务器压力较小的时间端进行一些大数据的更新，或定时执行某些任务**

- **使用springboot定时任务**

1、在配置类（如启动类）中添加 EnableScheduling注解 ：开启定时任务注解

2、创建一个类，编写需要定时执行的方法；在该类上添加@Component注解 ：类扫描到spring容器中，作为其组件进行依赖注入； 在对应的方法上添加 @Scheduled注解：该方法为定时任务执行（也可以直接在该类上添加 @Scheduled注解 ：该类的所有方法都为异步任务执行）

3、在应用运行后，自动执行

**通过添加@Scheduled中的属性，来定义定时任务的执行方式：**

1、fixedRate  上次任务执行多少毫秒后再次执行，但是在任务单线程模式下，后一个任务会在前一个任务每完成时发生阻塞
2、fixedDelay  上次任务执行完后，延迟多少毫秒再次执行，因此不存在单线程模式下阻塞
3、initialDelay  配合fixedRate、fixedDelay使用，来确定项目启动后任务第一执行的延迟时间（若不指定，则项目启动后就执行第一次任务）
以上者三种都支持String类型，用于进行配置文件的占位符注入fixedDelayString = "${time.fixedDelay}"；便于在配置文件中修改定时时间
4、cron     用于专门确定某个时间点执行任务

**cron使用一种固定的匹配规则来简化时间定的配置，cron表达式规则：**

不同框架和环境规则会有所不同，以java环境下使用springboot定时任务为例：

一共有六个参数，以空格分隔；参数含义从左往右依次为：

- second  0-59   秒

- min    0-59  分

- hour  0-23  时

- day of month   1-31   一个月中的第几天

- month  1-12 或月份英文简写    月份

- day of week   0-7或星期英文简写（0从周末开始）  一个周的第几天

特殊字符：

- ‘，’   代表该参数有多选  1，3，4   即 1、3、4
- ’—‘  代表参数范围多选  0-10   即0到10
- ‘*’   代表全选该参数的所有值（和其他参数是and关系）

5、zone  设置时间调度器的时区

- **定时任务线程池配置**

定时任务默认线程池：

springboot默认自动配置一个TasScheduleBuilder对象，用于创建ThreadPoolTaskScheduler定时任务线程池（和JDK concurrent包中的concurrent包ScheduledExecutorService类似，带有时间调度功能、固定线程数的线程池对象。

默认配置：固定线程数为1，即单线程；线程前缀为 scheduling-

###由于定时任务线程池默认只有一个线程，因此每个定时任务都有自己唯一固定的线程池；也无法进行自定义的线程池配置

- **定时任务下，支持异步任务**

由于在fixedRate模式下，定时任务会发生阻塞，因此需要通过多线程执行来阻止阻塞的发生：

开启异步任务注解，然后再定时任务方法上添加@Async注解：当定时任务发生阻塞时，会创建一个异步任务线程池，启动一个线程执行该方法，但方法的执行时间还是通过定时任务线程池中时间调度器控制

- **注意**

1、springboot在1.5后，定期任务类中的依赖注入可以在任务启动前提前注入。当然也可以通过ApplicationContext来手动注入bean


2、springboot 不使用任务模块，自定义线程任务时，其类的成员变量无法进行依赖注入。为了保证线程安全

但是可以通过内部类来定义runnable，从而将外部类成员依赖注入成功的对象，在内部类线程任务中使用

### 3、邮件任务

- **邮件发送、接收依赖于两个应用层协议**

1、SMTP（简单邮件传输协议），用于发送邮件

2、POP3（邮局协议版本3），用于接收邮件

**IMTP（应用层协议，交互邮件访问协议）：也是用于接收邮件，但是弥补了POP3的不足：pop3只能下载邮件，在本地进行操作；而IMTP可以于邮件客户端进行双向通信，将客户端的操作反馈到服务器上，并且可以在服务器上进行管理筛选需要的邮件，然后进行下载**

- **邮件发送、接收过程**：

连接登录SMTP服务器----------编写邮件，设置发送邮件的目的地----------SMTP服务器进行邮件发送-------------POP服务器进行邮件接收，保存管理邮件----------客户端登录POP服务器，进行邮件查看管理

- springboot邮件任务实现

1、添加springboot邮件任务依赖：spring-boot-starter-mail

2、配置邮件发送必须的配置

````java
#配置邮件任务：该项目的邮件客户端信息（使用qq邮箱作为发件服务器）
spring.mail.username=756116832@qq.com
#并不是直接使用密码，而是使用第三方登录授权码
spring.mail.password=*********************
#配置qq邮箱SMTP服务器（发送邮件服务器）地址javascript
spring.mail.host=smtp.qq.com

#springboot默认使用TLS安全协议，对于SMTP服务器端口为25
#邮件发送额外配置,设置邮件发送时，使用ssl安全协议，对于SMTP服务器端口为465，因此不需要对端口进行指定（防止安全协议和端口不对应）
#spring.mail.properties.mail.smtp.ssl.enable=true
````

3、编写邮件发送类

````java
@Test
	void contextLoads() {
		// 简单消息邮件
		SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
		simpleMailMessage.setSubject("TEST");// 邮件主题
		simpleMailMessage.setText("今天星期几");// 邮件内容
		simpleMailMessage.setTo("yuanhuanpeipei@gmail.com");// 邮件发送目的地
		simpleMailMessage.setFrom("756116832@qq.com");//设置发送人的邮件地址（必须和配置文件中的）

		//发送简单消息邮件
		mailSender.send(simpleMailMessage);
	}

	@Test
	void contextLoads01() throws MessagingException {
		// 复杂消息邮件(多功能邮件)
		MimeMessage createMimeMessage = mailSender.createMimeMessage();
		MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(createMimeMessage, true);//打开邮件附件
		
		//通过MimeMessageHelper对象来给createMimeMessage添加邮件消息体
		mimeMessageHelper.setSubject("TEST");// 邮件主题
		mimeMessageHelper.setText("今天星期几");// 邮件内容
//		mimeMessageHelper.setText("<html><html>",true);//发送html
		mimeMessageHelper.setTo("yuanhuanpeipei@gmail.com");// 邮件发送目的地
		mimeMessageHelper.setFrom("756116832@qq.com");//设置发送人的邮件地址（必须和配置文件中的）
		mimeMessageHelper.addAttachment("搞笑图片.jpg", new File("C:\\Users\\OneMTime\\Desktop\\我的文档\\微信图片_20190725163441.jpg"));

		//发送简单消息邮件
		mailSender.send(createMimeMessage);
	}
````

4、由于每个邮箱账号每天发送邮件数有限（减少邮箱厂商应用服务器的压力，也可以出钱提高上限），因此常设置多个账号来进行邮件轮询发送，方法如下：

- 重写配置邮件发送类（JavaMailSenderImpl）的部分实现方法，让其创建的发送者对象配置的账号轮询使用

````java
@Configuration
@EnableConfigurationProperties(MailProperties.class) // 将MailProperties组件放入该配置类中
public class MailConfig extends JavaMailSenderImpl implements JavaMailSender {
	// 保存多个用户名和密码的队列（spring.mail.username=、
	// spring.mail.password=中定义多个客户端账号、密码，使用","分隔）
	private ArrayList<String> usernameList;
	private ArrayList<String> passwordList;
	// 轮询标识
	private int currentMailId = 0;

	private final MailProperties properties;

	public MailConfig(MailProperties properties) {
		this.properties = properties;

		// 初始化账号
		if (usernameList == null)
			usernameList = new ArrayList<String>();
		String[] userNames = this.properties.getUsername().split(",");
		if (userNames != null) {
			for (String user : userNames) {
				usernameList.add(user);
			}
		}

		// 初始化密码
		if (passwordList == null)
			passwordList = new ArrayList<String>();
		String[] passwords = this.properties.getPassword().split(",");
		if (passwords != null) {
			for (String pw : passwords) {
				passwordList.add(pw);
			}
		}
	}

	@Override
	protected void doSend(MimeMessage[] mimeMessages, Object[] originalMessages) throws MailException {
		super.setUsername(usernameList.get(currentMailId));
		super.setPassword(passwordList.get(currentMailId));

		// 设置编码和各种参数
		super.setHost(this.properties.getHost());
		super.setDefaultEncoding(this.properties.getDefaultEncoding().name());
		super.setJavaMailProperties(asProperties(this.properties.getProperties()));
		super.doSend(mimeMessages, originalMessages);

		// 轮询（多个客户端账号，轮询发送邮件）
		currentMailId = (currentMailId + 1) % usernameList.size();
	}

	private Properties asProperties(Map<String, String> source) {
		Properties properties = new Properties();
		properties.putAll(source);
		return properties;
	}

	@Override
	public String getUsername() {
		return usernameList.get(currentMailId);
	}
}
````

注意，此时配置文件中的账号密码为多个，使用“，”分隔

- 定义一个邮箱发送的方法，可以自定义收件人邮箱、邮件内容、邮件附件等

````java
@Component
public class SendMailUtils {
	@Autowired
	MailConfig mailConig;

	// 邮件发送（收件人邮箱、邮件内容）
	public String sendMailString(String email, String text) throws MessagingException, UnsupportedEncodingException {
		MimeMessage message = mailConig.createMimeMessage();
		MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
		InternetAddress from = new InternetAddress();
		from.setAddress(mailConig.getUsername());
		from.setPersonal("袁欢", "UTF-8");
		helper.setFrom(from);// 设置发送人的邮件地址、发送人姓名
		helper.setTo(email);// 邮件发送目的地
		helper.setSubject("测试邮件");// 邮件主题
		helper.setText(text, true);// 邮件内容（true为html形式）
		// 发送附件
//		helper.addAttachment("搞笑图片.jpg", new File("C:\\Users\\OneMTime\\Desktop\\我的文档\\微信图片_20190725163441.jpg"));
		mailConig.send(message);
		return text;
	}
}
````

- 最后，在指定controller层，进行邮件发送的方法调用