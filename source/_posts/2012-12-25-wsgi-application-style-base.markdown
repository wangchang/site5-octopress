---
layout: post
title: "WSGI应用常见的几种写法-基本形式"
date: 2012-12-25 11:05
comments: true
categories: 
- WSGI
- Web
---

Python WSGI标准[PEP-0333](http://www.python.org/dev/peps/pep-0333/)中说到一个WSGI是一个可调用callable的对象，这包含了一个函数，一个方法，一个类，或者拥有call方法的实例。下面我就对这句话做一个总结，举几个例子。这里的WSGI应用的功能都是返回请求的环境信息environ。
<!--more-->

##1 服务端
采用简单的WSGI服务器参考实现，如下：

    if __name__ == '__main__':
        from wsgiref.simple_server import make_server
        server = make_server('192.168.1.11', 7070, wsgi_application)
        server.serve_forever()

##2 函数方法
这是最简单的方法，先调用`start_response()`处理返回头信息，函数再返回可迭代对象，比如列表或者字符串。

    def app1(environ,start_response):
        start_response("200 OK",[("Content-type","text/plain")])
        content = []
        for k,v in environ.iteritems():
            content.append('%s:%s \n' % (k,v))
        return content#返回的列表或者字符串

相应的服务端：

    server = make_server('192.168.1.11', 7070, app1)

##3 带有call方法的实例
类定义中实现call方法，WSGI应用是这个类的一个实例。

    class app2(object):
        def __init__(self):
           pass
        def __call__(self,environ,start_response):
           start_response("200 OK",[("Content-type","text/plain")])
           content = []
           for k,v in environ.iteritems():
               content.append('%s:%s \n' % (k,v))
           return content#返回的列表或者字符串

    application = app2()

服务端：

    server = make_server('192.168.1.11', 7070, application)

##4 类class
用类作为WSGI应用不太一样，调用这个类时会产生一个类的实例，这个实例随后会需要__iter__迭代返回值。

    class app3(object):
        def __init__(self, environ, start_response):
            self.environ = environ
            self.start = start_response

        def __iter__(self):
            status = '200 OK'
            response_headers = [('Content-type', 'text/plain')]
            self.start(status, response_headers)
            content = ''
            for k,v in self.environ.iteritems():
                content += '%s:%s \n' % (k,v)
            yield content#返回的是字符串

服务端：

    server = make_server('192.168.1.11', 7070, app3)

这里注意，和上面的比较可以看出，如果需要把类的实例作为WSGI应用，则类中需要实现call方法，并且作为WSGI应用的应该是这个类的实例。

##5 方法method
用一个方法来作为WSGI应用，那么这个方法不可能是实例方法（上面已经讲过），一种方式肯定就是类中的静态方法了，类中的静态方法，就当做一个全局函数一样理解吧。

    class app4(object):
        def __init__(self):
            pass
        @staticmethod
        def wsgi(environ,start_response):
            start_response("200 OK",[("Content-type","text/plain")])
            content = []
            for k,v in environ.iteritems():
                content.append('%s:%s \n' % (k,v))
            return content#返回的是列表或者字符串

服务端：

    server = make_server('192.168.1.11', 7070, app4.wsgi)

那么，类方法，可以用么，当然可以，`classmethod`和`staticmethod`在使用上可以看做只是参数有个区别而已，如下：

    class app5(object):
        def __init__(self):
            print 'This is app5'
            pass
        @classmethod
        def wsgi(cls,environ,start_response):
            start_response("200 OK",[("Content-type","text/plain")])
            content = []
            for k,v in environ.iteritems():
                content.append('%s:%s \n' % (k,v))
            return content #返回列表或者字符串

服务端：

    server = make_server('192.168.1.11', 7070, app5.wsgi)

