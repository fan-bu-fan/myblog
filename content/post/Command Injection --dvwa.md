---
title:      "Command Injection ---dvwa"
description: "主要用来记录dvwa靶场 中 命令行注入 破解方式"
author:     "伍帆"
date: 2022-06-28
image: "/img/home-bg.jpg"
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/Command Injection/"
categories: [ "网安" ]
---
##  Command Injection 简介
>Command Injection，即命令注入，是指通过提交恶意构造的参数破坏命令语句结构，从而达到执行恶意命令的目的。PHP命令注入攻击漏洞是PHP应用程序中常见的脚本漏洞之一，国内著名的Web应用程序Discuz!、DedeCMS等都曾经存在过该类型漏洞。

## LOW 级别破解
>查看源代码
![blind](/img/Command/low.png)
> 可以看到，服务器通过判断操作系统执行不同ping命令，但是对ip参数并未做任何的过滤，导致了严重的命令注入漏洞。

    window和linux系统都可以用&&来执行多条命令
输入`127.0.0.1&&net user`
![blind](/img/Command/1-1.png)

    Linux下输入127.0.0.1&&cat /etc/shadow甚至可以读取shadow文件，可见危害之大。

## Medium 级别破解
>查看源代码
![blind](/img/Command/mediun.png)
> 可以看到，相比Low级别的代码，服务器端对ip参数做了一定过滤，即把”&&” 、”;”删除，本质上采用的是黑名单机制，因此依旧存在安全问题。
#### 使用& 代替 &&
输入`127.0.0.1&&net user`
![blind](/img/Command/2-2.png)
因为被过滤的只有”&&”与” ;”，所以”&”不会受影响 
输入`127.0.0.1&net user`
![blind](/img/Command/2-1.png)
>这里需要注意的是”&&”与” &”的区别：
>* Command 1&&Command 2
>   >先执行Command 1，执行成功后执行Command 2，否则不执行Command 2
>* Command 1&Command 2
>   >先执行Command 1，不管是否成功，都会执行Command 2
 
#### 使用`&;&`进行绕过
由于使用的是str_replace把”&&” 、”;”替换为空字符，因此可以采用以下方式绕过：
`127.0.0.1&;&ipconfig`
    
    这是因为”127.0.0.1&;&ipconfig”中的” ;”会被替换为空字符，
    这样一来就变成了”127.0.0.1&& ipconfig” ，会成功执行
![blind](/img/Command/2-3.png)


## High 级别破解
>查看源代码
![blind](/img/Command/high.png)
>相比Medium级别的代码，High级别的代码进一步完善了黑名单，但由于黑名单机制的局限性，我们依然可以绕过

    黑名单看似过滤了所有的非法字符，但仔细观察到是把”| ”
    （注意这里|后有一个空格）替换为空字符，于是 ”|”成了“漏网之鱼”。
输入 `127.0.0.1| net user`


##  可能遇到的问题

#### 页面返回的结果为乱码的情况
>如下情况
![blind](/img/Command/3-1.png)
> 解决方法
![blind](/img/Command/3-2.png)
> 最后结果
![blind](/img/Command/3-3.png)

