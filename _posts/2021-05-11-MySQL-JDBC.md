---
title: MySQL和idea杂记
author: lijiabao
date: 2021-05-11 12:30
categories: 杂记
tags: 杂记
---

## MySQL

MySQL在Windows服务的关闭和重启：

```powershell
net stop mysql  # 关闭MySQL服务
net start mysql  # 开启MySQL服务
```

MySQL的时区设置：

临时修改：在MySQL命令行中输入`set time_zone='+08:00'`，这个操作MySQL关闭之后就失效

永久修改，在配置文件的[mysqld]项相面添加一项：`default-time-zone='+08:00'`，修改配置文件之后，重启MySQL生效

查看时区设置：`show variables like '%time_zone';`，如果显示SYSTEM说明并未对时区进行修改设置



## IDEA

对于IDEA的控制台乱码问题，在VM option设置项中添加`-Dfile.encoding=UTF-8`设置编码为utf-8



## JDBC

### JDBC驱动安装

1. 首先需要安装数据库的驱动，也就是你需要使用的数据库

2. 根据你使用的数据库去找寻JDBC的驱动，比如MySQL的驱动就是connector/j，需要去数据库的官网下载：https://dev.mysql.com/downloads/connector/j/

3. 下载好之后，驱动的使用方法：进入Eclipse或者IDEA，在项目文件中新建lib文件夹，将驱动的jar包放入到文件夹，并将这个路径加入到library。或者使用maven的话，直接将依赖项添加到pom.xml

   ```xml
   <!-- MySQL的JDBC数据库驱动 -->
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>8.0.19</version>
   </dependency>
   ```

### 简单连接使用

1. jdbc驱动加载：`Class.forName('com.mysql.cj.jdbc.driver');`
2. 使用DriverManager获取数据库连接`Connection conn = DriverManager.getConnection(url, user, password);`
3. 利用连接创建statement对象：`Statement stat = conn.createStatement();`
4. 利用statement对象执行数据库操作：`ResultSet result = stat.executeQuery(sql);`
5. 结果处理，结果防止在一个结果集对象中，可以利用迭代器来获取所有结果，`resutl.next();`获取结果
6. 关闭连接，将result，stat和conn都关闭

### 数据库连接

#### MySQL

驱动class：`com.mysql.cj.jdbc.Driver`或者`com.mysql.jdbc.Driver`

连接语句：`jdbs:mysql://localhost:3306/database-name`

增加配置项的连接语句：`"jdbc:mysql://localhost:3306/db_admin?serverTimezone=Hongkong&useUnicode=true&characterEncoding=utf8&useSSL=false"`

#### Oracle数据库

驱动class：`oracle.jdbc.driver.OracleDriver`

连接语句url：`jdbc:oracle:thin:@127.0.0.1:1521:db-name`

#### PostgreSQL数据库

驱动class：`org.postgresql.Driver`

连接语句url：`jdbc.postgresql://localhost/db-name`

#### SQLServer2000数据库

驱动class：`com.microsoft.jdbc.sqlserver.SQLSeverDriver`

连接语句url：`jdbc.microsoft.sqlserver://localhost:1433;DatabaseName=db-name`

SQLServer2005数据库

驱动class：`com.microsoft.sqlserver.jdbc.SQLServerDriver`

连接语句url：`jdbc.sqlserver://localhost:1433;DatabaseName=db-name`

#### DB2数据库

驱动class：`com.ibm.db2.jcc.DB2Driver`

连接语句url：`jdbc.db2://127.0.0.1:50000/db-name`

