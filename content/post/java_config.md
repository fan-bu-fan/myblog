---
title:      "java环境配置"
description: "本文主要描述记录如何在本机中配置java环境"
author:     "伍帆"
date: 2022-06-26
image: "/img/home-bg.jpg"
published: true
tags:
    - java
    - 环境配置
    - 工具
URL: "/2022/6/java环境配置/"
categories: [ "工具集" ]
---
![java_config](/img/java_config/java_config.png)
## java jdk下载
jdk版本不要过高，否则可能会出现问题  以下以java11的安装为例进行说明

我们可以通过[java官网](https://www.oracle.com/java/technologies/downloads/#java8)去进行下载：

进入官网后，下载64位即可

![java 官网](/img/java_config/java官网.png)
>💡 但是通过官网下载需要我们注册与登录到官网之中，为了方便 可以直接点击下面的链接去进行jar包的下载：
https://download.java.net/openjdk/jdk11/ri/openjdk-11+28_windows-x64_bin.zip

## java jdk的安装
解压下载的jar包后 选择目录就可以一路傻瓜式安装即可

这里我安装的目录是：
![java 安装目录](/img/java_config/java安装目录.png)

## java环境配置
打开系统属性面板，点击系统变量按钮
![java环境配置1](/img/java_config/环境配置1.png)
![java环境配置2](/img/java_config/环境配置2.png)
把刚刚安装的java的路径给粘上去
![java环境配置3](/img/java_config/环境配置3.png)

##  java 环境验证
使用 win+R 命令 打开cmd.exe，在终端中输入java -version后出现版本号代表搭建成功
>💡 一定要重新打开一个新的终端去进行验证

![验证java](/img/java_config/验证java.png)