---
title:      "CSRF ---dvwa"
description: "主要用来记录dvwa靶场 中 跨站请求伪造 破解方式"
author:     "伍帆"
date: 2022-06-28
image: "/img/home-bg.jpg"
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/CSRF/"
categories: [ "网安" ]
---

## CSRF 简介
>CSRF，全称Cross-site request forgery，翻译过来就是跨站请求伪造，是指利用受害者尚未失效的身份认证信息（cookie、会话等），诱骗其点击恶意链接或者访问包含攻击代码的页面，在受害人不知情的情况下以受害者的身份向（身份认证信息所对应的）服务器发送请求，从而完成非法操作（如转账、改密等）。CSRF与XSS最大的区别就在于，CSRF并没有盗取cookie而是直接利用

## LOW 级别破解
>查看源代码
![blind](/img/csrf/low.png)
> 可以看到，服务器收到修改密码的请求后，会检查参数password_new与password_conf是否相同，如果相同，就会修改密码，并没有任何的防CSRF机制
### 构造链接

#### 基础链接构造
密码的修改在页面上如下所示：
![blind](/img/csrf/1-1.png)
实际上是如下链接：
`http://192.168.153.130/dvwa/vulnerabilities/csrf/?password_new=password&password_conf=password&Change=Change#`
当受害者点击了这个链接，他的密码就会被改成password（这种攻击显得有些拙劣，链接一眼就能看出来是改密码的，而且受害者点了链接之后看到这个页面就会知道自己的密码被篡改了）
    
    这个链接也太明显了吧，不会有人点的，没错，所以真正攻击场景下，我们需要对链接做一些处理
#### 构造短链接来隐藏URL
使用[短链接](http://45.runchang.top/conversion/5664618)生成器 去生成短链接
![blind](/img/csrf/1-2.png)
    需要提醒的是，虽然利用了短链接隐藏url，但受害者最终还是会看到密码修改成功的页面，所以这种攻击方法也并不高明。
#### 构造攻击页面
现实攻击场景下，这种方法需要事先在公网上传一个攻击页面，诱骗受害者去访问，真正能够在受害者不知情的情况下完成CSRF攻击。这里为了方便演示，就在本地写一个test.html，下面是具体代码。

```html
<img src="http://192.168.153.130/dvwa/vulnerabilities/csrf/?password_new=hack&password_conf=hack&Change=Change#" border="0" style="display:none;"/>

<h1>404<h1>

<h2>file not found.<h2>
```
当受害者访问test.html时，会误认为是自己点击的是一个失效的url，但实际上已经遭受了CSRF攻击，密码已经被修改为了hack。


## Medium 级别破解
>查看源代码
> ![blind](/img/csrf/medium.png)
> 可以看到，Medium级别的代码检查了保留变量 HTTP_REFERER（http包头的Referer参数的值，表示来源地址）中是否包含SERVER_NAME（http包头的Host参数，及要访问的主机名，这里是192.168.153.130），希望通过这种机制抵御CSRF攻击
> ![blind](/img/csrf/2-1.png)

过滤规则是http包头的Referer参数的值中必须包含主机名（这里是192.168.153.130）
我们可以将攻击页面命名为192.168.153.130.html（页面被放置在攻击者的服务器里，这里是10.4.253.2）就可以绕过了
![blind](/img/csrf/2-2.png)
![blind](/img/csrf/2-3.png)
