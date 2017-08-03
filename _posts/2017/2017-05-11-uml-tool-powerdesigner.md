---
layout: post
title: "UML建模工具之powerdesigner的安装及使用"
categories: UML
tags: UML powerdesigner
author: Kopite
---

* content
{:toc}


powerdesigner`版本号16.5`的安装及使用。



## 安装及破解

* 工具：`powerdesigner.exe 版本号16.5`、`pdflm16.dll`
* 双击运行powerdesigner安装包，选择`Next`
* `Please select the location...`中选择`Hong Kong`，`I Agree the terms...`
* `Destination directory`中可以修改安装路径，选择`Next`
* 选择`Next`
* 根据需要勾选General、Notation中的选项，然后选择`Next`
* 选择`Next...`，最后选择`Finish`
* 复制破解工具`pdflm16.dll`并替换powerdesigner安装路径下的该文件

## 数据库设计

* File菜单中选择`New Model`
* `Model type：`Physical Data Model，`Diagram：`Physical Diagram，`DBMS：`中选择MySQL 5.0，`Model name：`中输入名称
* 使用Toolbox中Physical Diagram模块的`Table工具`建表
* 双击创建的表，在弹出的General选项卡中输入表的名称及注释，如下图所示
![](/image/2017/2017-05-11-uml-tool-powerdesigner-1.jpg)
* 在Columns选项卡中添加字段，设置字段名称、类型、长度等信息，如下图所示
![](/image/2017/2017-05-11-uml-tool-powerdesigner-2.jpg)
* 为选中字段设置注释，单击Columns选项左上角的Properties，如下图所示
![](/image/2017/2017-05-11-uml-tool-powerdesigner-3.jpg)
* 使用Toolbox中Physical Diagram模块的`Reference工具`为表间建立关系，由从表拖向主表，双击关系，如下图所示<br>
![](/image/2017/2017-05-11-uml-tool-powerdesigner-4.png)
![](/image/2017/2017-05-11-uml-tool-powerdesigner-5.png)
* 最后，得到的Physical Diagram如下图所示
![](/image/2017/2017-05-11-uml-tool-powerdesigner-6.png)
* 导出sql脚本，Database菜单中选择`Generate Database`，为避免中文注释乱码还需要在`Format`选项卡中选择`Encoding`<br>
![](/image/2017/2017-05-11-uml-tool-powerdesigner-7.png)

* [参考：powerdesigner破解步骤](http://jingyan.baidu.com/article/1974b289a836ccf4b1f77436.html)
* [参考：powerdesigner生成sql脚本](http://blog.csdn.net/kunkun378263/article/details/43701231)
* [下载：pdflm16.dll工具](http://pan.baidu.com/s/1i5spIKl)