# 1、后台servlet处理JSON数据

#### 获得前台ajxj传来的json数据

JSONObject jsonObject=JSONObject.fromObject(request.getParameter("json"))

#### 调用方法处理该数据，获得处理后的数据

String    name =dao.getname();

#### 将数据转换为josn类型   （除了string对象、可以是map对象来转换为json数组）

new JSONObject.fromObject(name)

new JSONArray().fromObject(map)

#### 通过response.getWriter,创建一个字符输出流，直接将数据输入到网站

PrintWriter out=new response.getWriter();

#### print方法可以输出对象，通过输出流将数据传入前台

#### （也可以使用Writer方法，但是要将JSON数据转化为String类型）

out.print(new JSONObject.fromObject(name))

#### 刷新输出流，关闭输出流

out.flush();
out.close();

#### *****PrintWriter为缓冲流，在执行close前，应该执行flush（），让缓冲区的数据也写入目标，之后再关闭

## JSON数据

#### 数据分类：

可分为json对象（JsonObject）、json数组（ArraryJson） 

语法格式为：

#### json对象（可以内含JSON数组、JSON对象）

myObj =

 {
"name":"runoob",
"alexa":10000,
"sites": {
"site1":"www.runoob.com",
"site2":"m.runoob.com",
"site3":"c.runoob.com"
              }
 }

#### JSON数组

#### [ "Google", "Runoob", "Taobao" ]

#### *****后台可以通过Map来转换为JSON对象;或者直接创建JSON对象，同通过put方式添加数据

**json对象常用方法：**





# 2、数据格式处理：

#### POST中处理请求消息体数据中的编码格式（处理中文乱码）;

 request.setCharacterEncoding("UTF-8");

#### 处理响应消息体数据类型、编码格式（一般与页面本身静态内容编码一致）；

	response.setContentType("text/html;charset=UTF-8");
#### *****该设置要在数据写入之前使用

#### *****数据类型是为了让浏览器知道如何显示该消息体内容



# 3、DAO层执行sql语句

## 事物管理：

在创建Cconnection对象后，关闭自动提交事务（手动进行事务管理）

#### conn.setAutoCommit(false);

sql语句执行成功：	conn.commit（）; 提交事务

sql语句执行失败：        conn.rollback();回滚事务

## JDBC:

1、创建JDBC连接工具类，提供Connection连接对象

2、在JDBC工具类中，通过调用一个数据处理工具，拼接出JDBC连接需要的url、username、password

# 4、并发模块

分为Vertx多线程（使用异步执行代码，默认线程安全）、和多线程（需要自己通过同步来保证线程安全）

Vertx多线程：

1、通过Vert.x实现，通过设置监听器，来创建该模块线程，部署多个Verticle

2、在一个监听器中创建一个Vertx实例，Vertx内部使用线程池（默认池大小为cpu核心数两倍）

3、Vertx的线程池会异步处理该Vertx实例部署的所有Verticle模块（类似线程体）



普通多线程：

1.创建 Callable 接口的实现类，并实现 call() 方法，该 call() 方法将作为线程执行体，并且有返回值。

2.根据实际需要编写线程体逻辑代码（一般进行循环，当发生什么事件时，线程开始执行一次代码）

3.在另一个类（如主程序、监听器）中建立线程池，然后将callable接口的实例类（线程）放入线程池同步处理

```
ExecutorService executor=Executors.newSingleThreadExecutor();
CycleCarTask task=new CycleCarTask();
executor.submit(task);
```

# 5、使用Properties文件

创建一个properties对象

Properties properties = new Properties();

获取项目的决定路径

 string url=Thread.currentThread().getContextClassLoader().getResource("").toString();

创建一个输入流，通过properties文件的决定路径，并设置内容编码格式

InputStreamReader insReader = new InputStreamReader(new FileInputStream(url + propertyname + ".properties"),"UTF-8");

通过输入流的读取获得properties对象

properties.load(insReader);

该对象可以获取数据：

properties.getProperty(Key)    通过key获得值

properties.toString()    直接获得整个文本数据

# 6、java实现Excel导入导出

## 使用POI插件进行处理

//所需jar包:commons-io-2.2.jar;poi-3.11-20141221.jar
//通过poi进行excel导入数据
public class PoiExcel {
public static void main(String[] args) throws SQLException {
	String title[]= {"名字","课程","分数"};
	//1.创建Excel工作簿
	HSSFWorkbook workbook=new HSSFWorkbook();
	//2.创建一个工作表
	HSSFSheet sheet=workbook.createSheet("sheet2");
	//3.创建第一行
	HSSFRow row=sheet.createRow(0);
	HSSFCell cell=null;
	//4.插入第一行数据
	for (int i = 0; i < title.length; i++) {
		cell=row.createCell(i);
		cell.setCellValue(title[i]);
	}
	//5.追加数据
	Data data=new Data();
	ResultSet rs=data.getString();
	while(rs.next()) {
		HSSFRow row2=sheet.createRow(rs.getRow());
		HSSFCell cell2=row2.createCell(0);
		cell2.setCellValue(rs.getString(1));
		cell2=row2.createCell(1);
		cell2.setCellValue(rs.getString(2));
		cell2=row2.createCell(2);
		cell2.setCellValue(rs.getString(3));
	}
	//创建一个文件,将Excel内容存盘
	File file=new File("e:/sheet2.xls");
	try {
		file.createNewFile();
		FileOutputStream stream=FileUtils.openOutputStream(file);
		workbook.write(stream);
		stream.close();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
	
}
}

//将Excel表中内容读取
public class PoiRead {
public static void main(String[] args) {
	//需要解析的Excel文件
	File file=new  File("e:/sheet2.xls");
	try {
		//获取工作簿
		FileInputStream fs=FileUtils.openInputStream(file);
		HSSFWorkbook workbook=new HSSFWorkbook(fs);
	    //获取第一个工作表
		HSSFSheet hs=workbook.getSheetAt(0);
		//获取Sheet的第一个行号和最后一个行号
	   int last=hs.getLastRowNum();
	   int first=hs.getFirstRowNum();
	   //遍历获取单元格里的信息
	   for (int i = first; i <last; i++) {
		HSSFRow row=hs.getRow(i);
		int firstCellNum=row.getFirstCellNum();//获取所在行的第一个行号
		int lastCellNum=row.getLastCellNum();//获取所在行的最后一个行号
		for (int j = firstCellNum; j <lastCellNum; j++) {
			HSSFCell cell=row.getCell(j);
			String value=cell.getStringCellValue();
			System.out.print(value+" ");
		}
		System.out.println();
	}
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

# 接口调用：

后台调用外部网页接口，获得数据：

	/** 日志 */
	private static Logger log = LogManager.getLogger(HttpRequestUtil.class);
	 
	/**
	 * 向指定URL发送GET方法的请求
	 * 请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
	 * @param url
	 *            发送请求的URL
	 * @return URL 所代表远程资源的响应结果
	 */
	public static String sendGet(String url) {
	    该类被设计用作 StringBuffer 的一个简易替换，StringBuilder不是线程安全的，而StringBuffer是线程安全的。但StringBuilder在单线程中的性能比StringBuffer高。
		StringBuilder result = new StringBuilder();
		BufferedReader in = null;
		try {
			URL realUrl = new URL(url);
			// 打开和URL之间的连接
			URLConnection connection = realUrl.openConnection();
			// 设置通用的请求属性
			connection.setRequestProperty("accept", "*/*");
			connection.setRequestProperty("connection", "Keep-Alive");
			connection.setRequestProperty("user-agent",
					"Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
			connection.setRequestProperty("Accept-Charset", "UTF-8");
			connection.setRequestProperty("contentType", "UTF-8");
			// 建立实际的连接
			connection.connect();
			// 遍历所有的响应头字段
			// for (String key : map.keySet()) {
			// System.out.println(key + "--->" + map.get(key));
			// }
			// 定义 BufferedReader输入流来读取URL的响应
			in = new BufferedReader(new InputStreamReader(
					connection.getInputStream(), "utf-8"));
			String line;
			while ((line = in.readLine()) != null) {
				result.append(line);
			}
		} catch (Exception e) {
​			System.out.println("发送GET请求出现异常！" + e);
​			result = new StringBuilder("{\"resCode\":\"1\",\"errCode\":\"1001\",\"resData\":\"\"}");
​			e.printStackTrace();
​			log.error("远程服务未开启", e);
​		}
​		// 使用finally块来关闭输入流
​		finally {
​			try {
​				if (in != null) {
​					in.close();
​				}
​			} catch (Exception e2) {
​				e2.printStackTrace();
​			}
​		}

		return result.toString();
	}
	 
	/**
	 * 向指定 URL 发送POST方法的请求
	 * 
	 * @param url
	 *            发送请求的 URL
	 * @param param
	 *            请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
	 * @return 所代表远程资源的响应结果
	 */
	public static String sendPost(String url, String param) {
	 
		PrintWriter out = null;
		BufferedReader in = null;
		StringBuilder result = new StringBuilder();
		try {
			URL realUrl = new URL(url);
			// 打开和URL之间的连接
			URLConnection conn = realUrl.openConnection();
			// 设置通用的请求属性
			conn.setRequestProperty("accept", "*/*");
			conn.setRequestProperty("connection", "Keep-Alive");
			conn.setRequestProperty("user-agent",
					"Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
			conn.setRequestProperty("Accept-Charset", "UTF-8");
			conn.setRequestProperty("contentType", "UTF-8");
			// 发送POST请求必须设置如下两行
			conn.setDoOutput(true);
			conn.setDoInput(true);
			// 获取URLConnection对象对应的输出流
			out = new PrintWriter(conn.getOutputStream());
			// 发送请求参数
			out.print(param);
			// flush输出流的缓冲
			out.flush();
			// 定义BufferedReader输入流来读取URL的响应
			in = new BufferedReader(
					new InputStreamReader(conn.getInputStream()));
			String line;
			while ((line = in.readLine()) != null) {
				result.append(line);
			}
		} catch (Exception e) {
//			System.out.println("发送 POST 请求出现异常！" + e);
			result = new StringBuilder("{\"resCode\":\"1\",\"errCode\":\"1001\",\"resData\":\"\"}");
			e.printStackTrace();
			log.error("远程服务未开启", e);
		}
		// 使用finally块来关闭输出流、输入流
		finally {
			try {
				if (out != null) {
					out.close();
				}
				if (in != null) {
					in.close();
				}
			} catch (IOException ex) {
				ex.printStackTrace();
			}
		}

		return result.toString();
	}
# Redis：

开源支持网络交互的、可基于内存也可持久化的KEY-Value数据库

Value支持五种数据类型：

1.字符串（strings）
2.字符串列表（lists）
3.字符串集合（sets）
4.有序字符串集合（sorted sets）
5.哈希（hashes）

1.key不要太长，尽量不要超过1024字节，这不仅消耗内存，而且会降低查找的效率；
2.key也不要太短，太短的话，key的可读性会降低；
3.在一个项目中，key最好使用统一的命名模式，例如user:10000:passwd。

Redis持久化方式：

RDB(Redis  DataBase)和AOF（Append Only  File）

1、RDB，简而言之，就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上；

2、AOF，则是换了一个角度来实现持久化，那就是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了。



# Shell：

用户界面和命令行就是这个另外开发的程序，就是这层“代理”。在Linux下，这个命令行程序叫做 **Shell**

## Shell 是一种脚本语言