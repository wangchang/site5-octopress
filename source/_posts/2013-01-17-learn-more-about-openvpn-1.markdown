---
layout: post
title: "OpenVPN学习-原理"
date: 2013-01-17 12:46
comments: true
categories: 
- OpenVPN
---

上次搭建好了实验室的VPN，后来有一些问题，主要是速度问题，所以又继续学习OpenVPN的，原理学习等做个记录。OpenVpn的技术核心是虚拟网卡，其次是SSL协议实现。
<!--more-->
##OpenVPN工作原理
openvpn驱动部分实现了网卡处理和字符设备。网卡处理网络数据，字符设备完成与应用层的数据交互。{% highlight 使用openvpn必须修改路由表 %}。

关于虚拟网卡，TUN/TAP，感觉要学习的很多，我现在暂时的理解就是虚拟网卡可以相应的截断数据，然后交给应用程序处理。虚拟网卡没法直接发送数据，而是要交给应用层通过其他方式发送。如下图：

{% img /images/201301/OpenVPN.png 'OpenVPN流程' 'OpenVPN流程' %}

总结，OpenVPN中发送数据:

* 应用程序发送网络数据

* 网络数据根据修改后的路由表把数据路由到虚拟网卡

* 虚拟网卡把数据放到数据队列中

* 字符设备从数据队列中取数据，然后送给应用层

* 应用层把数据转发给物理网卡

* 物理网卡发送数据

接收过程：

* 物理网卡接受到数据，并传到应用空间

* 应用守护程序通过字符设备，把数据传给驱动网卡

* 数据通过虚拟网卡重新进入网络堆栈

* 网络堆栈把数据传给上层真实的应用程序。

可以参考以下：
<http://hi.baidu.com/lmcbbat/item/4088fc0f5758cd8e03ce1b65>
<http://wenku.baidu.com/view/4259aaacdd3383c4bb4cd23b.html>

##OpenVPN的工作模式
OpenVPN中虚拟网卡驱动可以使用tun或者tap，对应了OpenVPN支持的两种模式，

* Routed VPN，L3 VPN

适用于不需广播的点对点IP (point-to-point)通信。比起桥接网络隧道来略显得更有效率些而且更易配置。此时vpn连上后相当于一个路由器。在这种模式下，`VPN服务器需要向客户推送路由表，以让到达某些特定地址的数据发送到VPN服务器。对应于`server.conf`中的配置为`dev tun`。

* bridged VPN，L2 VPN

用于IP协议或非IP协议的隧道。这种类型的隧道更适合于使用广播(broadcast)的应用，比如某些Windows网络游戏。配置起来稍微复杂些。此时vpn连上后相当于一个交换机。在这种模式下`所有2层的帧，包括ARP,DHCP,以太等等都会发送到VPN服务器。`对应于`server.conf`中的配置问`dev tap`。这个模式下会把服务器网卡挂在一个桥br0上。

可以参考<https://help.ubuntu.com/12.04/serverguide/openvpn.html>

##一些典型的配置

###如果需要翻墙，希望国内流量走本地链路，国外走VPS等，则需要用Routed VPN模式，Server需要向客户端推送路由表。
>[设置多网关国内流量不走国外线路教程](http://www.eastdesign.net/eastern-blog/mac-os-x-lion下vpn设置多网关国内流量不走国外线路教程/)

###OpenVPN支持LDAP,RADIUS,CA验证登陆：
>[配置OpenVPN使用User/Pass方式验证登录](http://ylw6006.blog.51cto.com/470441/1009004)

>[用openvpn连通两个私网网段配置步骤](http://yyri.blog.163.com/blog/static/14894395120116190149472/?fromdm&fromSearch&isFromSearchEngine=yes)

###OpenVPN协议的分析：
>[OpenVPN协议解析-网络结构之外](http://bbs.c114.net/thread-617226-1-1.html)

