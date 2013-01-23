---
layout: post
title: "关于DNS污染的一些学习"
date: 2013-01-23 12:40
comments: true
categories: 
- DNS
- Wall
---

最近github的事整的很麻烦，于是学习了一下DNS的一些东西。

<http://chao.lu/2013/01/anti-dns-pollute/>

师弟的一篇文章，写的好啊。了解了：

* DNS报文默认是UDP的，但是如果使用TCP来进行DNS查询的话，基本可以避免DNS污染。

* 拿到正确的DNS，配合ssl通信的话，墙的干扰很小，github等就可以正常访问了。

* 有服务器的话，搭建一个DNS服务器，用TCP从8.8.8.8拿到正确的解析，这样就可以解决DNS污染问题，可以使用软件pdnsd来搭建。具体还可以参见[这篇文章](http://kafei.in/archives-2/749.html)

* 如果不想走DNS服务器这条路，那可以改host文件，目前google上有两个项目：[huhamhire-hosts](https://code.google.com/p/huhamhire-hosts/downloads/list)和[smarthosts](http://code.google.com/p/smarthosts/)

不使用VPN的情况下，试试解决DNS污染还是能解决我目前的一些问题，还是满足了啊！

