---
title:      "Burp Suite的安装与使用"
description: "主要用来记录 Burp Suite工具的安装与使用方法"
author:     "伍帆"
date: 2022-06-27
image: "/img/home-bg.jpg"
published: true
tags:
    - 工具
    - 渗透测试
URL: "/2022/6/Burp Suite 安装与使用/"
categories: [ "工具集" ]
---



> 前置条件
> * [java 环境已经配置好](https://fan-bu-fan.github.io/2022/6/java%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE/)


#  Burp Suite的下载安装及破解教程
#### 下载地址
[点击进入官网进行下载](https://portswigger.net/burp/releases)

~~~
  !!!这里我们选择专业版，并且建议选择JAR。方便我们后续对其进行破解
~~~

![官网地址](/img/BurpSuit/官网下载.png)

下载后就是一个jar包,对于jar包我们不能直接打开 但是可以通过 `java -jar ****.jar ` 命令行的方式进行打开

![打开jar包](/img/BurpSuit/运行BurpSuite.png)

然后通过点击run 按钮进行安装，但是在安装的过程中会让你填写licence 这个时候我们就需要下载破解机对其进行破解了
#### 破解方式
[点击下载破解机](https://github.com/h3110w0r1d-y/BurpLoaderKeygen/releases)

![img.png](/img/BurpSuit/破解机下载.png)

将下载的jar包 和 burpSuite的jar包放在同一个目录下面
![存放目录](/img/BurpSuit/存放目录.png)

把这个jar包和Burp Suite的jar包放到同一个目录下面

通过 **`java -jar .\BurpLoaderKeygen.jar`** 命令 打开破解机 并按照如下步骤进行破解
![存放目录](/img/BurpSuit/破解1.png)
![存放目录](/img/BurpSuit/破解2.png)
![存放目录](/img/BurpSuit/破解3.png)
![存放目录](/img/BurpSuit/破解4.png)
![存放目录](/img/BurpSuit/破解5.png)
![存放目录](/img/BurpSuit/破解6.png)
![存放目录](/img/BurpSuit/破解7.png)
![存放目录](/img/BurpSuit/破解8.png)
#### 设置快捷启动方式
建立一个文档，输入`java -jar [破解机的下载路径]`（破解机的下载路径 jar和路径之间是有空格的！！！！！！！！！！！！不然打不开）编辑好后保存，改后缀为bat即可
>我的破解机存放的路径为： D:\bigProgram\Java\BurpLoaderKeygen.jar  所以我新建的文档里 只需要写 java -jar D:\bigProgram\Java\BurpLoaderKeygen.jar

![存放目录](/img/BurpSuit/快捷启动.png)

如上，以后我只需点击run.bat文件就能启动burpSuit了

# Burp Suite的代理和浏览器设置

#### 浏览器设置代理
  以火狐浏览器为例[查看代理安装方式](https://blog.csdn.net/Hou_Lang_/article/details/122047560)

#### 证书的安装
  <span id="jump">为浏览器安装相应的证书</span>
    
   * way1:(推荐) 
    
      <font color = red> 一定</font> 要打开浏览器中的代理
    
       ![img.png](/img/BurpSuit/开启代理.png)

       然后在浏览器中输入 https://burp  如果出现如下画面则代表访问成功，然后我们点击 CA Certificate进行证书的安装
       ![img.png](/img/BurpSuit/ca.png)

   * way2:(不推荐)
    
      [自行导出证书](https://blog.csdn.net/supassxu/article/details/81448908)的方式

> 下面我们就开始正式使用Burp Suite 工具啦。

# Burp Suite 模块的使用
#### Dashboard 仪表盘
    左上角Tasks是任务区,新建的扫描任务或正在运行的任务都会在该区域显示

    左下角是Event lot是事件区,burpsuit报错或者事件的状态(扫描开始,扫描结束)都会在该区域显示

    右上角Issue activity ,该区域显示扫描出的漏洞,可以通过High,Medium,low查看不同危险等级的漏洞

    右下角是某个漏洞的详细情况展示
    
  ![img.png](/img/BurpSuit/dashBoard.png)
  
  仪表盘的功能和简介可参考[dashBoard新建扫描](https://www.bilibili.com/video/BV1aq4y1X7oE?p=4)


#### Target 模块
    Burp Target 组件主要包含站点地图、目标域、Target 工具三部分组成，
    他们帮助渗透测试人员更好地了解目标应用的整体状况、当前的工作涉及哪些
    目标域、分析可能存在的攻击面等信息

  >[详细介绍文档](https://t0data.gitbooks.io/burpsuite/content/chapter5.html)

  >[快速了解视频](https://www.bilibili.com/video/BV1aq4y1X7oE?p=7)
    
#### Proxy 模块
    辅助我们进行网页抓包
  >[快速了解视频](https://www.bilibili.com/video/BV1aq4y1X7oE?p=6)

  >[详细操作文档](https://blog.csdn.net/weixin_45557138/article/details/107382941)
#### Intruder 模块
    Burp Intruder作为Burp Suite中一款功能极其强大的自动化测试工具，通常被系统安全渗透测试人员
    被使用在各种任务测试的场景中
  >[详细文档介绍](https://t0data.gitbooks.io/burpsuite/content/chapter8.html)

  >[快速了解视频](https://www.bilibili.com/video/BV1aq4y1X7oE?p=9)

#### Repeater 模块
    Burp Repeater作为Burp Suite中一款手工验证HTTP消息的测试工具，通常用于多次重放请求响应和
    手工修改请求消息的修改后对服务器端响应的消息分析
  >[详细介绍文档](https://t0data.gitbooks.io/burpsuite/content/chapter9.html)

  >[快速了解视频](https://www.bilibili.com/video/BV1aq4y1X7oE?p=8)
#### Sequencer 模块
    Burp Sequencer作为Burp Suite中一款用于检测数据样本随机性质量的工具，通常用于检测访问令牌是
    否可预测、密码重置令牌是否可预测等场景，通过Sequencer的数据样本分析，能很好地降低这些关键数据被
    伪造的风险
  >[详细介绍文档](https://t0data.gitbooks.io/burpsuite/content/chapter10.html)

#### Decoder 模块
    Burp Decoder的功能比较简单，作为Burp Suite中一款编码解码工具
  >[详细介绍文档](https://t0data.gitbooks.io/burpsuite/content/chapter11.html) 

#### Compare 模块
    Burp Comparer在Burp Suite中主要提供一个可视化的差异比对功能，来对比分析两次数据之间的区别
  >[详细介绍文档](https://t0data.gitbooks.io/burpsuite/content/chapter12.html)

#### burpsuite 和sqlmap 的联动
  >[burpsuite 安装sqlmap插件](https://blog.csdn.net/tiankai30/article/details/119378078)

# 常见问题记录
#### 抓包过程中显示“有软件正在阻止firefox安全的连接此网站”
    
  出现此现象的主要原因是我们的浏览器没有配置burpSuite证书，我们可以通过[以上方法](#jump)去进行证书的获取及安装

#### 访问不了证书下载页面
  在访问 `https://burp` 之前一定要确保自己已经把 burpSuite的代理模式打开了
  ![img.png](/img/BurpSuit/开启代理.png)

#### 火狐浏览器自动向http://detectportal.firefox.com发包的问题
   可以通过参考[此文章](https://blog.csdn.net/kongzhian/article/details/111595144?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-1-111595144-blog-115273979.pc_relevant_antiscanv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-1-111595144-blog-115273979.pc_relevant_antiscanv4&utm_relevant_index=2 )进行解决


# 补充扩展资料
[BurpSuite/Burp Suite 2021最新系列渗透视频](https://www.bilibili.com/video/BV1aq4y1X7oE?spm_id_from=333.337.search-card.all.click)

[Burp Suite 实战指南](https://t0data.gitbooks.io/burpsuite/content/chapter4.html)

<style>
blockquote {
  border-left: 2px dashed #333 !important;
  background: linear-gradient(to bottom, #efe 0%,#fef 100%) !important;
}
</style>