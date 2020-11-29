# 权限框架:

# springboot安全框架  spring securtiy

**spring security：是一个基于spring开发，强大、容易定制的实现认证和授权的框架**

spring securtiy是通过后台来进行页面的跳转验证，因此不满足现在前后端分离的REST架构风格，需要进行一定的自定义配置

### 1、spring security的功能：

1、Authentication（认证）：用户登录

2、Authorization（授权）：判断用户权限、可以访问那些资源

3、安全防护：防止跨站请求伪造（CSRF安全验证）、session攻击等

​	跨站请求伪造（CSRF）：当用户登录了某个网站，此时cookie中保存了该访问的认证信息；然后此时通过一个其他页面诱导用户点击，触发访问刚才网站的脚本，因此就能够请求就能通过cookie中的认证信息，继续进行访问操作（即利用了网站对于用户页面浏览器的信任）

​	跨站脚本攻击（XSS）：通过向用户发送给网站的请求传参中，添加恶意的HTML代码（类似于后台的sql注入，但是是为了破坏前台代码），然后就可以获取当前客户端下的cookie数据，实现session攻击，盗取该用户身份

​	session攻击：通过某种方式获取用户登录后的sessionID：

​				1、会话劫持：通过某种手段直接从客户端获取当前用户的sessionID

​				2、会话固定：通过重置用户的sessionID，让用户通过该sessionID进行站点登录；从而固定用户使用的sessionID

4、结合springMVC非常方便使用

### 2、spring security 和 apache Shiro的区别：

- spring security的优点：

1、基于spring开发，因此整合spirng家族框架更加方便；而shiro和spring需要进行配置整合

2、spring security功能更加丰富，如安全防护方面

3、springsecurity社区资源更加庞大

- apache shiro的优点：

1、使用配置比较简单，更容易上手

2、shiro依赖性低，不需要任何框架，就可以单独使用

### 3、spring security 认证、授权功能实现流程：

- **spring security主要配置：**

1、spring security拦截什么资源，不拦截什么资源

2、对应资源需要什么角色权限（配置授权规则）

3、定义认证方式：HttpBasic、FormLogin（最常用的表单登录）

​	HttpBasic：使用Http基础认证机制，当客户端访问资源响应一个401状态码时，浏览器会自动弹出一个登录对话框，然后登录后通过authorization请求头将账号密码加密后的密钥入后台，最后进行账号解析认证

4、指定认证使用的用户数据集

5、定义认证、授权的异常处理器

6、配置spring security安全防护

- **spring security底层通过springsecurityFilterChain （过滤器链）实现所有认证、授权功能，核心过滤器有：**

1、UsernamePasswordAuthenticaitonFilter：基于FormLogin的用户登录认证

2、BasicAuthenticaitonFilter：基于HttpBasic的用户登录认证

3、FilterSecurityInterceptor： 用于实现包含url资源访问的安全（拒绝不合格的用户请求，并抛出对应异常）

由多个组件构成（如FilterInvocationSecurityMetadataSource、AccessDecisionManager），定义授权规则

4、SecurityContextPersistenceFilter，用于将SecurityContext放入session的过滤器，通过sessionid获取已登录用户信息，从而进行后面的权限验证**（因此spring security默认使用的cookie-session的认证方式）**

5、AnonymousAuthenticationFilter，当前用户没有登录时，securityContext就没有在SecurityContextPersistenceFilter中被初始化，此时就需要生成要给匿名securityContext，用于后面未登录操作的判断

6、ExceptionTranslationFilter，异常处理过滤器，处理系统认证授权中抛出的异常：`AuthenticationException` 和 `AccessDeniedException`

- **spring security 认证流程（formLogin方式）：**

1、UsernamePasswordAuthenticationFilter   获取处理用户登录信息

2、providerManager  对认证用户数据集提供者进行管理（判断选择哪个作为认证用户的数据集）

3、AuthenticationProvider(为接口，spring security提供一个默认实现类DaoAuthenticationProvider)  将用户登录信息与提供的认证数据进行匹配认证，返回认证结果信息

4、AuthenticationSuccessHandler、AuthenticationFailureHandler 根据认证结果，执行相应认证处理器

### 4、自定义用户认证：

- 自定义提供用户信息（必须，否则就需要在配置中手动固定添加用户名、密码、权限）

在用户类的service中 implements UserDetailsService：

需要重写loadUserByUsername（String username）方法，返回值未UserDetailes（可以使用框架提供的user，包括用户名、密码、权限集合；也可以使用自定义User，但需要implements UserDetails，重写getAuthorities()，返回值为该用户的权限对象集合ArrayList<GrantedAuthority>），用于提供用户信息（可以从内存、数据库中获取）

- 自定义AuthenticationProvider（认证提供者），用于自定义用户信息认证逻辑（一般不用自定义，对于账号密码的认证，spring security已经写的很完善了）
- 自定义额外认证（添加除用户账号、密码外的其他认证方式，如图形验证码认证等）

1、自定义额外认证过滤器，继承springboot提供的OncePerRequestFilter，实现过滤功能（对于/login登录url中的额外认证属性进行判断

2、自定义继承AuthenticationException的字类异常，在认证不通过时，抛出并指定对应原因

3、在自定义处理Authenticaiton登录认证的失败异常处理器中，添加对于额外认证信息错误的异常判断，返回对应原因的json字符串

4、在spring security的配置类WebSecurityConfig中，添加该过滤器：

````java
//将自定义的图形验证码认证过滤器放在用户认证过滤器前
		http.addFilterBefore(new ImageCodeAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
		
````



### 5、自定义登录成功、失败、权限不足异常处理器：

**为什么要自定义处理器：spring security默认处理时，会进行一些页面跳转，不满足前后端分离的需求**

分别实现AuthenticationSuccessHandler、AuthenticationFailureHandler、AccessDeniedHandler接口，并重写对应的方法；方法中会传入HttpServletRequest request, HttpServletResponse response和异常对象（成功时会传入Authentication（认证对象，保存了用户认证信息））

然后我们可以在处理器方法中，使用原生serlvet方式，返回对应的json信息

### 6、在登陆后获取用户信息

spring security提供一个SecurityContextHolder工具类。当用户登录成功后，会在当前线程下，创建一个当前session所在线程的线程局部变量，来保存Authentication认证对象（用户的登录信息）。

使用如下方式获取：

````java
//SecurityContextHolder.getContext()为静态方法，不需要进行对象注入
Authentication authentication = SecurityContextHolder.getContext()为静态方法.getAuthentication();
		Object principal = authentication.getPrincipal();
		String username = "";
		String password ="";
		if (principal != null) {
			if (principal instanceof UserDetails) {
				User user = (User) principal;
				username = user.getUsername();
				password= user.getPassword();

			}
		}
````



### 7、基于前后端分离下，springboo基于SpringSecurity处理认证授权实例

#### 1、后端用户安全权限管理模块数据库设计（RBAC模型）

**RBAC（基于角色的访问控制）模型：用户关联角色、角色关联权限（权限对应于web资源，即url）**

在前后端分离下，权限数据库主要包含五张表：用户信息表、用户----角色表、页面资源表、

角色表、角色----资源表

**根据用户登录获取其对应用户信息、用户角色、角色对应资源，通过login、checkLogin接口来返回给前端，进行页面资源的交互渲染；即不同用户，登录使用的页面目录、操作都不同**

#### 2、后端项目构建

1、创建springboot项目，添加springweb模块、spring security模块、spring mybatis模块和mysql驱动

2、使用mybatis-generator代码生成工具，生成数据库中用户表、角色表、资源表的jopo、mapper类和mapper.xml(可以省略，使用mybatis注解方式)

3、配置对于数据库的数据源（单数据源可直接使用默认配置），并打开mybati实体映射时的驼峰规则

#### 3、配置整合spring security

1、用户信息表的实体类实现UserDetails，重写Collection<? extends GrantedAuthority>   getAuthorities（）方法

2、创建UserService类，实现UserDetailsService，重写UserDetails loadUserByUsername(String username)方法

3、创建UrlFilterInvocationSecurityMetadataSource类，实现FilterInvocationSecurityMetadataSource，重写Collection<ConfigAttribute> getAttributes(Object object)方法，获取当前url所需权限角色集

4、创建UrlAccessDecisionManager类，实现AccessDecisionManager，实现void  decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)方法，判断当前用户权限是否满足当前url的权限需求，不满足则抛出异常

5、创建spring security的配置类：WebSecurityConfig，继承WebSecurityConfigurerAdapter，实现如下配置：

````java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	//spring security 默认使用cookie-session进行用户认证，通过cookie中的jsessionID来匹配对于的session
	//和框架保存好的用户信息
	
	@Autowired // url资源安全过滤器（获取url资源允许访问的角色集合）
	UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource;
	@Autowired // url资源禁止访问管理器（用于判断用户是否可以访问该资源）
	UrlAccessDecisionManager urlAccessDecisionManager;
	@Autowired
	UserService userService;

	// 配置不需要被WebSecurity权限模块管理的请求
	@Override
	public void configure(WebSecurity web) throws Exception {
		 web.ignoring().antMatchers("/static/**","/login_p","/error");
	}

	// 配置默认用户验证使用的服务类
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		//验证用户密码时，使用BCryptPasswordEncoder进行加密比对（因此注册时，密码存数据库前，应进行加密）
		auth.userDetailsService(userService).passwordEncoder(new BCryptPasswordEncoder());
	}

	// 配置webSecurtiy权限规则
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// 配置请求授权的过滤器、管理器(定义授权规则)
		http.authorizeRequests().withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {
			@Override
			public <O extends FilterSecurityInterceptor> O postProcess(O object) {
				object.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource);
				object.setAccessDecisionManager(urlAccessDecisionManager);
				return object;
			}
		});
		
		//将自定义的图形验证码认证过滤器放在用户认证过滤器前
		http.addFilterBefore(new ImageCodeAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
		
		// 开启自动配置好的登录功能(http.formLogin(),之后可以修改自动配置好的属性)
		// 定义登录页面、登录url、用户名参数名、密码参数名（只能为表单提交post请求）、
		http.formLogin().loginPage("/login_p").loginProcessingUrl("/login").usernameParameter("username")
				.passwordParameter("password").permitAll();//允许所有角色访问/login（不会经过请求授权的过滤器、管理器）
		
		
		// 定义登录失败处理器(返回不同错误)
		http.formLogin().failureHandler(new AuthenticationFailureHandler() {
			@Override
			public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
					AuthenticationException exception) throws IOException, ServletException {
				response.setContentType("application/json;charset=utf-8");
				PrintWriter out = response.getWriter();
				Result result = null;
				//用户名不存在、密码错误、账号被禁、登录失败（服务器报错，如无法连接数据库）
				//若不想暴露用户名是否存在的信息，可以前两个异常共同处理
				if (exception instanceof UsernameNotFoundException ) {
					result = ResultUtils.error("用户名不存在！");
				} else if(exception instanceof BadCredentialsException) {
					result = ResultUtils.error("密码错误！");
				} 
				else if (exception instanceof DisabledException) {
					result = ResultUtils.error("账户被禁用，登录失败，请联系管理员!");
				} else {
					result = ResultUtils.error("登录失败!"+exception.getMessage());
				}
				out.write(result.toString());
				out.flush();
				out.close();
			}
		});

		// 定义登录成功处理器
		http.formLogin().successHandler(new AuthenticationSuccessHandler() {
			@Override
			public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
					Authentication authentication) throws IOException, ServletException {
				response.setContentType("application/json;charset=utf-8");
				PrintWriter out = response.getWriter();
				HttpSession session = request.getSession();
				String token = session.getId();
				Object principal = authentication.getPrincipal();//用户信息，保存到redis.以sessionId作为key
				System.out.println(principal);
				out.write(ResultUtils.success(token).toString());
				out.flush();
				out.close();
			}
		});

		// 开启自动配置好的用户注销功能
		http.logout().permitAll();
        //（spring security 默认开启）关闭CSRF安全验证，避免该机制影响springboot框架的使用（CSRF 跨站请求伪造，通过利用用户在站点1登录的cookie验证信息，来通过站点2的请求校验 ）
		http.csrf().disable();
		//定义权限不足异常处理器
		http.exceptionHandling().accessDeniedHandler(new AccessDeniedHandler() {
			@Override
			public void handle(HttpServletRequest request, HttpServletResponse response,
					AccessDeniedException accessDeniedException) throws IOException, ServletException {
				response.setContentType("application/json;charset=utf-8");
				PrintWriter out = response.getWriter();
				out.write(ResultUtils.error(accessDeniedException.getMessage()).toString());
				out.flush();
				out.close();
			}
		});

	}
````



# Apache Shiro

java 安全权限框架：

# java web常用用户验证方式：

#### 1、cookie-Session：由于http协议是无状态的，因此导致服务器并不知道该请求是哪个用户发送的。

​	1、因此，在服务器端在用户第一次访问，并创建session时，会根据用户信息来生成sessionID（作为客户端的身份标识），保存到当前会话（服务器session）中

​    2、在请求响应后，返回一个存储jsessionId的cookie交给客户端，来保存自己客户端的’身份标识‘；

​	3、客户端在下次进行请求时，会将cookie传入服务器，服务器会自动对比cookie中的jsessionID来匹配服务器中的Sessionid,保证客户端于服务器端的会话

**前三步在session手动创建后，服务器会自动完成这些操作；只有当session被创建时，服务器才会创建返回cookie**

​	4、然后我们可以通过取出cookie中的JsessionID（session中的SessionID）来获取内存、数据库中保存好的当前用户的信息，从而进行用户身份验证

**该方式存在的问题：**

​		当使用集群服务器时，只有客户端访问的服务器才会建立session；就无法保证客户端访问所有服务器时，都能进行正常的身份验证，因此当一台服务器建立session后，就需要同步所有服务器的session（大大增加了服务器性能开销、并且在扩展服务器时很不方便）

#### 2、Token：很好解决了session产生的问题，session需要服务端一直进行维护，从而保存用户信息，而Token可以通过算法密钥将用户信息加密，保存到客户端下;另外其他平台（Android、Ios移动设备原生不支持cookie）

1、客户端进行用户登录后，服务器通过加密算法，将用户信息生成未一个token（字符串）

2、通过JSON形式，响应传回给客户端

3、客户端将Token保存在cookie或local Storage（本地存储）

4、在下次请求时，将Token通过http的header传递给服务器进行解析

5、若token不存在，则证明没有登录；存在则解析数据，验证用户信息

缺点：由于Token的使用不受服务器session状态的影响，因此Token在生成后，无法进行权限变更，在到期之前一直有效。

**对于安全方面的优点：有效防止CSRF**

由于浏览器只能自己带上Cookie，而不能将Cookie中Token放入http的header中；因此不会受到CSRF的影响

#### Token和cookie-Session都有一个很大的安全隐患：

**会被黑客通过XSS的方式进session攻击，获取sessionID、Token；**

解决方式：

1、可以通过设置cookie的HttpOnly参数为true，此时js脚本就无法获取当前客户端下的cookie，也就无法被XSS获取cookie中的数据

​	网站可以使用后台过滤器，在客户端第一次访问网站时，返回的cookie中开启HttpOnly设置（一般默认为true）

2、对于http参数进行特殊字符过滤，防止js脚本的注入

#### 3、Token-redis：Token只保存一个key，用户信息保存到redis中，给所有服务器集群使用

该方式就是通过Token来替换了session保存sessionID的方式，将用户信息管理的key由客户端管理；没有遵循Token的定义，而只是将Token作为一个关键key，来存储获取redis中的用户信息

对于Token的有效期，可以通过redis的过期机制控制

#### 4、JWT（json web token）：是一种特殊的Token，遵循Token的定义，将用户信息加密保存。

