---
layout: post
title: "在Ubuntu12.04上配置OpenVPN"
date: 2013-01-06 13:07
comments: true
categories: 
- OpenVPN
---
本文主要描述了如何在Ubuntu12.04 Server上搭建一个OpenVPN服务器，因为现在OpenVPN版本有升级，和以前的使用有一些不同，所以可能网上其他一些教程就不适用了。我搭建VPN服务器主要有以下作用：
>服务器两块网卡，一块eth0接入Internet,IP为222.197.180.135，一块eth1接入实验室内部网络192.168.1.0/24，一块eth2接入另外一个实验室网络192.168.10.0/24；
>用户处于校园网内，无法访问Internet，但是可以访问服务器的外网地址eth0，无法访问到实验室内部网络；
>最终，用户连入VPN服务器，就可以连接到Internet以及实验室内部网络。
>系统:Ubuntu12.04 OpenVPN:2.11
OpenVPN还是比较复杂，所以我是先跑起来，再进行深入的学习。
<!--more-->
##1 安装

    sudo apt-get install openvpn

此时，配置目录为/etc/openvpn。
##2 基本配置
###2.1 配置证书

    cd /etc/openvpn/
    sudo mkdir easy-rsa    #准备证书相关文件
    sudo cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0/* ./easy-rsa/
    sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz ./
    sudo gzip -d /etc/openvpn/server.conf.gz    #解压服务器配置文件
    cd easy-rsa/
    sudo vim vars    #修改生成证书的相关变量
        
这里的环境变量最好简单点，有以下地方作修改：

    export KEY_SIZE=1024    #证书大小，如果你是偏执狂可以开到2048，会牺牲一些速度来换取高到没有必要的安全性
    export CA_EXPIRE=3650    #CA许可过期时间，默认10年
    export KEY_EXPIRE=3650    #证书过期时间，改成1到3年吧10年太长了
    export KEY_COUNTRY="US"    #国家，中国是"CN"
    export KEY_PROVINCE="CA"    #省份，如上海是"SH"
    export KEY_CITY="SanFrancisco"    #城市，如上海是"Shanghai"
    export KEY_ORG="Fort-Funston"    #组织，填公司名或根域名
    export KEY_EMAIL="me@myhost.mydomain"    #邮箱，填证书相关事务的邮箱
    export KEY_EMAIL=mail@host.domain #和上一条重复，删掉或者注释掉
    export KEY_CN=changeme #证书通用名，和下一条的NAME一样即可
    export KEY_NAME=changeme #证书名，openvpn.mydomain.com即可
    export KEY_OU=changeme #这是啥？？懒得查了我填成和组织一样了
    export PKCS11_MODULE_PATH=changeme #重复，删掉或注释掉
    export PKCS11_PIN=1234 #重复，删掉或注释掉

做出相应的更改就好，最好简单点，好记。
###2.2 生成证书
在/etc/openvpn/easy-rsa/目录下，有一些脚本文件，用来生成证书。

    cd /etc/openvpn/easy-rsa/
    source vars #更新环境变量
    ./clean-all
    ./build-ca #生成一个CA许可文件
    ./build-key-server myservername #生成服务器证书，记住这里的名字哈
    ./build-dh    #生成迪菲赫尔曼参数文件

以上执行完成以后，当前目录下会产生一个keys的文件，进入并拷贝相应的认证内容。

    cp myservername.crt myservername.key ca.crt dh1024.pem /etc/openvpn/ #这里myservername是之前产生服务器证书时候的名字

这一步主要就是把产生的认证文件拷贝的OpenVPN配置目录下。
###2.3 配置服务器server.conf
配置`/etc/openvpn`下面的server.conf，具体的配置内容解释我以后会写,我们需求情况下的配置文件如下：

    local 222.xxx.xxx.xxx
    port 1194
    proto udp
    dev tun
    ca /etc/openvpn/ca.crt
    cert /etc/openvpn/myservername.crt
    key /etc/openvpn/myservername.key
    dh /etc/openvpn/dh1024.pem
    server 10.8.0.0 255.255.255.0
    script-security 3
    push "route 192.168.1.0 255.255.255.0"
    push "route 192.168.10.0 255.255.255.0"
    push "redirect-gateway def1 bypass-dhcp bypass-dns"  #让流量全部经过VPN服务器
    push "dhcp-option DNS 61.139.2.69"
    push "dhcp-option DNS 8.8.8.8"
    duplicate-cn
    keepalive 10 120
    comp-lzo
    persist-key
    persist-tun
    verb 3
    
###2.4 产生客户端证书
客户端也需要相应的证书才能连接服务器。

    cd /etc/openvpn/easy-rsa/
    source vars
    ./build-key client1
    mkdir /etc/openvpn/client
    cp client1* /etc/openvpn/client

产生名为clinet1的用户证书，可以创建多个，也可以用一个证书让多个用户一起使用。
###2.5 配置客户端配置文件

    sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client

客户端的配置内容如下：

    client
    dev tun
    proto udp
    remote 222.xxx.xxx.xxx 1194
    resolv-retry infinite
    nobind
    user nobody
    group nobody
    persist-key
    persist-tun
    ca ca.crt
    cert wangchang.crt
    key wangchang.key
    comp-lzo
    verb 3
    auth-user-pass

###3 进阶配置
[设置OpenVPN Server配置文件让客户端所有网络流量都走VPN连接](http://blog.csdn.net/itocenter/article/details/6183008)
###4 使用
服务端：

    service openvpn start

客户端，需要openvpn这个软件，一般分为Windows下和Linux下，关于使用方法，可以参考[中科大OpenVPN的使用说明](http://openvpn.ustc.edu.cn/)。    

<https://help.ubuntu.com/12.04/serverguide/openvpn.html>
<http://www.ironpike.net/zh-hans/archives/141>
<http://www.xiaohui.com/dev/server/20070514-install-openvpn.htm>
<http://wenku.baidu.com/view/75643ef14693daef5ef73dd4.html>
<http://openvpn.ustc.edu.cn/>
<http://www.lutiaotiao.com/main/title/21/>
<http://blog.csdn.net/dasss/article/details/1745289>
