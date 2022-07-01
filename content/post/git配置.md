---
title:      "git配置"
description: "给电脑配置git 方便进行文件版本控制管理"
author:     "伍帆"
date: 2022-07-01
image: "/img/home-bg.jpg"
published: true
tags:
    - 工具
URL: "/2022/6/git配置/"
categories: [ "工具集" ]
---

## windows下git的安装
[安装教程](https://www.jb51.net/article/245698.htm)


## windows下 git 和github 联动

#### 1.打开git bash页面
找到git安装的目录，并点击git-bash.exe 
![git](/img/git/1-1.png)
![git](/img/git/1-2.png)

#### 2.配置git
第一步：输入ssh-keygen -t rsa -C “邮箱地址”(随便写一个邮箱就行)。（）

第二步：回车之后，要求输入一个保存密钥的路径，括号中的是默认位置。建议直接回车，使用默认路径。

第三步：要求设置密码，直接回车两次，就可以生成密钥了。（刚才的默认路径下会生成两个文件：id_rsa和id_rsa.pub。id_rsa.pub中的全部内容就是密钥）
![git](/img/git/2-1.png)


#### 3.github 官网注册和配置
第一步：[登录官网](https://github.com/),注册账号
第二步：点击进入个人配置页面点击 头像
![git](/img/git/3-1.png)
第三步：点击 `ssh and gpg keys` 按钮
![git](/img/git/3-2.png)
第四步：点解右上角的`new ssh key` 配置ssh keys 
![git](/img/git/3-3.png)
>其中名字随便区，key 为刚刚在git中配置中生成的id_rsa.pub的全部内容

![git](/img/git/3-4.png)



#### 4.进行配置验证
输入 `ssh -T git@github.com` 如果显示success则代表配置成功
![git](/img/git/4-2.png)
>**但是**如果显示`ssh:connect to host github.com port 22: Connection timed out`
![git](/img/git/4-1.png)
> 则需参考[此博文](https://blog.csdn.net/qq_38330148/article/details/109371362)的解决方式


