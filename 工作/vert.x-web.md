# vert.x-web

## 1、创建http服务器

1. 设置监听端口、响应头的内容格式和编码、响应内容
2. 处理从上到下匹配，可以通过**next()**方法来继续执行下一个处理器
3. 可以通过**order()**方法来改变处理器的优先级
4. Vert框架下的处理器，都是通过事件循环来处理的（为非阻塞式），但可以使用**blockingHandler**处理，该处理器被线程调用时，不会使用事件循环处理（阻塞式）

## 2、使用路由来处理请求

1. 通过匹配url来设置不同的处理
2. 路由可以通过url来捕获参数
3. 路由可以使用**正则表达式**来匹配url
4. 可以通过**请求方式**来区分匹配处理器
5. 可以通过**请求MIME类型**来区分匹配处理器，通过**consumes("text/html")**匹配
6. 通过可**接受的MIME类型**来匹配路由处理

## 3、子路由

可以通过设置二级路由来增加路由处理器的匹配区分度，防止同时开发时，出现处理器匹配url相同情况

通过主路由对二级路由进行挂载，使得url增加了“**/main**”这字段

**Mainrouter.mountSubRouter("/main",router);**

## 4、处理404错误

当请求没有匹配到所有路由时，将自动执行**errorHandler**处理（官方默认的处理程序），但以也可以对该处理器进行自定义

## 5、处理请求体

1、首先需要默认调用**请求体处理器**，在执行其他路由处理前创建（通过route()实现）

**router.route().handler(BodyHandler.create());**

***** 可以在创建BodyHandler时，设置请求体大小**setBodyLimit()**等其他设置

## 6、处理文件上传

1、任何文件上载都将自动流式传输到uploads目录

2、使用一般handler进行处理

**Set<FileUpload> uploads = routingContext.fileUploads();**

## 7、处理cookie

1、首先需要默认调用**cookie处理器**，在执行其他路由处理前创建（通过route()实现）

**router.route().handler(CookieHandler.create());**

## 8、处理session

1、先创建cookie处理器

**roter.route().handler(CookieHandler.create());**

2、创建session商店
**SessionStore store = ClusteredSessionStore.create(vertx);**

3、通过session商店来创建session处理器对象

**SessionHandler sessionHandler = SessionHandler.create(store);**

4、然后将sessionHandler对象挂载到路由器中
**router.route().handler(sessionHandler);**

5、之后再其他路由处理中，就可以直接通过路由上下文调用session

## 9、身份验证/授权

## 10、提供静态资源

？？？？？？