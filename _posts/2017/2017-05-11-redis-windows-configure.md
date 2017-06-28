---
layout: post
title: "redis在windows平台的配置及常用命令"
categories: redis
tags: redis windows nosql
author: Kopite
---

* content
{:toc}


redis`版本号Redis server v=3.2.100`在windows平台的配置。



## 配置

* 将压缩包解压后，运行命令行并切换至解压目录
* 运行命令`D:\Redis-x64-3.2.100>redis-server.exe  redis.windows.conf`，启动redis服务
* 测试，双击打开`D:\Redis-x64-3.2.100\redis-cli.exe`，运行`127.0.0.1:6379> set name songyushi`，在设置密码的情况下会报`(error) NOAUTH Authentication required.`
* 输入`127.0.0.1:6379> help`可以查看帮助信息
* 关闭服务，直接关闭窗口即可
* 在解压目录下可以通过输入以下命令，将redis安装为windows服务，使其开机自启动
```
D:\Redis-x64-3.2.100>redis-server --service-install redis.windows.conf
```

* 在执行完上一条指令后，redis即已作为windows服务，但此时redis服务并未启动，需要输入以下命令
```
D:\Redis-x64-3.2.100>redis-server --service-start
[17060] 11 May 17:07:55.404 # Redis service successfully started.
```
* 停止redis服务，在解压路径下使用命令`redis-server --service-stop`
* 从windows中卸载redis服务，在解压路径下使用命令`redis-server –service-uninstall`
* 命令行切换至解压目录下，运行登录命令`redis-cli.exe -h 127.0.0.1 -p 6379`
```
D:\Redis-x64-3.2.100>redis-cli.exe -h 127.0.0.1 -p 6379 //未设置redis密码情况下
127.0.0.1:6379> config get requirepass //获取redis密码
```

## 设置密码

* 打开`D:\Redis-x64-3.2.100\redis.windows.conf`文件，设置`requirepass`属性 
```
requirepass redisadmin //此处行前不能有空格
```

* 设置密码后，需要重新登录
```
D:\Redis-x64-3.2.100>redis-cli.exe -h 127.0.0.1 -p 6379 -a redisadmin
127.0.0.1:6379> config get requirepass //获取redis密码
1) "requirepass"
2) "redisadmin"
```

## 主从复制

此处以一个master、一个slave为例来实现主从复制
* 将压缩包解压至两个不同路径，一个用来存放master，另一个存放slave
* 修改master`D:\Redis-x64-3.2.100\redis.windows.conf`中的`bind`属性

```
bind 127.0.0.1 //指定redis只接收来自于该ip的请求，如果不进行设置，将处理所有请求
```

* 修改slave`D:\RedisSlave\redis.windows.conf`中的以下属性

```
port 6380


bind 127.0.0.1


requirepass redisslaveadmin //slave的密码


slaveof 127.0.0.1 6379 //设置该数据库为其他数据库的从数据库，master的host及port


masterauth redisadmin //master连接需要密码验证，master的密码
```

* 将slave安装为windows服务，使其开机自启动，进入解压路径运行以下指令

```
redis-server --service-install redis.windows.conf --service-name redisslave --port 6380 //安装


redis-server --service-start --service-name redisslave //启动


redis-server --service-stop --service-name redisslave //停止


redis-server --service-uninstall --service-name redisslave //卸载
```

* 登录，测试

```
D:\Redis-x64-3.2.100>redis-cli.exe -h 127.0.0.1 -p 6379 -a redisadmin //master
127.0.0.1:6379> set name songyushi
OK
127.0.0.1:6379> get name
"songyushi"

D:\RedisSlave>redis-cli.exe -h 127.0.0.1 -p 6380 -a redisslaveadmin //slave
127.0.0.1:6380> get name
"songyushi"
```

## 常用命令


* [参考：redis命令参考](http://doc.redisfans.com/)