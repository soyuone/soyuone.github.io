---
layout: post
title: "使用TortoiseSVN进行协作开发"
categories: 版本控制
tags: 版本控制 windows
author: Kopite
---

* content
{:toc}


对于普通开发者而言，通常会选择使用`TortoiseSVN`作为SVN的客户端，本文对TortoiseSVN`版本号1.9.6`在windows平台的使用进行整理。
<br>
<br>
TortoiseSVN与普通软件不同，它并未提供任何窗口来执行版本管理操作，只是一个`Shell`扩展，并且已经被整合到windows资源管理器中，使用TortoiseSVN时，只需要右键单击任何文件夹、文件，即可弹出TortoiseSVN工具菜单。



## 下载和安装

* 登录[TortoiseSVN官方网站](https://tortoisesvn.net/)，选择合适的TortoiseSVN版本进行下载，此处选择下载`1.9.6版本`，下载后得到`TortoiseSVN-1.9.6.27867-x64-svn-1.9.6.msi`文件
* `Next` -> `Next`
* 选择安装路径
* 单击`Install`，开始安装
* 单击`Finish`，安装完成

## 建立SVN资源库

* 新建文件夹，例如`E:\svnRepository`
* 打开文件夹，在空白处右键选择`TortoiseSVN` -> `Create repository here`
* 弹出对话框，提示创建成功，单击`OK`，自动创建目录结构，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-1.png)

## 将项目发布到服务器

使用TortoiseSVN将项目发布到服务器，假设在磁盘上开发了一个`svnProject`项目，现在希望将该项目发布到SVN服务器上，按如下步骤进行：
* 右键单击该项目对应的文件夹，即`D:\workspace\svnProject`
* 在弹出的菜单中，选择`TortoiseSVN` -> `Import`
* 在`URL of repository:`处输入SVN资源库的URL，在`Import message`处填写备注信息，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-2.png)

* 单击`OK`开始发布
* 发布完成后弹出`Import Finished!`提示，可以查看发布的文件，单击`OK`，将项目发布到SVN服务器完成，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-3.png)

## 从服务器下载项目

从服务器下载项目，如果项目组其他开发者想从服务器上下载刚刚发布的`svnProject`项目，按如下步骤进行：
* 进入用于保存待下载项目的目标磁盘空间，例如`D:\workspace\svnDownload`
* 在空白处右键单击，选择`SVN Checkout...`
* 在弹出的`Checkout`对话框`URL of repository:`处输入SVN资源库的URL，在`Checkout directory:`处指定将项目下载到本地磁盘的指定位置，点击`OK`开始下载，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-4.png)

* 项目下载成功后弹出`Checkout Finished!`提示，单击`OK`，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-5.png)

从SVN服务器上下载的项目、文件上都有一个绿色的小钩，这表明该项目、该文件与服务器项目状态一致。

## 同步本地文件

在多人协同工作的环境下，远程资源库中的某些文件可能已经被其他开发者修改过，SVN服务器上保存了各文件修改后的最新版本。同步操作能够把最新版本下载到本地，从而允许在别人修改的最新版本上进行修改，这样既可以避免版本冲突，也可以避免浪费精力和重复劳动。
<br>
<br>
对于多人协同开发的环境，通常推荐总是`先同步后工作`，即每次准备开发之前，总应该先同步一次，从而保证总是在项目的最新版本上进行开发，同步本地文件按如下步骤进行：
* 进入存放项目的文件夹，例如`D:\workspace\svnDownload\svnProject`
* 在空白处右键单击，选择`SVN Update`
* 弹出`Update Finished!`对话框，对话框中会列出所有被同步过的文件，单击`OK`同步完成

## 提交修改

从服务器下载了指定项目之后，开发者就可以对项目进行开发，当项目开发到一定阶段后，接下来应该将自己手中所做的修改提交到服务器，这样其他开发者也能得到该项目最新修改过的版本。
<br>
<br>
SVN项目修改过的文件的图标以及该文件所在文件夹的图标都会增加一个红色的感叹号，这表明该文件或该文件夹被修改过，需要将该修改提交到服务器，提交修改按如下步骤进行：
* 进入存放项目的文件夹，例如`D:\workspace\svnDownload\svnProject`
* 在空白处右键单击，选择`SVN Commit...`
* 在弹出的`Commit`对话框`Message:`处添加注释，在`Changes made(double-click on file for diff):`处选择需要提交的文件
* 单击`OK`开始提交，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-6.png)

* 弹出`Commit Finished`对话框，单击`OK`提交完成，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-7.png)

## 添加文件和目录

随着项目开发的进行，需要向项目中新增一些文件，但新增的文件并不会自动处于SVN的管理之下，例如在`D:\workspace\svnDownload\svnProject\src\main\java\svnProject`路径下新建`service`文件夹、新增`Main.java`文件，新增的文件夹和文件上没有任何标记，这表明还未处于SVN管理之下，需要将该文件添加到SVN中。
<br>
<br>
向SVN中添加文件按如下步骤进行：
* 进入存放项目的文件夹，例如`D:\workspace\svnDownload\svnProject`
* 在空白处右键单击，选择`TortoiseSVN` -> `Add...`
* 在弹出的`Add`对话框中会列出所有需要添加的文件和文件夹，选择需要添加的文件和文件夹，单击`OK`开始添加，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-8.png)

* 在弹出的`Add Finished!`对话框中，单击`OK`添加完成，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-9.png)

`注意：`上述步骤只是将该文件和文件夹置于TortoiseSVN管理之下，还并未提交到服务器。为了将所添加的文件夹、文件提交到服务器，还需要对它们执行提交操作。

## 删除文件和目录

通过TortoiseSVN删除指定的文件夹或文件，按如下步骤进行：
* 选中需要删除的文件夹或文件
* 右键单击选中的文件夹或文件，选择`TortoiseSVN` -> `Delete`，删除成功

`注意：`上述步骤只是从TortoiseSVN管理下、本地磁盘删除了该文件夹或文件，还并未提交到服务器。为了将所做的删除操作提交到服务器，还需要执行提交操作。

## 查看文件或目录的版本变革

TortoiseSVN提供了图形界面方式来查看文件、文件夹的版本变革历史，并比较任意两个版本之间的差异，查看文件或文件夹的版本变革历史按如下步骤进行：
* 选中需要查看的文件或文件夹
* 右键单击选中的文件或文件夹，选择`TortoiseSVN` -> `Fevision graph`，弹出的对话框会显示所选中文件或文件夹的版本历史

## 从以前版本重新开始

TortoiseSVN提供了很方便的操作允许从某个文件的以前版本重新开始开发，从指定文件的以前版本重新开始开发按如下步骤进行：
* 选中需要重新开始的一个或多个文件
* 右键单击选中的文件，选择`TortoiseSVN` -> `Update to revision...`
* 在弹出的对话框中的`Revision`处输入希望重新开始的版本号，单击`OK`，选中的文件将恢复到指定的版本，接下来可以在此版本上继续开发，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-10.png)

* 弹出`Update Finished!`对话框，单击`OK`

## 创建分支

有时会试探性的添加一些新功能，这时就需要在开发主线上创建一个分支（Branch），进而在分支上进行开发，避免损坏原有的稳定版本，创建分支按如下步骤进行：
* 选定需要创建分支的文件或文件夹（可以是整个项目所在的文件夹）
* 右键单击选中的文件或文件夹，选择`TortoiseSVN` -> `Branch/tag...`
* 在弹出的对话框中的`To path:`处指定新分支的名称，单击`OK`，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-11.png)

* 弹出`Copy Finished!`对话框，单击`OK`创建分支完成，可以在该文件的版本变革图中看到该分支的效果，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-12.png)

## 沿着分支开发

为了切换到指定分支继续开发，可按如下步骤进行：
* 选中拥有分支的文件或文件夹
* 右键单击选中的文件或文件夹，选择`TortoiseSVN` -> `Switch...`
* 在弹出的对话框中的`To path:`处指定切换到哪个分支，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-13.png)

* 单击`OK` -> 弹出`Switch Finished!`框 -> 单击`OK`，当前文件就会切换到指定分支，接下来对该文件所做的修改都将沿着该分支开发

例如，切换到`/svnSong`分支后继续对该页面进行修改，修改完成后将所做的修改提交到SVN服务器，再次查看该页面的版本变革历史，如下图所示：

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-14.png)

如果开发者沿着分支开发了一段时间后，想继续维护开发主线上的开发，则还可以切换回开发主线继续开发，再次查看该页面的版本变革历史，如下图所示：

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-15.png)

## 合并分支

当项目沿着分支试探性的开发新功能达到一定的稳定状态后，可以将开发主线和开发分支合并到一起，从而将分支中的新功能添加到开发主线中，合并时按如下操作步骤进行：
* 选中拥有分支的文件或文件夹
* 右键单击选中的文件或文件夹，选择`TortoiseSVN` -> `Switch...`，切换到主线分支
* 右键单击选中的文件或文件夹，选择`TortoiseSVN` -> `Merge...`，弹出如下对话框

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-16.png)

* 选择合适的合并项之后，单击`Next`
* 弹出`Merge`对话框，在`URL to merge from`处指定合并哪个分支，在`specific range`处指定合并哪个版本，如下图所示

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-17.png)

* 单击`Next`，弹出如下对话框

![](/image/2017/2017-07-29-svn-tortoisesvn-windows-18.png)

* 单击`Merge`，开始合并
* 弹出`Merge Finished!`对话框，单击`OK`合并完成

## 使用Eclipse作为SVN客户端

为了使用Eclipse作为SVN客户端，需要为Eclipse安装`Subclipse`插件，在`Eclipse MarketPlace...`中搜索并安装`Subclipse`插件。
<br>
<br>
使用Eclipse从SVN资源库中下载项目按如下步骤进行：
* 单击Eclipse中`File`菜单 -> `Import...`
* `SVN` -> `从SVN检出项目`，表示从SVN资源库中导入项目
* 单击`Next`，弹出`选择/新建位置`对话框，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-19.png)

* 如果是第一次使用Eclipse作为SVN客户端，在上图所示的对话框中并不存在有效的资源库，可以选择`创建新的资源库位置`，表示创建新的资源库，然后单击`NEXT`，将进入资源库属性设置对话框，在`URL:`处输入SVN资源库的URL，例如`file:///E:/svnRepository`，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-20.png)

* 单击`Next`，弹出`选择文件夹`对话框，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-21.png)

* 选择需要下载的文件夹，如果下载整个项目，则直接选中资源库的根结点，然后单击`Next`，将弹出`检出为`对话框，如下图所示<br>
![](/image/2017/2017-07-29-svn-tortoisesvn-windows-22.png)

* 单击`Finish`，即可将项目检出到Eclipse中

如果需要在Eclipse中对一个或多个文件执行同步、提交等常规操作，右键 -> `Team`。

### 建立SVN资源库

* 单击Eclipse中`Window`菜单 -> `Show View` -> `Other...` -> `SVN` -> `SVN资源库` -> 单击`OK`
* 在`SVN 资源库`窗口的空白处，右键单击 -> `新建` -> `资源库位置`，在`URL:`处输入SVN资源库的URL，例如`file:///E:/svnRepository`，单击`Finish`

### 将项目发布到服务器

* 选中需要上传的项目，右键 -> `Team` -> `Share Project...`
* 选择`SVN`，单击`Next`
* ...

### 从服务器下载项目

* 在`SVN 资源库`窗口中，选中需要检出的项目，右键单击 -> `检出为...`

### 提交修改

* 选中需要提交的项目，右键 -> `Team` -> `提交...`

### 删除SVN资源库中的项目

* 在`SVN 资源库`窗口中，选中要删除的项目，右键 -> `删除`

* [参考：TortoiseSVN的下载、安装以及常用操作](https://jingyan.baidu.com/article/358570f6638aa4ce4724fcf7.html)
* [参考：如何在Eclipse中使用SVN](https://jingyan.baidu.com/article/2c8c281daaeaaa0009252a64.html)