Class视图    
=====  

易看懂, 易修改, 这种代码是不是看起来既清爽又舒服呢?     
清爽简洁的语法背后映射的是简洁易懂高效的逻辑        
  
我们可不可以设计一个class, 让视图函数封装在这个类里面           
有请求时, 我们请求到这个类,  根据是get请求还是post请求, 而后再调用相应的功能,  可以吗?       
 
## class视图封装    
Django中已经帮我们封装了一个View类方法, 拿来用就可以了    
[Django官方最新文档查看](https://docs.djangoproject.com/en/3.0/ref/class-based-views/base/)    
节选文档:    
```Python
from django.http import HttpResponse
from django.views import View

class MyView(View):

    def get(self, request, *args, **kwargs):
        return HttpResponse('Hello, World!')
```

```Python
from django.urls import path
from myapp.views import MyView

urlpatterns = [
    path('mine/', MyView.as_view(), name='my-view'),
]
```
可以看到view的使用,  我们在`views.py`文件中创建一个class, 然后在urls中引用这个class就可以了  

## Views视图函数  
![class-views-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/class-views-1.jpg)  


## Ulrs文件  
![class-views-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/class-views-2.jpg)  



### 几个比较   
[刘江博客](http://www.liujiangblog.com/blog/37/)    
* 通过HTTP请求方法的不同，将代码分隔在不同的类方法中，比如GET和POST，而不是类函数中的条件判断；    
* 可以使用面向对象的技巧，比如混入  
* 两者没有绝对的好坏和压倒性优势之分      
优 点:   
1. 我们在上面的普通views函数中,  无论那种方式, 你都要写if判断请求方式       
2. 明确限定请求方式   
3. 类视图适用于大量重复性的视图编写工作    
> 在简单的场景下，没几个视图需要编写，或者各个视图差别很大的情况时，还是函数视图更有效！ 


## 代码块  
```Python
from django.views import View

class RegisterViews(View):
    def get(self, request):
        return render(request, 'register.html')

    def post(self, request):
        username = request.POST.get('user_name')
        password = request.POST.get('cpwd')
        email = request.POST.get('email')
        allow = request.POST.get('allow')
        if allow != 'on':
            return render(request, 'register.html', {'errmsg': '请同意协议'})
        if not all([username, password, email]):
            return render(request, 'register.html', {'errmsg': '数据不完整'})
            pass
        if email == 123:
            return render(request, 'register.html', {'errmsg': '邮箱校验失败, 请重新输入'})

        try:
            user = User.objects.get(username=username)
        except:
            user = None

        if user:
            return render(request, 'register.html', {'errmsg': '用户名存在, 请重新输入'})
        else:
            # 3.没问题, 业务处理  https://www.cnblogs.com/wcwnina/p/9246228.html
            user = User.objects.create_user(username, password, email)
            user.is_active = 0
            user.save()
            return HttpResponse('登录成功')
``` 
