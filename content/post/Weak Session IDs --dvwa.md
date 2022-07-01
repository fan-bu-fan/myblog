---
title:      "Weak Session IDs---dvwa"
description: "主要用来记录dvwa靶场 中 弱会话IDs 破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-29
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/weekSession/"
categories: [ "网安" ]
---
> 前置条件
> 浏览器安装[hackbar](https://github.com/Mr-xn/hackbar2.1.3)插件

## Weak Session IDs 简介
>当用户登录后，在服务器就会创建一个会话(session)，叫做会话控制，接着访问页面的时候就不用登录，只需要携带 Sesion去访问。 
> 
>sessionID作为特定用户访问站点所需要的唯一内容。如果能够计算或轻易猜到该sessionID，则攻击者将可以轻易获取访问权限，无需录直接进入特定用户界面，进而进行其他操作

## LOW 级别破解
>查看源代码
```php
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id'])) {
        $_SESSION['last_session_id'] = 0;
    }
    $_SESSION['last_session_id']++;
    $cookie_value = $_SESSION['last_session_id'];
    setcookie("dvwaSession", $cookie_value);
}
?> 
```
>low级别未设置过滤，直接用bp抓包，可以清楚的看到dvwaSesion的cookie，每重放一次，dvwaSesion增加一

#### 通过构造参数绕过验证过程
抓取数据包
![](/img/session/1-1.png)
获得 里面的cookie值：`dvwaSession=**; PHPSESSID=k368j2ca7pc2km9bn9qnuf0iuj; security=low`
更改dvwaSesion 的值+1然后,重新开个浏览器页面点击f12 调出hackbar
![](/img/session/1-3.png)

然后输入 对应的url 和上面更改后的cookie值 
![](/img/session/1-2.png)

## Medium 级别破解
>查看源代码
```php
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = time();
    setcookie("dvwaSession", $cookie_value);
}
?> 
```

medium级别是基于时间戳生成dvwaSesion的，关于时间戳转换，直接查找转换器进行转换即可

    其他步骤和low级别的破解是一样的

## High 级别破解
>查看源代码
```php
<?php

$html = "";

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id_high'])) {
        $_SESSION['last_session_id_high'] = 0;
    }
    $_SESSION['last_session_id_high']++;
    $cookie_value = md5($_SESSION['last_session_id_high']);
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], false, false);
}

?> 
```

>high级别使用了PHP setcookie()函数，来设置cookie:

对其进行抓包
![](/img/session/2-1.png)

抓包发现，dvwaSesion值很像md5加密，使用md5解密，发现是对从零开始的整数进行加密
![](/img/session/2-2.png)

因此我们可以使用md5对数字进行加密并赋值给dvwaSession,然后通过hackBar 去进行访问


## 学习资料
[视频资料](https://www.bilibili.com/video/BV1u54y167jS?spm_id_from=333.337.search-card.all.click&vd_source=f0d060adc4a7afa21464a1ed20e176c0)


