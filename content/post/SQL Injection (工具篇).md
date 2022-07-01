---
title:      "SQL Injection(工具篇) ---dvwa"
description: "主要用来记录dvwa靶场 中SQL Injection 使用工具 的破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-28
published: true
tags:
    - 靶场
    - dvwa
    - sqlmap
URL: "/2022/6/tool sql injection/"
categories: [ "网安" ]
---

## 前置准备
> * [sqlmap的安装](https://fan-bu-fan.github.io/2022/6/sqlMap%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/)
> 
> * [BurpSuite的安装](https://fan-bu-fan.github.io/2022/6/Burp%20Suite%20%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8/)
> 
> * [BuprSuite中配置sqlmap](https://blog.csdn.net/tiankai30/article/details/119378078)

## low级别破解
#### 1.抓包查看数据包的情况
![1-1](/img/tools_sqlJection/1-1.png)
![1-1](/img/tools_sqlJection/1-2.png)

#### 2.尝试使用sqlmap去查看是否有注入漏洞
点击run 按钮 使用sqlmap去进行探测
![1-1](/img/tools_sqlJection/1-3.png)
![1-1](/img/tools_sqlJection/1-4.png)
#### 3.用–-dbs来获取其所有数据库
![1-1](/img/tools_sqlJection/1-5.png)

#### 4.用–-tables来获取表
![1-1](/img/tools_sqlJection/1-6.png)
`-D '***' --tables` 获得指定数据库下的表
![1-1](/img/tools_sqlJection/1-7.png)
#### 5.用–-column来获取感兴趣的列
![1-1](/img/tools_sqlJection/1-8.png)
#### 6.破解用户名密码
![1-1](/img/tools_sqlJection/1-9.png)

## Medium级别破解
#### 1.抓包查看数据包的情况
    是post级别的
![1-1](/img/tools_sqlJection/2-1.png)

#### 2.尝试使用sqlmap去查看是否有注入漏洞
点击run 按钮 使用sqlmap去进行探测
![1-1](/img/tools_sqlJection/2-2.png)
![1-1](/img/tools_sqlJection/2-2.png)
发现所有sql注入漏洞的

    剩下的步骤和low中破解的步骤一样

## High 级别的破解
    high 级别有两个跳转页面

#### 1.截取跳转页面的数据包
![1-1](/img/tools_sqlJection/3-4.png)
![1-1](/img/tools_sqlJection/3-1.png)

#### 2.将截取的数据包存放在sqlmap 目录下
![1-1](/img/tools_sqlJection/3-5.png)

#### 3. 查看结果显示页面的urL
![1-1](/img/tools_sqlJection/3-6.png)

#### 4.使用sqlmap 工具进行sql注入
使用`python sqlmap.py -r 1.txt --batch --dbs --second-url "http://127.0.0.1/vulnerabilities/sqli/"`
其中--second-url 代表跳转到结果显示页面 
![1-1](/img/tools_sqlJection/3-3.png)

    剩下的步骤和low级别的步骤一样