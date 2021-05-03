---
title: MySQL安装和基本使用
author: lijiabao
date: 2021-04-14 23:30
categories: ["MySQL", "Database"]
tags: ["MySQL", "Database"]
---

简要介绍如何安装MySQL以及一些简单的配置和使用

## MySQL安装配置

安装分为压缩版和安装包版本安装

压缩包下载：[下载地址](https://downloads.mysql.com/archives/community/)

安装包下载：[下载地址](https://dev.mysql.com/downloads/)

### 安装

#### 压缩版

1. 将下载好的压缩包解压，然后将其放到你想要安装的安装目录下面，如果为了方便使用命令，可以将这个安装路径下的bin目录添加到环境变量中

2. 在安装目录下面添加一个MySQL配置文件`my.ini`，然后加入下面的内容，最好不要有注释内容：

   ```ini
   [mysqld]
   # 设置服务端使⽤用的字符集为utf-8
   character-set-server=utf8
   # 绑定IPv4地址
   bind-address = 0.0.0.0
   # 设置mysql的端⼝口号
   port = 3306
   # 设置mysql的安装⽬目录
   basedir=D:\mysql-8.0.12-winx64
   # 设置mysql数据库的数据的存放⽬目录
   datadir=D:\mysql-8.0.12-winx64\data
   # 允许最⼤大连接数
   max_connections=2000
   # 创建新表时将使⽤用的默认存储引擎
   default-storage-engine=INNODB
   # 设置mysql以及数据库的默认编码
   [mysql]
   default-character-set=utf8
   [mysql.server]
   default-character-set=utf8
   # 设置客户端默认字符集
   [client]
   default-character-set=utf8
   ```

3. 数据data目录的初始化，这个data目录就是上述配置文件的datadir所指：

   ```shell
   # unix和linux下的初始化
   mkdir data-file-name
   chown mysql:mysql data-file-name  # mysql是服务器所在的用户和所属组
   chmod 750 data-file-name
   mysqld --initialize --user=mysql  # 设置用户所属为mysql
   # 如果使用--initialize-insecure,就是不设置密码的初始化
   
   # Windows下的初始化
   mysqld --initialize --console
   mysqld --initialize-insecure --console
   ```

4. 将mysql安装为一个Windows服务：`mysqld --install`

5. 启动mysql服务`net start mysql`

6. 初始化后的操作

   ```shell
   mysql -u root -p   # 使用了initialize需要输入密码登陆，密码在data数据的err后缀的文件中保存
   mysqld -u root --skip-password  # 使用insecure初始化，可以不用密码登陆
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password'
   # 使用上面的这条命令修改root密码
   
   # 创建用户
   CREATE USER 'root'@'127.0.0.1' IDENTIFIED BY 'root-password'  # 只能本地访问
   CREATE USER 'root'@'::1' IDENTIFIED BY 'root-password'  # ::1指特定的主机可以访问
   
   # 测试MySQL的服务器状态
   mysqlshow -u root mysql -p  # 查看mysql数据库下的内容，需要输入用户密码
   mysqladmin version status proc
   mysql test
   ```

   

#### 安装包版

双击安装包文件按照图形界面，并且结合自己的需要安装需要额功能，安装好了之后可哟条件环境变量或者直接在安装的过程中勾选添加到环境变量的选项也可。安装好之后启动服务

### 使用

#### 连接MySQL服务和退出

```shell
mysql -h host -u root -p  # 指定主机地址和用户，并输入密码即可登陆连接
mysql -u root -p   # 登陆本地MySQL服务，需要输入密码之后连接
mysql  # 不需要其他选项的时候，如果设置了匿名登陆是可以成功的
## 连接好了之后就会进入mysql，此时输入quit即可退出
mysql>quit  # 退出，unix下可以按ctrl+D退出
```

#### MySQL查询命令

```sql
/* 下面的命令都是mysql内的命令，只能连接了mysql服务之后才能使用*/
SELECT VERSION(), CURRENT_DATE;  # 查询当前mysql版本号和当前日期，忽略大小写
SELECT SIN(PI()/4), (4+1)*5;   # 分隔符号是分号，和Java的语句结束符是一样的
/* 如果每条语句短的，可以在一行写多条语句*/
SELECT VERSION(); SELECT NOW();  # 分号是MySQL的语句结束符
/* 当语句很长的时候，也可以多行书写一条语句*/
SELECT
USER()
,
CURRENT_DATE;
/* 类似上面的多行写一条语句也是可以的，可以更清晰*/
/* 如果不想继续查询命令了，可以使用 \c 取消命令*/
SELECT USER() \c  #就可以取消这条查询命令了
/* 如果你输入命令之后回车没有反应，可能正在查询
# 如果出现  mysql>  说明又可以输入下面一条命令了
# 出现 -、'、`、"> 这种符号后面跟着一个> 代表命令缺少配对的符号，正在等待再次输入*/
```

#### 创建和使用数据库

##### database操作相关命令：

```sql
/* 展示现有的数据库*/
SHOW DATABASES;
/* 进入使用数据库,use命令和quit一样不需要分号终止符*/
USE databse-name  # database-name就是上一步操作中查看到的数据库名字
/* 授权数据库访问权限*/
GRANT ALL ON name.* TO 'your-mysql-name'@'your-client-host';
/* 上述命令就是将数据库name下的所有资源访问权限给到用户和主机如上的用户*/

/* 创建用户数据库*/
CREATE DATABSE database_name;  # 创建名为database_name的数据库
/* 进入该数据库*/
USE database_name  # 开始使用数据库database_name
/* 直接从命令行进入数据库的命令*/
mysql -h host -u root -p database_name # 直接访问database_name数据库
/* 如果想直接输入密码，在-p之后不接空格直接输入密码
# 查看当前使用的数据库*/
SELECT DATABASE();
```

##### 表格table相关命令

```sql
/* 创建table表格数据，也就是数据库地下的数据表*/
CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
	species VARCAHR(20), sex CHAR(1), birth DATE, death DATE);
	
CREATE TABLE pet (
    id INT(4) AUTO_INCREMENT COMMENT '宠物id',
	name VARCHAR(20) NOT NULL COMMENT '宠物名称',
    sex char(1) DEFAULT 'f' COMMENT '宠物性别',
    ....
    death DATE COMMENT '宠物死亡日期',
    PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8
/* 上面这条命令创建了一个有6行的数据表，分别是name，owner，species，sex，birth和death
# VARCHAR是数据类型，可变字符，CAHR是字符数据，DATE是日期数据类型
# 显示数据库下的表格名*/
SHOW TABLES;
/* 查看表格的具体属性和特征*/
DESCRIBE pet;
/* 删除表格*/ 
DROP TABLE tablename;
```

##### 加载数据到表格

```sql
/* 加载文件中的数据到数据库的表格中*/
LOAD DATA LOCAL INFILE '/path/file-name.txt' INTO TABLE pet;
/* Windows下以使用\r\n为行终止符创建文件时，需要使用下面的语句*/
LOAD DATA LOCAL INFILE '/path/filename.txt' INTO TABLE pet LINES TERMINATED BY '\r\n';
/* 插入数据到表格中*/
INSERT INTO pet VALUES ('dog', 'dd', 'ddd', 'f', '1999-01-01', NULL); 
```

##### 从表格中取出数据

**表格中提取数据：**

```sql
SELECT what_to_select
FROM which_table
WHERE condition_to_satisfy

/*### 上述命令解释
# what_to_select挑选你想查看的内容，比如*代表所有
# which_table代表哪一个表格
# condition_to_satisfy指定一个或者多个满足提取数据的条件*/

/* 取出所有数据*/
SELECT * FROM pet;

/* 取出特定的行*/
SELECT * FROM pet WHERE name = 'dog';
SELECT * FROM pet WHERE birth >= '1998-1-1';
/* 在条件判断的时候支持使用AND和OR逻辑符号，表示且和非*/
SELECT * FROM pet WHERE species = 'snake' OR species = 'bird';
SELECT * FROM pet WHERE species = 'snake' AND sex = 'f';
/* AND的优先级更高一点
# 两者支持混合使用，可以使用括号包裹使语句更加明晰*/
SELECT * FROM pet WHERE (species = 'snake' AND sex = 'm')
OR (species = 'bird' AND sex = 'f');

/* 取出特定的列*/
SELECT name, birth FROM pet;
SELECT owner FROM pet;
/* 取出无重复的单独的某一列内容，也就是查看无重复的内容*/
SELECT DISTINCT owner FROM pet;
/* 添加筛选条件的列选取*/
SELECT name, species, birth FROM pet
WHERE species = 'dog' OR species = 'cat';
```

**给行排序：**

```sql
/* 按照指定的列进行排序，默认是升序*/
SELECT name, birth FROM pet ORDER BY birth;
/* 降序排列*/
SELECT name, birth FROM pet ORDER BY birth DESC;
/* 多列排序，还可以为多列指定不同的排序方向*/
SELECT name, species, birth FROM pet ORDER BY species, birth DESC;
```

**日期计算：**

```sql
/* 选择name、birth、当前日期CURDATE()并以年为单位计算宠物的年龄并作为年龄列选择*/
SELECT name, birth ,CURDATE(),
TIMESTAMPDIFF(YEAR, birth, CURDATE()) AS age   -- 将TIMESTAMPDIFF操作之后的结果作为age列的内容
FROM pet;
/* 上述命令会选择出四列，name，birth，当前日期和年龄age*/

/* 上述命令增加了一个按照名字排序的操作*/
SELECT name, birth ,CURDATE(),
TIMESTAMPDIFF(YEAR, birth, CURDATE()) AS age 
FROM pet ORDER BY name;

/* 添加一个判断操作并对判断之后满足条件的结果排序*/
SELECT name, birth, death, TIMESTAMPDIFF(YEAR, birth, death) AS age
FROM pet WHERE death IS NOT NULL ORDER BY age;

/* 
date的一些常见操作，提取字段中的时间日期相关的内容：
YEAR()获取年
MONTH()获取月
DAYOFMONTH()获取每月中所处的天数
DATE_ADD(原日期, 日期间隔)示例：DATE_ADD(CURDATE(), INTERVAL 1 MONTH)月份加一
*/
SELECT name, birth MONTH(birth) FROM pet;
SELECT name, birth FROM pet WHERE MONTH(birth) = 5;  -- 找出满足出生日期是五月的宠物

/* 找出出生日期比现在日期的月份晚一个月的宠物*/
SELECT name, birth FROM pet
WHERE MONTH(birth) = MONTH(DATE_ADD(CURDATE(), INTERVAL 1 MONTH));
/* 另外一种实现方法*/ 
SELECT name, birth FROM pet
WHERE MONTH(birth) = MOD(MONTH(CURDATE()), 12) + 1;
/* MOD(some, int)，取余运算，上面的命令可以让日期为12的情况自动转换到下一个月，也就是1月
如果日期计算中出现不合理的日期，会出现警告信息
警告信息查看命令如下
*/
SHOW WARNINGS;  -- 查看警告信息
```

**空值的处理：**

```sql
/* 空值测试：
IS NULL, IS NOT NULL  判断操作，返回1或0
对于和NULL的运算符操作，都不生效，只会返回NULL值
排序操作，升序的时候NULL在最前面，降序NULL在最后面
0,'',1都不是NULL
*/
SELECT 1 IS NULL, 1 IS NOT NULL;  -- 测试1是不是空值NULL
```

**模式匹配：（类似正则表达式）**

```sql
/* Mysql也提供了标准的模式匹配，和UNIX的vi、grep和sed的类似
_匹配任意单个字符，%匹配任意数量的字符（包含零字符）
SQL的模式匹配默认大小写敏感，使用模式匹配，使用LIKE和NOT LIKE比较符，而不是使用>< =之类的
*/
SELECT * FROM pet WHERE name LIKE 'b%';  -- 匹配任意以b开头的名字
SELECT * FROM pet WHERE name LIKE '%fy';  -- 匹配以fy结尾的名字
SELECT * FROM pet WHERE name LIKE '%w%';  -- 匹配包含w字符的名字
SELECT * FROM pet WHERE name LIKE '_____';  -- 准确匹配包含5个字符的名字，也就是使用五个_ 符号

/* MySQL还可以使用扩展的正则表达式语法
正则匹配的匹配模式：
1. c：大小写敏感
2. i：大小写不敏感
3. m：多行模式匹配，识别行终止符
4. n：符号.匹配行终止符
5. u；仅unix行终止符

当需要使用正则表达式匹配的时候，使用下面的几个命令：

REGEXP_LIKE()函数：匹配字符串。
语法：REGEXP_LIKE(expr, pat[, match_type])
示例：REGEXP_LIKE('Michael!', '.*');REGEXP_LIKE('a', '^[a-d]');
示例：REGEXP_LIKE('abc', 'ABC', 'c');大小写敏感匹配 REGEXP_LIKE('a', BINARY 'A');二进制数匹配

REGEXP_REPLACE()：用指定的字符串替换匹配的子串。
REPLACE的语法：REGEXP_REPLACE(expr, pat, repl[, pos[, occurrence[, match_type]]])
示例：REGEXP_REPLACE('a b c', 'b', 'X'); REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X', 1, 3);

REGEXP_SUBSTR()：返回匹配正则表达式的子串
子串匹配的语法：REGEXP_SUBSTR(expr, pat[, pos[, occurrence[, match_type]]])
示例：REGEXP_SUBSTR('abc def ghi', '[a-z]+'); REGEXP_SUBSTR('abc def ghi', '[a-z]+', 1, 3);

REGEXP_INSTR()：匹配的正则表达式字符串的开始索引
语法：REGEXP_INSTR(expr, pat[, pos[, occurrence[, return_option[, match_type]]]])
示例：REGEXP_INSTR('dog cat dog', 'dog', 2); REGEXP_INSTR('dog cat dog', 'dog');

NOT REGEXP 正则匹配的反向，不匹配正则。示例：expr NOT REGEXP pat, expr NOT RLIKE pat
REGEXP 是否字符串匹配正则表达式。示例：expr REGEXP pat, expr RLIKE pat
RLIKE 是否正则表达式匹配字符串。示例：expr REGEXP pat, expr RLIKE pat
*/
SELECT * FROM pet WHERE REGEXP_LIKE(name, '^.{5}$');  -- 使用正则表达式的条件判断
```

**行计数：**

```sql
/* sql中计数使用COUNT(*)*/ 
SELECT COUNT(*) FROM pet;  -- 给整个pet表进行计数，计算多少行数据
SELECT owner, COUNT(*) FROM pet GROUP BY owner;  -- 按照owner分组后进行COUNT计数
SELECT species, COUNT(*) FROM pet GROUP BY species;
SELECT sex, COUNT(*) FROM pet GROUP BY sex;
/* 上述分组操作在你开启了ONLY_FULL_GROUP_BY的sql模式的时候，如果没有指定GROUP BY操作，会报错
但是如果没有开启并不会报错
*/
SET sql_mode = 'ONLY_FULL_GROUP_BY';   -- 设置完全分组匹配的sql模式

/* 其他操作组合而成的一些组合操作*/
SELECT species, sex, COUNT(*) FROM pet
WHERE sex IS NOT NULL
GROUP BY species, sex;  -- 找出性别不为空的种族和性别以及行数

SELECT sepcies, sex, COUNT(*) FROM pet
WHERE species = 'dog' OR species = 'cat'
GROUP BY species, sex;  -- 找出品种为狗或者猫的数据项和行数
```



