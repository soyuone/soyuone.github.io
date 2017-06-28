---
layout: post
title: "高性能mysql：查询性能优化"
categories: database
tags: database mysql
author: Kopite
---

* content
{:toc}


本文系阅读`《高性能MySQL》，Baron Schwartz等著`一书中`第六章 查询性能优化`的笔记，本章关注的是`合理的设计查询`，如果查询写的很糟糕，即时`库表结构再合理`、`索引再合适`，也无法实现高性能。
<br>
<br>
`查询优化`、`索引优化`、`库表结构优化`需要齐头并进，一个不落。在获得编写mysql查询的经验的同时，也将学习到如何为高效的查询设计表和索引。同样的，也可以学习到在优化库表结构时会影响到哪些类型的查询。
<br>
<br>
本章将从查询设计的一些基本原则开始——这也是在发现查询效率不高的时候首先需要考虑的因素。然后会介绍一些更深的查询优化的技巧，并会介绍一些mysql优化器内部的机制。之后将展示mysql是如何执行查询的，学习如何去改变一个查询的执行计划。最后，了解mysql优化器在哪些方面做的还不够，并探索查询优化的模式，以帮助mysql更有效的执行查询。
<br>
<br>
本章的目标是深刻的理解mysql如何真正的执行查询，并明白高效和低效的原因。



## 查询速度慢的原因

在尝试编写快速的查询之前，需要清楚一点，真正重要的是响应时间。通常来说，查询的生命周期大致可以按照顺序来看：`从客户端，到服务器，然后在服务器上进行解析，生成执行计划，执行，并返回结果给客户端`。其中执行可以认为是整个生命周期中最重要的阶段，这其中包括了大量为了检索数据到存储引擎的调用以及调用后的数据处理，包括排序、分组等。
<br>
<br>
在完成这些任务的时候，查询需要在不同的地方花费时间，包括网络，CPU计算，生成统计信息和执行计划、锁等待（互斥等待）等操作，尤其是向底层存储引擎检索数据的调用操作，这些调用需要在内存操作、CPU操作和内存不足时导致的I/O操作上消耗时间。根据存储引擎不同，可能还会产生大量的上下文切换以及系统调用。
<br>
<br>
在每一个消耗大量时间的查询案例中，都能看到一些不必要的额外操作、某些操作被额外的重复了很多次、某些操作执行的太慢等。优化查询的目的就是减少和消除这些操作所花费的时间。

## 优化数据访问

查询性能低下最基本的原因是访问的数据太多。某些查询可能不可避免的需要筛选大量数据，但这并不常见。大部分性能低下的查询都可以通过减少访问的数据量的方式进行优化。对于低效的查询，可通过下面两个步骤进行分析：
* 确认程序是否在检索大量超过需要的数据。这通常意味着访问了太多的行，但有时候也可能是访问了太多的列
* 确认mysql服务器层是否在分析大量超过需要的数据行

### 向数据库请求不需要的数据

有些查询会请求超过实际需要的数据，然后这些多余的数据会被程序丢弃。这会给mysql服务器带来额外的负担，并增加网络开销，另外也会消耗应用服务器的CPU和内存资源，典型案例如下所示：

* 查询不需要的记录。常常会误以为mysql会只返回需要的数据，实际上mysql却是先返回全部结果集再进行计算。例如，取出100条记录，但是只是在页面上显示前面10条，mysql会查询出全部的结果集，客户端的程序会接收全部的结果集数据，然后抛弃其中大部分数据。最简单有效的解决办法就是在这样的查询后面加上`LIMIT`。

* 多表关联时返回全部数据列。例如，查询某机构中的医生信息：

```sql
SELECT * FROM tb_user INNER JOIN cc_stdoc ON tb_user.USERID = cc_stdoc.stdoc_no WHERE cc_stdoc.hsp_code = '001X';
```

上述查询将返回两个表的全部数据列，应该用下面的查询，只取需要的列：

```sql
SELECT tb_user.* FROM tb_user INNER JOIN cc_stdoc ON tb_user.USERID = cc_stdoc.stdoc_no WHERE cc_stdoc.hsp_code = '001X';
```

* 总是取出全部列。每次看到`SELECT *`的时候都需要用怀疑的眼光审视，是不是真的需要返回全部的列？很可能不是必需的。取出全部列，会让优化器无法完成索引覆盖扫描这类优化，还会为服务器带来额外的I/O、内存和CPU的消耗。因此，一些DBA严格禁止`SELECT *`的写法，这样做有时候还能避免某些列被修改带来的问题。

* 重复查询相同的数据。很容易出现这样的错误——不断的重复执行相同的查询，然后每次都返回完全相同的数据。比较好的方案是，当初次查询的时候将这个数据缓存起来，需要的时候从缓存中取出，这样性能显然会更好。

### 扫描额外的记录

在确定查询值返回需要的数据以后，接下来应该看看查询为了返回结果是否扫描了过多的数据。对于mysql，最简单的衡量查询开销的三个指标如下：
1. 响应时间
2. 扫描的行数
3. 返回的行数

这三个指标都会记录到mysql的慢日志中，所以检查慢日志记录是找出扫描行数过多的查询的好办法。

* 响应时间。响应时间是两部分之和：`服务时间`和`排队时间`。`服务时间`是指数据库处理这个查询真正花了多长时间，`排队时间`是指服务器因为等待某些资源而没有真正执行查询的时间——可能是等I/O操作完成，也可能是等待行锁，等等。在不同类型的应用压力下，响应时间并没有什么一致的规律或者公式。诸如存储引擎的锁（表锁、行锁）、高并发资源竞争、硬件响应等诸多因素都会影响响应时间。

* 扫描的行数和返回的行数。分析查询时，查看该查询扫描的行数是非常有帮助的，这在一定程度上能说明该查询找到需要的数据的效率高不高。较短的行的访问速度更快，内存中的行也比磁盘中的行的访问速度要快的多。

* 扫描的行数和访问类型。在评估查询时，需要考虑从表中找到某行数据的成本。如果查询没有办法找到合适的访问类型，那么解决的最好办法通常就是增加一个合适的索引，索引让mysql以更高效、扫描行数最少的方式找到需要的记录，示例如下：

```sql
mysql> EXPLAIN SELECT item_name,item_code FROM cc_item WHERE item_code = 'YP0000
000002'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: cc_item
         type: ref
possible_keys: item_code
          key: item_code
      key_len: 153
          ref: const
         rows: 29
        Extra: Using index condition
1 row in set (0.00 sec)
```

从`EXPLAIN`的结果可以看到，mysql在索引item_code上使用了`type: ref`访问类型来执行查询，mysql预估需要访问29行数据，如果没有合适的索引会怎样？使用没有索引的item_name列进行查询，如下所示：

```sql
mysql> EXPLAIN SELECT item_name,item_code FROM cc_item WHERE item_name = '葡萄糖
注射液'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: cc_item
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 63257
        Extra: Using where
1 row in set (0.00 sec)
```

访问类型变成了一个`type: ALL（全表扫描）`，现在mysql预估需要扫描63257条记录来完成此查询，这里的`Extra: Using where`表示mysql将通过`WHERE`条件来筛选存储引擎返回的记录。一般情况下，mysql能够使用如下三种方式应用`WHERE`条件，从好到坏依次为：
  * 在索引中使用`WHERE`条件来过滤不匹配的记录，这在存储引擎层完成
  * 使用索引覆盖扫描（`Extra: Using index condition`）来返回记录，直接从索引中过滤不需要的记录并返回命中的结果，这在mysql服务器层完成，但无须再回表查询记录
  * 从数据表中返回数据，然后过滤不满足条件的记录（`Extra: Using where`），这在服务器层完成，mysql需要先从数据表读出记录然后过滤

如果发现查询需要扫描大量的数据但只返回少数的行，那么通常可以尝试下面的技巧：
  * 使用索引覆盖扫描，把所有需要用的列都放到索引中，这样存储引擎无须回表取对应行就可以返回结果
  * 改变库表结构
  * 重写这个复杂的查询，让mysql优化器能够以更优化的方式执行此查询

## 重构查询

有时候，可以将查询转换一种写法让其返回一样的结果，但是性能更好。但也可以通过修改应用代码，用另一种方式完成查询，最终达到一样的目的。

### 复杂查询还是多个简单查询

mysql内部每秒能够扫描内存中上百万行数据，相比之下，mysql响应数据给客户端就慢的多了。在其它条件都相同的时候，使用尽可能少的查询当然是更好的。但是有时候，将一个大查询分解为多个小查询是很有必要的。

### 切分查询

有时候对于一个大查询，需要分而治之，将大查询切分成小查询，每个查询功能完全一样，只完成一小部分，每次只返回一小部分查询结果。
<br>
<br>
例如，删除大量数据时，如果用一个大的语句一次性完成的话，则可能需要一次锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但重要的查询。将一个大的DELETE语句切分成多个较小的查询可以尽可能小的影响mysql的性能，同时还可以减少mysql复制的延迟。

### 分解关联查询

很多高性能的应用都会对关联查询进行分解。简单的，可以对每一个表进行一次单表查询，然后将结果在应用程序中进行关联，有如下优势：
* 让缓存的效率更高
* 将查询分解后，执行单个查询可以减少锁的竞争
* 在应用层做关联，可以更容易对数据库进行拆分，更容易做到高性能和可扩展
* 查询本身效率也可能会有所提升
* 可以减少冗余记录的查询。在应用层做关联查询，意味着对于某条记录应用只需要查询一次，而在数据库中做关联查询，则可能需要重复的访问一部分数据，这样的重构还可能会减少网络和内存的消耗
* 更进一步，这样做相当于在应用中实现了`哈希关联`，而不是使用mysql的循环嵌套关联。某些场景下，`哈希关联`的效率要高很多

## 查询执行过程

msyql的查询执行过程，如下图所示：

![](/image/2017/2017-06-15-database-mysql-high-performance-query-1.png)

1. 客户端发送一条查询给服务器
2. 服务器先检查查询缓存，如果命中了缓存，则立刻返回存储在缓存中的结果，否则进入下一阶段
3. 服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划
4. mysql根据优化器生成的执行计划，调用存储引擎的API来执行查询
5. 将结果返回给客户端

### 通信协议

mysql客户端和服务器之间的通信协议是`半双工`的，在任何一个时刻，要么是由服务器向客户端发送数据，要么是由客户端向服务器发送数据。

### 查询缓存

在解析一个查询语句前，如果`查询缓存`是打开的，那么mysql会优先检查这个查询是否命中`查询缓存`中的数据。如果当前的查询恰好命中了`查询缓存`，那么在返回查询结果之前mysql会检查一次用户权限。如果权限没有问题，mysql会跳过所有其它阶段，直接从`查询缓存`中拿到结果并返回给客户端。

### 查询优化



## 查询优化器的局限性



## 查询优化器的提示



## 优化特定类型的查询

### COUNT()查询

`COUNT()`是一个特殊的函数，有两种非常不同的作用，示例详见[选择优化的数据类型](https://soyuone.github.io/2017/06/07/database-mysql-high-performance-data-type/):
* 可以统计某个列值的数量，在统计列值时要求列值是非空的（不统计`NULL`），如果在COUNT()的括号中指定了列或者列的表达式，则统计的就是这个表达式有值的结果数
* 可以统计结果集的行数，当mysql确认括号内的表达式值不可能为空时，实际上就是在统计行数。最简单的就是使用`COUNT(*)`时，这种情况下，它会忽略所有的列而直接统计所有的行数

一个常见的错误：在括号内指定了一个列却希望统计结果集的行数，如果希望查询结果集的行数，最好使用`COUNT(*)`，这样写意义清晰，性能也会更好。
<br>
<br>
一个常见的问题：在同一个查询中统计同一个列的不同值的数量，以减少查询的语句量，以下三种查询方式均可实现
```sql
SELECT SUM(IF(first_name = 'm', 1, 0)) AS M, SUM(IF(first_name = 'a', 1, 0)) AS A FROM people;
SELECT COUNT(first_name = 'm' OR NULL) AS M, COUNT(first_name = 'a' OR NULL) AS A FROM people;
SELECT SUM(first_name = 'm') AS M, SUM(first_name = 'a') AS A FROM people;
```

上述查询中：
* `IF(expr1,expr2,expr3)`中expr1为真时，返回expr2；expr1为假时，返回expr3
* 第二种查询方式，只要满足条件设置为真，不满足条件设置为`NULL`

### 关联查询

* 确保`ON`或者`USING`子句中的列上有索引。在创建索引的时候就要考虑到关联的顺序，当表A和表B用列C关联时，如果优化器的关联顺序是B、A，那么就不需要在B表的对应列上建索引，没有用到的索引只会带来额外的负担。`一般来说，除非有其它理由，否则只需要在关联顺序中的第二表的相应列上创建索引`
* 确保任何的`GROUP BY`和`ORDER BY`中的表达式只涉及到一个表中的列，这样mysql才有可能使用索引来优化

#### JOIN

```sql
CREATE TABLE `tb_user` (
  `userid` int(10) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `email` varchar(50) NOT NULL COMMENT '用户邮箱',
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(100) NOT NULL COMMENT '登录密码',
  `passwordsalt` varchar(50) NOT NULL COMMENT '密码盐',
  `userphone` varchar(20) DEFAULT '' COMMENT '用户手机号',
  `createtime` datetime NOT NULL COMMENT '创建时间',
  `updatetime` datetime DEFAULT NULL COMMENT '最后更新时间',
  `version` int(11) NOT NULL COMMENT '乐观锁版本号',
  PRIMARY KEY (`userid`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COMMENT='用户信息表';
```

```sql
CREATE TABLE `tb_classify` (
  `id` int(10) NOT NULL AUTO_INCREMENT COMMENT '分类id',
  `classifyname` varchar(50) NOT NULL COMMENT '分类名称',
  `userid` int(10) NOT NULL COMMENT '所属用户id',
  PRIMARY KEY (`id`),
  KEY `FK_fk_classify_user_1` (`userid`),
  CONSTRAINT `FK_fk_classify_user_1` FOREIGN KEY (`userid`) REFERENCES `tb_user` (`userid`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8 COMMENT='文章分类信息表';
```

* `JOIN`等价于`INNER JOIN`，前后两表中至少都有一个匹配时才返回结果，`ON`前后的字段摆放顺序对查询无影响，示例如下：

```sql
SELECT u.userid, c.classifyname FROM tb_user u JOIN tb_classify c ON c.userid = u.userid;
SELECT u.userid, c.classifyname FROM tb_user u JOIN tb_classify c ON u.userid = c.userid;
SELECT u.userid, c.classifyname FROM tb_user u JOIN tb_classify c ON u.userid = c.userid WHERE u.userid = '3';
```

前两个查询中`ON`前后的字段摆放顺序颠倒，对查询结果无影响。由于c.userid中没有匹配u.userid = 3的记录，因此前两个查询结果中不含userid = 3的记录，第三个查询结果无返回记录。

* `LEFT JOIN`等价于`LEFT OUTER JOIN`，从左侧表返回所有的行，即使在右侧表中没有匹配的行，`ON`前后的字段摆放顺序对查询无影响。下述示例查询中，结果含userid = 3的记录：

```sql
SELECT u.userid, c.classifyname FROM tb_user u LEFT JOIN tb_classify c ON u.userid = c.userid;
```

* `RIGHT JOIN`等价于`RIGHT OUTER JOIN`，从右侧表返回所有的行，即使在左侧表中没有匹配的行，`ON`前后的字段摆放顺序对查询无影响。下述示例查询中，结果含userid = 3的记录：

```sql
SELECT u.userid, c.classifyname FROM tb_classify c RIGHT JOIN tb_user u ON c.userid = u.userid;
```

* `FULL JOIN`等价于`FULL OUTER JOIN`，只要在两侧的的某个表中存在匹配行就会返回，`mysql不支持FULL JOIN，但可通过UNION实现FULL JOIN`

* 当关联表的字段名称一样时，可以使用`USING()`，以减少sql语句的长度：

```sql
SELECT u.userid, c.classifyname FROM tb_classify c RIGHT JOIN tb_user u USING(userid);
```

### GROUP BY和DISTINCT

`GROUP BY`和`DISTINCT`都可以使用索引来优化，这也是最有效的优化办法。如果需要对关联查询做`GROUP BY`，并且是按照查找表中的某个列进行分组，那么通常采用查找表的`标识列`分组的效率会比其它列更高，示例如下，userid为标识列：

```sql
SELECT u.userid, c.classifyname, COUNT(*) FROM tb_classify c RIGHT JOIN tb_user u USING(userid) GROUP BY userid;
```

在分组查询的`SELECT`中直接使用非分组列通常都不是什么好主意，因为这样的结果通常是不定的，当索引改变，或者优化器选择不同的优化策略时都可能导致结果不一样。事实上，建议将mysql的`SQL_MODE`设置为包含`ONLY_FULL_GROUP_BY`，这时mysql会对这类查询直接返回一个错误，提醒需要重写此查询，[参见：SQL_MODE参数](https://soyuone.github.io/2017/05/10/database-mysql-windows-configure/)。
<br>
<br>
如果没有通过`ORDER BY`子句显式的指定排序列，当查询使用`GROUP BY`子句时，结果集会自动按照分组的字段进行排序。如果不关心结果集的顺序，而这种默认排序又导致了需要文件排序，则可以使用`ORDER BY NULL`，让mysql不再进行文件排序。也可以在`GROUP BY`子句中直接使用`DESC`或者`ASC`关键字，使分组的结果集按需要的方向排序。

### LIMIT

在系统中需要进行分页操作时，通常会使用`LIMIT`加上偏移量的办法，同时加上合适的`ORDER BY`子句。如果有对应的索引，通常效率会不错，否则mysql需要做大量的文件排序操作。

```sql
SELECT classifyname FROM tb_classify LIMIT m,n;
```

上述示例中，m是指记录开始的编号，从0开始(表示第一条记录)，n是指从第m+1条开始取，共取n条记录。
<br>
<br>
在偏移量非常大的时候，比如`LIMIT 10000, 20`这样的查询，这时mysql需要查询10020条记录然后只返回最后20条，前面10000条记录都将被抛弃，此时的代价非常高。优化此类分页查询的一个最简单的办法就是尽可能的使用索引覆盖扫描，而不是查询所有的列。然后根据需要做一次关联操作再返回所需的列。对于偏移量很大的时候，这样做的效率会提升非常大。

### UNION

mysql总是通过创建并填充临时表的方式来执行`UNION`查询，因此很多优化的策略在`UNION`查询中都没法很好的使用。经常需要手工的将`WHERE`、`LIMIT`、`ORDER BY`等子句下推到`UNION`的各个子查询中（例如，直接将这些子句冗余的写一份到各个子查询）。
<br>
<br>
除非确实需要服务器消除冗余的行，否则就一定要使用`UNION ALL`。如果没有`ALL`关键字，mysql会给临时表加上`DISTINCT`选项，这会导致对整个临时表的数据做唯一性检查，这样做的代价很高。
<br>
<br>
`UNION`内部的`SELECT`语句必须拥有相同数量的列，列也必须拥有相似的数据类型。同时，每条`SELECT`语句中列的顺序必须相同。另外，`UNION`结果集中的列名总是等于`UNION`中第一个`SELECT`语句中的列名，示例如下所示：

```sql
mysql> SELECT id, classifyname AS classify, userid FROM tb_classify_s;
+----+--------------+--------+
| id | classify     | userid |
+----+--------------+--------+
|  1 | 中国文学     |      1 |
|  2 | 日本文学     |      2 |
|  8 | 西方文学     |      4 |
+----+--------------+--------+
3 rows in set (0.00 sec)
```

```sql
mysql> SELECT id, classifyname AS name, userid FROM tb_classify;
+----+-----------------+--------+
| id | name            | userid |
+----+-----------------+--------+
|  2 | 东方文学        |      1 |
|  3 | 东西方文学      |      2 |
|  8 | 西方文学        |      3 |
+----+-----------------+--------+
3 rows in set (0.00 sec)
```

```sql
mysql> SELECT id, classifyname AS classify FROM tb_classify_s UNION SELECT id, classifyname AS name FROM tb_classify;
+----+-----------------+
| id | classify        |
+----+-----------------+
|  1 | 中国文学        |
|  2 | 日本文学        |
|  8 | 西方文学        |
|  2 | 东方文学        |
|  3 | 东西方文学      |
+----+-----------------+
5 rows in set (0.00 sec)
```

```sql
mysql> SELECT id, classifyname AS classify FROM tb_classify_s UNION ALL SELECT id, classifyname AS name FROM tb_classify;
+----+-----------------+
| id | classify        |
+----+-----------------+
|  1 | 中国文学        |
|  2 | 日本文学        |
|  8 | 西方文学        |
|  2 | 东方文学        |
|  3 | 东西方文学      |
|  8 | 西方文学        |
+----+-----------------+
6 rows in set (0.00 sec)
```

上述两个查询中，`UNION`只会选取不同的值，`UNION ALL`会获取所有的值。

### 自定义变量

`自定义变量`是一个用来存储内容的临时容器，在连接mysql的整个过程中都存在，可以在任何允许使用表达式的地方使用这些`自定义变量`，示例如下：

```sql
SET @target := 5;
SET @max_id := (SELECT MAX(id) FROM tb_classify);

SELECT @target;
SELECT @max_id;
SELECT id, classifyname, userid FROM tb_classify WHERE id > @target;
SELECT id, classifyname, userid FROM tb_classify WHERE id = @max_id;
```

不能使用自定义变量的情景：
* 使用自定义变量的查询，无法使用查询缓存
* 不能在使用常量或者标识符的地方使用自定义变量，如表名、列名和`LIMIT`子句中
* 自定义变量的生命周期是在一个连接中有效，所以不能用它们来做连接间的通信
* 如果使用连接池或者持久化连接，自定义变量可能让看起来毫无关系的代码发生交互（通常是代码bug或者连接池bug）
* 需要注意在不同mysql版本间的兼容性问题
* 不能显式的声明自定义变量的类型，如果希望变量是`整数`类型，最好在初始化时就赋值`0`，`浮点型`赋值为`0.0`，`字符串`赋值为`''`，自定义变量的类型在赋值时就会改变，mysql的自定义变量是一个动态类型
* mysql优化器在某些场景下可能会将这些变量优化掉，导致代码不按预想的方式运行
* 赋值的顺序和赋值的时间点并不总是固定的，这依赖于优化器的决定
* 赋值符号`:=`的优先级非常低，赋值表达式应该使用明确的括号
* `使用未定义变量不会产生任何语法错误`

使用自定义变量的一个最常见的问题就是没有注意到在赋值和读取变量的时候可能是在查询的不同阶段。例如，在`SELECT`子句中进行赋值然后在`WHERE`子句中读取变量，变量取值并不如所想，示例如下：

```sql
mysql> SET @target := 0;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT id, classifyname, userid, @target := @target + 1 AS cnt FROM tb_classify WHERE @target <= 1;
+----+-----------------+--------+------+
| id | classifyname    | userid | cnt  |
+----+-----------------+--------+------+
|  2 | 东方文学        |      1 |    1 |
|  3 | 东西方文学      |      2 |    2 |
+----+-----------------+--------+------+
2 rows in set (0.00 sec)
```

原因在于`WHERE`和`SELECT`是在查询执行的不同阶段被执行，解决此问题的办法是让变量的赋值和取值发生在执行查询的同一阶段：

```sql
mysql> SET @target := 0;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT id, classifyname, userid, @target AS cnt FROM tb_classify WHERE (@target := @target + 1) <= 1;
+----+--------------+--------+------+
| id | classifyname | userid | cnt  |
+----+--------------+--------+------+
|  2 | 东方文学     |      1 |    1 |
+----+--------------+--------+------+
1 row in set (0.00 sec)
```

自定义变量用于`UNION`的示例，假设需要编写一个`UNION`查询，其第一个子查询作为分支条件先执行，如果找到了匹配的行，则跳过第二个分支：

```sql
mysql> SELECT GREATEST(@found := -1, id) AS id, classifyname, 'classify' AS source FROM tb_classify WHERE id = '8'
    -> UNION ALL
    -> SELECT id, classifyname, 'classify_s' AS source_s FROM tb_classify_s WHERE id = '8' AND @found IS NULL
    -> UNION ALL
    -> SELECT 1, 'reset', 'reset' FROM DUAL WHERE (@found := NULL) IS NOT NULL;
+----+--------------+----------+
| id | classifyname | source   |
+----+--------------+----------+
|  8 | 西方文学     | classify |
+----+--------------+----------+
1 row in set (0.00 sec)
```

其中，`GREATEST(value1,value2,...)`是横向取几列的最大值，区别于`MAX(expr)`从一列中纵向取最大值；`DUAL`是虚拟表，用于满足`SELECT ... FROM ... `格式的需要。
<br>
<br>
上述示例查询中，一旦在第一个表中找到记录，就定义变量`@found`——通过在结果列中做一次赋值来实现，然后将赋值放在函数`GREATEST()`中以避免返回额外的数据。为了明确结果的来源，新增了一个区分列。最后，在查询的末尾将变量重置为`NULL`，保证遍历时不干扰后面的结果。

* [参考：SQL教程](http://www.w3school.com.cn/sql/index.asp)