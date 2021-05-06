---
title: MySQL高级技巧
author: lijiabao
date: 2021-05-06 23:30
categories: ["MySQL", "Database"]
tags: ["MySQL", "Database"]
---


介绍一些MySQL中的高级技巧和性能优化的内容


# 高级技巧

## 视图view

> MySQL5添加了对视图的支持

**<font color='red'>视图是虚拟的表</font>**，与包含数据的表不一样，视图主要用于检索数据（select之后）的查询

```sql
-- 多表联表查询
select cust_name, cust_contact
from customers, orders, orderitems
where customers.cust_id = orders.cust_id
and orderitems.order_num = orders.order_num
and prod_id = 'TNT2';

-- 使用视图
select cust_name, cust_contact
from productcustomers
where prod_id = 'TNT2';
-- productcustomers是一个视图，作为视图，它不包含表中应该有的任何列或数据，包含的是一个SQL查询（与上面联结查询相同的查询操作）
```

### 使用视图的原因

视图的常见应用：

- **重用SQL语句**
- **简化复杂的SQL操作**：编写查询后，可以方便重用它而不必要知道它的基本查询细节
- **使用一部分表**：使用表的组成部分而不是整个表
- **保护数据**：可以给用户授予表的特定部分的访问权限而不是整个表的访问权限
- **更改数据格式和表示**：视图可以返回与底层表的表示和格式不同的数据

视图创建之后，可以用与表基本相同的方式利用它们：可以对视图执行select操作、过滤和排序数据、将视图联结到其他视图或者表，还可以添加和更新数据（有一些限制）

<font color='red'>视图仅仅是用来查看存储在别处的数据的一种结构，视图本身并不包含数据，返回的数据是从其他的表中检索出来的，添加或者更改这些表中的值时，视图将返回改变过的数据</font>

> 使用视图操作的性能问题：
>
> 视图不包含数据，因此使用视图的时候，必须执行查询操作，如果使用了的多个联结和过滤创建复杂视图或者嵌套了视图，会发现视图下降的很厉害，因此对于大量视图的应用前，应进行测试

### 视图的规则和限制

- 和表一样，视图必须唯一命名（不能给视图取与别的视图或者表相同的名字）
- 对于可以创建的视图数目没有限制
- 为了创建视图，必须有足够的访问权限
- 视图可以嵌套
- 视图中如果有order by，则外部视图查询出现的order by将覆盖视图中的
- 视图不可以索引，也不可以有关联的触发器或者默认值
- 视图可以和表一起使用

### 视图使用

- 创建视图：`create view`语句
- 查看创建视图的语句：`show create view viewname`
- 删除视图：`drop view viewname`
- 更新视图：先用drop再使用create,也可以直接使用`create or replace view`，如果需要更新的视图不存在，则创建一个视图，存在替换原有的视图

```sql
-- 视图最常见的应用就是隐藏复杂的SQL，通常会涉及联结 --------
-- 创建一个名为producttcustomers的视图，联结了三个表
create view productcustomers as
select cust_name. cust_contact, prod_id
from customers, orders, orderitems
where customers.cust_id = orders.cust_id
and orderitems.order_num = orders.order_num;

--  使用视图来查找特定的数据
select cust_name, cust_contact
from procuctcutomers
where prod_id = 'TNT2';

-- 视图格式化检索的数据 ------
create view viewname as
select concat(rtrim(name),'入学时间',rtrim(register_date))
from student
order by registerdate;

-- 使用视图过滤不想要的数据 ------------
create view viewname as
select name, register_date, subjectname
from student inner join subject
on student.id = subject.studentid
where register_date is not null
order by register_date;
-- 视图中使用的where子句和使用视图时的where子句会自动组合

-- 使用计算字段的视图
create view viewname as
select id*id as new_id, name,
date_add(register_date,interval 2 days) as new_date
from student;

-- 基本上之前使用的查询语句都可以使用到视图的创建中来
```

> 视图使用时尽量创建可重用的视图，创建不受特定数据限制的视图是不错的方法。扩展视图的范围使得视图可重用，因此就不再需要创建和维护多个视图

### 视图更新

通常，视图是可以更新的（可以使用insert、update和delete），更新一个视图将会更新其基表，对视图增加行或删除行，实际是对其基表增加或者删除行

但是有些视图是不可更新的，即当MySQL不能正确确定被更新的基数据，就不允许更新，也就是，使用了以下操作的：

- 分组group by
- 联结join
- 子查询
- union合并
- 聚合函数
- distinct 唯一值

<font color='red'>但是尽管有这么多的更新限制，但是实际上，视图主要是用来检索数据而不是更新数据的</font>

## 索引（index）

### 索引的优点

- 索引大大减少了服务器需要扫描的数据量
- 索引可以帮助服务器避免排序和临时表
- 索引将随机IO变成顺序IO
- 索引对于InnoDB（索引支持行级锁）非常重要，可以让查询锁更少的元组
- InnoDB在耳机索引上使用共享锁（读锁），访问主键索引需要排他锁（写锁）

### 索引缺点

- 虽然索引提高了查询速度，但是会降低更新表的速度，如：update、insert和delete，更新表时，不仅要保存数据，还要保存索引
- 建立索引会占用磁盘空间的索引文件，如果在大表创建多种组合索引，索引文件会膨胀的很快
- 如果某个数据列包含许多重复内容，创建索引没有太大效果
- 对于非常小的表，全表扫描更高效
- MySQL同一个数据表索引限制16个

### 索引分类

#### 底层数据结构分类

##### hash索引

InnoDB中，对于热点数据（经常被访问）会自动生成hash索引，又被叫做自适应hash索引

##### B+树索引

除了全文索引和hash索引，InnoDB和MyISAM的索引都是B+树实现

#### 是否在主键创立索引

##### 主键索引

MySQL根据主键来组织数据的，每张表必须有主键索引，主键索引只能有一个，不能为null且必须保证唯一性。创建表的时候如果没有指定主键索引，会自动生成一个隐藏的字段作为主键索引

##### 辅助索引

如果不是主键索引，则就可以称之为辅助索引，又称为辅助索引或者二级索引。

主键索引的叶子结点存储了完整的数据行，但是<font color='red'>辅助索引的叶子结点存储的是主键索引，通过辅助索引查询数据，会首先查找到主键索引，然后再到主键索引找对应的数据行。</font>

#### 其他类型索引

##### 唯一索引

不允许具有索引值相同的行，禁止重复的索引或键值。系统在创建该索引时自动检查是否有重复键值，并在每次插入更新操作执行时检查，有重复值就会操作失败抛出异常

主键索引一定时唯一索引，唯一索引不一定是主键索引，唯一索引只是将索引设置了一个唯一性

##### 全文索引

fulltext全文索引，主要用于全文本搜索使用，InnoDB加入了全文索引的支持，但是不支持中文的索引，主要用来进行关键字或者关键词查询文本。

### 索引使用

```sql
-- 创建索引的方法：
-- 1. 在创建表的时候增加索引
create table table_name (
	id int not null auto_increment,
    name varchar(10),
    age int(3) default 20,
    identityCard varchar(18),
    description text,
    key name (`name`),
    primary key id (`id`),
    unique key identityCard (`identituCard`)
    fulltext text (`text`)
)engine=InnoDB default charset=utf8;

-- 2. 表创建好之后，使用修改表命令增加索引
alter table database_name.table_name add fulltext index (column_name);  -- 增加全文索引到指定列
alter table table_name add index (column_name);  -- 增加普通索引
alter table table_Name add unique (column_name);  -- 增加唯一索引
alter table table_name add primary key (column_name);  -- 增加主键索引

-- 3. 使用create index命令
/*
create [unique | fulltext | spatial] index index_name [index_type]
on table_name (column_key_part,...) [index_option];
*/
-- 在description列创建全文索引
create fulltext index text_description on student (description);
```

## 存储过程（stored procedure）

存储过程：就是将多条语句放入到一个完整的操作中的一个方式，和众多编程语言的函数类似，或者说类似于批处理。

### 存储过程的优点

- 通过把处理封装在容易使用的单元中，简化复杂的操作
- 保证某些操作下使用的都是同一个存储过程，使用的代码都一样，<font color='red'>减少出现错误的可能性，保证数据的一致性</font>
- 简化对变动的管理，只需对存储过程中的代码进行修改，使用这个存储过程的人都可以不用知道更改细节，<font color='red'>保证 操作的安全性</font>
- <font color='red'>提高性能</font>：使用存储过程比使用单独的SQL语句要快
- 存储过程可以编写功能强大且灵活的代码

总结起来存储过程优点就是<font color='red'>简单、安全、高性能、更灵活</font>

### 存储过程的缺点

- 存储过程的编写更加的复杂，需要更高技能和经验
- 可能没有创建存储过程的安全访问权限：有些数据库管理员会限制存储过程的创建权限，有时候可能允许使用但是不允许创建存储过程

### 存储过程使用

#### 创建存储过程

```sql
-- 简单创建存储过程的示例  ------
create procedure procedure_name()
begin
	select avg(id) as priceaverage from student;
end;

/*
上述代码的解释：
create procedure是创建存储过程的语句
procedure_name是存储过程的名字，用来后面调用存储过程使用
()用来指定参数在括号内部，就算没有参数也需要指定该括号
begin和end中间用来指定存储过程的sql操作
*/

/* 
使用MySQL的命令行客户机的需要注意：
 MySQL语句的默认分隔符为;号，而存储过程中的语句分隔符也是;号
 如果命令行程序要解释存储过程内部的;号，会出现语法错误
 解决方法是临时更改命令行程序的分隔符号
 */
delimiter //  -- 更改分隔符为双斜线
-- 创建存储过程
create procedure procedure_name()
begin
	select select avg(id) as priceaverage from student;
end //

delimiter ;  -- 将分隔符改回去，除了\号，其他任何字符都可以作为分隔符
```

#### 执行存储过程

```sql
-- 调用存储过程
call procedure_name();
-- 计算并返回产品的最低、最高和平均值
```

#### 删除存储过程

```sql
-- 删除存储过程，新建一个同名存储过程前，需要先删除这个存储过程
drop procedure procedure_name;  -- 只需要给出存储过程名
drop procedure if exists procedure_name;  -- 仅当存在时删除
```

#### 使用参数的存储过程

存储过程一般不显示结果，只是把结果返回给你指定的变量

变量：内存中一个特定位置，用来临时存储数据

```sql
-- 创建使用变量的存储过程
create procedure procedure_name(
	out pl decimal(8,2),
    out ph decimal(8,2),
    out pa decimal(8,2)
)
begin
	select Min(id) into pl from student;
	select Max(id) into ph from student;
	select Avg(id) into pa from student;
end;
-- 此存储过程接受3个参数
/*
MySQL参数类型：
in：传递给存储过程
out：从存储过程传出
inout：对存储过程传入传出
参数指定的示例：arg_type var_name date_type()
*/

-- 含参数的存储过程调用，需要和定义时的参数数量一致才可正确调用
call procedure_name(
	@pricelow,  -- 这些变量是存储过程中保存结果的变量
    @pricehigh,
    @priceaverage
);
-- MySQL的变量必须以@开始

-- 另外一个使用参数的存储过程的示例
create procedure procedure_name(
	in re_date date,
    out n char(10)
)
begin
	select name from student
	where register_date = re_date into n;
end
-- 调用
call procedure_name('2021-05-02',@find_name);
```

#### 建立智能有效的存储过程

存储过程的强大只有包含业务规则和智能处理时，它们的高效好用才能体现

```sql
-- Name: orderTotal
-- Parameters: onumber = order number
--             taxable = 0 if not taxable, 1 if taxable
--             otatal  = order total variable
create procedure orderTotal(
	in onumber int,
    in taxable boolean,
    out ototal decimal(8,2)
) comment 'obtain order total' -- 存储过程解药介绍的注释
begin
	-- 声明变量，声明的是局部变量
	declare total decimal(8,2);
	-- 声明税率百分率，局部变量声明支持默认值
	declare taxrate int default 6;
	-- 获取订单总数
	select sum(price*quantity) from orderitems
	where order_num = onumber into total;
	-- 判断是否可收税
	if taxable then
		select total+(total/100*taxrate) into total;
	end if;
	-- 将数据输出到out变量中
	select total into ototal;
end

/*
if语句语法：
if condition then
	...
elseif condition then
	...
else
	...
end if;
*/

-- ===== 查看存储过程 =======
-- 查看创建存储过程的语句
show create procedure procedure_name;
-- 获取和人何时创建等信息的语句
show procedure status;   -- 列出所有存储过程
show procedure status like 'irdertotal';
-- 限制输出，使用like啦
```



## 游标cursor

> MySQL5添加了对游标的支持

> 游标：一个存储在MySQL服务器上的数据库查询，不是一条select语句，而是被select语句检索出来的结果集，存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。<font color='red'>游标主要用于交互式应用，如：滚动屏幕并对数据进行浏览或者修改，游标只能够用于存储过程和函数</font>

### 游标使用步骤

游标使用的步骤：

1. 创建定义游标，这个创建过程并没有检索数据，只是定义使用
2. 声明后，需要打开游标以供使用，这个过程是把select语句的数据查询出来
3. 对于有数据的游标，根据需要取出（检索）各行
4. 结束游标使用时，必须关闭游标

游标声明后，可根据需要频繁打开和关闭游标，游标打开后，可根据需要频繁执行读取操作

### 创建游标

```sql
-- 游标创建需要在存储过程中定义或者函数中定义
create procedure procedure_name()
begin
	declare ordernumbers CURSOR  -- 创建名为ordernumbers的游标
	for  -- for子句后面附带select语句
	select order_num from orders;
end;
```

存储过程处理完成之后，游标就会消失（仅存在于存储过程）

### 打开和关闭游标

```sql
-- 打开游标
open cursor_name;
-- 关闭游标
close cursor_name;

/*
游标关闭后，没有重新打开，就不能再使用，但是使用了声明过的游标不需要再声明，直接打开就好
如果没有明确关闭游标，当到达end语句之后就会自动关闭它
*/
create procedure procedure_name()
begin
	declare ordernumbers cursor
	for
	select order_num from orders
	where order_num > 10000;
	-- 打开游标
	open ordernmbers;
	-- 关闭游标
	close ordernumbers;
end;
```

### 使用游标数据

在游标打开之后，可以使用fetch语句分别访问它的每一行。

fetch指定检索什么数据（所需的列），检索出来的数据存储到哪，它还向前移动游标中的内部行指针，使下一条fetch语句检索下一行（不重复读取同一行）

```sql
-- ======== 获取一条数据 =========
create procedure procedure_name()
begin
	-- 声明局部变量
	declare o int;
	-- 声明游标
	declare ordernumbers cursor
	for
	select order_num from orders;
	-- 打开游标
	open ordernumbers;
	-- 获取订单数
	fetch ordernumbers into o;
	-- 关闭游标
	close ordernumbers;
end;

-- ======== 循环读取数据 =========
create procedure procedure_name()
begin
	-- 声明局部变量
	declare done boolean default 0;
	declare o int;
	-- 声明游标
	declare ordernumbers cursor
	for
	select order_num from orders;
	-- 声明句柄，设置当SQLSTATE出现'02000'（repeat没有更多行读取时，出现这个条件）,执行set操作
	declare continue handler
	for SQLSTATE '02000' set done=1;
	-- 打开游标
	open ordernumbers;
	-- 获取订单数
	-- 创建一个循环读取所有行
	repeat
		fetch ordernumbers into o;
	-- 结束循环
	until done end repeat
	-- 关闭游标
	close ordernumbers;
end;
```

## 触发器trigger

触发器：是用来在某些语句之后自动执行的一条语句，只能响应下列语句：

- delete语句
- insert语句
- update语句

### 创建触发器

触发器创建需要给出下列信息（触发器只能在表中创建，视图和临时表都不支持）：

- 唯一的触发器名（只在表中唯一，同一个数据库的两个表可以具有相同名字的触发器）
- 触发器关联的表
- 触发器响应的活动（delete、update或insert）
- 触发器什么时候执行（处理之前或者之后）

```sql
-- 简单的触发器创建
create trigger trig_name  -- 创建触发器并指定触发器名
after insert   -- 在什么操作的前或者后
on `tablename`   -- 在指定的表设定触发器
for each row select 'product added';
-- for each row指定对每一行插入操作都执行触发器
```

创建触发器的注意事项：

- 每个表每个事件只能有一个触发器
- 每个表只允许6个触发器，insert、update和delete操作的之前之后
- 一个触发器不能与多个事件或多个表关联
- 触发器只能在表中创建

### 删除触发器

```sql
-- 删除触发器
drop trigger trig_name;
-- 触发器不能更新或者覆盖，想要修改触发器必须先删除
-- 删除之后再重新创建
```

### 使用触发器

#### insert触发器

- insert触发器中，可以引用一个名为NEW的虚拟表，访问被插入的行
- before insert触发器，new中的值可以被更新（允许更改被插入的值）

```sql
-- 插入触发器的例子
create trigger trig_name
after insert
on student
for each row
set @hello = NEW.name;

-- 使用begin end包裹的语句
create trigger trig_name
after insert
on student
for each row
begin
	if new.studentid = '20290012' then
		set new.name = 'jiabao';
	else
		set new.name = 'lijiabao';
	end if;
end;
```

#### delete触发器

- 在delete触发器代码内，你可以引用一个名为OLD的虚拟表，访问被删除的行
- OLD值全部是只读的，不能更新

```sql
-- 创建一个删除操作触发器，将删除的数据插入到另外一个表中，以防止误删，从而可以恢复
create trigger delete_trig
after delete
on student
for each row
begin
	insert into archive_student(id, name, register_date, studentid)
	values(OLD.id, OLD.name,OLD.register_date, OLD.studentid);
end;
```

<font color='red'>使用begin和end语句块的好处就是在内部可以放入多条SQL语句</font>

#### update触发器

- update出发去代码中，可以引用一个OLD的虚拟表访问更新之前的值，也可以引用一个名为NEW的虚拟表访问更新之后的值
- before update触发器，NEW的值也可能被更新
- OLD之中的值全部是只读的，不能更新

```sql
-- 使用update触发器
create trigger trig_name
before update on student
for each row
begin
	insert into backup
	values(OLD.name, OLD.register_date, OLD.studentid);
	select name from OLD into @name;
end;
```

### 触发器特征

- 创建触发器可能需要特殊的**访问权限**
- 触发器的**执行是自动的**，只要insert、update和delete可以执行，绑定的触发器就可以执行
- 可以使用触发器来保证数据的一致性（大小写和格式之类的）
- 触发器最有用的使用是用来进行**审计跟踪**：将每一次更改记录到另外一个表中
- 触发器中并**不能使用call函数**，因此就不可以调用存储过程，只能将存储过程的代码复制到触发器中

## 事务处理（transaction）

> 支持事务处理的引擎是InnoDB，而MyISAM引擎不支持明确事务处理

**事务处理**：用来维护数据库的完整性，是一种机制，来保证成批的MySQL操作要么完全执行，要么全不执行，如果操作中没有错误发生，就将操作之后的结果持久化（commit）到数据库，失败了则进行回滚（rollback）恢复操作之前的状态

### 事务处理控制

事务的开始：`start transaction`，这个语句标识事务开始

#### 回滚（rollback）

回滚rollback命令是用来回退撤销MySQL语句的

```sql
select * from student;
start transaction;
update student set name = 'lijiabao' where name = 'jiabao';
select * from student;
rollback;  -- 进行回滚，撤销之前的更改操作
select * from student;

/*
rollback使用的指导原则：
1. rollback只能在一个事务中执行使用（start transaction;语句之后）
2. rollback用来管理撤销insert、update和delete操作的
3. 不能够回滚撤销create和drop操作
4. rollback执行之后就代表了事务的结束
*/
```

#### 提交持久化（commit）

普通的不在事务内部的SQL语句都会有一个隐含提交操作，这是自动进行的，但是在事务内部的操作并不会自动执行提交操作，需要手动执行commit命令来进行提交持久化

```sql
-- 持久化提交
start transaction;
select * from student;
update student set name = 'lijiabao' where name = 'jiabao';
commit;
select * from student;
```

rollback和commit操作执行之后，就相当于一个事务的结束，之后的SQL语句操作就又都是自动提交的

#### 保存点（savepoint）

对于稍微复杂的事务，往往提交和回滚只需要执行到特定的位置就行，不用完全撤销或者全部提交。在这个时候，就需要使用到一种占位符，回退和提交可以选择到这个位置来进行。这个占位符称之为保存点

保存点创建：`savepoint point_name;`，保存点的名字唯一，以方便回滚或者提交

```sql
-- 保存点的使用
start transaction;
select * from student;
update student set name = 'jiabao' where name = 'lijiabao';
savepoint point1;
select * from student;
delete from student;
select * from student;
rollback to point1;  -- 回退到保存点
-- 回退到保存点之后，保存点之前的操作就相当于提交执行了
```

保存点使用原则：

- 每个保存点名字需要唯一，便于MySQL直到应该回滚到哪里
- 保存点越多越好，保存点越多，就可以更灵活的回退
- 保存点在事务完成之后，自动释放，MySQL5之后，可以使用`release savepoint point_name`明确的释放保存点

#### 修改默认的提交行为

默认的MySQL设置了自动提交所有更改，所有语句执行完之后，立即提交生效，你可以通过命令`set autocommit=0`关闭自动提交，关闭自动提交后，需要手动执行commit命令才可以持久化到数据库中。autocommit只针对每一个MySQL连接，而不是服务区级别的。

如果需要开启自动提交，使用命令`set autocommit=1`

```sql
-- ======= 常用的事务逻辑步骤 =======
set autocommit=0;   -- 关闭自动提交
start transaction;   -- 开启一个事务
select * from student;
update student set name = 'jiabao' where name = 'lijiabao';
savepoint point1;
...   -- 写事务中需要执行的语句即可
select * from student;
rollback;  -- 也可使用commit，标志着transaction结束
set autocommit=1;  -- 重新开启自动提交
```



### 事务处理的特点

- 原子性（atomic）：事务内部的MySQL操作要么全部成功，要么全部失败
- 一致性（consistency）：事务的运行并不会影响数据库中数据的一致性，比如转账：这个账户往另外一个账户转账，这个账户金额变少则另外一个账户金额增多，银行总额不变。
- 隔离性（isolation）：不同事务之间的操纵是相互隔离互不影响的，不会出现两个事务交互处理的情况。如果在转账过程中出现两个事务交错执行，这个账户先减少了钱，然后跳到另外一个查询余额的事务中执行，这个时候查询的余额就不对了，因为前一个事务还未持久化。
- 持久性（duration）：事务执行成功之后，事务对于数据的更改就会持久化到数据库中

几种常见的事务操作中的问题：

- 脏读：一个事务读取到了另外一个事务未提交的数据
- 幻读：一个事务查询两次，两次得到的数据条数不一致
- 不可重复读：一个数据查找两次，得到的数据不一致

### 事务的隔离级别

- 读未提交：事务A可以读取到事务B修改但是未提交的数据，这种隔离级别下容易发生脏读、不可重复读和幻读的问题（较少使用）
- 读已提交：事务A只能在事务B修改提交之后才能读取到事务B修改的数据，解决了脏读的问题，但可能出现不可重复读和幻读的问题（较少使用）
- 可重复读：事务AB只能在事务都提交之后才可以读取修改后的数据，解决了脏读和不可重复读问题但是可能出现幻读
- 可串行化：通过加锁的方式来解决上述的三种问题，读读操作不会阻塞，读写、写读、写写操作都会被加锁阻塞。

### 查看和设置隔离级别

#### 查看隔离级别

```sql
-- 方法一：
show variables like 'transaction_isolation';

-- 方法二：
select @@transaction_isolation;
```

#### 设置隔离级别

```sql
set [global|session] transaction isolation level level_name;

/*
level隔离级别值：
1. repeatable read  可重复读
2. read committed  读已提交
3. read uncommitted   读未提交
4. serializable  可串行化

global关键字：只对执行完该语句之后产生的会话起作用，已存在的会话无效
session关键字：对当前会话的所有后续事务有效
可以在事务中间执行，不影响事务，对后续事务生效
无关键字：只对当前会话的下一个事务有效，下一事务完成后，后续事务不再生效，不能在开启的事务中间执行
*/


-- 方法二：通过服务启动
-- 启动服务时加上--transaction-isolation=level
```



## 全球化和本地化（字符集编码）

- 字符集：字母和符号的集合
- 编码：某个字符集成员的内部表示
- 校对：规定字符如何比较的指令

> 校对为什么重要：因为不同的字符集不同语言下的排序情况不一样，同时是否区分大小写也有比较大的影响

### 字符集和校对顺序使用

```sql
-- 查看所支持的字符集完整列表
show character set;

-- 查看所有可用的校对以及他们适合的字符集，有的字符集不只一种校对
show collation;

-- 通常系统安装时定义了一个默认的字符集和校对，也可在创建数据库时，指定默认的字符集和校对
-- 确定所用的字符姐和校对，使用下面的两条命令
show variables like 'character%';  -- 查看系统默认字符集
show variables like 'collation%';  -- 查看系统默认校对

/*
字符集很少是服务器范围的设置
不同的表甚至不同的列都可能需要不同的字符集，两者都可在创建时指定
*/
-- 对表进行字符集和校对的设置
create table `table_name` (
	column1 int,
    column2 varchar(10)
)engine=InnoDB default  charset set gb2312
collate gb2312_general_ci;

-- 对列进行字符集和校对的设置
create table `table_name` (
	column1 int,
    column2 varchar(10),
    column3 varchar(10) character set gb2312 collate gb2312_genneral_ci
)default character set utf8 collate utf8_general_ci;

-- 校对主要用于在order by子句检索出来的数据排序起重要作用
-- 需要指定和创建表时用的不同排序顺序时，可以使用下面命令
select * from student
order by name collate gb2312_general_ci;

-- collate子句还可以在其他的如：group by、having、聚集函数和别名等出现
```

MySQL确定使用的字符集和校对的方式：

- 如果指定了character set和collate两者，则使用这两者的值
- 如果只指定了character set，则使用此字符集和其默认的校对
- 如过没有指定，则使用数据库的默认值

## 安全管理

### 访问控制

**访问控制：**就是给用户提供他们所需的访问权，且仅提供他们所需的访问权，访问控制的实现通过创建和管理用户账号来实现的。

> MySQL Administator提供了图形用户界面来管理用户及其账号的权限

root用户账号对整个MySQL服务拥有整个服务器的完全控制权

实际生产和工作当中，不要使用root，应该创建一系列的用户账号，用来管理或者提供给用户华人开发人员使用等

<font color='red'>访问控制主要是用来防止错误和用户的恶意企图</font>

### 用户管理

MySQL的用户账号和信息存储在mysql数据库中

```sql
use mysql;  -- 使用mysql数据库
select user from user;  -- 查看用户信息
```

#### 创建用户账号

```sql
-- 创建一个用户，指定身份认证通过密码 p@$$wOrd 进行验证登陆
create user user_name identified by 'p@$$wOrd';  -- 使用identified by口令创建用户

-- 更改用户密码
set password for user_name = 'new_password';
set password = 'new_password';  -- 为当前用户更改密码
alter user user_name identified by 'new_password';  -- 更改密码

-- grant语句也可以用于创建用户
grant

-- 直接在mysql数据库的user表中插入行来增加用户，但是为了安全，不这样使用
```

#### 重命名用户

```sql
-- rename user语句来进行重命名
-- rename user old_name to new_name;
rename user jiabao to lijiabao;  -- mysql5之后支持的rename操作

-- 还可以直接更新user表来重命名
update mysql.user set user = new_name where user = old_name;
```

#### 删除用户账号

```sql
-- drop user语句删除用户
-- drop user user_name;
drop user lijiabao;

-- mysql5之后，drop user可以删除用户账号和所有相关的权限
-- mysql5之前，只能删除账号而不能删除相关权限，需要用revoke删除权限，再删除账号
```

#### 设置访问权限

创建了用户账号之后，需要紧接着分配访问权限。

<font color='red'>新创建的用户没有访问权限，但是可以登陆mysql，但是不能看到数据，不能执行任何数据库操作</font>

```sql
-- 查看赋予给用户的权限
show grant for jiabao;
-- grant usage on *.* to 'jiabao'@'%'，其中usage表示没有权限
-- 标准的用户定义user@host，用户名和主机名，不指定主机使用默认主机名%（授予用户权限而不管主机名）

-- ======= 设置权限 =======
/*
grant语句要求你至少给出以下信息：
1. 要授予的权限
2. 被授予访问权限的数据库或表
3. 用户名
*/
-- 示例
grant select on test.* to jiabao;
-- 将test数据库的所有表的查询权限给到用户jiabao，也就是只给予了可读权限
grant all on test.* to ''@'host';  -- 给与权限到匿名用户

-- 撤销权限操作，revoke
revoke select on test.* from jiabao;  -- 删除前一步操作赋予的权限

/*
权限赋予更改样式：
grant/revoke [grant_type] on [grant_range] to/from user@host;
*/
```

**grant和revoke控制访问权限的几个层次：**

- 整个服务器，`grant all`和`revoke all`
- 整个数据库，使用`on database_name.*`
- 指定的表：`on database_name.table_name`
- 指定的列：`on database_name.table_name.column_name`
- 指定的存储过程

| 可给予的权限           | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| all                    | 除了grant option外的所有权限                                 |
| alter                  | 使用alter table的权限                                        |
| alter routine          | 使用alter procedure和drop procedure的权限                    |
| create routine         | 使用create procedure的权限                                   |
| create temporary table | 使用create temporary table                                   |
| create user            | 使用create user、drop user、rename user和revoke all privileges |
| create view            | 使用create view                                              |
| delete                 | 使用delete                                                   |
| drop                   | 使用drop table                                               |
| event                  | 允许使用事件调度器                                           |
| execute                | 使用call和存储过程                                           |
| file                   | 使用select into outfile和load data infile                    |
| grant option           | 使用grant和revoke                                            |
| index                  | 使用create index和drop index                                 |
| insert                 | 使用insert                                                   |
| lock tables            | 使用lock tables                                              |
| process                | 使用show full processlist                                    |
| preference             | 允许创建外键                                                 |
| reload                 | 使用flush                                                    |
| replication client     | 服务器位置访问                                               |
| replication slave      | 复制从属使用                                                 |
| select                 | 使用select                                                   |
| show databases         | 使用show databases                                           |
| show view              | 使用show create view                                         |
| shutdown               | 使用mysqladmin shutdown（关闭MySQL）                         |
| super                  | 使用change master、kill、logs、purge、master和set global，还允许mysqladmin调试登陆 |
| trigger                | 允许触发器操作                                               |
| update                 | 使用update                                                   |
| usage                  | 无访问权限                                                   |



## 数据库维护

### 备份数据

和所有数据一样，MySQL数据也必须经常备份，MySQL数据是给予磁盘的文件，普通备份系统就能备份数据

- 使用命令行程序mysqldump转储所有数据库内容到某个外部文件
- 可用命令行使用程序mysqlhotcopy从一个数据库复制所有数据（并非所有数据引擎都支持）
- 可使用MySQL的`backup table`或`select into outdile`转储所有数据到某个外部文件，这两条语句都接受将要创建的系统文件名，这个系统文件需不存在，不然会出错。通过`restore table`来复原数据。

> 为了保证所有数据写入到磁盘，需要在进行备份前使用`flush tables`语句

### 数据库维护

- `analyze table table_name`用来检查表键是否正确
- `check table table_name`：针对许多问题对表进行检查，MyISAM表上还对索引进行检查。check table发现和修复问题
- 如果MyISAM表访问产生不正确和不一致的结果，可能需要使用`repair table`来修复相应的表，不应该常使用这个语句，如果需要经常使用，可能会有更大的问题需要解决
- 如果从一个表中删除大量数据，应该使用`optimize table`来收回所用的空间，从而优化表的性能

### 诊断启动问题

排除系统启动问题的时候，首先应该使用手动启动服务器，MySQL服务通过在命令行执行mysqld启动，下面是几个常用重要的mysqld的命令行选项：

- --help显示帮助
- --safe-mode装载减去某些最佳配置的服务器
- --verbose显示全文本消息
- --version显示版本信息然后退出

### 查看日志

主要的日志文件：

- **错误日志**：<font color='re'>包含启动和关闭问题以及任意关键错误的细节</font>，此日志通常名为hostname.err，位于data目录中，日志名可以使用--log-error命令行选项更改
- **查询日志**：记录所有MySQL活动，<font color='red'>在诊断问题时非常有用</font>，此日志文件可能会很快变得非常大，不应该长期使用，此日志名为hostname.log，位于data目录，可以使用--log命令行选项更改
- **二进制日志**：<font color='red'>记录更新过数据的所有语句</font>，此日志通常名为hostname-bin，位于data目录中，可以使用--log-bin命令行选项更改，在版本5中添加
- **缓慢查询日志**：记录执行缓慢的任何查询，<font color='red'>这个日志在确定数据库何时需要优化很有用</font>，此日志通常名为hostname-slow.log，位于daya目录中，可以使用--log-slow-queries命令行选项更改

> 使用日志时，可以使用`flush logs`刷新和重新开始所有日志文件

## 性能调优

- MySQL具有特定的硬件建议，学习使用时，并没有很大的硬件要求，但实际生产中，还是需要遵守官方的硬件建议
- 关键的DBMS应该运行在自己的专用服务器上
- MySQL使用的是一系列预先配置好的设置，但是有时候需要调整内存分配、缓冲区大小等（查看当前设置，`show variables`，`show status`)
- MySQL是多用户多线程的DBMS，某一个任务执行缓慢，所有任务都会在执行缓慢。遇到性能不良时，使用`show processlist`查看所有活动进程，然后还可以使用`kill`命令杀死某个进程（管理员）
- 对于每一个查询操作，总有许多种方法，应该试验找出性能最优的一种方法进行操作
- 存储过程比一条一条执行MySQL语句要更快
- 总是使用正确的数据类型，不要使用会过度影响性能的数据类型
- 不要检索比需求多的数据，就是不使用`select  * from`
- 需要使用索引来改善数据检索的性能，但是如果数据不经常被查询，可以不适用索引
- 使用union而不是多条or语句的连接
- like模糊搜索很慢，尽量使用fulltext
- 数据库是不断变化的，性能优化可能需要时时进行

> <font color='red'>每条规则在某些条件下都会被打破</font>