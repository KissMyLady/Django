URL反向解析
====

## 使用方法: 

### 1. 总url配置: 

```python
from django.contrib import admin
from django.urls import path, re_path
from django.urls import include

urlpatterns = [
    path('admin/', admin.site.urls),
    #re_path(r'^', include('app_bookinfo.urls'))
    re_path(r'^', include(('app_bookinfo.urls', 'app_bookinfo'), namespace='app_bookinfo'))  # 反向解析
]
```

### 2. 次级url配置: 

```python
from django.contrib import admin
from django.urls import path, re_path
from app_bookinfo import views

urlpatterns = [
    re_path(r'^reverse_index$',   views.reverse_index), 
    re_path(r'^re_test$',    views.re_test, name='re_test'), # 反向解析 
]
```

### 3. 视图views配置: 
```python
from django.shortcuts import render, redirect
from django.http import HttpResponse
from app_bookinfo.models import BookInfo, HeroInfo

def reverse_index(request):
    return render(request, 'reverse_index.html')

def re_test(request):
    return render(request, 'reverse_test.html')
```

### 4. HTML配置: 
```html
<a href="/re_test"><h3>正常re_test测试页面: href="/re_test"</h3></a><br>

<a href="{%url 'app_bookinfo:re_test'%}"><h3>方向解析</h3></a><br>
```


