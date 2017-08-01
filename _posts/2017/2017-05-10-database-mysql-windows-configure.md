---
layout: post
title: "mysql在windows平台的安装和配置"
categories: Database
tags: Database mysql windows
author: Kopite
---

* content
{:toc}


mysql`版本号5.6.26 MySQL Community Server (GPL)`在windows平台的安装配置，在变量`Path`的变量值中添加`D:\mysql\mysql-5.6.26-winx64\bin;`。



## 添加my.ini文件方式

* 在mysql解压根目录中添加my.ini文件，注意`basedir`及`datadir`参数
```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
# 设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=D:\mysql\mysql-5.6.26-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\mysql\mysql-5.6.26-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB 
# 大小写敏感
lower_case_table_names = 0
```

* 以管理员身份启动命令行，切换至\bin目录下，按顺序输入以下命令
```
mysqld -install mysql
```
```
net start mysql //正常会显示mysql服务正在启动 mysql服务已经启动成功
```
```
mysql -u root –p //第一次登录没有密码，按回车键即可
```
```
mysqladmin -u root password admin //为数据库设置密码，不需要登录mysql
```
```
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('admin'); //修改密码，需要登录mysql
```

### The service already exists

报错的原因是之前安装过mysql服务并未删掉，打开命令行输入以下命令，查看是否有名为mysql的服务
```
C:\Users\SOYU>sc query mysql

SERVICE_NAME: mysql
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
如果有mysql服务，运行`sc delete mysql`命令删掉即可

## 不添加my.ini文件方式

* 以管理员身份运行命令行，切换至\bin目录下安装服务

## 常用命令

* 查看mysql数据库中的所有用户

```sql
mysql> SELECT DISTINCT CONCAT('User: ''', user, '''@''', host, ''';') AS query FROM mysql.user;
+---------------------------+
| query                     |
+---------------------------+
| User: 'root'@'127.0.0.1'; |
| User: 'root'@'::1';       |
| User: ''@'localhost';     |
| User: 'root'@'localhost'; |
+---------------------------+
4 rows in set (0.00 sec)
```

* 查看当前登录用户

```sql
mysql> SELECT USER();
+----------------+
| USER()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

* 查看某个用户的权限

```sql
mysql> SHOW GRANTS FOR 'root'@'localhost'\G
*************************** 1. row ***************************
Grants for root@localhost: GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*4ACFE3202A5FF5CF467898FC58AAB1D615029441' WITH GRANT OPTION
*************************** 2. row ***************************
Grants for root@localhost: GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION
2 rows in set (0.00 sec)
```

```sql
mysql> SELECT * FROM mysql.user WHERE user = 'root'\G
*************************** 1. row ***************************
                  Host: localhost
                  User: root
              Password: *4ACFE3202A5FF5CF467898FC58AAB1D615029441
           Select_priv: Y
           Insert_priv: Y
           Update_priv: Y
           Delete_priv: Y
           Create_priv: Y
             Drop_priv: Y
           Reload_priv: Y
         Shutdown_priv: Y
          Process_priv: Y
             File_priv: Y
            Grant_priv: Y
       References_priv: Y
            Index_priv: Y
            Alter_priv: Y
          Show_db_priv: Y
            Super_priv: Y
 Create_tmp_table_priv: Y
      Lock_tables_priv: Y
          Execute_priv: Y
       Repl_slave_priv: Y
      Repl_client_priv: Y
      Create_view_priv: Y
        Show_view_priv: Y
   Create_routine_priv: Y
    Alter_routine_priv: Y
      Create_user_priv: Y
            Event_priv: Y
          Trigger_priv: Y
Create_tablespace_priv: Y
              ssl_type:
            ssl_cipher:
           x509_issuer:
          x509_subject:
         max_questions: 0
           max_updates: 0
       max_connections: 0
  max_user_connections: 0
                plugin: mysql_native_password
 authentication_string:
      password_expired: N
```

* 查看当前数据库

```sql
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| test       |
+------------+
1 row in set (0.00 sec)
```

* 查看当前会话的事务隔离级别

```sql
mysql> SELECT @@SESSION.tx_isolation;
+------------------------+
| @@SESSION.tx_isolation |
+------------------------+
| REPEATABLE-READ        |
+------------------------+
1 row in set (0.00 sec)
```

* 查看全局的事务隔离级别

```sql
mysql> SELECT @@GLOBAL.tx_isolation;
+-----------------------+
| @@GLOBAL.tx_isolation |
+-----------------------+
| REPEATABLE-READ       |
+-----------------------+
1 row in set (0.00 sec)
```

## 锁的调试

对于一个mysql连接，或者说一个线程，任何时刻都有一个状态，该状态表示了mysql当前正在做什么。`在一个查询的生命周期中，状态会变化很多次`。有很多种方式能查看当前的状态，最简单的是使用`SHOW PROCESSLIST`/`SHOW FULL PROCESSLIST`命令，该命令返回结果中的`Command`列就表示当前的状态：
* Sleep，线程正在等待客户端发送新的请求
* Query，线程正在执行查询或者正在将结果发送给客户端
* Locked，在mysql服务器层，该线程正在等待表锁。在存储引擎级别实现的锁，例如`InnoDB`的行锁，并不会体现在线程状态中。对于`MyISAM`来说这是一个比较典型的状态，但在其它没有行锁的引擎中也经常会出现
* Analyzing and statistics，线程正在收集存储引擎的统计信息，并生成查询的执行计划
* Coping to tmp table [on disk]，线程正在执行查询，并且将其结果集都复制到一个临时表中，这种状态一般要么是在做GROUP BY操作，要么是文件排序操作，或者是UNION操作。如果这个状态后面还有on disk标记，那表示mysql正在将一个内存临时表放到磁盘上
* Sorting result，线程正在对结果集进行排序
* Sending data，表示多种情况：线程可能在多个状态间传送数据，或者在生成结果集，或者在向客户端返回数据

```sql
mysql> SHOW PROCESSLIST;
+----+------+----------------+------+---------+------+-------+------------------
+
| Id | User | Host           | db   | Command | Time | State | Info
|
+----+------+----------------+------+---------+------+-------+------------------
+
|  1 | root | localhost:4044 | NULL | Sleep   | 1798 |       | NULL
|
|  2 | root | localhost:4045 | test | Sleep   | 1796 |       | NULL
|
|  3 | root | localhost:4046 | test | Sleep   | 1794 |       | NULL
|
|  4 | root | localhost:4047 | test | Sleep   |  257 |       | NULL
|
|  5 | root | localhost:4687 | NULL | Query   |    0 | init  | SHOW PROCESSLIST
|
+----+------+----------------+------+---------+------+-------+------------------
+
5 rows in set (0.00 sec)
```

使用`SHOW PROCESSLIST`可以查看正在运行的线程及各个线程的执行状态，但只显示前100条。查看当前全部线程，可使用`SHOW FULL PROCESSLIST`：

```sql
mysql> SHOW FULL PROCESSLIST;
+----+------+----------------+------+---------+------+-------+------------------
-----+
| Id | User | Host           | db   | Command | Time | State | Info
     |
+----+------+----------------+------+---------+------+-------+------------------
-----+
|  1 | root | localhost:4044 | NULL | Sleep   | 1843 |       | NULL
     |
|  2 | root | localhost:4045 | test | Sleep   | 1841 |       | NULL
     |
|  3 | root | localhost:4046 | test | Sleep   | 1839 |       | NULL
     |
|  4 | root | localhost:4047 | test | Sleep   |  302 |       | NULL
     |
|  5 | root | localhost:4687 | NULL | Query   |    0 | init  | SHOW FULL PROCESSLIST |
+----+------+----------------+------+---------+------+-------+------------------
-----+
5 rows in set (0.00 sec)
```

死锁时，可通过`KILL Id`命令杀死指定的连接：

```sql
mysql> KILL 7;
Query OK, 0 rows affected (0.00 sec)
```

服务器级别的锁要比存储引擎中的锁容易调试的多，各个存储引擎的锁互不相同，并且存储引擎可能不提供任何方法来查看内部的锁，InnoDB在`SHOW ENGINE INNODB STATUS`的输出中显露了一些锁信息：

```sql
mysql> SHOW ENGINE INNODB STATUS\G
*************************** 1. row ***************************
  Type: InnoDB
  Name:
Status:
=====================================
2017-06-22 17:52:03 b04 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 15 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 0 srv_active, 0 srv_shutdown, 9003 srv_idle
srv_master_thread log flush and writes: 9003
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 3
OS WAIT ARRAY INFO: signal count 3
Mutex spin waits 9, rounds 5, OS waits 0
RW-shared spins 3, rounds 90, OS waits 3
RW-excl spins 0, rounds 0, OS waits 0
Spin rounds per wait: 0.56 mutex, 30.00 RW-shared, 0.00 RW-excl
------------
TRANSACTIONS
------------
Trx id counter 33284
Purge done for trx's n:o < 32817 undo n:o < 0 state: running but idle
History list length 422
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 0, not started
MySQL thread id 13, OS thread handle 0xb04, query id 289 localhost ::1 root init

SHOW ENGINE INNODB STATUS
---TRANSACTION 0, not started
MySQL thread id 10, OS thread handle 0x12b4, query id 279 localhost 127.0.0.1 root cleaning up
---TRANSACTION 0, not started
MySQL thread id 2, OS thread handle 0x38c, query id 254 localhost 127.0.0.1 root cleaning up
---TRANSACTION 33283, not started
MySQL thread id 8, OS thread handle 0x13cc, query id 256 localhost 127.0.0.1 root cleaning up
--------
FILE I/O
--------
I/O thread 0 state: wait Windows aio (insert buffer thread)
I/O thread 1 state: wait Windows aio (log thread)
I/O thread 2 state: wait Windows aio (read thread)
I/O thread 3 state: wait Windows aio (read thread)
I/O thread 4 state: wait Windows aio (read thread)
I/O thread 5 state: wait Windows aio (read thread)
I/O thread 6 state: wait Windows aio (write thread)
I/O thread 7 state: wait Windows aio (write thread)
I/O thread 8 state: wait Windows aio (write thread)
I/O thread 9 state: wait Windows aio (write thread)
Pending normal aio reads: 0 [0, 0, 0, 0] , aio writes: 0 [0, 0, 0, 0] , ibuf aio reads: 0, log i/o's: 0, sync i/o's: 0
Pending flushes (fsync) log: 0; buffer pool: 0
800 OS file reads, 242 OS file writes, 10 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 0, seg size 2, 0 merges
merged operations: insert 0, delete mark 0, delete 0
discarded operations: insert 0, delete mark 0, delete 0
Hash table size 276707, node heap has 3 buffer(s)
0.00 hash searches/s, 0.00 non-hash searches/s
---
LOG
---
Log sequence number 344413232
Log flushed up to   344413232
Pages flushed up to 344413232
Last checkpoint at  344413232
0 pending log writes, 0 pending chkp writes
9 log i/o's done, 0.00 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total memory allocated 137428992; in additional pool allocated 0
Dictionary memory allocated 88710
Buffer pool size   8192
Free buffers       7759
Database pages     430
Old database pages 0
Modified db pages  0
Pending reads 0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 0, not young 0
0.00 youngs/s, 0.00 non-youngs/s
Pages read 430, created 0, written 233
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
No buffer pool page gets since the last printout
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s

LRU len: 430, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Main thread id 1232, state: sleeping
Number of rows inserted 0, updated 0, deleted 0, read 14
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================

1 row in set (0.00 sec)
```

## SQL_MODE

mysql能够工作在不同的`SQL_MODE`模式下，`SQL_MODE`定义了mysql支持的sql语法，以及对数据执行何种确认检查。可配置在`my.cnf`(Linux系统)或`my.ini`文件中，也可在客户端中进行配置，并可分别通过全局、当前会话方式进行配置。

### 简介

* sql语法支持类：
  * `ONLY_FULL_GROUP_BY`，对于`GROUP BY`聚合操作，如果在`SELECT`中的列、`HAVING`或者`ORDER BY`子句中的列没有在`GROUP BY`中出现，那么这个sql不合法
  * `ANSI_QUOTES`，启用后不能用双引号来引用字符串，因为双引号将被解释为识别符
  * `PIPES_AS_CONCAT`，启用后将视`||`为字符串的连接操作符，而非`或`运算符
  * `NO_TABLE_OPTIONS`
  * `NO_AUTO_CREATE_USER`，禁止`GRANT`创建密码为空的用户

* 数据检查类：
  * `NO_ZERO_DATE`，启用后认为形如0000-00-00 00:00:00此类的零日期非法，与是否设置严格模式有关
    1. 如果设置了严格模式，则`NO_ZERO_DATE`自然满足，但如果是`INSERT IGNORE`或`UPDATE IGNORE`，此类零日期依然允许且只显示警告
    2. 如果在非严格模式下，设置了`NO_ZERO_DATE`，效果与`1.`一样，允许零值但显示警告。倘若没有设置`NO_ZERO_DATE`时，零值将被当作完全合法的值
  * `NO_ZERO_IN_DATE`，与`NO_ZERO_DATE`类似，不同的是控制月份和天是否可为零值，例如， 2010-01-00是否合法
  * `NO_ENGINE_SUBSTITUTION`，启用后，使用`ALTER TABLE`或`CREATE TABLE`指定存储引擎时，若需要的存储引擎被禁用或未编译，将抛出错误。不设置此值时，`CREATE TABLE`用默认的存储引擎替代，`ALTER TABLE`不进行更改并抛出一个警告
  * `STRICT_TRANS_TABLES`，表示只对支持事务的表启用严格模式，`STRICT_TRANS_TABLES`并不是几种模式的组合
  * `STRICT_ALL_TABLES`，表示对所有存储引擎的表都启用严格模式

常设置的SQL_MODE：`ANSI`、`TRADITIONAL`、`STRICT_TRANS_TABLES`，其中ANSI和TRADITIONAL是组合项：
* `ANSI`：REAL_AS_FLOAT, PIPES_AS_CONCAT, ANSI_QUOTES, IGNORE_SPACE
* `TRADITIONAL`：STRICT_TRANS_TABLES, STRICT_ALL_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_AUTO_CREATE_USER, NO_ENGINE_SUBSTITUTION

无论何种SQL_MODE，出现错误就意味着单条sql执行失败，对于支持事务的表，将导致当前事务回滚；如果没有在事务中执行，或者存储引擎不支持事务的表，可能导致数据不一致。mysql认为，相比直接报错终止，数据不一致的问题更严重。

### 查看

* 查看当前会话的SQL_MODE配置：

```sql
mysql> SELECT @@SESSION.SQL_MODE;
+------------------------+
| @@SESSION.SQL_MODE     |
+------------------------+
| NO_ENGINE_SUBSTITUTION |
+------------------------+
1 row in set (0.00 sec)

mysql> SHOW VARIABLES LIKE 'SQL_MODE'; //通过SHOW VARIABLES LIKE 命令
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| sql_mode      | NO_ENGINE_SUBSTITUTION |
+---------------+------------------------+
1 row in set (0.00 sec)
```

* 查看全局的SQL_MODE配置：

```sql
mysql> SELECT @@GLOBAL.SQL_MODE;
+------------------------+
| @@GLOBAL.SQL_MODE      |
+------------------------+
| NO_ENGINE_SUBSTITUTION |
+------------------------+
1 row in set (0.00 sec)
```

### 配置

* 配置当前会话的SQL_MODE，仅对当前会话有效：

```sql
mysql> SET SESSION SQL_MODE = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN
_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBS
TITUTION';
Query OK, 0 rows affected, 3 warnings (0.02 sec)

mysql> SELECT @@SESSION.SQL_MODE;
+-------------------------------------------------------------------------------
------------------------------------------------------------+
| @@SESSION.SQL_MODE
                                                            |
+-------------------------------------------------------------------------------
------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_
DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------
------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT @@global.SQL_MODE;
+------------------------+
| @@global.SQL_MODE      |
+------------------------+
| NO_ENGINE_SUBSTITUTION |
+------------------------+
1 row in set (0.00 sec)
```

* 配置全局的SQL_MODE，重启电脑后该全局配置依然有效，并且会话的SQL_MODE也变为此全局配置：

```sql
mysql> SET GLOBAL SQL_MODE = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_
DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBST
ITUTION';
Query OK, 0 rows affected, 3 warnings (0.00 sec)

mysql> SELECT @@GLOBAL.SQL_MODE;
+-------------------------------------------------------------------------------
------------------------------------------------------------+
| @@GLOBAL.SQL_MODE
                                                            |
+-------------------------------------------------------------------------------
------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_
DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------
------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT @@SESSION.SQL_MODE;
+------------------------+
| @@SESSION.SQL_MODE     |
+------------------------+
| NO_ENGINE_SUBSTITUTION |
+------------------------+
1 row in set (0.00 sec)
```

* [参考：官方手册](https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html)
* [参考：mysql SQL_MODE说明](http://seanlook.com/2016/04/22/mysql-sql-mode-troubleshooting/)
* [参考：mysql SQL_MODE详解](http://blog.itpub.net/29773961/viewspace-1813501/)
* [参考：mysql SQL_MODE修改](http://blog.csdn.net/wulantian/article/details/8905573)