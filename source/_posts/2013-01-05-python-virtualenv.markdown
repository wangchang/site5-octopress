---
layout: post
title: "Python中的VirtualEnv(一)"
date: 2013-01-05 18:43
comments: true
categories: 
- Python
- Virtualenv
---
##1 什么是virtualenv？
VirtualEnv是一个工具，用于在一台机器上创建多个独立的python运行环境。特点如下：

* 隔离项目之间的第三方包依赖，如A项目依赖django1.2.5，B项目依赖django1.3

* 为部署应用提供方便，把开发环境的虚拟环境打包到生产环境即可,不需要在服务器上再折腾一翻。

* 解决库之间的版本依赖，比如同一系统上不同应用依赖同一个库的不同版本。

* 解决权限限制，比如你没有root权限。

* 尝试新的工具，而不用担心污染系统环境。

<!--more-->
##2 使用virtualenv
一般来讲，我们称virtualenv创建的一个独立Python环境为一个“沙盒”。执行下面的命令安装：

    easy_install virtualenv

然后创建一个沙盒，pro1：

    root@node4:~# virtualenv pro1
    New python executable in pro1/bin/python
    Installing setuptools............done.
    Installing pip...............done.

当前目录下就会创建一个pro1的目录，沙盒的所有内容都在该目录下。在ENV/bin目录下有个定制的python解释器，它会使用ENV/lib/pythonX.X/site-packages目录下的库。如果一个python程序有`#!/path/to/ENV/bin/python`，那么这个程序也是使用的是虚拟环境中的库文件。使用的pip工具是ENV/bin/pip，通过pip安装的库都放在上述目录中。一个虚拟环境的特点就是{% highlight 使用自己的python库，不会使用其他环境的库文件，默认情况下也不会使用系统中的python库文件 %}。这中间还有几个比较重要的参数：

    virtualenv --system-site-packages ENV

如果使用这个，那么你的虚拟环境中的库文件是继承系统site-packages（/usr/lib/python2.7/site-packages）中的。

启用这个环境：

    root@node4:~# source pro1/bin/activate

这一步其实就是更改了环境变量PATH，提示符也会变成`(pro1)root@node4:~#`。之前说到沙盒的解释器会优先查找沙盒的site-packages目录，如果找不到，就会查找系统的site-packages目录。我们可以在创建沙盒时加入参数--no-site-packages来禁止它查找系统的目录。如下可以关闭这个虚拟环境：

    (pro1)root@node4:~/pro1/bin# deactivate

##3 Bootstrap
默认创建ENV的时候不会对环境操作，但是如果想在创建ENV的时候执行一些特殊的动作，比如创建一个web应用(说白了，就是给环境Hook一下，自定义环境)，那么就需要Bootstrap脚本。
具体的写法，可以参见下面的参考文章。

<http://www.cnblogs.com/cheungjustin/archive/2011/12/08/2281041.html>
<http://www.virtualenv.org/en/latest/>
<https://virtualenv-chinese-docs.readthedocs.org/en/latest/>
<http://blogs.360.cn/blog/how-360-uses-python-1-virtualenv/>


