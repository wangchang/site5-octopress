---
layout: post
title: "通用TUN/TAP设备简介"
date: 2013-01-18 14:58
comments: true
categories: 
- Tun
- Tap
- OpenVPN
---
##总览
TUN与TAP是操作系统内核中的虚拟网络设备。不同于普通靠硬件网路板卡实现的设备，这些虚拟的网络设备全部用软件实现，并向运行于操作系统上的软件提供与硬件的网络设备完全相同的功能。

TAP 等同于一个以太网设备，它操作第二层数据包如以太网数据帧。TUN模拟了网络层设备，操作第三层数据包比如IP数据封包。这两种设备针对网络包实施不同的封装。利用tun/tap驱动，可以将tcp/ip协议栈处理好的网络分包传给任何一个使用tun/tap驱动的进程，由进程重新处理后再发到物理链路中。

操作系统通过TUN/TAP设备向绑定该设备的用户空间的程序发送数据，反之，用户空间的程序也可以像操作硬件网络设备那样，通过TUN/TAP设备发送数据。在后种情况下，TUN/TAP设备向操作系统的网络栈投递（或“注入”）数据包，从而模拟从外部接受数据的过程。

以上来自[百度百科](http://baike.baidu.com/view/8570754.htm)

##详细
TUN/TAP设备为用户空间程序提供了数据包的接收和传输能力，它可以被看做一个简单的点到点(point to point)或者一个以太设备，普通的网卡设备是从物理介质中收包，而TUN/TAP则是从用户态程序中收到数据包，普通网卡设备是通过物理介质发送数据，而TUN/TAP设备则是向用户空间程序发送数据。

在使用上，一个TUN/TAP设备你可以看做是一个文件描述符，不同的是系统会有关于这个文件描述符的路由。

根据设备类型的选择，用户态程序通过TUN/TAP借口就能够读写IP包（通过TUN接口）或者以太帧（通过TAP接口），至于使用哪个接口，实际上是系统ioctl（）函数的参数决定的。因为TUN/TAP接口是内核软件实现的，使用的时候就需要打开`/dev/net/tun`。

__Tun/tap设备的工作过程：__

Tun/tap设备提供的虚拟网卡驱动，从tcp/ip协议栈的角度而言，它与真实网卡驱动并没有区别。从驱动程序的角度来说，它与真实网卡的不同表现在tun/tap设备获取的数据不是来自物理链路，而是来自用户区，Tun/tap设备驱动通过字符设备文件来实现数据从用户区的获取。发送数据时tun/tap设备也不是发送到物理链路，而是通过字符设备发送至用户区，再由用户区程序通过其他渠道发送。

__发送过程：__

使用tun/tap网卡的程序经过协议栈把数据传送给驱动程序，驱动程序调用注册好的hard_start_xmit函数发送，hard_start_xmit函数又会调用tun_net_xmit函数，其中skb将会被加入skb链表，然后唤醒被阻塞的使用tun/tap设备字符驱动读数据的进程，接着tun/tap设备的字符驱动部分调用其tun_chr_read()过程读取skb链表，并将每一个读到的skb发往用户区，完成虚拟网卡的数据发送。

__接收数据的过程：__
当我们使用`write()`系统调用向tun/tap设备的字符设备文件写入数据时，`tun_chr_write`函数将被调用，它使用`tun_get_user`从用户区接受数据，其中将数据存入skb中，然后调用关键的函数`netif_rx(skb)` 将`skb`送给`tcp/ip`协议栈处理，完成虚拟网卡的数据接收。

##FAQ

* TUN/TAP主要用来干嘛？

答：TUN/TAP主要用来做隧道用

* 这种Virtual network device是如何工作的？

答：Virtual network device（TUN/TAP）可以被看做是一个简单的点到点设备或者以太设备，从用户程序接收数据，并向用户程序发送数据。

一个简单的例子，你再tap0接口上配置了IPX，无论何时，当内核发送一个IPX报文到tap0接口的时候，tap0接口把这个报文传递给应用程序，这个应用程序再加密报文，经过压缩处理以后再通过系统的TCP或者UDP协议栈发送到对端。在对端的应用程序通过系统TCP或者UDP协议栈接收到数据，进行解密解压后，把报文再写入到TAP设备上，这样，内核此时就处理来自TAP接口的数据，并且在内核看来，这个数据{% hightlight 就像是从实际的网络设备处接收到的一样。 %}

* TUN 和 TAP的区别？

答：TUN用来处理IP报文，TAP处理以太帧。

##参考资料
<http://www.kernel.org/doc/Documentation/networking/tuntap.txt>

<http://www.ibm.com/developerworks/cn/linux/l-tuntap/>

<http://blog.csdn.net/macrossdzh/article/details/5734736>

[VTun 工作原理详解](http://blog.csdn.net/wangxing1018/article/details/4169179){% highlight 这篇文章强烈推荐，要了解TUN/TAP在OpenVPN中如何操作的话，一定要读读！ %}
