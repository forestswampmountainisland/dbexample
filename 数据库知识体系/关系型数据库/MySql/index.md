# MySql

MySql是一种关系型数据库，由Oracle开发，免费的数据库产品

## 为什么使用MySql而不是其他的关系型数据库

MySql开源，可以被用来修改满足定制要求

MySql免费

性能好

## MySql操作

1. 连接数据库

mysql -h host -u user -p

填写主机，端口，用户名，密码

2. 选择数据库

use [db]

3. 显示所有数据库

show databases

4. 显示所有表

show tables

5. 显示列

show [columns] from [db]

6. 显示数据库状态

show status

7. 显示权限

show grants

8. 显示建数据库的Sql

show create database

9. 显示建表的Sql

show create table

10. 检索单列

select [column] from [table]

11. 检索多列

select [column1], [column2] from [table]

12. 检索所有列

select * from [table]

13. 检索出不同值的行

select distinct [column] from [table]

14. 检索并限制结果

select [column] from [table] limit [offset], [limitCount]

offset从0开始

limitCount大于结果数量时，会返回结果的数量

15. 给搜索结果排序

select [column1] from [table] order by [column2]

排序的column可以不是搜索的column

排序默认是升序

16. 按多列排序

select [column1], [column2] from [table] order by [column1], [column2], [column3]

17. 指定排序方向

select [column] from [table] order by [column] desc

若有多列，需要指定多个desc

18. 过滤数据

select [column] from [table] where [clause] order by [column]

clause成为子句，可以想象成结果为boolean值的关系式，比如是id = 1

order by 应该在where之后

clause 举例:

* id = 1
* id <> 1
* id between 1 and 5
* id > 1

19. 空值检查

空值为null,行允许某一列为空值

select [column] from [table] where [column2] is null

20. 使用操作符(operator)

operator包括:

* and
* or
* in
* not
等

and和or属于逻辑操作符, 使用方法为 [clause] [operator] [clause2], 优先级 and > or,可以使用()来改变优先级

in操作符用来指定范围,使用方法之一为 [column] in ([value1], [value2], ...),当column的值为任一value则为true

in的另一种使用方法为in where clause

not可以与in, between或者exist一起使用，表示取反

21. 通配符(wildcard)

* % 任意字符出现任意次数
* _ 匹配单个字符

22. 正则表达式(regular expression)

REGEXP

23. 计算字段

* Concat
* LTrim
* Rtrim
* as
* +, -, *, /

24. 函数

函数的可移植性不强，所以慎用

字符串
* RTrim
* Upper
* Length
* Concat
* Lower
* Left
* Right
* Locate
* Soundex
* Substring

日期
* AddDate
* AddTime
* CurDate
* CurTime
* Date
* DateDiff
* Date_Add
* Date_Format
* Day
* DayOfWeek
* Hour
* Minute
* Month
* Now
* Second
* Time
* Year

数值
* Abs
* Cos
* Exp
* Mod
* Pi
* Rand
* Sin
* Sqrt
* Tan

汇总
* Count
* Avg
* Sum
* Max
* Min

25. 分组

select Count([column]) from table group by [column2] having Count([column]) > 2

select gender, Count(*) from user group by gender;

where 过滤行, having过滤分组

select from where group by having order by limit

26. 子查询

* 子查询过滤

select customId from orders where orderid in (select orderId from orderItem where produc2`tId = 'TNT2')

* 子查询计算

select customers.custName, (select count(*) from orders where orders.custId = customer.custId) as orders from customers

27. 联结表(join)

分开存储在不同表的原因:

* 表太大降低性能
* 有重复数据一旦改动会很麻烦

联结会使多张表做笛卡儿积

join类型

* cross join

select [column1], [column2] from [table1], [table2]

全部表做笛卡儿积

* inner join 

select [column1] from [table1] inner join [table2] on [table1].[column1] = [table2].[column2]

做笛卡尔积但是会有条件

* auto join

select p1.prod_id from products as p1, products as p2 where p1.vend_id = p2.vend_id and p2.prod_id = 'DTNTR'

相同两张表做笛卡尔积

* Natural join

* 外部联结

select [column1] from [table1] [left|right] outer join [table2] on [table1].[column1] = [table2].[column2]

相比于inner join没有关联的行也会出现在结果中，值会为Null

left outer join 会使用左边所有的行与右边匹配如果没有匹配右边的行的记录会为null

right outer join相反

28. 组合查询

前提使所union的查询语句查询的列，数据类型,计算列一致

会将行合并

Union

[select clause1] union [select clause2]

Union会自动去掉重复的行，如果不想去掉可以使用Union all

Union可以使用order by, order by 作用于union后的集合

29. 全文本搜索

MySql搜索引擎中支持全文本搜索的有MyISAM

创建表时需指明Fulltext

Match([column]) Against([text])

30. 插入值

insert into [table] values([values])(,([values]))*

insert into [table and columns] values([value])(,([values]))*

insert into [table1] select [columns] from [table2]

31. 更新数据

update (ignore)? [table] set [column] = [value],([column] = [value]*) where [clause]

32. 删除数据

delete from [table] where [clause]

33. 创建表

create table [table] ([columnDefs]) (if not exists)*

null值

Auto_increment 一张表最对只有一个列可以是auto_increment的

default默认值

引擎类型:

InnoDB 支持事务
Memory 快速，但只存内存
MyISAM 支持全文搜索

34. 更新表

alter table [table] add [column] [columnDef]
alter table [table] drop [column]
rename table [table] to [newTable]

35. 删除表

drop table [table]

36. 视图

类似与一个Sql查询，但是对这个查询进行了封装，外部调用宛如在调用一个表一样，可以简化Sql操作

可以隐藏表的一些细节

可以改变数据的格式和表示

视图与表的权限可以不一样

37. 创建视图

create view [viewName] as [clause]

show create view [viewName]

drop view [viewName]

create or replace view [viewName] as [clause]

视图中可以查询视图

38. 存储过程

预编译的SQL语句

作用：
封装
完整性
简化对变动的管理
提升性能

缺点
复杂
需要权限

call [procedure] [params]

create procedure [procedure]([paramsDef]) begin [clause] end

drop procedure [procedure] 

create procedure [procedure](out|in|inout [name] [type]) begin select [column] into [name] from table; end

存储过程同一个查询只准返回最多一个列

39. 游标(cursor)

与存储过程一起使用的迭代器

40. 触发器(trigger)

自动执行sql的语句

只支持delete,insert,update

create trigger [name] before|after [clause]

delete trigger

41. 事务

并非所有引擎都支持事务处理

优点：完整性

InnoDB支持，MyISAM不支持

start transaction

rollback

commit

savepoint [name]

rollback to [savepoint]

42. 用户管理

mysql数据库user表

create user [name] identified by [password]

drop user [name]

rename user [name] to [newName]

show grants for [name]

grant [operation] on [database].[table] to [name]

revoke [operation] on [database].[table] from [name]

43. 备份数据

mysqldump

mysqlhotcopy

backup table

analyze table

44. 日志