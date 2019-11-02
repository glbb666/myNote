# SQL标准写法
1. 关键字大写
2. 库，表，字段需要加上``
# MySQL连接

首先我们用管理员身份打开cmd
```mysql
mysql -u root -p
Enter password:******
```
# Navicat本地数据库端口号获取
1.在cmd中执行命令
```
netstat -ano
```
这样可以获取本机所有被占用的端口号
2.这里的端口号会有一个对应的pid，我们可以根据这个pid找到应用程序

# MySQL数据库操作
## 1. MySQL创建数据库
```mysql
CREATE DATABASE `数据库名`；
```
## 2. MySQL 删除数据库

```mysql
DROP DATABASE `数据库名`;
```
## 3. MySQL 选择数据库

```mysql
USE `RUNOOB`;
```
## 4. MySQL 展示数据库
```mysql
SHOW DATABASES;
```
# MySQL数据表
## 1.MySQL创建数据表
```mysql
CREATE TABLE `table_name`(column_name column_type)
```
## 2.MySQL删除数据表
```mysql
DROP TABLE `table_name`
```
## 3.约束
1. primary key 主键
主键约束字段不能为空，不能重复，一张表只能有一个主键
```mysql
CREATE TABLE `table_name`{
	column_name column_type primary key
}
```
2. foreign key 外键
```mysql
CREATE TABLE SPJ(
	SNO varchar(5) FOREIGN KEY REFERENCES S (SNO) NOT NULL,
	PNO varchar(5) FOREIGN KEY REFERENCES P (PNO) NOT NULL,
	JNO varchar(5) FOREIGN KEY REFERENCES J (JNO) NOT NULL,
	QTY int,
);```
一般会和references一起出现，如下：
3. references
A表Sno需要B表Sno数据，结合外键
​```mysql
eg：
foreign key(Sno)  references B(Sno)
```
4. unique 唯一键
```

```
5. check 约束
```mysql
CREATE TABLE `table_name`{
	Ssex char(2) DEFAULT '男' check（Ssex in('男', '女’）
}
```
如果是数字范围，还可写成这样
```mysql
check（age>=1 and age<=100）
此处也可以用 age between ** and**
not between ** and **
```
6. default 默认值
未添加数据时的默认值
```mysql
CREATE TABLE `table_name`{
	column_name column_type DEFAULT '20'//设置20为默认值
}
```
7. not null
不能为空
```mysql
CREATE TABLE `table_name`{
	column_name column_type DEFAULT '20'//设置20为默认值
}
```
8. indentity(种子值，增量值） 自增
9. zerofill 若长度未达到规定，自动在字段前填充0

# MySQL数据操作

## 1.MySQL插入数据(INSERT INTO)

MySQL 表中使用 **INSERT INTO** SQL语句来插入数据。

你可以通过 mysql> 命令提示窗口中向数据表中插入数据，或者通过PHP脚本来插入数据。

```mysql
INSERT INTO table_name (field1,field2,...,filedN) VALUES (value1，value2,...,valueN);
```
还可以使用 **INSERT IGNORE INTO**来插入数据
```mysql
INSERT IGNORE INTO table_name (field1,field2,...,filedN) VALUES (value1，value2,...,valueN);
```
两者的区别是：IGNORE会忽略插入的完全相同的记录，这样即使主键重复，也不会报错，因为被忽略了。

如果数据是字符型，必须使用单引号或者双引号，如："value"。

## 2.MySQL删除数据(DELETE 语句)
你可以使用 SQL 的 DELETE FROM 命令来删除 MySQL 数据表中的记录。

你可以在 mysql> 命令提示符或 PHP 脚本中执行该命令。

语法
以下是 SQL DELETE 语句从 MySQL 数据表中删除数据的通用语法：

```
DELETE FROM table_name [WHERE Clause]
```
- <font color=red>如果没有指定 WHERE 子句，MySQL 表中的所有记录将被删除。</font>
- 你可以在 WHERE 子句中指定任何条件
- 您可以在单个表中一次性删除记录。
当你想删除数据表中**指定的记录**时 WHERE 子句是非常有用的。

## 3.MySQL更新数据(UPDATE 语句)
如果我们需要修改或更新 MySQL 中的数据，我们可以使用 SQL UPDATE 命令来操作。.

```
UPDATE table_name SET field1=new-value1, field2=new-value2
[WHERE Clause]
```
- 你可以同时更新一个或多个字段。
- 你可以在 WHERE 子句中指定任何条件。
- 你可以在一个单独表中同时更新数据。
当你需要更新数据表中指定行的数据时 WHERE 子句是非常有用的。
## 4.MySQL查询数据(SELECT)
```mysql
SELECT column_name,column_name
FROM table_name
[WHERE Clause]
[LIMIT N][ OFFSET M]
```
- 改名（将原sname改名为NAME）
```mysql
select sname NAME from student;
```
- lower小写
```mysql
select lower sdept from student
```
- group by 列名 按值分组
- 查询一些计算过后的值
```mysql
select Sname,2014-Sage as birth from Student;//2014-Sage,一个算术表达式
```
- distinct消除重复
```mysql
select distinct sno from sc;
```
# MySQL WHERE子句
```mysql
SELECT field1, field2,...fieldN FROM table_name1, table_name2...
[WHERE condition1 [AND [OR]] condition2.....
```
注意：
如果在node中需要判断字符串相等，这样写是不对的
```mysql
let searchSql = 'SELECT weekly_email FROM user WHERE weekly_email="+user.email;
```
我们需要这样写
```mysql
let searchSql = 'SELECT weekly_email FROM user WHERE weekly_email="'+user.email+'"';
```
1. 比较大小
= 等于 >大于 <小于 >=大于等于 <=小于等于 <>/!=不等于 !>不大于 !<不小于
2. 确定范围
```mysql
between and
```
3.确定集合
```mysql
SELECT Sname
FROM Studen
WHERE Sdept IN (A,B,C);
```
4. 字符匹配
[not] like [ESCAPE '<换码字符>']
- %任意长度（可以为0）
- _单个任意字符
```mysql
//若like后不含有通配符，可以取代=
select * from student where sno like '201321'
//若含有通配符
select sname from student where sname like '_阳%'查找名字中第二个字为阳的学生姓名
//如含有换码字符
select sname from student where sname like '\_阳%' ESCAPE '\'//这里'\'后面的'_'不再具有通配符的含义，转义为普通的'_'字符
```

5. 涉及空值的查询
```
SELECT Sno,Cno
FROM SC
WHERE Grade IS NULL;//注意，这里的IS不能用=替代
```
6. 多重条件查询
```
SELECT Sno,Cno
FROM SC
WHERE Sdept = 'CS' OR Sdept = 'IS'
```
# ORDER By子句
```mysql
select * from Student
where Cno='3'
order by 列名 asc升序/desc降序
```
# Group By子句
将查询结果按某一列或多列的值分组
例如:求某一科的平均成绩，就要把平均成绩按照科目分组
```
select avg(grade) from cno
group by subject
```
# 聚集函数
- count(*)
- count([distinct|all] 列名)
- sum([distinct|all] 列名)
- avg([distinct|all] 列名)
- max([distinct|all] 列名)
- min([distinct|all] 列名)
distinct代表取消默认值，all代表不取消默认值

# 建立视图
```
CREATE VIEW 视图名 (列名1，列名2)
AS
子查询
[WITH CHECK OPTION]
```
其中，子查询可以是任意的SELECT语句

组成视图的属性列明必须全部省略或指定，如果省略，由select目标列的诸字段组成，但在下列情况下必须明确指定组成视图的所有列名
- 某个目标列不是单纯的属性名，而是聚集函数或列表达式
- 多表连接时选出了几个同名列作为视图的字段
- 需要在视图中为某个列启用新的更合适的名字
若在定义视图时加上`WITH CHECK OPTIONS`子句，以后对该视图进行插入、修改和删除操作的时候，关系数据都会自动检查插入的数据是否符合（WHERE）中的内容

视图除了可以建立在表上，还可以建立在视图上
- 行列子集视图：若一个视图时从单个基本表导出的，并且只是去掉了基本表的某些行和某些列，但是保留了主码，则称这类视图为行列子集视图
- 分组视图：
# MySQL中的一些小tip
## 1.int(5)究竟是什么意思

在我们给数据库增加新的列的时候，经常需要填写数据类型和位数，那么，int(5)是什么意思呢？难道代表int类型的五位数吗？

答案显然是否定的，int类型存储的始终是四个字节的整数，int(5)其实是和fillzero属性连起来使用的，也就是说，当数字的位数不足5位的时候，数据库会帮忙自动补零。当超过时不进行操作。

## 2.为什么数据库突然不能添加外键了？
是数据库存储引擎的问题。
InnoDB：支持外键
MyISAM：不支持外键

# 数据库安全

假设用户的名字叫LILI，我们现在给她一些权限
1.给用户进行某些操作的权限
```mysql
GRANT INSERT ON customer TO LILI;//给用户插入customer表的权限
```
2.给用户某项具体的权限=>修改价格的权限
```mysql

TE(PRICE) ON ORDERS TO LILI
```
# 范式
- 2NF
非主属性必须完全依赖于任何一个候选码
比如有一个成绩表（科目号，学生号，学生姓名，成绩）
其中主键是科目号和学生号
学生姓名只由学生号决定，这是部份依赖
成绩由学生号和科目号共同决定，这是完全依赖
- 3NF
教师号，系，住所
住所依赖于系
系依赖于教室号
住所间接依赖于教室号
- BCNF






