---
layout: post
title: "WSGI应用常见的几种写法-高级形式 "
date: 2012-12-26 09:39
comments: true
categories: 
- WSGI
- Web
- Python
---

上一节中说到了WSGI应用的几种基本形式，这一节就来说一些高级的用法，所谓高级，我理解的也就是使用装饰器`@`，WebOb，以及中间件的形式，当然，还有一些更有意思的用法，比如Paste.Deploy这种载入WSGI应用的形式，我以后会做总结。还是和前篇[WSGI应用常见的几种写法-基本形式](http://op.wachang.net/blog/2012/12/wsgi-application-style-base/)一样，本文实现的WSGI应用都是一个回显请求环境信息的程序。
<!--more-->
##1 WebOb
WebOb就是将请求和相应封装成Request和Response类，就可以使用一个类方法简化真个操作，如下，直接将一个字符串作为resp_body传给给Response对象，而不用考虑WSGI中的可迭代对象，这里要注意，Response对象`是一个WSGI应用`，所以在最后的时候我们使用`resp(environ,start_response)`就返回了，我们不关心返回的细节问题了，而在标准WSGI中，这一步才只是WSGI应用的入口函数。

    def app6(environ, start_response):
        req = Request(environ)
        resp_body = ''
        for k,v in req.environ.iteritems():
            resp_body += '%s:%s\n' % (k,v)
        resp = Response(body=resp_body)
        return resp(environ, start_response)
服务端：
    
    server = make_server('192.168.1.11', 7070, app6)

##2 用装饰器@，Controller
用装饰器@以后，重要的思想就是把WSGI应用关于返回流程的处理（包括header，状态等）和返回Body的处理分开，这样的话实际上一个应用只关注于产生返回的Body就可，而其他的处理流程则交给装饰器controller来完成，这个controller，不要理解成控制神马的，只是一个名字，其原意是作为WSGI中`一个资源的控制`，不用太操心就当做一个名字就好。

    def controller1(func):
        def application(environ,start_response):
            #do something else
            resp_body = func(environ)
            start_response("200 OK",[("Content-type","text/plain")])
            return resp_body
        return application

    @controller1
    def app7(environ):
        content = []
        for k,v in environ.iteritems():
            content.append('%s:%s \n' % (k,v))
        return content

服务端：

    server = make_server('192.168.1.11', 7070, app7)

##3 结合WebOb和装饰器
上面两种方式可能看起来没多大用，但是二者结合到一起，那就不一般了，这样子的话我们用装饰器controller来处理environ,并封装Request和Response，最后让应用函数来处理返回信息，只给出一个字符串Body就可以了。这会大大简化WSGI应用的开发流程。

    def controller2(func):
        def replacement(environ, start_response):
            req = Request(environ)
            try:
                resp_body = func(req)
            except exc.HTTPException, e:
                resp_body = e
            resp = Response(body=resp_body)#body must be a string
            resp.status = '200 very OK'
            return resp(environ, start_response)
        return replacement

    @controller2
    def app8(req):
        ret = ''
        for k,v in req.environ.iteritems():
            ret = ret + '%s:%s \n' % (k,v)
        return ret

服务端：

        server = make_server('192.168.1.11', 7070, app8)

以上是用装饰器来装饰一个函数，还有一种用法，用装饰器来装饰一个类。

    def controller3(cls):
        def replacement(environ, start_response):
            req = Request(environ)
            instance = cls(req, **req.urlvars)
            method = req.method
            if method == 'GET':
                func = getattr(instance, method.lower())
                resp_body = func()
                if isinstance(resp_body, basestring):
                    resp = Response(body=resp_body)
            return resp(environ,start_response)
        return replacement

    class App9(object):
        def __init__(self,req,**args):
            self.req = req
        def get(self):
            body = ''
            for k,v in self.req.environ.iteritems():
                body += '%s"%s \n' % (k,v)
            return body

    app9 = controller3(App9)

上面关于类的装饰器可以仔细琢磨一下哈，相应的服务端：

    server = make_server('192.168.1.11', 7070, app9)

##4 总结
两篇文章我一共说明了9种WSGI应用的写法，当然WSGI只是一个协议，相应的实现还是很灵活的，以后再实践中再分享一些新的用法。整个代码我使用Gist存放，连接在此：<https://gist.github.com/4378216>


