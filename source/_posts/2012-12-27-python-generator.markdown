---
layout: post
title: "Python中的生成器语法Yield"
date: 2012-12-27 10:54
comments: true
categories: 
- Python
---
总体感觉，生成器Generation比迭代器复杂很多，特别是生成器在Python新版本2.7中又有很多新的用法。这篇文章我是把几篇文章的内容归纳了一下，自己测试了以后做的一个总结。所以建议先读完这篇文章然后再细细去研究一下参考文章。Python中生成器的标准是PEP 342.
<!--more-->
## Part I
来自文章[Python函数式编程指南（四）：生成器](http://www.cnblogs.com/huxi/archive/2011/07/14/2106863.html)

生成器是迭代器，同时也并不仅仅是迭代器，不过迭代器之外的用途实在是不多，所以我们可以大声地说：生成器提供了非常方便的自定义迭代器的途径。生成器就是一种迭代器。生成器拥有next方法并且行为与迭代器完全相同，这意味着生成器也可以用于Python的for循环中。另外，对于生成器的特殊语法支持使得编写一个生成器比自定义一个常规的迭代器要简单不少，所以生成器也是最常用到的特性之一。

在Python中，yield是生成器关键字，一般而言，yield是在函数内使用的，拥有yield语句的对象就是一个生成器。这里要切记:{% highlight yield是一个表达式 %},格式为:`yield 参数`。比如`yield 5`，5就是表达式`yield 5`的参数，而这个`表达式返回的就是自己的参数`。

    >>> def gen():  
    ...     print 'S 1'
    ...     yield 1
    ...     print 'S 2'
    ...     yield 2
    ... 
    >>> a = gen()
    >>> a
    <generator object gen at 0x7f816c591820>

注意两点，函数内有yield，就是一个生成器函数，所以`a=gen()`不会执行print语句，而是生成一个生成器对象。第一次调用生成器对象的next方法时，生成器才开始执行（构造生成器对象时不执行），直到遇到yield时暂停执行（挂起），并且yield的参数将作为此次next方法的返回值。 

    >>> a.next() 
    S 1
    1

之后每次调用生成器对象的next方法，生成器将从上次暂停执行的位置恢复执行生成器函数，直到再次遇到yield时暂停，并且同样的，yield的参数将作为next方法的返回值。

    >>> a.next() 
    S 1
    1
    >>> a.next()
    S 2
    2

如果当调用next方法时生成器函数结束（遇到空的return语句或是到达函数体末尾），则这次next方法的调用将抛出StopIteration异常（即for循环的终止条件）。

    >>> a.next()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    StopIteration

生成器函数在每次暂停执行时，函数体内的所有变量都将被封存(freeze)在生成器中，并将在恢复执行时还原，并且类似于闭包，即使是同一个生成器函数返回的生成器，封存的变量也是互相独立的。这里还有几个特点：

* 从上面可以看出，生成器yield语句执行后会暂定到当前位置，下次执行时再行当前位置开始继续往下执行。

* 生成器函数可以带参数，你看到了，这就是函数定义的嘛。

* 生成器中不能有return 值语句（可以有空的return语句），因为它返回的是yield语句的参数。

##Part II
参考文章[Python 深入理解yield](http://blog.chinaunix.net/uid-26922865-id-3382980.html)

先看一个问题，上面说到yield x会返回参数x，如果有`a = yield x`，那么又是怎样的呢？

    >>> def gen():
    ...     print 'S 1'
    ...     a = yield 1 #(1)位置
    ...     print 'This is a',a #（2）位置
    ...     print 'S 2'
    ...     b = yield 2
    ...     print 'This is b',b
    ... 
    >>> a = gen()
    >>> a
    <generator object gen at 0x7f8169402870>
    >>> a.next()
    S 1
    1
    >>> a.next()
    This is a None
    S 2
    2

结果很有意思，来分析一下，从结果来看，函数next（）方法以后，到（1）位置，返回的是yield的参数，1，相当于`return 1`，所以屏幕有打印结果，然后挂起，再调用next（）方法时候，到达（2）位置，但是a = None，也就是说，`a = yield x实际上就返回x（return x），这个x并不会赋值给变量a`.

send(msg) 方法，你可以向 generator 发送消息。执行一个 send(msg) 会恢复 generator 的运行，然后发送的值将成为当前 yield 表达式的返回值。然后 send() 会返回下一个被 generator yield 的值，如果没有下一个可以 yield 的值则引发一个异常。,其实next()和send()在一定意义上作用是相似的，区别是send()可以传递yield表达式的值进去，而next()不能传递特定的值，只能传递None进去。因此，我们可以看做c.next() 和 c.send(None) 作用是一样的。需要提醒的是，第一次调用时，请使用next()语句或是send(None)，不能使用send发送一个非None的值，否则会出错的，因为没有yield语句来接收这个值。说白了，就是这么个道理：

__在a = yield x中，我使用send(msg)，那么yield语句接收到这个msg后，就会把msg赋值给a.__

    >>> def gen():
    ...     print 'S 1'
    ...     a = yield 1 #(1)位置
    ...     print 'This is a',a 
    ...     print 'S 2'
    ...     b = yield 2#（2）位置
    ...     print 'This is b',b
    ... 
    >>> b = gen()
    >>> b.send(None) #相当于b.next（）
    S 1
    1
    >>> b.send('HaHa')
    This is a HaHa
    S 2
    2

b.send(None),执行到(1)位置，yield 1返回了1，然后等待，b.send（msg）以后，将从（1）位置继续执行，将发送的值'HaHa'作为当前yield表达式的返回值，将msg赋值给a，然后继续执行，将下一个yield语句的参数作为send()函数的返回值，到位置(2) 以后返回2，然后等待。这里也可以得到，{% highlight next和send（）方法的返回值很特殊，返回的是yield表达式的参数 %}，如下：

    def h():
    print 'Wen Chuan',
    m = yield 5 # Fighting!
    print m
    d = yield 12
    print 'We are together!'

    c = h()
    m = c.next() #m 获取了yield 5 的参数值 5
    d = c.send('Fighting!') #d 获取了yield 12 的参数值12，m=Fighting
    
我细细解释一下，c.send（'Fighting'），从位置（1）开始执行，将发送值‘Fighting’作为当前yield表达式的返回值，m='Fighjting',将下一个yield的参数12，作为当前send()函数的返回值，d=12.如果你还不是很明白，那么接着看，马上恍然大悟了。

##Part III
[Python 学习]2.5版yield之学习心得](http://blog.donews.com/limodou/archive/2006/09/04/1028747.aspx)可以把 yield 想象成下面的伪代码，一切都明白了：

    x = yield i  <==> 

    return(i)
    x = wait_and_get_next_msg()

第一句话return(i)含义就是yield 的参数i将作为send()函数的返回值，第二句` x = wait_and_get_next_msg()`含义就是阻塞，然后等待下一次的send(msg)函数触发，并把下一次的send(msg)函数中的msg传递给x，作为yield x表达式的返回值。
