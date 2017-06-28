---
layout: post
title: "VMware Workstation安装CentOS"
categories: Linux
tags: Linux CentOS VMware
author: Kopite
---

* content
{:toc}


VMware Workstation`版本号10.0.7 build-2844087`安装CentOS`版本号CentOS-7-x86_64-DVD-1511`。



## 虚拟机配置

* 工具：VMware Workstation`版本号10.0.7 build-2844087`、`CentOS-7-x86_64-DVD-1511.iso`
* 启动VMware Workstation，单击`选择创建新的虚拟机`
* 弹出`新建虚拟机向导`窗口，选择`自定义(高级)`
* `硬件兼容性(H):`使用默认项`Workstation 10.0`，点击`下一步`
* 选择`稍后安装操作系统`，点击`下一步`
* `客户机操作系统`中选择`Linux(L)`，版本选择`CentOS 64 位`，点击`下一步`
* `虚拟机名称(V):`中填入`CentOS`，`位置(L):`中选择虚拟机文件的保存位置，点击`下一步`
* `处理器数量(P):`表示虚拟机使用CPU的个数，一般是1个，`每个处理器的核心数量(C):`表示虚拟机使用的CPU是几核，此处使用默认参数
* `此虚拟机的内存(M):`使用`2048MB`，不能超过`最大推荐内存`，点击`下一步`
* `网络连接`中选择`网络地址转换(NAT)`，点击`下一步`
* `I/O控制器类型`中使用推荐项`LSI Logic(L)`，点击`下一步`
* `虚拟磁盘类型`中使用推荐项`SCSI(S)`，点击`下一步`
* `磁盘`中选择`创建新虚拟磁盘`，点击`下一步`
* `最大磁盘大小(GB)`中使用默认值`20GB`，选择`将虚拟磁盘存储为单个文件`，点击`下一步`
* `磁盘文件`中使用默认值`CentOS.vmdk`，点击`下一步`
* 点击`完成`

![](/image/2017/2017-06-19-centos-linux-vmware-workstation-1.png)

## 安装CentOS 7

* `我的计算机`中选中`CentOS`，`虚拟机`下拉菜单中选择`设置`
* `CD/DVD(IDE)`项中选择`使用ISO映像文件`，单击`确定`
* 单击`启动此客户机操作系统`，出现如下提示

![](/image/2017/2017-06-19-centos-linux-vmware-workstation-2.png)

![](/image/2017/2017-06-19-centos-linux-vmware-workstation-3.png)

进入BIOS，进入`Configuration`菜单，`Intel Virtual Technology`项的值改为`Enabled`

* 单击`启动此客户机操作系统`，提示`键盘挂钩超时值未设置为VMware Workstation建议的值...`，关闭CentOS安装程序，`虚拟机`下拉菜单中选择`设置`，切换至`选项`，`常规`菜单中`增强型虚拟键盘`项改为`在可用时使用(推荐)`，单击`确定`，备份并删除CentOS虚拟机目录下后缀为`.vmx.lck`的文件夹；依旧提示时选择`确定`
* 选择`Test this media & install CentOS 7`，回车键确认
* 回车键开始安装
* `您在安装过程中想使用哪种语言？`中选择`中文`、`简体`，单击`继续`
* `软件`->`软件选择`，选中GNOME桌面，如下图所示，单击`完成`

![](/image/2017/2017-06-19-centos-linux-vmware-workstation-4.png)

* `系统`->`安装位置`->`其他存储选项`->`我要配置分区`，单击`完成`，`新CentOS 7安装`->`点这里自动创建他们(C)`，如下图所示

![](/image/2017/2017-06-19-centos-linux-vmware-workstation-5.png)

* 单击`完成`，`接受更改`
* 开启网络，`系统`->`网络和主机名`，单击`开启`，单击`完成`，如下图所示

![](/image/2017/2017-06-19-centos-linux-vmware-workstation-6.png)

* 选择`开始安装`
* 设置`ROOT密码`，单击`完成`，`创建用户`中输入`全名`，选中`将此用户做为管理员`，单击`完成`
* 安装完成后，单击`重启`
* 接受许可信息
* 安装完成

* [参考：VMware Workstation安装CentOS](http://blog.csdn.net/alex_my/article/details/38142229)
* [参考：在虚拟机中安装CentOS](https://jingyan.baidu.com/article/eae0782787b4c01fec548535.html)
* [参考：解决VMware中虚拟机中无法使用键盘的故障](https://jingyan.baidu.com/article/d713063525f54113fdf475ed.html)