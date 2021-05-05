---
title: MySQL-CRUD
author: lijiabao
date: 2021-05-04 20:30
categories: ["MySQL", "Database"]
tags: ["MySQL", "Database"]
---


介绍mysql常用的增删改查操作


# MySQL基础使用

## DML（数据库管理语言）

### show命令使用

```sql
-- 查看show命令的帮助信息
help show  -- 可以查看到show的帮助文档

-- 展示所有数据库
show databases;

-- 展示所有数据表
show tables;

-- 展示创建指定数据表或者数据库的MySQL语句
show create database `database-name`;
show create table `table-name`;

-- 展示服务器状态信息
show status;

-- 展示警告信息
show warnings;

-- 展示错误信息
show errors;

-- 展示授权用户
show grants;
```

### 数据库管理

```sql
create database if not exists `databse-name`;  -- 创建数据库

drop database `data-basename`;  -- 删除数据库
```

### 数据表管理

#### 创建表

```sql
/* 创建数据表的模板
create table if not exists `table-name` 
(
    `这里写字段名` data-type filed-property comment '注释内容写在这里',
    `字段1` int(4) not null auto_increment 
)
*/
create table if not exists `student` 
(
    id int(4) not null auto_increment,
    name varchar(10),
    register_date date,
    primary key (`id`)
)engine=INNODB default charset=utf8;

/*引擎engine类型：
1. InnoDB：可靠的事务处理引擎，不持支全文本搜索，保存在磁盘中
2. MEMORY：功能和MyISAM，不过数据存储在内存，速度快，适合临时表创建
3. MyISAM：性能极高的引擎，支持全文本搜索，不支持事务处理，保存在磁盘中
*/
```

#### 查看表信息

```sql
show tables;  -- 查看该数据库下的所有表

describe `table-name`;  -- 查看指定表的情况，列名和数据类型以及索引等的情况
show columns from `table-name`;  -- 查看指定表的所有列情况
```

#### 删除和重命名表

```sql
-- 删除表：drop table `table-name`;
drop table student;  -- 该操作永久删除，没有确认操作

-- 重命名表：rename table `old-name` to `new-name`;
rename table student to stu_table;
rename table student to stu_table, subject to sub_table;  -- 重命名多表
```

#### 修改表

对于表的定义修改更新，使用`alter table`命令，但是理想情况下，表中存储数据之后，就不应该再更新这个表，表的涉及需要花费大量的时间考虑，以便后期不用再对表进行大的改动。

```sql
/* 表修改的模板：
alter table 
*/

/*
添加一列:
alter table `tablename`
add column-name datatype;
*/
alter table subject add select_date date;

/*
创建外键：
alter table `table-name` add constraint ’约束名'
foreign key (`列名`) references `reference-table-name` (`refe-列名`);
*/
alter table `studentid`
add constraint 'FK_studentid'  -- 外键名
foreign key (`studentid`) references student (`id`);
```

#### 外键管理

**外键：表示表中的一个字段被另外一个表中的字段引用**。外键的作用就是限制使用外键的表中的数据只能插入引用表主键中存在的值。

> <font color='red'>注意：</font>外键关联的必须是表的主键，并且主键和外键的数据类型必须一致。外键不可以跨引擎

```sql

create table `student` 
(
    id int(4) not null auto_increment,
    name varchar(10),
    register_date date,
    primary key (`id`)
)engine=INNODB default charset=utf8;

-- 外键约束创建方法一：创建表的时候创建
create table `subject`
(
    studentid int(4) auto_increment,
    subjectname varchar(10) default 'Java',
    constraint 'FK_studentid'  -- 外键约束名
    foreign key (`studentid`) references student (`id`)
);
-- 外键约束二：修改已有的表添加约束
alter table `studentid`
add constraint 'FK_studentid'  -- 外键约束名
foreign key (`studentid`) references student (`id`);

---- 删除外键约束 ----
alter table `table-name` drop foreign key '外键约束名'

---- 删除外键 ----
alter table `table-name` drop `foreign-key-column`
```

### 插入数据

```sql
/*
插入数据：
1. 插入完整的行：
insert into `table-name`
values (`和前面column对应的值`);
2. 插入行的一部分：
这种方法是可以省略列插入数据的，当满足下面要求时：
列的定义中允许出现null值
定义的时候该列设置了默认值
insert into `table-name` (`column-name`) 
values (`和前面column对应的值`);
3. 插入多行：下面的多行插入比多个插入语句插入快，性能更优
insert into `table-name` (`列名`, [`列名`])
values (`列对应的值`),(`另外一行列对应的值`), ....(`列对应的值`);
4. 插入查询的结果：
使用这种插入方法可以将其他表中的数据插入到你想插入数据的表中指定的列
select into `table-name`('列名')
select '对应的需要插入数据的列' from 'table';
*/

----- 1. 插入整行-----
insert into student values (1,'jiabao','2021-05-02', '20290025');
-- 不支持省略某一列或者某些列

----- 2. 插入一部分 -----
insert into student (name, register_date, studentid)
values('jiabao', '2021-05-02', '20290012')

----- 3. 插入多行 -----
insert into student (name)
values ('jiabao'),('baojia'),('帅哥');
-- 使用多条语句插入
insert into student (name) values('Lijia');
insert into student (reguster_date) values ('2025-05-02');

----- 4. 插入查询的数据 -----
insert into student (name, register_date, studentid)
select name, register_date, studentid from student; 
-- 从其他的表中查找数据项来插入数据
insert into student (name)
select studentname from subject;
```

### 更新数据

```sql
/*
更新数据模板：
update `table-name` set column-name = '修改后的列值',[column-name = '' ...] where condition;
如果不加上where语句判断，将会将所有的数据都更新为这个值
*/
update student set name = 'lijiabao' where name = 'jiabo';
```

### 删除数据

```sql
/*
删除数据的模板：
使用delete：
delete from `table-name` where condition
使用truncate：
truncate table `table-name`  删除所有的行，速度更快，本质就是删除原理来的表并且重新创建一个空表
*/
-- 不删除表本身，只删除特定的行或者所有的行
delete from student where name = 'jiabao';

-- 先删除表，然后再创建一个空表
truncate table student;  
```

### 更新数据和删除数据的原则

- 如果不打算更新或者删除所有的行的数据，一定需要带上where语句
- 保证每个表都有主键，尽可能像where子句那样使用它
- 在使用更新或者删除操作的时候，先使用select进行测试，保证是选中的自己想要更新或者删除的数据，防止where子句书写不正确，更新或者删除了错误的数据
- 使用强制实施引用完整性的数据库，这样MySQL将不允许删除具有与其他表相关联的数据的行
- <font color='red'>MySQL没有撤销操作，一旦删除或者更新数据错误，没法恢复</font>

## DQL（数据库查询语言）

### 基本语法和简单查询

```sql
----- 基本查询的语法 -----
/*
select *（全部列）或者指定列名,[...列名] from `table-name` as 别名1
left join/right join/inner join `another-table` as 别名2
on 别名1.name = 别名2.name
where condition
group by 列名,[...列名]
having condition
order by 列名,[...列名]
limit num1, num2
其中的num1表示从哪一行开始，num2表示输出多少行
*/

----- 1. 检索单列/多列 -----
select name from student;  -- 单列
select name,studentid from student; -- 多列

----- 2. 检索所有列 -----
select * from student;

----- 3. 检索满足某些条件的列 -----
-- 查询满足两个要求的所有行列内容 
select * from student
where name = 'lijiabao' and register_date > '2020-09-01'; 
-- 查询满足两个要求中一个的行列内容
select * from student
where name = 'lijiabao' or register_date > '2021-05-01'

----- 4. 检索不重复的行 -----
select distinct name from student;

----- 5. 限制行的输出 -----
-- 使用limit子句，只有一个数字表示限制输出行数
-- 两个数字的表示从第几行开始限制多少行的输出
-- 限制，只输出前五行
select * from student limit 5;
-- 限制，从第二行开始输出五行
select * from student limit 2, 5;

----- 6. 使用完全限定的表名 -----
select student.name from student;
-- 下面的限定是将查询限定到test数据库的student表中，然后再去限定在student表中查询name列
select student.name from test.student;
```

### 排序数据

```sql
/*
order by 子句是用来对某一列或者某些列进行排序
*/
----- 1. 根据单列排序 -----
select * from test.student
order by register_date desc; -- desc表示降序
select * from test.student
order by register_date asc; -- asc表示升序,默认的排序是升序排序

----- 2. 根据多列排序 -----
select * from test.student
order by register_date, studentid;

----- 3. 指定排序的方向 -----
-- desc和asc两种，升序和降序
-- 多列排序时，可以分别为每一列指定排序方向
select * from student
order by name asc, register_date desc;
```

### 过滤数据和模糊查询

```sql
------- where 语句过滤数据 -------
-- 查找数据中name为jiabao的行
select * from student
where name = 'jiabao';

/*
where子句的操作符：
比较操作符：
<>不等于, >大于, <小于, =等于, !=不等于, <=小于等于, >=大于等于, between a and b 介于a和b之间

逻辑操作符：
and与, or或, not非
and的优先级要高于or

模糊操作符：
like：like支持的通配符,name like 'li%';
（%匹配0个，一个或者多个字符，_匹配一个或者单个字符）
between a and b：介于 between 1 and 8;
in：在某个集合中，name in ('jiabao', 'lijiabao');
*/

----- 1. 简单的过滤查询 -----
select * from student where name='jiabao';


----- 2. 多条件合并过滤数据 -----
-- 或操作符的使用
select * from student
where name = 'jiabao' or name = 'baojia';
-- 与操作符的使用
select * from student
where name = 'jiabao' and register_date = '2021-05-02';
-- 多个操作符混合使用
select * from student
where (name = 'jiabao' and register_date = '2021-05-02') or name = 'lijiabao';


----- 3. 不匹配查询 -----
-- 非空查询
select * from student
where register_date not null;
-- 不等于操作符使用
select * from student
where name != 'jiabao';


----- 4. null值检查
-- 空值查询
select * from student
where name is null;
-- 非空查询
select * from student
where name is null;


----- 5. 模糊查询 -----
-- 使用like的模糊查询
-- 通配符%表示匹配任意0个，一个或者多个字符，但是不能匹配null
-- 通配符_表示匹配任意一个字符
/*
通配符查询操作的优缺点和使用技巧：
1. 通配符功能性强大，但是其操作所需的时间更长，因此需要适度使用，如果其他的操作可以完成，不要使用通配符
2. 必须使用通配符时，尽量不要把通配符放在搜索模式的开始处，因为这样搜索的效率更低
3. 通配符匹配位置不对，可能查找的数据不合适
*/
select * from student
where name like 'li%'; -- 匹配以li开头的name
select * from student
where name like 'li_';  -- 匹配以li开头的后面只有一个字符的name

-- 使用in的模糊查询，指定条件范围，处于这个范围内都可匹配
/* 
使用in操作符的优点：
1. 在条件选项比较多的时候，in语法更加直观清楚
2. 使用in可以减少计算次序的管理
3. in操作的速率比or速率更快
4. in还可以包含select语句，使得能够更动态建立where子句
*/
select * from student
where name in ('jiabao', 'lijiabao');

-- 使用between的模糊范围查询
select * from student
where register_date between '2021-05-01' and '2021-06-01';


------ 6. 否定not查询 ------
-- not操作符号是用来否定后面跟着的条件的
select * from student
where name not in ('jiabao', 'lijiabao');
```

### 正则表达式搜索

正则表达式是用来匹配文本的特殊的串（字符集合），MySQL中的正则表达式仅仅只是正则表达式语言的一个子集，只支持完整正则表达式的一部分语法。

#### 正则表达式的几种用法

**正则匹配的匹配模式：**

1. c：大小写敏感
2. i：大小写不敏感
3. m：多行模式匹配，识别行终止符
4. n：符号.匹配行终止符
5. u；仅unix行终止符

当需要使用正则表达式匹配的时候，使用下面的几个命令：

- REGEXP_LIKE()函数：匹配字符串。
  语法：`REGEXP_LIKE(expr, pat[, match_type])`
  示例：`REGEXP_LIKE('Michael!', '.*');REGEXP_LIKE('a', '^[a-d]');`
  示例：`REGEXP_LIKE('abc', 'ABC', 'c');`大小写敏感匹配 `REGEXP_LIKE('a', BINARY 'A');`二进制数匹配

- REGEXP_REPLACE()：用指定的字符串替换匹配的子串。
  REPLACE的语法：`REGEXP_REPLACE(expr, pat, repl[, pos[, occurrence[, match_type]]])`
  示例：`REGEXP_REPLACE('a b c', 'b', 'X'); REGEXP_REPLACE('abc def ghi', '[a-z]+', 'X', 1, 3);`

- REGEXP_SUBSTR()：返回匹配正则表达式的子串
  子串匹配的语法：`REGEXP_SUBSTR(expr, pat[, pos[, occurrence[, match_type]]])`
  示例：`REGEXP_SUBSTR('abc def ghi', '[a-z]+'); REGEXP_SUBSTR('abc def ghi', '[a-z]+', 1, 3);`

- REGEXP_INSTR()：匹配的正则表达式字符串的开始索引
  语法：`REGEXP_INSTR(expr, pat[, pos[, occurrence[, return_option[, match_type]]]])`
  示例：`REGEXP_INSTR('dog cat dog', 'dog', 2); REGEXP_INSTR('dog cat dog', 'dog');`

- NOT REGEXP 正则匹配的反向，不匹配正则。示例：`expr NOT REGEXP pat, expr NOT RLIKE pat`
  REGEXP 是否字符串匹配正则表达式。示例：`expr REGEXP pat, expr RLIKE pat`
  RLIKE 是否正则表达式匹配字符串。示例：`expr REGEXP pat, expr RLIKE pat`

#### 正则和like的区别

**正则表达式可以匹配正则字符串出现过的列值所在的行，只要正则表达式的字符串出现过就可以匹配**

**而like需要匹配整个列值才会返回所在的行，需要完全匹配才可以**

#### 区分大小写

正则表达式正常并不区分大小写，因此为了区分大小写，应该使用BINARY关键字`where name regexp binary 'lijia.*'`，如果出现指令集错误`ERROR 3995 (HY000): Character set 'utf8_general_ci' cannot be used in conjunction with 'binary' in call to regexp_like.`，需要使用强制类型转换将列转换为binary类型：`where cast(name as binary) regexp binary 'lijia.*'`

#### MySQL正则的语法

mysql正则的转义符需要两个斜杠`\\`

| 正则语法 | 描述                              | 示例                                                |
| -------- | --------------------------------- | --------------------------------------------------- |
| `|`      | 或匹配，搜索多个内容其中一个即可  | 'table \| database'，匹配table或者database都可      |
| `[]`     | 匹配列表中的其中一个字符          | '[123]'，匹配1，2，3其中一个即可                    |
| `[^]`    | 不匹配列表中的字符                | `[^123]`，不匹配123数字                             |
| `[a-b]`  | 匹配范围                          | '[0-9] \| [a-z]'，匹配0到9或者字母a到z              |
| `.`      | 匹配除了特殊字符之外的所有字符    | '.' 可以匹配任意单个字符，除了某些特殊字符          |
| `\\.`    | 匹配.符号                         | 'hello.world'不能匹配hello.world                    |
| `*`      | 表示匹配前面字符0次、一次或者多次 | 'a*b'可以匹配aab或者ab或者b                         |
| +        | 匹配一个或者多个                  | 'a+b'可以匹配ab，aab，aaab                          |
| ?        | 匹配0次或者一次                   | 'a?b'可以匹配b，ab                                  |
| {n,m}    | 匹配n次到m次                      | 'a{1,3}b'匹配ab,aab,aaab                            |
| {n}      | 匹配n次                           | 'a{3}b'匹配aaab                                     |
| {n,}     | 匹配最少n次                       | 'a{2,}b'匹配aab,aaab等                              |
| ^        | 文本的开始                        | `^abc`，匹配以abc开头的文本，在[]内部起到否定的作用 |
| $        | 文本的结尾                        | `abc$`，匹配以abc结尾的文本                         |

| 字符类表达式 | 描述                                                  |
| ------------ | ----------------------------------------------------- |
| `[:alnum:]`  | 匹配任意数字和字母，和`[0-9a-zA-Z]`效果一样           |
| `[:digit:]`  | 匹配任意数字，和`[0-9]`类似                           |
| `[:alpha:]`  | 匹配任意字母，同`[a-zA-Z]`                            |
| `[:blank:]`  | 匹配空格和制表符，同`\\t`                             |
| `[:lower:]`  | 匹配任意小写字母，`[a-z]`                             |
| `[:upper:]`  | 匹配任意大写字母，`[A-Z]`                             |
| `[:print:]`  | 任意可打印字符                                        |
| `[:graph:]`  | 同`[:print:]`但是不包含空格                           |
| `[:cntrl:]`  | ascii空值字符，（0-31和127的ascii字符）               |
| `[:space:]`  | 包含空格在内的任意空白字符，`\\t``\\v``\\n``\\f``\\r` |
| `[:xdigit:]` | 任意16进制数字                                        |
| `[:punct:`]  | 即不在`[:alnum:]`也不在`[:cntrl:]`中的任意字符        |
| `[[:<:]]`    | 词的开始                                              |
| `[[:>:]]`    | 词的结尾                                              |

### 字段处理

#### concate函数拼接字段

`concate(string,...string)`：传入多个字符串参数，然后这个函数将字符串拼接出来

```sql
-- as是将某一列数据起一个别名输出
select concat(name,studentid) as all_info from student; -- 拼接name列字段和studentid字段
```

#### trim函数删减空格

```sql
-- rtrim删除字符串右边的空格，ltrim删除左边的空格
select concat(rtrim(name), rtrim(studentid)) from student;
```

### 算数运算

直接在列的选取时，可以直接对列进行算数运算

```sql
-- 列的基本数学运算
select id*id as id from student;
select id+id as id from student;
select id/id as id from student;
select id-id as id from student;
```

### 函数使用

函数主要有四类：

- 文本处理函数
- 数值函数
- 时间和日期函数
- 系统函数

#### 文本处理函数

查看帮助信息：`help func_name`即可查看函数的说明文档

| 函数          | 描述                                 |
| ------------- | ------------------------------------ |
| `left()`      | 返回串最左边的字符，`left(str, num)` |
| `length()`    | 返回串的长度                         |
| `locate()`    | 找出串的一个子串                     |
| `lower()`     | 将串转换为小写                       |
| `upper()`     | 将串转换为大写                       |
| `ltrim()`     | 删除串左边的空格                     |
| `rtrim()`     | 删除串右边的空格                     |
| `right()`     | 返回串右边的字符                     |
| `soundex()`   | 返回串的soundex值                    |
| `substring()` | 返回字符串的一个字串                 |

#### 数值函数

| 函数     | 描述                                     |
| -------- | ---------------------------------------- |
| `abs()`  | 绝对值                                   |
| `cos()`  | 返回角度的余弦值                         |
| `sin()`  | 返回角度的正弦值                         |
| `tan()`  | 返回角度的正切值                         |
| `sqrt()` | sqrt(num)，返回num的平方根               |
| `exp()`  | exp(num)，返回e的num次方                 |
| `mod()`  | mod(num1, num2)，取余，还可num1 mod num2 |
| `pi()`   | 圆周率的值                               |
| `rand()` | 返回一个随机数                           |
| `asin()` | 得出arcsin值                             |

#### 时间和日期函数

| 函數            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `adddate()`     | 增加一个日期(天，周，月，年等)默认添加天，adddate('2021-05-03', interval 2 week)，adddate('2020-05-03', 14) |
| `addtime()`     | 增加一个时间两个参数都是时间表达式，addtime('12:25:22', 5)，addtime('12:25:22', '1:34:38') |
| `curdate()`     | 返回当前的日期                                               |
| `date()`        | 返回日期时间的日期部分                                       |
| `datediff()`    | 返回两个日期之间的差值                                       |
| `date_add()`    | 日期运算函数                                                 |
| `date_sub()`    | 日期计算相减的函数                                           |
| `date_format()` | 日期格式化函数                                               |
| `day()`         | 返回时间日期中的天数                                         |
| `dayofweek()`   | 返回日期处于一周的第几天                                     |
| `hour()`        | 返回小时数                                                   |
| `minute()`      | 返回分钟数                                                   |
| `month()`       | 返回月份数                                                   |
| `now()`         | 返回当前的时间和日期                                         |
| `second()`      | 返回日期的秒数                                               |
| `time()`        | 返回日期时间的时间部分                                       |
| `year()`        | 返回日期的年份                                               |

<font color='red'>注意：</font>MySQL的日期格式，日期必须为格式`yyyy-mm-dd`，最好总是使用四位数字的年份

示例：`select * from student where date(register_date) = '2021-05-02'`

### 数据分组查询

数据分组和聚合是两个经常搭配使用的操作，分组就是把数据分成多个逻辑组，便于后续使用聚合函数聚合计算。

```sql
/*
分组语法：
使用group by子句来进行分组操作
group by name就是以name列来进行分组操作
*/
------ 分组的例子 ------
select name, count(distinct studentid)
from student
group by name;

-- group by 子句可以进行多列分组
select name, count(distinct studentid)
from student
group by name id;

-- 分组子句必须位于where子句后面，order by 子句的前面
select name, count(distinct studentid) as num
from student
group by name
order by num;

----- 过滤分组 ------
select name, count(*) as num
from student
group by name
having num > 2 -- 此处的having子句是用来进行分组过滤条件判断的语句
order by num;

/*
having子句和where子句的区别：
having是用在分组之后的条件判断，而where语句是用来在分组前进行条件判断的
*/
```

#### 分组和排序

分组：对行进行分组，但是输出的结果不一定是分组的顺序，<font color='red'>只能选择列或者表达式列来进行分组，而且必须使用每个选择的列表达式</font>，需要使用聚合函数
排序：一定是排序产生的结果，<font color='red'>任意的列都可以用来进行排序</font>，不一定需要使用聚合函数

<font color='red'>由于group by分组后的结果不一定是排序好的数据，因此最好使用order by再进行分组</font>

```sql
select name, register_date, count(*) as num
from student
group by name, register_date
having register_date > '2021-05-01';
```

> Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.student.name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by。<font color='red'>这个报错信息是因为MySQL默认设置了完全分组匹配，这个设置导致必须将选择列包括在分组子句才可</font>

```sql
select name, register_date, sum(id) as sum
from student
where register_date != '2021-05-01'
group by name, register_date
having name in ('jiabao', 'lijiabao', 'lijialin')
order by register_date
limit 2;
-- 上面的语句顺序就是标准的select语句的顺序
```

### 数据聚合查询

#### 聚合函数

| 函数               | 描述                       |
| ------------------ | -------------------------- |
| `avg()`            | 返回某一列的平均值         |
| `count()`          | 返回某一列的行数           |
| `count(distinct)`  | 返回某一列无重复的行数     |
| `max()`            | 返回某一列的最大值         |
| `min()`            | 返回某一列的最小值         |
| `sum()`            | 返回某一列的和             |
| `std()`            | 返回某一列的标准差         |
| `json_arrayagg()`  | 返回结果集为单个json列表   |
| `json_objectagg()` | 返回结果集为单个json对象   |
| `group_concat()`   | 将分组结果连接为一个字符串 |

**示例：**

```sql
------ 1. 平均值处理 ------
-- 查询某一列的平均值
select avg(id) as avg_id from student;
-- 查询满足某些条件的行或者列的平均值
select avg(id) as avg_id from student
where name = 'jiabao';
--- 平均值函数avg只能作用于单列，并且avg会忽略值为null的行

------ 2. 计数函数  -------
/*
两种count函数的使用方式：
1. count(*)对表中行的数目进行计数，不管是null还是非空都计数
2. count(`列名`)对表中指定列名的列进行计数，忽略null值
*/
select count(*) from student;
select count(name) from student;
```

#### 聚合不同的值

为了聚合不同的值，需要使用distinct参数

```sql
----- 聚集不同值的示例 ------
select avg(distinct id) from student;
select count(distinct name) from student;
select sum(distinct id) from student;
```

#### 聚合函数的结合使用

```sql
-- 聚合函数之间是可以混合搭配使用的
select count(*) as item_num,
min(id) as min_num,
max(id) as max_num,
sum(distinct id) as sum_num,
avg(id) as avg_num
from student;
```

### 联表查询

联结表是SQL最强大的功能之一，是很多查询操作可以进行的重要基础。

#### 关系表

通过某一个具有相同信息的列来进行表之间的相互关联

使用关系表的原因：

- 如果对某些相同的信息重复出现在不同的表内，会浪费时间和存储空间
- 这个共有的列改变只需要改动一次即可

使用外键就可以创建一个关系表，外键就是一个表中的一列数据，它引用了另外一个表的主键

**可伸缩性**：能够适应不断增减的工作量而不失败，设计良好的数据库或者应用程序将其称为可伸缩性好

#### 联表查询的好处

- 可以在一条select语句中查出多表中存储的数据
- 联结并不存在，只是根据查询需要进行表的连接共同查询数据

#### 简单联结创建

```sql
select name, subjectname, register_date
from student,subject
where subject.studentid = student.id
order by register_date;
-- 上述代码的where子句指示两个表之间的联结是通过id列和studentid来进行的
-- 上述代码为了更好的消除二义性，使用了完全限定表名
-- 对于列的 选取为了更好的消除二义性，也最好使用完全限表名，更安全不出错
```

联结两个表的时候，如果没有where子句来作为过滤条件的话，匹配的话每一行都会与另外一列的每一行进行匹配输出，这个时候输出的行数就是笛卡尔积。

使用了where子句过滤的话，就只会匹配那些给定条件的行，比如上面代码的联结条件。

**可以对表使用表别名来减少代码量，可以使用as语句或者也可以省略as，表的别名只在查询执行使用，并不会返回到客户机**

```sql
----- 表别名使用 -----
-- 使用as参数
select name, subjectname
from student as stu
inner join subject as sub
on sub.studentid = stu.id;

-- 省略as参数
select name, subjectname
from student stu
inner join subject sub
on sub.studentid = stu.id;
```

<font color='red'>注意：</font>对于联结查询来说，应该保证联结都有where子句，否则返回的数据会比想象的数据要多

### 联表查询的联结方式

#### 内联结

**等值联结**：基于两个表之间的相等测试，这种联结方式也称为内联结

```sql
-- 常见的内联结方式
select name, register_date, subjectname
from student as stu
inner join subject as sub
on stu.id = sub.studentid  -- 联结表用on子句判断
order by register_date;

-- ANSI SQL规范首选INNER JOIN依法，首选内联结方法
-- 尽管使用where子句定义联结比较简单，但是使用明确的联结语法能够确保不会忘记联结条件，有时候也会影响一定的性能
```

#### 左联结

```sql
-- 常见的内左联结方式
select name, register_date, subjectname
from student as stu
left join subject as sub
on stu.id = sub.studentid  -- 联结表用on子句判断
order by register_date;
```

#### 右联结

```sql
-- 常见的右联结方式
select name, register_date, subjectname
from student as stu
right join subject as sub
on stu.id = sub.studentid  -- 联结表用on子句判断
order by register_date;
```

#### 自联结

>  子联结和子查询有些类似，子联结通常作为外部语句代替从相同表中使用子查询语句检索，因为有时候处理联结远比处理子查询更快

```sql
-- 子联结查询
select p1.id, p1.name
from student as p1, student as p2
where p1.id = p2.id
and p2.register_date = '2021-05-02';
-- 使用表别名的原因：是因为不使用两个别名会出现二义性，不知道具体去使用哪一列数据来进行比较

-- 子查询方法实现
select id, name
from student
where id in 
(select id from student
where register_date = '2021-05-02');
```

#### 自然联结

自然联结排除多次出现，使得每个列只返回一次

事实上，我们所有用到的内部都是自然联结

```sql
--- 自然联结
select stu.*, sub.subjectname, gra.grade
from student as stu, subject as sub, grade as gra
where stu.id = sub.studentid
and sub.name = gra.studentname
and register_date = '2021-05-02';
```

#### 外部联结

外部联结包含了那些在相关表中没有关联的行，这一种类型的联结称为外部联结

```sql
-- 外部联结
select student.name, student.register_date, subject.subjectname
from student
left outer join subject
on student.id = subject.studentid;

-- 使用外部联结时需要使用left或者right来指定参考哪一边的表数据
```

#### 多表联结

```sql
-- 多表联结的例子
select name, register_date, subjectname, gradeNo
from student, subject, grade
where student.id = subject.studentid
and student.name = grade.name
and register_date > '2021-05-01'
order by register_date;

-- 另外一种等价的联结
select name, register_date, subjectname, gradeNo
from student stu
inner join subject sub
on student.id = subject.studentid
inner join grade as gra
on student.name = grade.name
where register_date > '2021-05-01'
order by register_date;
```

**<font color='red'>要点</font>：**为了找出某一个操作的最佳性能，应该多练习多找寻最佳的实践，并没有绝对正确的操作

> mysql的性能会受到操作类型、表中数据两、是否存在索引或键以及其他的一些条件的影响

#### 几种联结方式的区别

- 左联结：是一种不关联的联结，会返回左表中的数据（会以左边的表中数据为基准），不管右表中的数据是不是空值，但是如果使用了右联结，就不会返回该行数据

- 右联结：是一种不关联的联结，会返回右表中的数据（以右表中的数据为基准），不管左表中的数据是不是空值，但是如果使用了左联结，就不会返回该行数据

- 内联结：是一种等值联结方式

- 自然联结：不会出现重复值的一种联结方式

- 外部联结：外部联结可以包含了在两个表没有关联的行数据

#### 联结使用的指导原则

- 应该注意使用的联结方式，一般情况都是使用内部联结，但是有时候外部联结也是有效的
- 联结使用时需要指定正确的联结条件，否则会返回错误的数据
- 联结时应该总是提供联结条件，否则会得出笛卡尔积
- 联结可以作用与多个表，还可以对不同的表使用不同的联结方式，这是合法可行的，但是在使用之前应该首先单独测试每一种联结，确保数据返回正确，并且可以用来排序鼓掌所在

### 子查询

- **子查询**：就是嵌套在其他查询中的查询

- 子查询的顺序：从内向外进行查询处理

- 子查询的数量：并没有限制子查询的数量，但是考虑到性能限制，不能嵌套过多的子查询

- **相关子查询**：涉及外部查询的子查询，对于涉及外部查询的子查询往往可能会存在多义性，这个时候最好使用完全限定列。

#### 子查询过滤数据

```sql
select * from student
where name in (
	select name from student
    where register_date is not null
);
-- 这个操作首先进行的是内部的select语句查询，再进行外部的查询处理
-- 子查询的效率并不总是最有效的方法
```

#### 作为计算字段使用

```sql
select name, register_date,
(select count(*) from subject 
where subject = 'java' and
subject.studentid = student.id) as num
from student
order by num;
-- 此处字段列的选取使用了完全限定列名
```

### 组合查询

组合查询的实现通过UNION操作分来组合数条SQL查询，利用UNION，可给出多条select语句，将结果组合成单个结果集。

#### UNION使用

在多个查询语句之间放入关键字UNION即可

```sql
select * from student where name = 'jiabao'
union
select * from student where name = 'lijialin'
union
select * from student where register_date = '2021-05-02';
```

#### UNION规则

- UNION必须由两条或者两条以上的select语句组成，语句之间使用关键字UNION分隔
- UNION之间的查询必须包含相同的列、表达式或聚集函数，列之间的顺序可以不相同
- 列数据类型必须兼容，不一定需要完全相同，但必须是DBMS可以隐式转换的数据类型

#### UNION包含/取消重复行

默认情况下UNION自动去除了重复的行，如果你需要返回所有的所有的匹配行，可以使用`union all`关键字而不是`union`

```sql
-- union,自动删除重复行
select * from student where name = 'jiabao'
union
select * from student where register_date = '2021-05-02';

-- union all ,包含重复行
select * from student where name = 'jiabao'
union all
select * from student where register_date = '2021-05-02';
```

#### 对组合结果进行排序

对于组合完成之后的结果可以直接使用order by 子句进行排序操作

```sql
-- 对组合之后的结果进行排序
select * from student where name = 'jiabao'
union
select * from student where register_date = '2021-05-02'
order by name;
```

<font color='red'>UNION查询是支持不同表之间的组合查询，只要满足数据类型、列/表达式列/聚合函数列的数量一致即可</font>

### 全文本搜索

全文本搜索功能有些引擎并不能使用：比如InnoDB，但是MyISAM是可以使用的

#### 为什么使用全文本搜索

虽然前面提到了像like关键字（支持通配符匹配文本）和正则表达式（可以查找匹配较为复杂的匹配模式），但是上述的几个搜索机制存在一定的**限制**：

- 性能：通配符和正则表达式匹配通常要求mysql尝试匹配表中所有行，因此随着行数的增长，搜索就比较耗费时间
- 明确控制：正则表达式和通配符很难明确的控制匹配或者不匹配什么
- 智能化选择结果：如一个特殊词的搜索将会返回包含该词的所有航，而不区分包含单个匹配的行和包含多个匹配的行，同样的，特殊词的搜索不会返回不包含该词而包含其他词的行。

全文本搜索可以解决这几个限制

#### 全文本搜索的使用

1. 首先将需要被搜索的列进行索引，而且需要随着数据的改变不断重新索引

   ```sql
   ------ 全文本搜索启用 ------
   /*
   全文本搜索的启用方式：
   1. 创建表时使用指定fulltext
   2. 稍后指定（这个时候已有数据必须立即索引
   */
   
   -- 创建表时启用全文本搜索
   create table student (
   	id int(4) not null auto_increment,
       name char(10) not null,
       register_date date not null,
       studentid char9(10) not null,
       article text null,
       primary key(id),
       fulltext(article)
   )engine=MyISAM;
   -- 定义好了之后，mysql自动维护该索引，在增加、更新或者删除行，索引自动更新
   ```

   > 在导入数据时不要使用fulltext,更新索引需要花费时间，如果正在导入数据到一个新表，应该先导入所有数据，然后再修改表，定义fulltext，这样，更有助于加快导入数据

2. 索引之后，select语句可以和match()和against()一起使用来执行搜索

   ```sql
   ------ 全文本搜索示例 ------
   select article from student
   where match(article) against('rabbit');
   -- 上述示例会将rabbit所在的行匹配出来
   -- match()指示MySQL对指定的列进行搜索
   -- against('str')指定str为搜索文本
   
   -- 通配符实现
   select article from student
   where article like '%rabbit%';
   
   ------  全文本搜索注意事项： ------
   /*
   1. match()传递的参数必须和fulltext定义相同，如果指定了多个列，必须列出他们，而且还需要次序正确
   2. 搜索不区分大小写，处于使用binary方式，否则全文本搜索不区分大小写
   3. 全文本搜索返回的是以文本匹配的良好程度排序的数据
   4. match（`列名`） against('特殊词')会返回不同行的匹配程度
   5. 如果指定多个搜索项，包含匹配数多的行比极少匹配的行等级值高
   */
   ```
#### 使用查询扩展

查询扩展（适用4.11或更高级版本）：用来设法放宽所返回的全文本搜索结果的范围，可以找出可能相关的结果，即使并不精确包含所有查找的词

使用查询扩展的时候，MySQL会对数据和索引进行两遍扫描来完成搜索：

1. 首先继续宁基本的全文搜索，找出与搜索条件匹配的所有行
2. MySQL检查这些匹配行并选择有用的词
3. MySQL再次进行全文本搜索，不仅使用原来的条件，而且还使用所有有用的词

```sql
----- 无查询扩展的全文本搜索 -----
select * from student
where match(article) against('rabbit');

----- 有查询扩展的全文本搜索 ------
select * from student
where match(article) against('rabbit' with query expansion);
```

#### 布尔文本搜索

布尔文本搜索即使没有fulltext索引也可以使用，性能随着数据量增加而降低

```sql
----- 布尔文本搜索示例 ------
-- 简单示例
select article
from student
where match(article) against('hello' in boolean mode);

-- 包含某个值不包含另外一个值的行
select article from student
where match(article) against('hello -world*' in boolean mode);
-- 上述代码就是匹配包含hello但是不包含以world开头的词的行

------ 布尔文本搜索操作符 -------
/*
+   包含，词必须出现
-   不包含，词必须不出现
>   包含，而且增加等级值
<   包含，而且减少等级值
()  把词组成一个表达式，允许这个表达式作为组被包含、排除、排列
~   取消一个值的排序值
*   词尾的通配符，表示任意值
""  定义一个短语，需要匹配整个短语或者排除这个短语
*/

------- 操作符使用示例 -------
-- 包含操作符示例
select article from student
where match(article) against('+hello +world' in boolean mode);
-- 包含hello和world的行

-- 不包含操作符
select article from student
where match(article) against('hello world' in boolean mode);
-- 匹配包含hello或者world其中一个的行

-- 短语操作符
select article from student
where match(article) against('"hello world"' in boolean mode);
-- 匹配包含hello world这个短语的行

-- 排除操作符
select article from student
where match(article) against('-"hello world"' in boolean mode);
-- 匹配不包含hello world短语的行

-- 等级变化操作符
select article from student
where match(article) against('>hello <world' in boolean mode);
-- 匹配包含hello 和 world的行，增加前者等级，降低后者的等级

-- 分组括号符
select article from student
where match(article) against('+hello +(<world)' in boolean mode);
-- 匹配hello 和 world,降低后者的等级
```

#### 全文本搜索的注意事项

- 全文本搜索仅在MyISAM引擎中支持
- 不具有词分隔符的语言（包含日语和汉语）不能恰当的返回全文本搜索结果
- 会忽略词中的单引号
- 索引全文本数据，短词会被忽略且从索引中排除，短词定义为那些具有3个或3个以下字符的词（可以更改）
- MySQL有一个内建的非用词（stopword）列表，这些词在索引全文本数据时总是被忽略，如果需要，这个列表可以被覆盖
- 许多次的出现频率过高，搜索这些词用处不大。MySQL规定了一个50%规则：如果一个词出现在50%以上的行中，则将这个词考虑为非用词
- 如果表中行数少于3行，全文本搜索不返回结果（每个词或者不出现，或者至少出现50%行中）

