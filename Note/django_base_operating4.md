视图.py  
====
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
路由urls.py   
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

完成
===


## 完整代码  
