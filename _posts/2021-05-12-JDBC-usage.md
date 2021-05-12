---
title: JDBC使用
author: lijiabao
date: 2021-05-11 23:30
categories: ["MySQL", "Database"]
tags: ["MySQL", "Database"]
---

主要介绍Java中的JDBC数据库管理API的使用



# JDBC使用

## 使用前的准备工作

### 安装数据库

找到自己需要使用数据库，亦或是自己比较熟悉的关系型数据库，常用的数据库：MySQL、Oracle、微软的SQLServer、DB2和PostgreSQL

去各自的官网依照安装向导将其安装好，并测试是否已经安装好可以使用

### 安装数据库对应的JDBC驱动

MySQL对应的JDBC驱动下载地址：[MySQL :: Download Connector/J](https://dev.mysql.com/downloads/connector/j/#:~:text= MySQL Connector%2FJ is the official JDBC driver,use with MySQL Server 8.0%2C 5.7 and 5.6.)

Oracle对应JDBC驱动下载：[Central Repository: com/oracle/database/jdbc/ojdbc8 (maven.org)](https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc8/)

下载好对应的驱动jar包之后，可以直接将其复制到项目文件下的lib文件夹下面，然后将这个文件add as library，添加至库中

或者在命令行操作时，将jar包加入到classpath中

```powershell
java --classpath .:/directory/ojdbc8-21.1.0.0.jar
```

## 使用步骤

### 1. 注册驱动器

主要有两种方法：

1. 使用`Class.forName()`

   ```java
   // mysql
   Class.forName("com.mysql.cj.jdbc.Driver");
   // oracle
   Class.forNmae("oracle.jdbc.driver.OracleDriver");
   ```

2. 使用`System.setProperty("jdbc.drivers", "com.mysql.cj.jdbc.driver")`
   或者 ：`java -Djdbc.drivers=com.mysql.cj.jdbc.driver:oracle.jdbc.driver.OracleDriver`

### 2. 连接数据库

```java
// jdbc数据库连接
String url = "jdbc:mysql://localhost:3306/test?useUnicode=true&useSSL=true&characterEncoding=utf8";
String user = "root";
String password = "密码";
Connection conn = DriverManager.getConnection(url, user, password);
```

### 3. 获取statement对象

```java
// 获取常规的Statement对象
Statement stat = conn.createStatement();
// statement的几种使用
stat.executeQuery(sql); // 执行查询操作，返回的对象是一个结果集
stat.executeUpdate(sql);  // 执行增删改操作，返回操作影响的行数
stat.execute(sql);  // 所有数据库操作都可执行，第一个执行结果是结果集就返回true，反之返回false，getResultSet()或者getUpdateCount()获取第一个执行结果

// 获取预备语句PrepareStatement
PreparedStatement stat = conn.prepareStatement(querySql);
// 预备语句的使用
// 预备语句就是提前设计好缺口，可以根据程序传入的参数或者设置好的参数执行相应的查询操作
String querySQL = ”select name, studentid, register_date" + 
    " from student" + 
    " where name = ?";   // 其中的?号就是提前预备号的缺口
// 为了使用这些预定的缺口，需要使用set方法将变量绑定到实际的值上来
stat.setString(1, "变量值");   // 第一个参数的数字代表预备变量的编号，第二个参数代表的是变量对应的值
stat.clearParameters();  // 清除设置的变量信息
// setXxx(); 根据变量类型不同选择不同的设置变量方法名，有Int、Double、Date等
ResultSet result = stat.executeQuery();  // 执行预定语句的查询操作并返回结果集
Int affectedRows = stat.executeUpdate();  // 执行更新操作，返回影响的行数
```

### 4. 获取结果集对象/操作影响行数

```java
// 获取查询结果集对象
ResultSet result = stat.executeQuery(sql);   // 返回的是结果集


// 结果集相关的一些操作
result.next();  // 查看下一行数据

// 获取数据有两种传参方式，一种是传递列名，一种是传递列所在的位置
result.getObject();  // 不确定列数据类型的时候使用这个方法获取列内容
result.getXxx();  // 其中的Xxx可以是Int、Double、Date、String等数据类型
result.findColumn(String columnName); // 根据指定的列名获取列的序号
result.updateObject();  // 返回或者更新列的值，并将值转换成指定的类型
```

### 5. 关闭连接

connection、resultset和statement对象属于资源对象，在使用结束之后需要关闭

```java
// 手动关闭执行close方法即可
result.close();
stat.close();
conn.close();

// 自动关闭，利用try-resource资源语句。自动关闭释放资源
try( Connection conn = DriverManager.getConnection();)
{
    ....
}
```



## Connection、ResultSet、Statement管理

每一个connection对象可以创建一个或者多个Statement对象

同一个Statement对象可以用于多个不想管的命令和查询，但是每个Statement对象只能有一个打开的ResultSet对象

如果需要执行多个查询操作并且都需要查看结果，需要创建多个Statement对象来执行操作

<font color='red'>实际上，并不需要同时处理多个结果集，如果结果集相互关联，可以先使用联表查询语句将结果查询到一个表中，而且执行一个组合查询的效率比多个结果集要更高效</font>

这三个对象都属于资源对象。因此，在使用完后就需要手动关闭，或者利用try-resource语句来达成自动关闭资源的操作

## 查询操作

### 读写LOB

除了常规的数据类型之外，许多的数据库还可以存储大对象，例如图片或者其他数据

BLOB：二进制大对象大

CLOB：字符型大对象

大对象的读取都是通过结果集中的方法来进行的

```java
// 读取二进制大对象
Blob blob= result.getBlob();  // 也是两种方法，一种传入列序号，一种传入列标签
// blob对象下面有getBinaryStream()读取数据，setBinaryStream()创建写入数据，getBytes()获取指定范围数据

// 利用connection对象可以创建Blob和Clob对象
conn.createBlob();
conn.cretaeClob();

// 读取字符型大对象
Clob clob = result。getClob();  // 合上面的二进制大对象类似的传参数方法
// clob对象下有getCharacterStream()和setCharacterStream()，还有一个getSubString()获取子字符串
```

### SQL转义

