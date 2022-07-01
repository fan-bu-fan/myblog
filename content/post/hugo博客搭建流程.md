---
title:      "hugo博客搭建流程"
description: "如果你想要尝试搭建自己的博客,不如参考以下步骤进行搭建"
author:     "伍帆"
date: 2022-07-01
image: "/img/home-bg.jpg"
published: true
tags:
    - 工具
URL: "/2022/7/hugo/"
categories: [ "瞎折腾" ]
---
>[hugo 官网](https://gohugo.io/)

## 前置条件
[git以及github的配置](https://fan-bu-fan.github.io/2022/6/git配置/)

## hugo下载与安装
#### 1.[Hugo安装包github下载](https://github.com/gohugoio/hugo/releases)
![hugo](/img/hugo/1-1.png)
就只有一个exe执行程序,我们可以解压存放到你想放的文件夹下 我把它放到了`D:\extendSoft\hugo\bin`文件夹下
![hugo](/img/hugo/1-2.png)
#### 2.环境变量的配置
找到hugo.exe存放的位置，给其配置环境变量
![hugo](/img/hugo/1-3.png)
可以在终端输入`hugo version`验证是否安装配置hugo成功
![hugo](/img/hugo/1-4.png)


## 创建站点项目

#### 1.确定hugo站点要安装的目录
我新建了`F:\hugo\sites`文件路径，准备将要搭建的hugo站点放到此目录下

#### 2.新站点的创建 并在本地启动
打开终端 输入`hugo new site ***` ***为你新站点的名字
我是又创建了 `hugo new site myblog2` myblog2去进行演示
![hugo](/img/hugo/1-5.png)

#### 3.进入到新创建的站点文件夹内并进行主题添加
进入到刚创建的myblog2文件夹内
![hugo](/img/hugo/1-6.png)
在[hugo 官网](https://gohugo.io/)选择自己喜欢的主题
![hugo](/img/hugo/1-7.png)
并点进去 找到下载地址
![hugo](/img/hugo/1-.png)
![hugo](/img/hugo/1-8.png)
进入到themes 文件夹内
![hugo](/img/hugo/1-9.png)
打开终端 输入git clone + 刚从github上拿到的链接
![hugo](/img/hugo/2-1.png)

用exampleSite下的配置文件替换站点下的配置文件
![hugo](/img/hugo/3-1.png)
![hugo](/img/hugo/3-2.png)

#### 4.启动本地博客
退回到themes 上一级文件夹 并在终端输入 `hugo server` 去进行运行
![hugo](/img/hugo/2-2.png)
![hugo](/img/hugo/3-3.png)


## 将项目部署到github上

#### 1.在github上创建托管仓库
进入github 点击 Repositores下的new按钮进行创建
![hugo](/img/hugo/4-1.png)
![hugo](/img/hugo/2-2.png)


#### 2.将本地文件托管到gitHub上
在站点目录myblog2下打开终端输入`hugo`该命令的作用将主题和md文件”编译成“html
![hugo](/img/hugo/4-4.png)
回车后会在myblog2下创建一个新的文件夹public
![hugo](/img/hugo/4-5.png)
`cd public` 终端进入public下依次输入以下命令，每条命令之后回车
```GIT
    git init
    git remote add origin https://github.com/fan-bu-fan/fanfan.github.io.git
    git add .
    git commit -m "第一次提交"
    git push -u origin master
```
此时回到你的gitHub的Reposipories下即可查看刚刚提交的内容
![hugo](/img/hugo/4-6.png)

我们来到github中此仓库的设置`settings`页面里 
![hugo](/img/hugo/4-3.png)
点击`page`按钮
![hugo](/img/hugo/5-1.png)
然后看到` Your site is published at https://fan-bu-fan.github.io/` 提示
这样我们就可以通过访问此链接去访问我们的博客了


## 可能遇到的问题

#### 1.Hugo无法解析markdown中的html标签
>在config.toml文件中加入下面内容
``` toml
[markup.goldmark.renderer]
unsafe = true
```









