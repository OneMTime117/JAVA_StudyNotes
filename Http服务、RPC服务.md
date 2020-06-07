## **REST**服务（Http服务）

REST ： Resource Representational State Transfer  资源（请求中的数据） 表现层（数据的形式，json、xml） 状态 变化（服务器需要对数据的操作，CRUD create/read/update/delete 增删改查）
用于定义web中客户端与服务端进行软件交互设计规范，统一了web、ios、android平台的服务接口规范，HTTP协议就实现这些规范：
1、根据不同的操作来指定http的访问方式（post、get、update、delete）：
这样才更满足http协议的设计，而不是把应用层的http当中只传输数据，因此url应该不能含有动词；否则当此时使用url来传递数据时，post、get请求都能满足，这就会破坏httpGET、POST方法的安全性和幂等性；
2、这样就可以把url当中一个资源，通过get、post、put、delete来进行对资源的不同操作；而不是每种操作都是一个url，不符合url的设计规范
3、最关键的REST服务应该满足超文本驱动，http就是超文本传送协议
RESTful API（使用了REST风格的api）：满足REST需求的服务接口设计规范的接口（服务），因此我们常把http服务称为REST服务

java常用访问Http RESTful服务方法：
1、java原生的HttpURLConnection（使用java.net包）

````java
		try {
			URL url = new URL(
					"http://10.64.140.1:6080/arcgis/rest");
			HttpURLConnection HttpConnection = (HttpURLConnection) url.openConnection();
			HttpConnection.setRequestMethod("GET");// 设定http访问url方式，如GET
			HttpConnection.setRequestProperty("ContentType", "application/x-www-form-urlencoded");// 添加http请求头属性，如数据内容格式Content-Type
			HttpConnection.setConnectTimeout(2000); // 设置连接超时时间，单位：ms
			HttpConnection.setReadTimeout(2000); // 设置读取超时时间，单位：ms
			HttpConnection.setDoInput(true); // 设置打开输入流 ： default = true
			HttpConnection.setDoOutput(true); // 设置打开输出流：default = false
			HttpConnection.setUseCaches(false); // 设置是否启用用户缓存： default = false
			OutputStream outputStream = HttpConnection.getOutputStream();// 通过http来连接创建输出流
			String pama = "name=11&id=1";
			outputStream.write(pama.getBytes());// 通过输出流将数据写入到服务器
			outputStream.flush();
			outputStream.close();

			// 打印返回状态、消息
			System.out.println(HttpConnection.getResponseCode());
			System.out.println(HttpConnection.getResponseMessage());

//			通过连接创建字节输入流，并将输入流转化为字节流
			InputStreamReader reader = new InputStreamReader(HttpConnection.getInputStream(), "utf-8");
			BufferedReader br = new BufferedReader(reader);
			String line = "";
			StringBuffer sb = new StringBuffer();
			while ((line = br.readLine()) != null) {
				sb.append(line);
			}
			br.close();
			reader.close();
			System.out.println(sb.toString());
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
````

2、创建HttpClient来进行调用REST服务（使用org.apache.http.client.HttpClient包）

添加依赖有：httpclient-4.5.3.jar、httpcore-4.4.8.jar、commons-logging-1.0.4.jar

````java
		CloseableHttpClient httpClient = HttpClients.createDefault();
		//get请求
		HttpGet httpGet = new HttpGet("http://10.64.140.1:6080/arcgis/rest/services");
		CloseableHttpResponse response = httpClient.execute(httpGet);
		
		//post请求
		HttpPost httpPost = new HttpPost("http://10.64.140.1:6080/arcgis/rest/services");
		//创建表单数据
		List<NameValuePair> formData= new ArrayList<NameValuePair>();
		formData.add(new BasicNameValuePair("name", "yh"));
		StringEntity formData1 = new UrlEncodedFormEntity(formData,"utf-8");
		httpPost.setEntity(formData1);
		CloseableHttpResponse response2 = httpClient.execute(httpPost);
		
		
		org.apache.http.HttpEntity entity = response.getEntity();
		String string = EntityUtils.toString(entity);
		response.close();
		httpClient.close();
		System.out.println(string);
````

3、使用springboot封装的REST template（spring-web中包含该依赖）

````java
//      GET请求		
		RestTemplate restTemplate = new RestTemplate();
		ResponseEntity<String> forEntity = RestTemplate.getForEntity("http://10.64.140.1:6080/arcgis/rest/services/JianGuan",
				String.class);//前一个参数为url，后一个为请求响应的body数据类型
		//getForEntity是通过get请求获取ResponseEntity对象，它是springboot对HTTP请求响应的封装，包括了几个重要的元素，如响应码、contentType、contentLength、响应消息体等。
		//而getForObject是对getForEntity进一步封装，直接获取响应的body数据
//		getForEntity、getForOject还支持url占位符添加参数
		String body = forEntity.getBody();
		System.out.println(body);		

//		POST请求
		HttpHeaders httpHeaders = new HttpHeaders();
//      请求头属性contentype: pplication/json; charset=UTF-8
		httpHeaders.setContentType(MediaType.APPLICATION_JSON_UTF8);
//		请求体BODY数据
		JSONObject jsonObject = new JSONObject();
//		请求实体对象（封装请求体、请求头），且指定请求体的数据类型JSONObject
		HttpEntity<JSONObject> httpEntity = new HttpEntity<JSONObject>(jsonObject, httpHeaders);		
		ResponseEntity<String> postForEntity = restTemplate.postForEntity("", httpEntity, String.class);
		String body2 = postForEntity.getBody();
//		postForEntity、postForOject也支持url占位符添加参数
````

## RPC服务

RPC：Remote Procedure Call 远程过程调用

RPC服务和HTTP服务都属于C/S架构，就进行Client、Server端的数据传输
RPC服务建立在传输层，即基于TCP/IP传输层协议；HTTP服务建立在应用层，即基于http应用层协议
**因此RPC服务相对HTTP服务传输速度更快一些**

RPC服务建立：在两个服务（项目，如微服务）中建立TCP客户端和TCP服务端。并通过SCOKET来使用tcp连接，进行数据传输

Socket：scoket是对TCP/IP协议的封装，自己本身不是协议，而是一个API，让我们更好的处理TCP/IP协议的网络通信
1、使用socket接口来创建连接时，可以指定传输层协议，可支持TCP、UDP,当使用TCP协议时，socket连接本质上就是TCP连接
2、socket连接本质上就是TCP连接，因此在使用socket连接时，不仅可以保持长连接，而且不需要像HTTP连接那样，客户端发送一次请求后才能将数据传回给客户端；因此服务器只有有新数据，就能直接传递给客户端

RPC服务一般需要一些相应的框架来进行封装使用；一般使用在大型项目中，使用RPC服务来完成服务间的调用。（Vertx支持RPC服务的封装）



