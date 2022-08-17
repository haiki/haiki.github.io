---
title: mysql
date: 2023-01-15 09:00:25
categories:
tags:
---

## 了解 SQL

### 数据库基础

数据库

表

模式

列 和 数据类型

行

主键

### 什么是 SQL

## MySQL 简介

MySQL是一种DBMS，即它是一种数据库软件。

### MySQL 工具

为了指定用户登录名ben，应该使用`mysql -u ben`。为了给出用户名、主机名、端口和口令，应该使用`mysql -u ben -p -h myserver -P 9999`。

命令输入在`mysql>`之后;
 命令用`;`或`\g`结束，换句话说，仅按Enter不执行命令;

 输入`help`或`\h`获得帮助，也可以输入更多的文本获得特定命令的帮助(如，输入`help select`获得使用SELECT语句的帮助);

 输入`quit`或`exi`t退出命令行实用程序。

## 使用 MySQL

关键字(key word) 作为MySQL语言组成部分的一个保留字。绝不要用关键字命名一个表或列。附录E列出了MySQL的关键字。

必须先使用USE打开数据库，才能读取其中的数据。 `USE customers;`

`SHOW DATABASES;`返回可用数据库的一个列表。

`SHOW TABLES;` 返回当前选择的数据库内可用表的列表。

`SHOW COLUMNS FROM customers;` 它对每个字段返回一行，行中包含字段名、数据 类型、是否允许NULL、键信息、默认值以及其他信息(如字段cust_id，auto_increment)。也可以用 `DESCRIBE customers`。

所支持的其他SHOW语句还有:

- `SHOW STATUS;` 用于显示广泛的服务器状态信息;
- `SHOW CREATE DATABASE;` 和 `SHOW CREATE TABLE;` 分别用来显示创建特定数据库或表的MySQL语句;
- `SHOW GRANTS;` 用来显示授予用户(所有用户或特定用户)的安全权限;
- `SHOW ERRORS;` 和 `SHOW WARNINGS` 用来显示服务器错误或警告消息。

## INSERT

INSERT是用来插入(或添加)行到数据库表的。插入可 以用几种方式使用:
 插入完整的行;

CREATE TABLE customers
(
  cust_id      int       NOT NULL AUTO_INCREMENT,
  cust_name    char(50)  NOT NULL ,
  cust_address char(50)  NULL ,
  cust_city    char(50)  NULL ,
  cust_state   char(5)   NULL ,
  cust_zip     char(10)  NULL ,
  cust_country char(50)  NULL ,
  cust_contact char(50)  NULL ,
  cust_email   char(255) NULL ,
  PRIMARY KEY (cust_id)
) ENGINE=InnoDB;

没有输出 INSERT语句一般不会产生输出。
对每个列必须提供一个值。如果某个列没有值,应该使用NULL值（假定表允许对该列指定空值）。第一列是个由MySQL自动增量。你不想给出一个值(这是MySQL的工作)，又不能省略此列(如前所述，必须给出每个列)，所以指定一个NULL值(它被 MySQL忽略，MySQL在这里插入下一个可用的cust_id值)。

```sql
INSERT INTO Customers
VALUES(NULL,
'qipei',
'nanshan street',
'shenzhen',
'guangdong',
'10010',
'CN',
NULL,
NULL);
```

 插入行的一部分
因为提供了列名，VALUES必须以其指定的次序匹配指定的列名，不 一定按各个列出现在实际表中的次序。

省略的列必须满足以下某个条件。
 该列定义为允许NULL值(无值或空值)。
 在表定义中给出默认值。这表示如果不给出值，将使用默认值。
如果对表中不允许NULL值且没有默认值的列不给出值，则 MySQL将产生一条错误消息，并且相应的行插入不成功。

```sql
INSERT INTO Customers(cust_name,
cust_contact,
cust_email)
VALUES(
    'peiqi',
    '1881157',
    'peiqi66@111.com'
);
```

`INSERT LOW_PRIORITY INTO` 指示MySQL 降低INSERT语句的优先级.

 插入多行;
// 如果是在cmd输入，第一条insert输入完就会执行了；因此要用脚本一次性执行？

```sql
INSERT INTO customers(cust_name,
cust_contact,
cust_email
) VALUES (
    'peiqi',
    '1881157',
    'peiqi66@111.com'
);
INSERT INTO customers(cust_name,
cust_city,
cust_email
) VALUES (
    'haiki',
    'shenzhen',
    'qipei@qq.com'
);
```

只要每条INSERT语句中的列名(和次序)相同，可以如下组合各语句:单条INSERT语句有多组值，每组值用一对圆括号括起来， 用逗号分隔。

```sql
INSERT INTO customers(cust_name,
cust_contact,
cust_email) 
VALUES (
    'peiqi',
    '1881157',
    'peiqi66@111.com'
),
(
    'haiki',
    '12227894',
    'qipei@qq.com'
);
```

 插入某些查询的结果。

INSERT SELECT，顾名思义，它是由一条INSERT语句和一条SELECT 语句组成的，将一条SELECT语句的结果插入表中。

在填充custnew时，不应该使用已经在customers 中使用过的cust_id值(如果主键值重复，后续的INSERT操作 将会失败)或仅省略这列值让MySQL在导入数据的过程中产 生新值。

这条语句将插入多少行有赖于custnew表中有多少行。

```sql
INSERT INTO customers(cust_name,
cust_contact,
cust_email)
SELECT cust_id,
cust_contact,
cust_email
FROM custnew;
```

## UPDATE

为了更新(修改)表中的数据，可使用UPDATE语句。可采用两种方 式使用UPDATE(不要省略WHERE子句):

 更新表中特定行;

```sql
UPDATE customers 
SET cust_email = 'haiki123@qq.com',
cust_contact = '12345'
WHERE cust_id = 2;
```

在UPDATE语句中使用子查询。TODO?

```sql
UPDATE customers
SELECT cust_id
cust_contact,
cust_email
FROM customers
WHERE cust_id = 2;
```

如果用UPDATE语句更新多行，并且在更新这些
行中的一行或多行时出一个现错误，则整个UPDATE操作被取消 (错误发生前更新的所有行被恢复到它们原来的值)。为即使是发
生错误，也继续进行更新，可使用IGNORE关键字，如下所示: `UPDATE IGNORE customers...`

为了删除某个列的值，可设置它为NULL(假如表定义允许NULL值)。

```sql
UPDATE customers
SET cust_contact = NULL
WHERE cust_id = 3;
```

 更新表中所有行。
不使用 WHERE 字句。

可以限制和控制UPDATE语句的使用，更多内 容请参见第28章。 TODO

## DELETE

为了从一个表中删除(去掉)数据，使用`DELETE`语句。可以两种方 式使用`DELETE`:
 从表中删除特定的行;

```sql
DELETE FROM customers
WHERE cust_id = 2;
```

`DELETE`不需要列名或通配符。`DELETE`删除整行而不是删除列。为了删除指定的列，请使用`UPDATE`语句。

  从表中删除所有行。
 不要省略`WHERE`子句。
 `DELETE`与安全 可以限制和控制`DELETE`语句的使用，更多内 容请参见第28章。


删除表的内容而不是表 `DELETE`语句从表中删除行，甚至是删除表中所有行。但是，`DELETE`不删除表本身。

更快的删除：
可使用 `TRUNCATE TABLE` 语句(TODO)，它完成相同的工作，但速度更 快(TRUNCATE实际是删除原来的表并重新创建一个表，而不 是逐行删除表中的数据)。

下面是许多SQL程序员使用UPDATE或DELETE时所遵循的习惯。
 除非确实打算更新和删除每一行，否则绝对不要使用不带WHERE 子句的UPDATE或DELETE语句。
 **保证每个表都有主键**(如果忘记这个内容，请参阅第15章)，尽可能 像WHERE子句那样使用它(可以指定各主键、多个值或值的范围)。 
 在对UPDATE或DELETE语句使用WHERE子句前，**应该先用SELECT进行测试，保证它过滤的是正确的记录，**以防编写的WHERE子句不
正确。
 **使用强制实施引用完整性的数据库**(关于这个内容，请参阅第15
章)，这样MySQL将不允许删除具有与其他表相关联的数据的行。

## MANAGE TABLE

### 创建表

为了用程序创建表，可使用SQL的CREATE TABLE语句。必须给出下列信息:

 新表的名字，在关键字CREATE TABLE之后给出; 

 表列的名字和定义，用逗号分隔。

```sql
CREATE TABLE customer (
    cust_id int NOT NULL AUTO_INCREMENT,
    cust_name char(50) NOT NULL,
    cust_contact char(50) NULL,
    cust_email char(50) NULL,
    PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
```

如果你仅想在一个表不存在时创建它，应该在表名后给出 `IF NOT EXISTS`。这样做不检查已有表的模式是否与你打算创建的表模式相匹配。它只是查看表名是否存在，并且仅在表名不 存在时创建它。

表中的列在创建时就要指定是否允许 NULL，只有两种选择。
理解`NULL`：不要把`NULL`值与空串相混淆。`NULL`值是没有值， 它不是空串。如果指定`''`(两个单引号，其间没有字符)，这 在`NOT NULL`列中是允许的。空串是一个有效的值，它不是无值。`NULL`值用关键字`NULL`而不是空串指定。

#### 主键

主键值必须唯一。即，表中的每个行必须具有唯一的主 键值。如果主键使用单个列，则它的值必须唯一。如果使用多个列，则 这些列的组合值必须唯一。

`PRIMARY KEY(order_num, order_item)`

主键为其值唯一标识表中每个行的列。**主键中只能使用不允许NULL值的列。**允许NULL值的 列不能作为唯一标识。

**每个表只允许一个AUTO_INCREMENT列，而且它必须被索引(如，通过使它成为主键)。**

如何在使用AUTO_INCREMENT列时获得这个值呢?可使 用last_insert_id()函数获得这个值，如下所示:`SELECT last_insert_id()`
此语句返回最后一个AUTO_INCREMENT值，然后可以将它用于 后续的MySQL语句。

如果在插入行时没有给出值，MySQL允许指定此时使用的默认值。默认值用`CREATE TABLE`语句的列定义中的`DEFAULT`关键字指定 `age int NOT NULL DEFAULT 18`。

不允许函数与大多数DBMS不一样，MySQL不允许使用函数作为默认值，它只支持常量。

使用默认值而不是NULL值 许多数据库开发人员使用默认值而不是NULL列，特别是对用于计算或数据分组的列更是如此。

#### 引擎类型

MySQL有一个具体管理和处理数据的内部引擎。 在你使用 `CREATE TABLE`语句时，该引擎具体创建表，而在你使用`SELECT` 语句或进行其他数据库处理时，该引擎在内部处理你的请求。

MySql 具有多种引擎，它们具有各自不同的功能和特性，为不同的任务选择正确的引擎能获得良好的功能和灵活性。

以下是几个需要知道的引擎:（所支持引擎的完整列表(及它们之间的不同)，请 参阅<http://dev.mysql.com/doc/refman/5.0/en/storage_engines.html）>
InnoDB 是一个可靠的事务处理引擎(参见第26章)，它不支持全文本搜索;
 MEMORY在功能等同于MyISAM，但由于数据存储在内存(不是磁盘) 中，速度很快(特别适合于临时表);
 MyISAM是一个性能极高的引擎，它支持全文本搜索(参见第18章)， 但不支持事务处理。

外键不能跨引擎 混用引擎类型有一个大缺陷。外键(用于强制实施引用完整性，如第1章所述)不能跨引擎，即使用一个引擎的表不能引用具有使用不同引擎的表的外键。

### 更新表

为了使用`ALTER TABLE`更改表结构，必须给出下面的信息:
 在`ALTER TABLE`之后给出要更改的表名(该表必须存在，否则将 出错);
 所做更改的列表。

添加列必须明确其数据类型。

```sql
ALTER TABLE customers
ADD cust_age int;
```

删除刚刚添加的列，可以这样做:

```sql
ALTER TABLE customers
DROP COLUMN cust_age;
```

ALTER TABLE的一种常见用途是定义外键。

```sql
ALTER TABLE ordersitem
ADD CONSTRAINT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES orders (order_num);
```

为了 对单个表进行多个更改，可以使用单条ALTER TABLE语句，每个更改用逗 号分隔。

小心使用ALTER TABLE 使用ALTER TABLE要极为小心，应该 在进行改动前做一个完整的备份(模式和数据的备份)。数据 库表的更改不能撤销，如果增加了不需要的列，可能不能删 除它们。类似地，如果删除了不应该删除的列，可能会丢失 该列中的所有数据。

#### 删除表

```sql
DROP TABLE customers;
```

删除表没有确认， 也不能撤销，执行这条语句将永久删除该表。

#### 重命名表

RENAME TABLE所做的仅是重命名一个表。

```sql
RENAME TABLE Customers TO customers;
```

对多个表重命名：

```sql
RENAME TABLE Customers TO customers,
orderitems TO orderItems;
```



## QA

1. 为什么一个完整的sql语句要以分号结束？
2. 空格会有什么影响？
   - MySQL语句中忽略空格。 语句可以在一个长行上输入，也可以分成许多行。
