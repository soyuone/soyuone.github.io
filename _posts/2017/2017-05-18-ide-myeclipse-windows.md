---
layout: post
title: "myeclipse在windows平台的安装及使用"
categories: IDE
tags: IDE myeclipse windows
author: Kopite
---

* content
{:toc}


myeclipse enterprise workbench`版本号2015 Stable 2.0`在windows平台的安装及使用。



## 安装

* 工具：`myeclipse-2015-stable-2.0-offline-installer-windows.exe 版本号2015 Stable 2.0`、`myeclipse-2015-stable-2.0 破解工具.zip`
* 双击运行myeclipse安装包，选择`Next`
* `I accept the terms...`，`Choose Installation Location`中可以修改安装路径，根据电脑系统选择合适版本
* 取消勾选`Launch MyEclipse 2015`，点击Finish
* 解压`myeclipse-2015-stable-2.0 破解工具.zip`，双击`\myeclipse2015_keygen\crack.bat`
* `Usercode:`中随意填入，右侧选择`BLUE`，单击`SystemId...`两次生成SystemId后单击`Active`激活
* Tools菜单中选择`save`保存信息，关闭破解工具
* 把`\patch-补丁包\plugins\`下的文件复制到myeclipse安装路径`\plugins\`下，覆盖相关文件
* 删除myeclipse安装路径`\plugins\com.genuitec.eclipse.mobile.phonegap.core_13.0.0.me201504281437`下的`PhonegapProjectManagerImpl$1.class`、`PhonegapProjectManagerImpl$2.class`，每个文件在文件夹中搜索后可以得到两个同名文件，一共删除四个文件
* 最后，在myeclipse`MyEclipse`菜单下选择`Subscription Information...`查看激活信息

## 生成Persistent Object

* myeclipse中新建`JPA Project`，如下图所示<br>
![](/image/2017/2017-05-18-ide-myeclipse-windows-1.png)
* 在Window菜单中选择`Open Perspective`，选择`MyEclipse Database Explorer`
* 右键`New`，选择Driver template，填入Driver name、Connection URL、User name、Password，在Add JARs中添加mysql-connector-java-5.1.38.jar，并选择适宜的Driver classname，如下图所示<br>
![](/image/2017/2017-05-18-ide-myeclipse-windows-2.png)
* 单击Test Driver，提示如下<br>
![](/image/2017/2017-05-18-ide-myeclipse-windows-3.png)
* 找到要生成Persistent Object的数据库，如下图所示<br>
![](/image/2017/2017-05-18-ide-myeclipse-windows-4.png)
* 选中要生成Persistent Object的表，按住Ctrl键可以同时选择多个表，右键选择`JPA Reverse Engineering...`<br>
![](/image/2017/2017-05-18-ide-myeclipse-windows-5.png)
![](/image/2017/2017-05-18-ide-myeclipse-windows-6.png)	
* 最后选择`Finish`，生成的一个Persistent Object如下所示

TbUser.java
```java
package com.persistent;

import java.sql.Timestamp;
import java.util.HashSet;
import java.util.Set;
import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import static javax.persistence.GenerationType.IDENTITY;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import javax.persistence.Version;

/**
 * TbUser entity. @author MyEclipse Persistence Tools
 */
@Entity
@Table(name = "tb_user", catalog = "sharelife")
public class TbUser implements java.io.Serializable {

	// Fields

	private Integer userid;
	private Integer version;
	private String email;
	private String username;
	private String password;
	private String passwordsalt;
	private String userphone;
	private Timestamp createtime;
	private Timestamp updatetime;
	private Set<TbArticle> tbArticles = new HashSet<TbArticle>(0);
	private Set<TbClassify> tbClassifies = new HashSet<TbClassify>(0);

	// Constructors

	/** default constructor */
	public TbUser() {
	}

	/** minimal constructor */
	public TbUser(String email, String username, String password,
			String passwordsalt, Timestamp createtime) {
		this.email = email;
		this.username = username;
		this.password = password;
		this.passwordsalt = passwordsalt;
		this.createtime = createtime;
	}

	/** full constructor */
	public TbUser(String email, String username, String password,
			String passwordsalt, String userphone, Timestamp createtime,
			Timestamp updatetime, Set<TbArticle> tbArticles,
			Set<TbClassify> tbClassifies) {
		this.email = email;
		this.username = username;
		this.password = password;
		this.passwordsalt = passwordsalt;
		this.userphone = userphone;
		this.createtime = createtime;
		this.updatetime = updatetime;
		this.tbArticles = tbArticles;
		this.tbClassifies = tbClassifies;
	}

	// Property accessors
	@Id
	@GeneratedValue(strategy = IDENTITY)
	@Column(name = "userid", unique = true, nullable = false)
	public Integer getUserid() {
		return this.userid;
	}

	public void setUserid(Integer userid) {
		this.userid = userid;
	}

	@Version
	@Column(name = "version", nullable = false)
	public Integer getVersion() {
		return this.version;
	}

	public void setVersion(Integer version) {
		this.version = version;
	}

	@Column(name = "email", nullable = false, length = 50)
	public String getEmail() {
		return this.email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	@Column(name = "username", nullable = false, length = 50)
	public String getUsername() {
		return this.username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	@Column(name = "password", nullable = false, length = 100)
	public String getPassword() {
		return this.password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	@Column(name = "passwordsalt", nullable = false, length = 50)
	public String getPasswordsalt() {
		return this.passwordsalt;
	}

	public void setPasswordsalt(String passwordsalt) {
		this.passwordsalt = passwordsalt;
	}

	@Column(name = "userphone", length = 20)
	public String getUserphone() {
		return this.userphone;
	}

	public void setUserphone(String userphone) {
		this.userphone = userphone;
	}

	@Column(name = "createtime", nullable = false, length = 19)
	public Timestamp getCreatetime() {
		return this.createtime;
	}

	public void setCreatetime(Timestamp createtime) {
		this.createtime = createtime;
	}

	@Column(name = "updatetime", length = 19)
	public Timestamp getUpdatetime() {
		return this.updatetime;
	}

	public void setUpdatetime(Timestamp updatetime) {
		this.updatetime = updatetime;
	}

	@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "tbUser")
	public Set<TbArticle> getTbArticles() {
		return this.tbArticles;
	}

	public void setTbArticles(Set<TbArticle> tbArticles) {
		this.tbArticles = tbArticles;
	}

	@OneToMany(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "tbUser")
	public Set<TbClassify> getTbClassifies() {
		return this.tbClassifies;
	}

	public void setTbClassifies(Set<TbClassify> tbClassifies) {
		this.tbClassifies = tbClassifies;
	}

}
```

* [参考：myeclipse-2015-stable-2.0 安装及破解](http://jingyan.baidu.com/article/22a299b5c136a99e19376a19.html)
* [下载：myeclipse-2015-stable-2.0-offline-installer-windows.exe](http://pan.baidu.com/s/1mhIsjvq)
* [下载：myeclipse-2015-stable-2.0 破解工具.zip](http://pan.baidu.com/s/1hszdFvA)
* [参考：myeclipse生成Persistent Object](http://www.cnblogs.com/lzb1096101803/p/4189096.html)