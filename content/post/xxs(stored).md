---
title:      "跨站脚本攻击（存储型）---dvwa"
description: "主要用来记录dvwa靶场 跨站脚本攻击（存储型）破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-30
published: true
draft: false
tags:
    - 靶场
    - dvwa
URL: "/2022/6/xss(stored)/"
categories: [ "网安" ]
---
>强烈建议先学习下[反射型XSS](https://fan-bu-fan.github.io/2022/6/xss(reflected)/)

## 存储型xss简介
>存储型XSS是一种持久化的XSS，它与反射型XSS最大的不同就是服务器在接收到我们的恶意脚本时会将其做一些处理，代码是`存储在服务器`中的。
比如微博、留言板这种，通常就是存储型XSS。当有人在留言内容中插入恶意脚本时，由于服务器要向每一个访客展示之前的留言内容，每次访问时都会触发代码执行。

## Low 级别破解
> 查看源码
```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = stripslashes( $message );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitize name input
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?> 
```
>没有过滤机制，只是检查下是否为空,这样就存在很明显的XSS漏洞了

输入`<script>alert(1)</script>` 可以看到会出来一个弹窗
![直接输入](/img/xss(stored)/1-2.png)
![直接输入](/img/xss(stored)/1-1.png)


接下来我们修改下Payload，弹出用户的cookie.输入`<script>alert(document.cookie)</script>`
![直接输入](/img/xss(stored)/1-3.png)
![直接输入](/img/xss(stored)/1-4.png)


## Medium 级别破解
> 查看源码
```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = str_replace( '<script>', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?> 
```

> 剥去message的HTML、XML以及PHP的标签。
> 对message采用htmlspecialchars()函数进行预定义字符转换为HTML实体。
> 对name的`<script>`进行替换删除。

上面代码对message过滤的很严，基本上把攻击的路堵死了。但是对于name只做了关键字过滤，因此可以对name区域下手

但是name在Web页面输入端对字数有限制，同样我们可以通过burp来修改发送报文，注意下`<Script>`用大小写来进行绕过

![直接输入](/img/xss(stored)/2-1.png)
我们可以用`<Script>`大小写来绕过弹个窗:`<Script>alert(1)</Script>`'
![直接输入](/img/xss(stored)/2-2.png)
![直接输入](/img/xss(stored)/2-3.png)

## High 级别破解
> 查看源码
```php
<?php

if( isset( $_POST[ 'btnSign' ] ) ) {
    // Get input
    $message = trim( $_POST[ 'mtxMessage' ] );
    $name    = trim( $_POST[ 'txtName' ] );

    // Sanitize message input
    $message = strip_tags( addslashes( $message ) );
    $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $message = htmlspecialchars( $message );

    // Sanitize name input
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
    $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Update database
    $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    //mysql_close();
}

?> 
```
>这个级别对name的过滤进行了些增强，把`<script>`做了正则表达式，如何绕过在反射型XSS里面也介绍了，这里就简单演示一种，利用`<image>`标签来绕过

同样我们可以通过burp来修改name中的值发送报文，注意下用`<img>`用大小写来进行绕过
![直接输入](/img/xss(stored)/3-1.png)
输入`<img src='1' onerror='alert(1)'>`
![直接输入](/img/xss(stored)/3-2.png)
![直接输入](/img/xss(stored)/3-3.png)