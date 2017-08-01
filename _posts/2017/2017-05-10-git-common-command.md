---
layout: post
title: "git在windows平台的常用命令"
categories: 版本控制
tags: 版本控制 git windows
author: Kopite
---

* content
{:toc}


git`版本号1.9.5.msysgit.1`在windows平台的常用命令。



## 查看git版本号

```
$ git version
git version 1.9.5.msysgit.1
```

## 查看远程分支

```
$ git branch -r
  origin/master
```

## 查看本地分支

```
$ git branch
* master
```

## 创建本地分支

```
$ git branch songyushi
```

## 切换至本地某一分支

```
$ git checkout songyushi
Switched to branch 'songyushi'
```

## 删除本地某一分支

```
$ git branch -D songyushi
Deleted branch songyushi (was 6bbfb71).
```

## 创建远程分支

```
$ git push origin songyushi:songyushi //提交本地songyushi分支作为远程的songyushi分支
```

![](/image/2017/2017-05-10-git-common-command-1.png)

```
$ git push origin songyushi:test //提交本地songyushi分支作为远程的test分支
```

![](/image/2017/2017-05-10-git-common-command-2.png)

## 删除远程分支

```
$ git push origin :test //origin后有空格
```

## 修改.gitignore文件后不起作用

```
$ git rm -r --cached .
$ git add .
$ git commit -m 'update .gitignore'
```
* [参考：git清除本地缓存](http://www.cnblogs.com/zzcc/p/5695883.html)

## Unable to create '···index.lock': File exists.

```
$ rm -f ./.git/index.lock
$ rm -Force ./.git/index.lock
```
* [参考：Unable to create '···index.lock': File exists.](http://stackoverflow.com/questions/7860751/git-fatal-unable-to-create-path-my-project-git-index-lock-file-exists)

## 放弃本地所有修改

```
$ git fetch --all
$ git reset --hard origin/master
```

## 本地代码库回滚

```
$ git reset --hard commit-id //回滚到commit-id，将commit-id之后提交的都回滚
$ git reset --hard HEAD~3 //将最近3次的提交进行回滚
```

## 标准流程

```
$ git status
```

```
$ git add -A
```

```
$ git commit -m "change code"
```

```
$ git fetch origin master
```

```
$ git merge origin/master
```

```
$ git checkout master
```

```
$ git merge songyushi
```

```
$ git push origin master
```

```
$ git checkout songyushi
```