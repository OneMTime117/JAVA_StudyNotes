# **MAVEN使用**

maven是一个项目管理工具，可以对java项目进行构建和依赖管理

## 1、maven的核心文件 pom.xml ：

POM(project object Model,项目对象模型) ；

是一个XML文件，包含项目的基本信息、构建信息、依赖信息；

可以配置：项目依赖、插件、执行目标、项目构建的 （配置文件）profile（可针对不同环境进行配置）、项目版本、项目开发者列表、相关邮件列表信息

pom.xml常用标签：

````java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0     https://maven.apache.org/xsd/maven-4.0.0.xsd">
	
	<modelVersion>4.0.0</modelVersion>//pom模型版本
	<parent>    //该pom继承了一个父pom（springboot启动器就使用了这种方式简化pom配置）
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	
     //本项目基本信息
	<groupId>com.yh</groupId> //全球标识符
	<artifactId>test</artifactId>//唯一标识符
	<version>0.0.1-SNAPSHOT</version>//版本号
	<packaging>war</packaging>//项目打包方式
	<name>test</name>//项目名
	<description>Demo project for Spring Boot</description>//项目说明

     //全局变量配置
	<properties>
		<java.version>1.8</java.version>  //内置变量 项目使用的java版本
		
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>
       //内置变量  修改项目源码、输出、编译时的编码格式 utf-8
		
        <springboot.version>1.5</springboot.version> //自定义变量，在之后的属性中，可以统一使用该变量值，通过  ${springboot.version} 来使用该变量
	</properties>

	//依赖：包括groupId、artifactId、version、scope
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	//项目build配置
	<build>		
        //build时使用插件（springboot项目构建统一提供spring-boot-maven-plugin来丰富 maven build 对于spring项目的支持）
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
		
        //定义项目资源文件在执行maven build时，所存放的位置
        <resources>       
    	//一般用于指定第三方jar（项目根目录下的lib文件夹中）编译后，资源存放路径
            <resource>
                <directory>${project.basedir}/lib</directory>
                <targetPath>BOOT-INF/classes/</targetPath>
                //指定资源文件名的过滤
                <includes>
                    <include>*/ **.jar</include>
                </includes>
                <filtering>true</filtering>//启动资源文件配置过滤器：将资源文件中的动态占位符（${key}），使用实际值代替(配置文件中的键值对是可以互相引用的)
            </resource>
        </resources>
	</build>
	
     //多环境配置
    <profiles>
		<profile>
			<id>dev</id>
			<properties>
				<activatedProperties>dev</activatedProperties>//环境标识，用于匹配 build配置中的profile参数
			</properties>
			<activation>
				<activeByDefault>true</activeByDefault>//设置为默认环境
				。。。。。//配置该环境下，项目的其他参数
			</activation>
		</profile>
		<profile>
			<id>test</id>
			<properties>
				<activatedProperties>test</activatedProperties>
			</properties>
		</profile>
	</profiles>


</project>
````



## 2、maven的配置文件 setting.xml,常用配置:

````java
<localRepository>D:\JAVA\maven-localRepository</localRepository>//设置本地仓库路径
    
<mirrors> //使用阿里云镜像，进行远程仓库jar下载
  <mirror>             
  <id>alimaven</id>             
  <name>aliyun maven</name>             
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>             
  <mirrorOf>central</mirrorOf>         
  </mirror> 
 </mirrors>
 
 //配置默认环境下，项目使用jdk1.8
<profiles> 
  <profile>  
    <id>jdk-1.8</id>  
    <activation>  
    <activeByDefault>true</activeByDefault>  
    <jdk>1.8</jdk>  
    </activation>  
    <properties>  
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target> 
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
    </properties>
 </profile>  
</profiles>
````

#### 3、maven仓库：

仓库分为：本地仓库、中央仓库、远程仓库

本地仓库：用于保存从远程仓库下载的jar包，给项目使用（在pom文件中添加依赖时，会先扫描本地仓库，没有在到远程仓库下载）

通过在setting.xml中的 < localRepository >标签进行设置

````java
<localRepository>D:\JAVA\maven-localRepository</localRepository>
````

中央仓库：为maven社区管理的公共远程仓库，提供绝大部分流行开源的java构件，但在由于在国外，下载速度慢

远程仓库：用于开发人员自己定义的仓库，便于其他人使用；如阿里云（Aliyun）远程仓库就是国内比较著名的公共远程仓库（推荐以该仓库作为maven的默认远程仓库）

通过在setting.xml中添加mirrors（仓库镜像）节点来实现

 ````java
 <mirrors>
  <mirror>             
  <id>alimaven</id>             
  <name>aliyun maven</name>             
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>             
  <mirrorOf>central</mirrorOf>         
  </mirror> 
  </mirrors>
 ````

mirrors节点可以拦截访问指定 < mirrorOf>central< /mirrorOf> id为central的仓库（中央仓库），从而转向访问mirror节点指定的远程仓库。达到替换默认远程仓库（中央仓库）的作用

远程仓库可以通过< repositories>依赖远程仓库、< pluginRepositories>插件远程仓库两个标签，在pom.xml和setting.xml中配置；**但pom.xml配置的优先级高（此时setting.xml直接不生效）**

仓库访问顺序：本地仓库  > 中央仓库 >远程仓库 >找不到时抛开异常    **仓库镜像会拦截指定id仓库的访问**

## 4、maven插件

maven插件用于：打包jar、war文件、编译源代码、单元文件测试、创建工程文档、创建工程报告

## 5、maven导入依赖

````java
    <dependencies>		
        <dependency>
        //本地仓库会以org/mybatis/spring/boot/mybatis-spring-boot-starter/2.0.1 目录结构保存
        //构件文件名为mybatis-spring-boot-starter-2.0.1.jar
			<groupId>org.mybatis.spring.boot</groupId>   
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>2.0.1</version>

			<scope>test</scope>
		</dependency>
	</dependencies>
````

  maven依赖的作用域分类：

* compile，缺省值，适用于所有阶段，会随着项目一起发布。
* provided，类似compile，期望JDK、容器或使用者会提供这个依赖，因此不会参与打包。如servlet.jar。
* runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段，不会编译。
* test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。
* system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

## 6、当需要使用maven导入本地依赖时：

使用maven命令行执行将第三方jar导入本地仓库：

````java
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.1.0 -Dpackaging=jar -Dfile=C:\Users\OneMTime\Desktop\ojdbc6-11.2.0.1.0.jar
````

groupId、artifactId、version应和pom.xml中的依赖坐标一致   file 为jar的路径

**使用maven导入第三方jar包和使用lib目录导入jar的区别：**

maven导入第三方包后，其他项目都可以直接通过 < dependency>标签来导入该依赖

而lib目录导入jar需要每个项目做相同操作，不方便版本管理

## 7、maven  build构建 生命周期：

validate  验证：验证项目是否正确且所有必须信息是可用的

compile  编译：对源代码进行编译（生成calss文件）

test         测试： 对单元测试框架进行运行测试

package 打包： 创建JAR/WAR包如在 pom.xml 中定义提及的包

verify      检查： 对集成测试的结果进行检查，以保证质量达标

install     安装： 安装打包的项目到本地仓库，以供其他项目使用

deploy    部署：拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程

**在maven  build执行前添加需要执行的命令（goals 目标）：指定需要执行到的生命周期阶段，常用 compile、test、package**

由于项目在开发过程中会不停修改部署，因此在重新编译、打包时，需要先进行clean，将maven执行后的文件删除，重新进行maven build

**因此一般在使用命令行时，前面应添加 clean  如  clean  package；也可以先执行calen命令、再打包**

**这些build的执行，都依赖于maven内置的插件；当然也可以使用第三方插件**

## 8、使用本地项目作为依赖：

可以通过本地项目Pom文件中的groupId、artifactId、version三个属性，直接作为依赖在其他本地项目使用，但要注意：

**这种MAVEN本地项目创建依赖过程，必须先对所依赖的项目进行maven安装，将项目打包到本地仓库中，之后其他使用该依赖的项目才能正常打包**

# Eclipse使用：

## 1、eclipse对项目的编译：

​	1、默认情况下，eclipse在项目修改保存后，会自动对修改的文件进行编译（但推荐取消，影响此时电脑性能和代码编写体验）

​	**通过  Project--Build Automatically    设置**

​	2、也可以通过eclipse手动编译，但会直接clean所有编译文件，然后重新编译整个项目

​    **通过   project ---clean** 

​	3、使用maven来进行编译，效果和eclipse类似；使用命令行：clean   compile

​	4、一般情况下，可以通过在本地执行项目，此时eclipse会对修改后的文件重新编译（日常开发时，使用）

**###eclipse在本地运行主方法（测试类、springboot jar项目都使本地运行了主方法）时，会触发eclipse编译，将修改过的java编译成calss文件；而对应war项目，需要在外部tomcat运行时，并不会触发编译，因此需要自己手动编译后，才能使修改的代码生效**



## 2、eclipse对项目进行打包：

​	1、需要直接运行的jar包：export---runnable jar file，直接指定主函数类，然后按不同方式打包：

​         runnable jar有三种打包的形式：

​				1、将需要的依赖jar文件转变为class文件保存

​				2、直接在生成的jar内部保存依赖jar文件，并提供ogr文件夹，存放一些calss文件用于读取内部jar文件 

​				3、将lib文件保存在生成jar文件的同目录下（推荐这样jar大小最小）

​	2、作为第三方jar使用： export---jar file，正常生成的**META-INF\MANIFEST.MF**主清单没有指定class-main(主类)和  calss - path（第三方jar路径）；因此无法直接运行，但可以作为第三方jar使用

​	3、作为war包在服务器上部署：export -war file

​		**对于一些使用框架的复杂项目，不推荐使用使用eclipse进行打包，会由于目录结构问题，导致项目无法正常运行；因此应使用maven进行打包**

## 3、eclipse  DeBug 调试：

- 单步执行：根据代码行数，一行行执行

​	F5：会进入到每行的方法中
​    F6：不会进入到每行的方法中
​    F7：直接快速执行到该方法的返回值代码，转到方法调用的上一级代码中

​	通过对这三个单步执行方式的切换使用，来找到自己想要观察的执行代码行

- 其他调试按钮

​    F8：resume 一直往下执行代码，直到下一个断点（可以通过在循环外添加断点，然后使用F8来跳过循环）
​    Skip All Breakpoints 屏蔽所有断点
​    Remove All Breakpoints  清除所有断点
​    Drop to Frame ：让程序从方法开头第一行重新开始执行

- 其他调试方法：

1、让程序运行达到某个条件时，断点生效：
   对于已经添加断点的行代码添加执行条件，进入BreakPoint视图，选择需要添加执行条件的断点
   1、勾选Hit count ，设置次数：该行代码执行第5次前，断点生效
   2、勾选conditional ，设置断点生效条件，入i==100；	

两种方式不能同时使用，否则断点设置无效
2、可以在variables视图中快速改变变量值，来跳过加快循环的执行
3、使用Step Filters 在断点调试时，跳过不需要的类。 在Windows -> Preferences -> Java -> Debug -> Step Filtering中设置，然后调试时，打开Use step Fileters
4、在Expressions视图中，可以对代码中的变量进行表达式计算( 也可以在代码中选择表达式、变量，然后右键watch)

## 4、eclipse字符集设置

eclipse工作窗口编码、分隔符类型：preference--General-workspace      

eclipse控制台窗口编码 ：run configurations--common（默认utf-8）

## 5、eclipse模板设置

模板：当新建class、方法等时，直接生成一些固定注解或代码

preferences---java ----code style---code Templates  ---Comments/code

comment： 当自动创建的基础代码（类、文件）时，会自动添加注解  也可以选择对应的类名、方法名。。。使用**ALT+SHIFT +J**   对手动的代码自动添加模板注解

code:用于配置eclipse生成器中的方法代码，如set、get等，一般不用修改

常用类、方法的固定注解：

```java
@author			创建人
@date			创建时间
@Description   描述当前类、方法作用
${tags}  可以显示方法的参数、返回值注解
```

