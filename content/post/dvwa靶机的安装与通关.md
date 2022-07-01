---
title:      "dvwa靶机的安装与通关"
description: "主要用来记录dvwa靶场的安装以及通过方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-26
published: true
tags:
    - 靶场
URL: "/2022/6/dvwa/"
categories: [ "网安" ]
---
## dvwa简介
DVWA（Damn Vulnerable Web Application）是一个用来进行安全脆弱性鉴定的PHP/MySQL Web应用，旨在为安全专业人员测试自己的专业技能和工具提供合法的环境，帮助web开发者更好的理解web应用安全防范的过程。

DVWA共有十个模块，分别是

    Brute Force（暴力（破解））

    Command Injection（命令行注入）

    CSRF（跨站请求伪造）

    File Inclusion（文件包含）

    File Upload（文件上传）

    Insecure CAPTCHA （不安全的验证码）

    SQL Injection（SQL注入）

    SQL Injection（Blind）（SQL盲注）

    XSS（Reflected）（反射型跨站脚本）

    XSS（Stored）（存储型跨站脚本）

需要注意的是，DVWA 1.9的代码分为四种安全级别：Low，Medium，High，Impossible
## dvwa 安装与配置

#### phpstudy 下载安装
进入phpstudy[官网]进行下载(https://www.xp.cn/download.html)
![download](/img/dvwa/phpstudy.png)
点击exe文件进行傻瓜式安装
![phpexe](/img/dvwa/phpstudy_exe.png)
打开phpstudy，启动Apache和MySQL
![start](/img/dvwa/启动.png)
#### dvwa 下载安装
    windows 下的安装方法
进入[官网](https://dvwa.co.uk/)点击download进行下载
![download](/img/dvwa/官网.png)
将下载好的dvwa靶机解压后放到安装phpstudy的www文件夹下
![存放dvwa](/img/dvwa/dvwa存放.png)
进入www文件下的dvwa文件下的config目录下 修改文件扩展名config.inc.php
![文件扩展](/img/dvwa/配置文件修改.png)
![文件扩展](/img/dvwa/修改后1.png)
打开config.inc.php文件按照 phpstudy中数据库一栏显示的数据库配置信息去修改我们的配置文件
![文件扩展](/img/dvwa/修改后3.png)
![文件扩展](/img/dvwa/修改后2.png)
    
    kali 下的安装方法
  [kali中安装dvwa详细步骤](https://blog.csdn.net/weixin_53067829/article/details/124027617)
## dvwa的访问与修改漏洞级别
#### dvwa 网站访问
打开浏览器访问 localhost/login.php 通过账号：admin 密码：password 进行访问
![文件扩展](/img/dvwa/login.png)

    如果输入localhost/login.php 页面时显示了如下页面，则我们需要进行一定的配置更改
  ![登录页面](/img/dvwa/新登录页面.png)
    * 点击phpstudy设置-配置文件-php.ini,将allow_url_include=Off改成allow_url_include=On，改完记得保存！
         ![登录页面](/img/dvwa/ON.png)
    * [按照此说明](https://blog.csdn.net/RBPicsdn/article/details/80059132 ) 完善dvwa/config/config.inc.php这个配置文件
#### 修改漏洞级别
点击DVWA Security 按钮可以修改漏洞等级
![漏洞等级](/img/dvwa/设置漏洞等级.png)


## dvwa 漏洞破解

### sql 手工注入

#### low 等级


#### medium 等级

#### high 等级





## 常见问题
#### phpstudy 中的mysql 启动不起来
>主要原因是之前已经在本地安装好了一个MySQL服务，而phpstudy里的MySQL服务与本地的MySQL占用的都是3306端口，产生了冲突
我们可以通过[方法](https://www.likecs.com/show-203537400.html)关闭本地mysql服务

