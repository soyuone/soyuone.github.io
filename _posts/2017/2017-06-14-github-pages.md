---
layout: post
title: "在github pages搭建个人主页"
categories: github
tags: github pages
author: Kopite
---

* content
{:toc}


本文对在[github pages](https://pages.github.com/)上搭建个人主页的流程进行整理。



## 安装Ruby

Ruby`版本号rubyinstaller-2.2.6-x64`在windows系统的安装教程：
* 双击安装包
* 选择语言`English`
* 接受证书的内容
* 选择安装路径，勾选`Add Ruby executables to your PATH`
* 点击Finish，安装完成
* 打开命令行，输入`ruby -v`，查看安装版本

```
C:\Users\SO>ruby -v
ruby 2.2.6p396 (2016-11-15 revision 56800) [x64-mingw32]
```

## RubyGems

Ruby安装完成后，程序中已经包含了RubyGems，从开始菜单找到`Start Command Prompt with Ruby`：
* 输入`gem update --system`对RubyGems进行更新
* 输入`gem -v`，查看版本号
* 输入`gem sources -l`，查看当前源

```
C:\Users\SO>gem sources -l
*** CURRENT SOURCES ***

https://rubygems.org/
```

* 输入`gem sources --remove https://rubygems.org/`，移除`https://rubygems.org/`

```
C:\Users\SO>gem sources --remove https://rubygems.org/
https://rubygems.org/ removed from sources
```

* 输入`gem sources -a https://gems.ruby-china.org/`，添加RubyGems国内镜像，报以下错误：

```
C:\Users\SO>gem sources -a https://gems.ruby-china.org/
ERROR:  SSL verification error at depth 1: unable to get local issuer certificat
e (20)
ERROR:  You must add /O=Digital Signature Trust Co./CN=DST Root CA X3 to your lo
cal trusted store
Error fetching https://gems.ruby-china.org/:
        SSL_connect returned=1 errno=0 state=error: certificate verify failed (h
ttps://gems.ruby-china.org/specs.4.8.gz)
```

为解决此[SSL证书问题](https://gems.ruby-china.org/)，改为输入`gem sources -a http://gems.ruby-china.org`

```
C:\Users\SO>gem sources -a http://gems.ruby-china.org
http://gems.ruby-china.org added to sources
```

* 输入`gem sources -l`，查看当前源

```
C:\Users\SO>gem sources -l
*** CURRENT SOURCES ***

http://gems.ruby-china.org
```

## 安装jekyll

从开始菜单找到`Start Command Prompt with Ruby`，输入`gem install jekyll`，安装jekyll。
<br>
<br>
jekyll安装成功后，命令行进入项目路径，输入`jekyll s`，报以下错误：

```
D:\workspace\soyuone.github.io>jekyll s
Configuration file: D:/workspace/soyuone.github.io/_config.yml
Configuration file: D:/workspace/soyuone.github.io/_config.yml
  Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one o
f its dependencies installed. In order to use Jekyll as currently configured, yo
u'll need to install this gem. The full error message from Ruby is: 'cannot load
 such file -- jekyll-paginate' If you run into trouble, you can find helpful res
ources at https://jekyllrb.com/help/!
jekyll 3.4.3 | Error:  jekyll-paginate
```

为解决此错误，开始菜单找到`Start Command Prompt with Ruby`，输入`gem install jekyll-paginate`

```
C:\Users\SO>gem install jekyll-paginate
Fetching: jekyll-paginate-1.1.0.gem (100%)
Successfully installed jekyll-paginate-1.1.0
Parsing documentation for jekyll-paginate-1.1.0
Installing ri documentation for jekyll-paginate-1.1.0
Done installing documentation for jekyll-paginate after 0 seconds
1 gem installed
```

命令行进入项目路径，输入`jekyll s`，提示如下：

```
D:\workspace\soyuone.github.io>jekyll s
Configuration file: D:/workspace/soyuone.github.io/_config.yml
Configuration file: D:/workspace/soyuone.github.io/_config.yml
            Source: D:/workspace/soyuone.github.io
       Destination: D:/workspace/soyuone.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 9.027 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'D:/workspace/soyuone.github.io'
Configuration file: D:/workspace/soyuone.github.io/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

* [参考：github pages](https://pages.github.com/)
* [参考：Ruby教程](http://www.runoob.com/ruby/ruby-tutorial.html)
* [参考：RubyGems镜像](https://gems.ruby-china.org/)
