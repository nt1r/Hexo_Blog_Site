---
title: Java 连接 Oracle 数据库
date: 2018-12-13 09:11:39
categories:
- CS
tags:
- Java
- Oracle
---
## Java 连接 Oracle 数据库

利用Java连接Oracle数据库需要使用JDBC（Java Database Connectivity），JDBC的使用主要分为6大步骤：

> 1. 加载Oracle驱动
> 2. 获取数据库连接
> 3. 创建SQL语句执行对象
> 4. 执行SQL语句
> 5. 处理数据库返回的结果集
> 6. 释放资源断开连接

**Demo**：假设Oracle数据库中有一张由姓名name和年龄age构成的tableTest表，name是VARCHAR2类型，age是NUMBER类型，如下图所示。

![testTable表中的内容](/blog/images/java_link_oracle_database_name_age.png)

我们要从上述的tableTest表中获得name为leqi的人所对应的age。

* **需要在项目中导入ojdbc6.jar才能正常运行下列代码**

## 建立与Oracle数据库的连接

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