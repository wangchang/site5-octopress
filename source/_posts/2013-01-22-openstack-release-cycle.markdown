---
layout: post
title: "OpenStack开发和发行周期简介"
date: 2013-01-22 13:57
comments: true
categories:
- OpenStack
---
主要是翻译wiki的资料，让各位明白一下OpenStack的开发流程以方便大家加入OpenStack的贡献之中。

##1 发布周期Release Cycle
六个月一个版本，在过程中会频繁的有一些里程碑版本milestone发布，可以[点击这里](http://wiki.openstack.org/Releases)找到当前的开发计划，整个发行周期会有四个阶段。

注意：任何时候，你都可以参与贡献，即使在新版本发布的时候。

注意：每一个子Project（Nova Quantum等）都可以独立的选择发布的内容，只要是在OpenStack版本发行之前，当然，OpenStack还是鼓励按照OpenStack的计划来发布。

##2 计划阶段Planning (Design, Discuss and Target)
计划阶段是一个周期的开始，主要关注于今后将做什么，这个阶段会进行相应的讨论，并创建相应的Blueprint，通常持续4周，可以[点击这里](http://wiki.openstack.org/Summit)查看相应的Summit。

OpenStack使用的是Launchpad来追踪这个阶段，主要是用来管理BP。在这个阶段末尾，高层和根据提交的BP选择一个进入到周期计划中，并给这些BP一定的优先级，这些在计划中的BP由发行管理系统追踪并指向了一个特定的milestone。

###2.1 Blueprint
Launchpad的BP主要是用来追踪项目的进展情况。一个BP实际上是一个解决方案的描述，尽量简介以及针对问题，整个BP的使用包括在项目主页的Blueprint主页注册一个BP,
分配(assign)BP任务，更新状态三个部分。每个BP会有一个PTL（project team leader，项目组老大）加入，这个OpenStack官方人员，他会和提出BP的人一起做以下这些事：

__优先级Priority__:

PTL根据BP内容以及release情况定义一个优先级，优先级包括essential,high,medium,low,undefined，其中undefined表示优先级没定义，优先级再BP被接受后会第一时间定义。

__定义Definition__:

PTL可能会要求你使用定义状态来表示BP的状态，这些包括：

Discussion：BP还需要讨论

Drafting：正在撰写实现细节

Review：实现细节已完成，进入审核

Pending approval：BP做好准备被PTL接受

Approved：BP被接受，可以开始实现

Superseded：BP被另外的BP代替

Obsolete：BP已无效

__实现Implementation__:

表征BP的实现情况，这一步必须的！

Unknown：实现进度未知，请更改！

Not started：实现0%.

Started：实现>0%

Blocked：实现被中止，可能需要讨论

Slow progress：实现没有停止，但是进度缓慢，可能赶不上目标里程碑版本

Good progress：一切都好:)

Beta available：实现基本完成，代码已经存在一个分支或者正在进行drafted review。

Needs code review：已经提出进行合并merge

Implemented：完成

还有其他的一些状态：

Informational：不需要代码改变

Deferred：BP被延缓到下一个发行版

Needs infrastructure：不使用

Deployment：不使用

整个BP中有以下角色：

* Approver：这个项目的PTL

* Drafter：BP撰写者

* Assignee：BP实施者

整个BP的生命周期大致如下：

1 创建一个BP 

2 指定一个Approver，就是PTL

3 指定Assignee

4 指明Series goal

5 指明工作会进入到哪一个里程碑中

6 PTL审了以后，更改Definition为Approved，设定Series goal，设定优先级

7 然后就可以开始开发了，过程和完成后记得更改状态

更多可以参见<http://wiki.openstack.org/Blueprints>

##3 实现阶段
这个阶段就是根据BP的内容完成代码和文档，这个过程中会有很过里程碑版本的迭代。

一旦相应的工作准备好了进入代码主分支，首先需要push代码到Gerrit review system让大家来审阅，如果有任何改动，都需要在相应的里程碑版本发布的前一周内进行提出。

这个过程中，使用了Launchpad来追踪BP的实现情况。但是，不用所有的功能都经过BP的追踪，这个很随意的，BP主要是用来讨论和追踪主要的功能，并不限制你更多的贡献代码。

再相应的里程碑版本发布的几天前，会从稳定版本中产生相应的milestone-proposed代码分支，此时的代码只接受BUG修改，而不接受功能的改变。

###3.1 GerritWorkflow 
Gerrit主要是用来审阅代码用：<https://review.openstack.org/>

使用Gerrit，需要Launchpad账号，以及ssh key。首先登陆以上网址，设置你的联系信息，SSH公钥以及Groups，然后设置git的ueser信息：

    git config --global user.name "Firstname Lastname"
    git config --global user.email "your_email@youremail.com"
    git config --list

然后安装git的一个子工具 git-review，用来完成所有的review流程。

    apt-get install git-review

{% highlight 注意，git-review的指令其实都是git指令的一些集合，加入参数- v就可以打印出git指令。 %}

我们开始使用，首先clone项目：

    git clone git://github.com/openstack/nova.git
    #让git-review识别Girrit
    cd nova
    git review -s
    git remote update
    git checkout master
    git pull origin master

完成以后，还需要创建一个topic branch（参考西面的BranchModel）来维护你的工作，如果是根据BP来工作的话名字就叫做bp/BLUEPRINT。

    git checkout -b TOPIC-BranchModel

然后写代码吧，最后就是commit了，commit以后还需要向review提交一下。

    git commit -a
    git review

更多，可以参考<http://wiki.openstack.org/GerritWorkflow>    
 
###3.2 BranchModel
OpenStack的BranchModel是参照[NVIE model](http://nvie.com/posts/a-successful-git-branching-model/)进行规范。在一个里程碑发布之前的一个时间点(branch point)，会从主分支产生一个milestone-proposed分支，这个分支经过测试以后再和主分支合并。

下面是一个版本号：

拥有里程碑版本和发行版本的项目(Nova, Glance)：

* 从master分支产生VERSION~DEVMILESTONE~DATE.TRUNKREV (nova-2011.3~d2~20110531.302.tar.gz)

* 从milestone-proposed分支产生 VERSION~MILESTONE~DATE.rMPREV (nova-2011.3~d1~20110531.r301.tar.gz)

* 在Milestone时间产生VERSION~MILESTONE (nova-2011.3~d1.tar.gz)

* 发布时间时产生VERSION (nova-2011.3.tar.gz)

里程碑版本号就是发行版本号的项目(Swift)：

* master分支产生VERSION~DATE.TRUNKREV (swift-1.4.1~20110531.302.tar.gz)

* milestone-proposed分支产生VERSION~DATE.rMPREV (swift-1.4.0~20110531.r301.tar.gz)

* 发行时版本变为VERSION (swift-1.4.0.tar.gz)

##4 预发布阶段Pre-release (Release Candidates dance)
主要精力在于BUG修复和测试，也就不多讲了！
