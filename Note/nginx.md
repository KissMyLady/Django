## Uwsgi部署常规流程
* 遵循wsgi协议的web服务器  

1. uwsgi的安装    
`pip  install  uwsgi`   
2, uwsgi的配置     
项目部署时，修改settings.py:  
```Python
DEBUG=FALSE
ALLOWED_HOSTS=[‘*’] 
```
3. 新建文件,  `wsgi.ini`        
```
[uwsgi]
# 使用nginx连接时使用
# socket=127.0.0.1:8080
# 直接做web服务器使用 python manage.py runserver ip:port
http=127.0.0.1:8080
# 项目目录
chdir = /home/zhuying/django/daily_fresh
# 项目中wsgi.py文件的目录，相对于项目目录
wsgi-file=daily_fresh/wsgi.py
processes=4   # 启动的工作进程数
threads=2     # 指定工作进程的线程数
master=True   # 后台运行
pidfile=uwsgi.pid    # 保存启动之后,主进程号
daemonize=uwsgi.log  # 后台运行, 保存日志信息
virtualenv=/home/zhuying/.virtualenvs/today
```
请注意, 最后的实际进程数: 4+2= 6个   

#### uwsgi的启动和停止:  
```
启动:uwsgi –-ini 配置文件路径 例如:  uwsgi –-ini uwsgi.ini   
停止:uwsgi --stop uwsgi.pid路径 例如:uwsgi –-stop uwsgi.pid   
```
![ScreenShot-00275](https://github.com/KissMyLady/Django/blob/master/Img/ScreenShot-0027jpg)   

这里会发现我们的图片和css加载不了, 我们需要一个服务器来提供这些后续文件支持   

在配置Nginx之前, 我们先看看Django源码是怎么实现wsgi的     
wsgi.py文件     
```Python
import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'daily_fresh.settings')

application = get_wsgi_application()
```
`get_wsgi_application`   
```Python
import django
from django.core.handlers.wsgi import WSGIHandler


def get_wsgi_application():
    """
    The public interface to Django's WSGI support. Return a WSGI callable.

    Avoids making django.core.handlers.WSGIHandler a public API, in case the
    internal WSGI implementation changes or moves in the future.
    """
    django.setup(set_prefix=False)
    return WSGIHandler()
```
返回`WSGIHandler`类   
```Python 
class WSGIHandler(base.BaseHandler):
    request_class = WSGIRequest

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.load_middleware()

    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        signals.request_started.send(sender=self.__class__, environ=environ)
        request = self.request_class(environ)
        response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%d %s' % (response.status_code, response.reason_phrase)
        response_headers = [
            *response.items(),
            *(('Set-Cookie', c.output(header='')) for c in response.cookies.values()),
        ]
        start_response(status, response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```
好像很高深的样子,  待功力提升再来研究研究Django的源码   

像我们之前搭建的mini框架, wsgi是这样实现的:   
```Python 
...
else:
    env = dict()
    body = mini.application(env, self.set_resopnse_header)
    header = 'HTTP/1.1 %s\r\n' % self.status
    for temp in self.headers:
        header += '%s:%s\r\n' % (temp[0], temp[1])
    header += '\r\n'
    response = header + body
    new_socket.send(response.encode('utf-8'))

def set_resopnse_header(self, status, headers):
    self.status = status
    self.headers = headers
```
```Python 
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'),('charset=utf-8')])
    return 'Hello World! Kiss My Lady'
```
在服务器里调用wsgi接口, 接口返回状态码与头, 返回body   
`application(env, self.set_resopnse_header)`这里传入了字典与接受状态码与请求头, 设计的非常巧妙  
![ScreenShot-00251](https://github.com/KissMyLady/WEB_Server/blob/master/Img/ScreenShot-00251.jpg)   

服务的功能就是转发, Django作为框架就是调用各种资源返回给服务器, 返回的这个接口就是威士忌`WSGI`    
![Snip20161117_8]()   
但是uwsgi作为服务器我们的页面都不能完整显示, 该换个办法了   

我们需要在uwsgi前面再加入一个Nginx服务器   
![ScreenShot-00276](https://github.com/KissMyLady/Django/blob/master/Img/ScreenShot-00276.jpg)   
像这样, 把静态资源挪动到Nginx服务器下   
</br>  

# 1. Nginx由来   
Nginx是异步框架的网页服务器，也可以用作反向代理、负载平衡器和HTTP缓存。  
该软件由伊戈尔·赛索耶夫创建并于2004年首次公开发布。 2011年成立同名公司以提供支持。  
2019年3月11日，Nginx公司被F5 Networks以6.7亿美元收购。 Nginx是免费的开源软件，根据类BSD许可证的条款发布。   
> 编写时间: C语言   
> 初始版本: 2004年10月4日，​15年前   
> 原作者: 俄罗斯, Igor Sysoev(伊戈尔·赛索耶夫)      
>　许可协议: 类BSD   
> 语言: C   
> 开发者: NGINX, Inc   

### [2. 特 点:](https://zh.wikipedia.org/wiki/Nginx)  
Nginx可以部署在网络上使用FastCGI脚本、SCGI处理程序、WSGI应用服务器或Phusion Passenger模块的动态HTTP内容，并可作为软件负载均衡器   

Nginx使用异步事件驱动的方法来处理请求。Nginx的模块化事件驱动架构, 可以在高负载下提供更可预测的性能。  

Nginx是一款面向性能设计的HTTP服务器，相较于Apache、lighttpd具有占有内存少，稳定性高等优势。
与旧版本（<=2.2）的Apache不同，Nginx不采用每客户机一线程的设计模型，而是充分使用异步逻辑从而削减了上下文调度开销，
所以并发服务能力更强。整体采用模块化设计，有丰富的模块库和第三方模块库，配置灵活。 
在Linux操作系统下，Nginx使用epoll事件模型，得益于此，Nginx在Linux操作系统下效率相当高。
同时Nginx在OpenBSD或FreeBSD操作系统上采用类似于epoll的高效事件模型kqueue。

#### 使用广, 多 
根据Netcraft在2016年11月网络服务器调查, Nginx被发现是所有“活跃”站点(被调查站点的18.22%)和百万最繁忙站点(被调查站点的27.83%)中使用次数最多的Web服务器。     
根据W3Techs的数据，前100万个网站中的37.7%，前10万个网站中的49.7%，以及前10000个网站中的57.0%被使用。   
据BuiltWith统计，在全球前10000个网站中，有38.2%的网站使用Nginx。   
维基百科使用Nginx作为其SSL终端代理。   
从OpenBSD 5.2版本（2012年11月1日）开始，Nginx成为了OpenBSD基础系统的一部分，提供了替代Apache 1.3系统的替代方案，但是后来被替换为OpenBSD自己的httpd(8)

### [国内认为特点--知乎:](https://zhuanlan.zhihu.com/p/34943332)    
Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。

反向代理服务器？   
经常听人说到一些术语，如反向代理，那么什么是反向代理，什么又是正向代理呢？
由于防火墙的原因，我们并不能直接访问谷歌，那么我们可以借助VPN来实现，这就是一个简单的正向代理的例子。
这里你能够发现，正向代理“代理”的是客户端，而且客户端是知道目标的，而目标是不知道客户端是通过VPN访问的。

当我们在外网访问百度的时候，其实会进行一个转发，代理到内网去，这就是所谓的反向代理，
即反向代理“代理”的是服务器端，而且这一个过程对于客户端而言是透明的   


# [3. Ubuntu安装Negix](https://segmentfault.com/a/1190000015797789)  
1. 安装  
```
sudo apt-get update
sudo apt-get install nginx
```
2. 测试安装   
```
sudo nginx -t
```
应该看到:  
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
在浏览器输入: `192.168.0.104`, 应该看到:         
![ScreenShot-00278](https://github.com/KissMyLady/Django/blob/master/Img/ScreenShot-00278.jpg)  


# 4. Nginx文件配置   
1. nginx 配置转发请求给uwsgi  
```Python 
location / {
	include uwsgi_params;
	uwsgi_pass uwsgi服务器的ip:port;
}
```
2. nginx配置处理静态文件        
django settings.py中配置收集静态文件路径:
STATIC_ROOT=收集的静态文件路径 例如:/var/www/dailyfresh/static;

django 收集静态文件的命令:
	python manage.py collectstatic
执行上面的命令会把项目中所使用的静态文件收集到STATIC_ROOT指定的目录下。

收集完静态文件之后,让nginx提供静态文件，需要在nginx配置文件中增加如下配置:
location /static {
	alias /var/www/dailyfresh/static/;
}


