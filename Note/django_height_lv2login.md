Django-Login第二种方法-类视图继承  
=====

如果我们的url转发有很多的url改怎么办, 复制粘贴我们也嫌麻烦  
```Python
re_path(r'^$',       login_required(UserInfoViews.as_view()))
re_path(r'^order',   login_required(UserOrderViews.as_view()))
re_path(r'^address', login_required(UserAddressViews.as_view()))
.....
```

## 类视图使用  
[类视图文档说明](https://yiyibooks.cn/xx/django_182/topics/class-based-views/mixins.html)    

Mixin
注意  
> 这是一个进阶的话题。需要建立在了解 基于类的视图的基础上。  
>　Django的基于类的视图提供了许多功能，但是你可能只想使用其中的一部分。例如，你想编写一个视图，它渲染模板来响应HTTP，但是你用不了TemplateView；或者你只需要对POST 请求渲染一个模板，而GET 请求做一些其它的事情。 虽然你可以直接使用TemplateResponse，但是这将导致重复的代码。  
> 由于这些原因，Django 提供许多Mixin，它们提供更细致的功能。例如，渲染模板封装在TemplateResponseMixin 中。Django 参考手册包含所有Mixin 的完整文档。    

![mxin-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/mixin-1.jpg)  

在定义视图的时候, 我们继承Mixin类就可以完成登录的判断了  

## 使用方法  
1. 在项目新建文件包`utils`, utils代表通用工具的意思, 对于一些通用的东西, 我们就可以定义在utils文件夹里面  

2. 新建文件`mixin.py`文件   

3. 在里面写代码  
```Python
from django.contrib.auth.decorators import login_required

class LoginRequireMixin(object):
    @classmethod
    def as_view(cls, **initkwargs):
        view = super(login_required, cls).as_view(**initkwargs)
        return login_required(view)
```

在视图中, 我们需要登录才能访问的, 直接让它继承这个`LoginRequireMixin`类就可以了     
```Python
class UserInfoViews(LoginRequireMixin, View):
    def get(self, request):
        return render(request, 'user_center_info.html', {'page': 'user'})


# /user/order
class UserOrderViews(LoginRequireMixin, View):
    def get(self, request):
        return render(request, 'user_center_order.html', {'page': 'order'})


# /user/site
class UserAddressViews(LoginRequireMixin, View):
    def get(self, request):
        return render(request, 'user_center_site.html', {'page': 'address'})
```
url地址正常配置就可以了, 他们看起来应该像是这样  
```Python
re_path(r'^$',       UserInfoViews.as_view()),
re_path(r'^order',   UserOrderViews.as_view()),
re_path(r'^address', UserAddressViews.as_view()),
```
此时, 我们就可以用很简单的方法实现用户登录才能访问的页面了  
这是第二种方法, 第一种方式是使用用函数嵌套实现的  
[登录装饰器配合使用](https://github.com/KissMyLady/Django/blob/master/Note/django_height_setlogin.md)  


