---
layout: post
title: "Ubuntu12.04安装配置Octopress"
date: 2012-12-27 16:19
comments: true
categories: 
- Octopress
---
这个文章实在我以前写的"在Ubuntu 12.04下用Github安装和维护Octopress"上的一个更新，主要更新了配置和自定义样式部分，同时，因为Octopress不一定是要用Github存放的，只要是支持静态页面的主机空间就可以部署Octopress,本文也加入了这些内容。
<!--more-->
##1 安装及使用
>OS:Ubuntu 12.04 
###1.1 安装Ruby环境###
利用RVM安装ruby,RVM管理Ruby的话感觉更爽一点,自己也懒得编译了.

    root@Node1:apt-get install curl git
    root@Node1:curl -L get.rvm.io | bash -s stable

根据RVM的要求,还需要将用户加入组以及更新配置.

    root@Node1:gpasswd -a root rvm
    root@Node1:source /etc/profile.d/rvm.sh

安装编译依赖以及Ruby

    root@Node1:apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf
    root@Node1:rvm install 1.9.2

OK,环境安装完成。
###1.2 安装Octopress###
先通过git克隆octopress的代码.再安装相应的组件.

    git clone git://github.com/imathis/octopress.git octopress
    cd octopress
    gem install bundler
    bundle install

然后安装默认主题:

    rake install

以上安装的是默认的主题，其实Octopress提供了很多的主题，安装也特别简单，可以参考[Octopress主题集](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-themes)，安装的方法也相当简答。到这里，整个安装也就完成了。
###1.3 配置和使用###

    config.yml
    rake new_post['name']
    rake new_page['page name']
    rake generate
    rake preview

上面是基本使用。
###1.4 部署###
####Github Page####
Octopress的部署现在一般两种部署方式，一是用Github Page功能把网站做到Github上，相关原理请自行查阅，需要做这么几个事：

    rake setup_github_pages 设置于github的关联
    rake generate 生成静态页面
    rake deploy 向github发送页面





##页面
Octopress在 .themes 目录提供了一个默认主题。当你安装Octopress时，HTML和Javascripts被复制到 /source，Sass样式表被复制到 /sass。 你可以随意修改上述文件。首先我们看看定义页面的内容，目录树入下：

    source/
        _includes/    # Main layout partials
        custom/     # <- Customize head, header, navigation, footer, and sidebar here
        asides/     # Theme sidebar partials
        post/       # post metadata, sharing & comment partials
    _layouts/     # layouts for pages, posts & category archives

##其他

* 通过修改`octopress/sass/custom/_layout.scss中$indented-lists: true`可以让文章中列表和每行对齐，这个对于文章中使用markdown语法的列表功能，显示效果不错。

* 在source目录下增加robots.txt，允许或者禁止搜索引擎抓取。
