mybatis-genneerator代码生成工具：

根据数据库表设计，自动生成jopo、dao层mapper、mapper.xml

使用方法：使用maven整合的mybatis-generator插件

1、创建maven项目，pom添加配置mybatis-generator-maven-plugin插件

````java
<!-- maven build 执行命令为：mybatis-gennerator：generate -e -e含义：打印maven build执行信息 -->
	<build>
		<plugins>
			<plugin>
				<!-- 使用MAVEN中使用 mybatis-generator-maven-plugin插件 -->
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.7</version>
				<configuration>
					<!--配置文件的位置 -->
					<configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
					<verbose>true</verbose>
					<overwrite>true</overwrite>
				</configuration>

				<!-- 添加mybatis-generator插件使用所需要的依赖：数据库连接驱动 -->
				<dependencies>
					<dependency>
						<groupId>mysql</groupId>
						<artifactId>mysql-connector-java</artifactId>
						<version>8.0.18</version>
						<scope>runtime</scope>
					</dependency>
				</dependencies>
			</plugin>
		</plugins>
	</build>
````

2、创建配置mybatis-gennerator工具需要的配置文件generaotrConfig.xml:

````java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

	<context id="mysqlgenerator" targetRuntime="MyBatis3">

		<commentGenerator> 
		<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
		 <property name="suppressAllComments" value="true" /> 
		</commentGenerator>
		
		<!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL 
			和 NUMERIC 类型解析为java.math.BigDecimal -->
	<!-- 	<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver> -->

        <!-- 数据库连接,xml中会对&符合进行转义处理，因此url参数使用&amp;分隔 -->
		<jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
			connectionURL="jdbc:mysql://localhost:3306/secutriy?serverTimezone=GMT&amp;useUnicode=true&amp;characterEncoding=UTF-8"
			userId="root" password="123456" >
			<!-- mysql数据库驱动高版本，会导致生成器获取对应表中的其他信息，生成多余的bean，通过该属性来避免 -->
			<property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>
        
	    <!-- 生成的javabean位置-->
		<javaModelGenerator  targetPackage="entity" targetProject="src/main/java">
		 <!-- enableSubPackages:是否让schema作为包的后缀 -->
         <property name="enableSubPackages" value="false" />
         <!-- 从数据库返回的值被清理前后的空格 -->
         <property name="trimStrings" value="true" />
		</javaModelGenerator>	
			

        <!-- mapperxml映射文件 -->
		<sqlMapGenerator targetPackage="sqlxml" targetProject="src/main/resources" >
		 <!-- enableSubPackages:是否让schema作为包的后缀 -->
         <property name="enableSubPackages" value="false" />
         <!-- 从数据库返回的值被清理前后的空格 -->
         <property name="trimStrings" value="true" />
		</sqlMapGenerator>

		<javaClientGenerator type="XMLMAPPER" targetPackage="mapper" targetProject="src/main/java" >
		 <!-- enableSubPackages:是否让schema作为包的后缀 -->
         <property name="enableSubPackages" value="false" />
         <!-- 从数据库返回的值被清理前后的空格 -->
         <property name="trimStrings" value="true" />
		</javaClientGenerator>

		<table tableName="menu"  domainObjectName="Menu" />
	</context>

</generatorConfiguration>
````

3、使用maven build允许插件：命令行为  mybatis-gennerator：generate -e

4、此时，根据配置文件的指定的目录，生成对应的jopo、mapper、mapper.xml

