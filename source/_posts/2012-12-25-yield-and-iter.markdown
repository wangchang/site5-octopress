---
layout: post
title: "yield and iter"
date: 2012-12-25 15:01
comments: true
categories: 
---

[](http://lamoop.diandian.com/post/2011-11-04/40029544769)
生成器是一个函数，迭代器和生成器是python中两个非常强大的特性，编写程序时你可以不使用生成器达到同样的效果，但是生成器让你的程序更加pythonic。创建生成器非常简单，只要在函数中加入yield语句即可。函数中每次使用yield产生一个值，函数就返回该值，然后停止执行，等待被激活，被激活后继续在原来的位置执行。下边的例子实现了同样的功能.

def fib():
    a,b = 0,1
    while 1:
        a,b = b,a+b
        yield a
for f in fib():
    if f < 10000:
        print f
    else:
        break
不理解的地方之一，被激活继续在原来的地方执行，这个原来的地方是不是yield的地方，测试了一下。
    
    def fib():
        a,b = 0,1
        print 'this is fun fib'
        while 1:
            a,b = b,a+b
            print 'this will be yielding'
            yield a

    for f in fib():
        if f < 100:
            print f
        else:
            break

this is fun fib
this will be yielding
1
this will be yielding
1
this will be yielding
2
this will be yielding
3
    

http://www.cnblogs.com/huxi/archive/2011/07/01/2095931.html



classmethod staticmethod
http://besteam.im/blogs/article/71/

python classmethod，staticmethod 
http://blog.sina.com.cn/s/blog_759a8e39010166ov.html
http://luozhaoyu.iteye.com/blog/1506376
http://blog.donews.com/limodou/archive/2006/09/04/1028747.aspx
http://blog.chinaunix.net/uid-26922865-id-3382980.html
http://www.cnblogs.com/xuxm2007/archive/2010/08/30/1812566.html
http://luozhaoyu.iteye.com/blog/1513198
