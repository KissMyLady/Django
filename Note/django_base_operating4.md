视图--模板
====
## 视图

后台管理页面做好了，接下来就要做公共访问的页面了。当我们刚刚在浏览器中输入 http://127.0.0.1:8000/admin/ 之后，浏览器显示出了后台管理的登录页面，那有没有同学想过这个服务器是怎么给我们找到这个页面并返回呢？/admin/是我们想要请求的页面，服务器在收到这个请求之后，就一定对应着一个处理动作，这个处理动作就是帮我们产生页面内容并返回回来，这个过程是由**视图**来做的。

对于django的设计框架MVT，用户在URL中请求的是视图，视图接收请求后进行处理，并将处理的结果返回给请求者。

使用视图时需要进行两步操作：

```txt
* 1.定义视图函数
* 2.配置URLconf
```

#### 1.定义视图

视图就是一个Python函数，被定义在views.py中。

视图的必须有一个参数，一般叫request，视图必须返回HttpResponse对象，HttpResponse中的参数内容会显示在浏览器的页面上。

打开booktest/views.py文件，定义视图index如下

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("index")
```

#### 2.配置URLconf

##### 查找视图的过程

请求者在浏览器地址栏中输入url，请求到网站后，获取url信息，然后与编写好的URLconf逐条匹配，如果匹配成功则调用对应的视图函数，如果所有的URLconf都没有匹配成功，则返回404错误。

一条URLconf包括url规则、视图两部分：

- url规则使用正则表达式定义。
- 视图就是在views.py中定义的视图函数。

需要两步完成URLconf配置：

- 1.在应用中定义URLconf
- 2.包含到项目的URLconf中

在booktest/应用下创建urls.py文件，定义代码如下：

```python
from django.contrib import admin
from django.urls import path, re_path
from app_bookinfo import views

urlpatterns = [
    re_path(r'',  views.index)
]
```

包含到项目中：打开test1/urls.py文件，为urlpatterns列表增加项如下：

```python
re_path(r'^', include('app_bookinfo.urls'))
```

test1/urls.py文件完整代码如下：

```python
from django.contrib import admin
from django.urls import path, re_path
from django.urls import include
urlpatterns = [
    path('admin/', admin.site.urls),
    re_path(r'^', include('app_bookinfo.urls'))  # 打开ip地址时， 默认代开bookinfo
]
```

#### 请求访问

视图和URLconf都定义好了，接下来在浏览器地址栏中输入网址：

```python
http://127.0.0.1:8000/
```

网页显示效果如下图，视图被成功执行了。

## Re_path匹配  
严格匹配开头与结尾的url规则   
```Python
from django.contrib import admin
from django.urls import path, re_path
from app_bookinfo import views
urlpatterns = [
    re_path(r'test_response',  views.test_response),
    re_path(r'^go1$', views.go1),
    re_path(r'^go2$', views.go2),
    re_path(r'^', views.index)]
``` 
方式: `^ + $`   
此时若在浏览器输入`go1`与`go2`, 可以看到是可以严格匹配规则的   

## views.py文件  
定义调哪个函数  
```Python
from django.template.loader import get_template
from django.shortcuts import render
from django.http import HttpResponse

def loadpage(request):
    temp_index = get_template('booktest/index.html')
    res_html = temp_index.render(locals())
    return HttpResponse(res_html)

def center(request):
    return HttpResponse('个人中心')

def page(request):
    return render(request,'booktest/index.html',
                  {'content':'Hello World', 'list':list(range(1,10))})

from booktest.models import BookInfo
def show_books(request, **kwargs):
    books = BookInfo.objects.all()
    return render(request ,'booktest/show_books.html' ,{'books':books})

def detial(request ,id):
    book = BookInfo.objects.get(id=id)
    heros = book.heroinfo_set.all()
    return render(request ,'booktest/detial.html' ,{'book' :book,'heros':heros})
```
urls.py   
====
```Python
from booktest import views
from django.urls import path, re_path

urlpatterns = [
    #re_path(r'show_books/4/'  ,views.detial),
    #re_path(r'show_books/1/'  ,views.detial),
    re_path(r'index'    ,views.index),
    re_path(r'loadpage' ,views.loadpage),
    re_path(r'center'   ,views.center),
    re_path(r'page'     ,views.page),
    re_path('^show_books(?P<show_books>/$)' ,views.show_books),
    path('show_books/<int:id>/'             ,views.detial)
]
```


模板template    
====
setting.py中设置  
```python
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```

将来要呈现的网页模板  
## index.html文件    
```Html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>模板文件</title>
</head>
<body>
<h1>模板文件</h1>
<br>使用模板变量

{{ content }}

<br>
使用列表
{{ list }}

<a href="#"></a>
for循环<br>
<ul>
    {% for i in list %}
        <li>{{ i }}</li>
    {% endfor %}

</ul>

</body>
</html>
```
## detial.html文件    
```Html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>英雄信息</title>
</head>
<body>

这是详情页
<h1>{{ book.b_title}}</h1>
英雄信息如下<br/>

<ul>
    {% for hero in heros %}
        <li>{{hero.h_name}}--{{hero.h_comment}}</li>
    {% empty %}
        <li>没有英雄信息</li>
    {% endfor %}
</ul>


</body>
</html>
```
## show_books.html文件  
```Html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Books</title>
</head>
<body>
图书目录如下
<ul>
    {%for book in books%}
        <li><a href="/show_books/{{book.id}}">{{book.b_title}}</a></li>
    {%endfor%}
</ul>

</body>
</html>

```


## 看完了  
- [返回Django主页](https://github.com/KissMyLady/Django)  
- [Django-初始化](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating.md)  
- [Django-M模块介绍](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating2.md)   
- [Django-admin管理](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating3.md)
- [Django-完整流程](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating5.md)
