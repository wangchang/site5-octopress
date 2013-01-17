---
layout: post
title: "如何使用Readthedocs"
date: 2013-01-05 20:17
comments: true
categories: 
- Readthedocs
- Python
---
Readthedocs（RTD）是专门用来给项目Project托管文档的服务，支持Git等各种版本管理，使用了reStructuredText结构的Sphinx文档格式，下面简单总结一下怎么使用Readthedocs.
<!--more-->
##安装Sphinx
sphinx主要用来生成文档，可以把文本文档生成PDF等格式，类似Octopress把markdown文档转换成静态html格式。安装完成后，在项目中新建docs文件夹：

    apt-get install sphinx-doc
    cd /path/to/Project
    mkdir docs
    sphinx-quickstart

sphinx-quickstart会生成编译文档的环境，重点有index.rst和conf.py文件。

    root@node4:~/xxx/docs# sphinx-quickstart 
    Welcome to the Sphinx 1.1.3 quickstart utility.

    Please enter values for the following settings (just press Enter to
    accept a default value, if one is given in brackets).

    Enter the root path for documentation.
    > Root path for the documentation [.]: 

    Finished: An initial directory structure has been created.

    You should now populate your master file ./source/index.rst and create other documentation
    source files. Use the Makefile to build the docs, like so:
       make builder
    where "builder" is one of the supported builders, e.g. html, latex or linkcheck.

    root@node4:~/huawei_quantum/docs# ls
    Makefile  build  source

##基本配置
将上一步产生的所有文件加入版本控制，编译index.rst文件，记住，所有的文档撰写都需要是 reStructuredText格式的。完成后执行：

    make html

之后commit所有文件到仓库，github等等。

##使用
在Readthedocs的控制面板中，加入你的Project，完成一些基本配置就OK了。整个RTD的工作流程就是，RTD会从你的仓库的docs目录下，读取相应的文档文件并进行编译，然后将得到的Html文件存放在RTD的服务器上，在导入项目的时候，你会拥有RTD的一个二级域名，访问这个域名就相当于访问了你项目的文档,这样，你的项目就拥有了一个很好的文档工具。相当炫，其他的一些RTD的特性，请参考文档。比如我就为我的博客创建了一个文档库，地址为<https://wang-changs-docs.readthedocs.org/>,当然，RTD上面还托管了很多其他的项目文档，有兴趣可以查查。

##参考资料
<https://docs.readthedocs.org/en/latest/index.html>
<http://sphinx-doc.org/rest.html>



