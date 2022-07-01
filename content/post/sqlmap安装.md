---
title:      "sqlMap的安装与使用"
description: "主要用来记录sqmMap工具的安装与使用方法"
author:     "伍帆"
image: "/img/home-bg.jpg"
date: 2022-06-26
published: true
tags:
    - 工具
    - 渗透测试
URL: "/2022/6/sqlMap安装与使用/"
categories: [ "工具集" ]
---

>sqlmap是一个自动化的sql注入工具，其主要功能是扫描，发现并利用给定的URL进行sql注入，目前支持数据库有mysql、oracle、access、postagesql、sql server、sqlite等

## sqlmap的安装
 进入sqlmap[官网](https://sqlmap.org/) 选择安装包进行下载安装

 ![sqlmap官网](/img/sqlmap/官网.png)

 >注：因为sqlmap基于Python语言开发的，所以前提需要将Python这个语言环境给安装上
 
 将下载好的sqlmap进行解压并且copy到python路径下
    ![存放路径](/img/sqlmap/存放.png)
   
## sqlmap的启动
  在桌面创建一个cmd的快捷方式，并命名为SQLMap
 * 第一步：右键—>新建—>快捷方式
    ![cmd](/img/sqlmap/cmd.png)
 * 第二步：输入快捷方式的名称,点击完成，桌面上就会显示快捷图标
    ![名称](/img/sqlmap/名称.png)
    ![快捷](/img/sqlmap/名称1.png)
 * 第三步：在新建的快捷方式上右键“属性”，将“起始位置”修改为自己sqlmap所在的路径，点击确定
    ![快捷路径](/img/sqlmap/快捷路径.png)
 * 第四步：双击已创建的快捷方式，输入sqlmap.py -h，出现如下信息则表示安装成功
   ![启动](/img/sqlmap/启动.png)
## sqlmap的使用


## 补充扩展资料
[sqlmap学习视频](https://www.bilibili.com/video/BV1bJ41177mD?spm_id_from=333.337.search-card.all.click)

[sqlmap 使用文档资料](https://blog.csdn.net/qq_43531669/article/details/113835025)