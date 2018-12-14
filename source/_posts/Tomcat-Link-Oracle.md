---
title: Tomcat 连接 Oracle 数据库
date: 2018-12-13 09:11:39
categories:
- CS
tags:
- Java
- Tomcat
- Oracle
---

# 配置环境

- Windows 10 64位
- JDK_1.8.0_181
- Tomcat 7.0.90
- Oracle 11g Release 2

确保上述环境的成功配置。

**需要在Tomcat服务器目录中导入ojdbc6.jar才能成功运行之后的代码**：ojdbc6.jar的位置在{Oracle_Home}\app\product\{Oracle_Version}\{Database_Name}\jdbc\lib\，例如 D:\Oracle\app\product\11.2.0\newDatabase\jdbc\lib\。将ojdbc6.jar复制到Tomcat目录中的lib文件夹下，例如 D:\apache-tomcat-7.0.90\lib\。

# Demo
假设Oracle数据库中有一张由姓名name和年龄age构成的名为tableTest的表，name是VARCHAR2类型，age是NUMBER类型，如下图所示。

![testTable表中的内容](/blog/images/tomcat_link_oracle_database_name_age.png)

本文将示范从上述tableTest表中获得姓名name的值为leqi的人所对应的年龄age。

# 主要步骤

利用Java连接Oracle数据库需要使用JDBC（Java Database Connectivity），JDBC的使用主要分为6大步骤：

> 1. 加载Oracle驱动
> 2. 获取数据库连接
> 3. 创建SQL语句执行对象
> 4. 执行SQL语句
> 5. 处理数据库返回的结果集
> 6. 释放资源断开连接

## 建立与Oracle数据库的连接

### 方法一：

```java
try {
	// 1. 注册类对象，加载驱动
	Class.forName("oracle.jdbc.driver.OracleDriver");
	
	// 数据库的URL地址
	String url = "jdbc:oracle:thin:@localhost:1521/orcl";
	
	// username and password
	String user = "username";
	String password = "password";

	// 2. 利用'DriverManager'获取数据库链接
	Connection myCnct = DriverManager.getConnection(url, user, password);
}

catch (Exception e) {
	e.printStackTrace();
}
```

### 方法二：
修改Tomcat服务器目录下的server.xml配置文件，该文件的位置在{Tomcat_Home}\conf\，例如 D:\apache-tomcat-7.0.90\conf\）。在server.xml文件中的<GlobalNamingResources>标签下添加<Resource>标签信息：

```xml
<!-- Global JNDI resources
   Documentation at /docs/jndi-resources-howto.html
-->
<GlobalNamingResources>
<!-- Editable user database that can also be used by
     UserDatabaseRealm to authenticate users
-->
	<Resource name="UserDatabase" auth="Container"
	          type="org.apache.catalina.UserDatabase"
	          description="User database that can be updated and saved"
	          factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
	          pathname="conf/tomcat-users.xml" />
	
	<!-- 需要添加的信息 -->
	<Resource 
		name="jdbc/orcl"
		auth="Container"
		type="javax.sql.DataSource"
		maxActive="100"
		maxIdle="30"
		maxWait="10000"
		username="username"
		password="password"
		driverClassName="oracle.jdbc.driver.OracleDriver"
		url="jdbc:oracle:thin:@127.0.0.1:1521/orcl" />
	<!-- 需要添加的信息 End -->

</GlobalNamingResources>
```
添加结束后保存该文件，重启Tomcat服务器使配置生效。完成重启后可使用下列代码建立与Oracle数据库进行连接。
```java
try {
	// 1. 初始化查找命名空间
    Context ctx = new InitialContext();
    Context envContext = (Context)ctx.lookup("java:/comp/env"); 
    DataSource ds = (DataSource)envContext.lookup("jdbc/orcl"); 

	// 2. 利用'DataSource'获取数据库链接
    Connection conn = ds.getConnection();
}

catch (Exception e) {
	e.printStackTrace();
}
```

- 上述两种方法孰优孰劣需要自己取舍：方法一将用户名和密码写在了代码中，不够灵活也不够安全；方法二读取Tomcat本地配置文件中的信息进行连接，一旦更换用户名或密码后需要修改对应的配置信息。

## 创建并执行SQL语句对象

```java
// 当表名和列名不全为大写时，记得使用转义符(\")；where语句中=的右侧值使用转义符(\')
String mySQL = "select \"Age\" from \"testTable\" where \"Name\" = \'leqi\'";

try {
	// 3. 创建SQL语句执行的对象'PreparedStatement'
	PreparedStatement myPreStat = myCnct.prepareStatement(mySQL);

	// 4. 执行SQL语句
	ResultSet myRs = myPreStat.executeQuery();
}

catch (SQLException e) {
	e.printStackTrace();
}
```

## 处理数据库返回的结果集

```java
try {
	// 5. 循环处理数据库返回结果中的ResultSet
	while (myRs.next()) {
		// your code here
		/* example:
		 * String value = myRs.getString("KeyName"));
		 */
	}
}

catch (SQLException e) {
	e.printStackTrace();
}
```

## 关闭与Oracle数据库的连接，释放资源

```java
try {
	// 6. 释放数据库资源，关闭与数据库的连接，注意顺序
	if (myRs != null) {
		myRs.close();
	}

	if (myPreStat != null) {
		myPreStat.close();
	}

	if (myCnct != null) {
		myCnct.close();
	}
}

catch (Exception e) {
	e.printStackTrace();
}
```

## Demo运行结果

```
Leqi's age is: 22
```
----------