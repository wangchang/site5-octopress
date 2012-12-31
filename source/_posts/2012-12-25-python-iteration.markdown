---
layout: post
title: "Python中的迭代器Iteration"
date: 2012-12-25 15:01
comments: true
categories: 
- Python
---
随着Python学习的深入，迭代器和生成器看来是必须要会的了，搜了一大堆资料，学习了一下，这里做个总结。
<!--more-->
##1 迭代器
迭代器是访问集合内元素的一种方式。迭代器对象从集合的第一个元素开始访问，直到所有的元素都被访问一遍后结束。主要是为类序列对象提供一种类序列操作的接口。

迭代器不能回退，只能往前进行迭代。这并不是什么很大的缺点，因为人们几乎不需要在迭代途中进行回退操作。

迭代器的另一个优点就是它不要求你事先准备好整个迭代过程中所有的元素。迭代器仅仅在迭代至某个元素时才计算该元素，而在这之前或之后，元素可以不存在或者被销毁。这个特点使得它特别适合用于遍历一些巨大的或是无限的集合，比如几个G的文件，或是斐波那契数列等等。这个特点被称为延迟计算或惰性求值(Lazy evaluation)。

对于原生支持随机访问的数据结构（如tuple、list），迭代器和经典for循环的索引访问相比并无优势，反而丢失了索引值（可以使用内建函数enumerate()找回这个索引值，这是后话）。但对于无法随机访问的数据结构（比如set）而言，迭代器是唯一的访问元素的方式。

注意，`迭代器是一个概念`，在使用中我们使用的是迭代器对象。对于迭代的实现，主要是通过对象的next()方法，而从根本上说，{% highlight 迭代器就是一个拥有next方法的对象 %}。下面说说Python中的方法：

##2 iter()函数
obj参数如果是序列，很简单，可以根据索引到迭代结束。如果obj是一个类，那么这个类需要实现__iter__()和next()方法。

    >>> a = [1,2,3,4,5] #这是一个序列
    >>> type(a)
    <type 'list'> #这是一个序列，不是迭代器
    >>> dir(a)
    ['__iter__', '__str__', 'append', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort'] #这个序列没有next方法，所以不能迭代的next方法。
    >>> b = iter(a)
    >>> b
    <listiterator object at 0x1af0b50> #通过iter()，产生一个迭代器b。
    >>> dir(b)
    ['__class__', '__delattr__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__iter__'，'next'] #看看b，有了next方法。
    >>> b.next()
    1
    ...
    >>> b.next()#迭代越界，抛出异常。
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    StopIteration

这里，有两点我比较有体会：

* 为啥列表有个\__iter\__方法？
  
这个方法就是把列表变成一个可迭代的对象，和iter（）函数实际上是调用__iter__函数的。

* 列表不是可以`for x in y`这种形式访问么，为啥还需要迭代器？

Python专门将关键字for用作了迭代器的语法糖。在for循环中，Python将in后面的对象自动调用工厂函数iter()，而这个iter()，又是调用的对象的__iter__()方法，获得迭代器，自动调用next()获取元素，还完成了检查StopIteration异常的工作，相当于for是实现了下面的代码：

    it = iter(list)
    try:
        while True:
            val = it.next()
            print val
    except StopIteration:
        pass

* 常用的几个内建数据结构tuple、list、set、dict都支持迭代器，字符串也可以使用迭代操作。你也可以自己实现一个迭代器，如上所述，只需要在类的\__iter\__方法中返回一个对象，这个对象拥有一个next()方法，这个方法能在恰当的时候抛出StopIteration异常即可。但是需要自己实现迭代器的时候不多，即使需要，使用生成器会更轻松。异常并不是非抛出不可的，不抛出该异常的迭代器将进行无限迭代。

iter()函数还有一个形式`iter(itr, 'c')`,意思是说，返回itr的iterator，而且在之后的迭代之中，迭代出来'c'就立马停止,但是，这里要求itr必须是callable的，即要实现__call__函数。

##3 类实现迭代
下面来自己实现一个迭代器，从上面我们看出三点，`一个迭代器对象，需要有next()方法，如果用类实现迭代器，需要实现\__iter\__()方法，这个方法返回一个迭代器（可以用iter()函数产生，或者自己写一个有nextf方法的对象）。如果要支持iter()两个参数类型，那么，这个类还需要实现__call__方法.`

    class Iter(object):
        def __init__(self):
            self.data = [1,2,3,4,5]
        def __iter__(self):
            return iter(self.data) #函数需要返回一个迭代器，这里实际上调用了列表的__iter__方法，生成一个迭代器，然后返回。

    a = Iter()#初始化一个实例
    iter_a = iter(a) #调用实例的__iter__方法，实际调用列表的__iter__方法。

    while True:
        try:
            print iter_a.next()
        except StopIteration:
            pass

    root@Node1:~/python# python iter.py 
    1
    2
    3
    4
    5

我们可以做一个总结：{% highlight iter(obj)方法会调用obj的__iter__()方法，返回一个可迭代的对象，而这个对象里面，必然含有next()方法，next方法可以控制迭代的结束。 %}

上面是根据列表产生的迭代器，所以这个迭代器就已经有next()方法了。所以我的返回没问题，上面说到，一个迭代器对象一定要有next（）方法，那么，我们试试在类里面的__iter__中产生一个含有next方法的对象：

    class Iter_n(object):
        def __init__(self,count):
            self.data = 1
            self.count = count
        def __iter__(self):
            return self
        def next(self):
            if self.data < self.count:
                self.data += 1
                return self.data
            else:
                raise StopIteration

    a = Iter_n(5)
    iter_a = iter(a)
    while True:
        try:
            print iter_a.next()
        except StopIteration:
            pass

    root@Node1:~/python# python iter.py 
    2
    3
    4
    5

上面还说到，iter（）两个参数的形式需要call方法，call方法的意思何在？

iter(obj,sential),会重复调用obj，所以obj是类的话需要实现__call__方法，这个类的实例才是可调用的，直到迭代器的下个值等于sential就停止。如下代码：

    class Iter_c(object):
        def __init__(self):
            self.data = [1,2,3,4,5]
            self.__name__ = 'Iter_c'
            self.iterobj = iter(self.data)

        def __call__(self):
            print '%s __call__' % self.__name__
            n = self.iterobj.next()
            print n
            return n

        def __iter__(self):
            return self.iterobj

    a = Iter_c()
    iter_a = iter(a,3)
    while True:
        try:
            print iter_a.next()
        except StopIteration:
            pass

运行结果：

    root@Node1:~/python# python iter.py 
    Iter_c __call__
    1
    1
    Iter_c __call__
    2
    2
    Iter_c __call__
    3

##4 深入研究
如果给一个迭代器再执行一次iter（）会怎样？

    >>> x = [1,2,3,4,5]
    >>> y = iter(x)
    >>> y
    <listiterator object at 0x1f483d0>
    >>> z = iter(y)
    >>> z
    <listiterator object at 0x1f483d0>

我们看到两个迭代器对象的地址是一样的，也就是说，`对一个迭代器对象执行iter（）得到的还是这个对象`    

我们来看看字典，我们常见的字典访问形式是：

    for k,v in dict.iteritems():
      do something

首先，回忆两点，for x in y会对y执行iter()函数，实际上是调用y的__iter__方法。而字典含有__iter__方法，但是字典的__iter__方法只是创建键的迭代器，而不是键值的迭代器，所以这么用是可以的：

    >>> for k in a:
    ...     print k
    ... 
    a
    b

而字典的iteritems（）方法，是产生键值的迭代器，所以dict.iteritems（）就产生了一个键值迭代器，含有next方法。

    >>> a 
    {'a': 1, 'b': 2}
    >>> b = a.iteritems()
    >>> b
    <dictionary-itemiterator object at 0x1f38998>

上面说到，for x in y会对y执行iter(y)，以及对一个迭代器对象执行iter()返回的是它本身，所以有下：

    >>> c=iter(b)
    >>> c
    <dictionary-itemiterator object at 0x1f38998>

注意，c和b的地址是一样的，他们是同一个迭代器。如此我们常见的字典访问方式你就懂了撒。

##5 总结
* iter(obj)调用obj的__iter__方法，返回一个迭代器对象，这个对象含有next方法。

* `for x in y`是一个语法糖，底层的实现就是iter（y）变成可迭代对象，然后执行next方法，直到抛出异常。
            
##参考资料
<http://www.cnblogs.com/huxi/archive/2011/07/01/2095931.html>
<http://luozhaoyu.iteye.com/blog/1513198>
