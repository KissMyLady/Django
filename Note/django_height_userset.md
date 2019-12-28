用户注册时的数据验证    
=====   
![uset-1](https://github.com/KissMyLady/Django/blob/master/Img/uset-1.jpg)   
当输入的数据不对时, 我们返回首页并返回一个错误     
具体代码, 复制粘贴即可         
```Python
def register_handle(request):
    # 一般web开发, 通用流程
    # 1. 如果访问地址有数据, 接收,   username, password, email
    username = request.POST.get('user_name')
    password = request.POST.get('cpwd')
    email =    request.POST.get('email')
    allow =    request.POST.get('allow')
    if allow != 'on':
        return render(request, 'register.html', {'errmsg':'请同意协议'})
    # 2. 校验
    if not all([username, password, email]):
        return render(request, 'register.html', {'errmsg':'数据不完整'})
        pass
    if not re.match(r"^[a-z0-9][\w\.\-]*@[a-z0-9\-]+(\.[a-z]{2,5}){1,2}$/", email):
        return render(request, 'register.html', {'errmsg':'邮箱校验失败, 请重新输入'})


    # 3. 没问题, 业务处理

    # 4. 有问题
    return HttpResponse('登录成功')
```
## 承接上文, HTML关键代码块     
```html
<form method="post" action="/user/register_handle">
% csrf_token %}
<ul>
	<li>
		<label>用户名:</label>
		<input type="text" name="user_name" id="user_name">
		<span class="error_tip">提示信息</span></li>					
	<li>
		<label>密码:</label>
		<input type="password" name="pwd" id="pwd">
		<span class="error_tip">提示信息</span></li>
	<li>
		<label>确认密码:</label>
		<input type="password" name="cpwd" id="cpwd">
		<span class="error_tip">提示信息</span></li>
	<li>
		<label>邮箱:</label>
		<input type="text" name="email" id="email">
		<span class="error_tip">提示信息</span></li>
	<li class="agreement">
		<input type="checkbox" name="allow" id="allow" checked="checked">
		<label>同意”第八宇宙用户使用协议“</label>
		<span class="error_tip2">提示信息</span></li>
	<li class="reg_sub">
		<input type="submit" value="注 册" name=""></li>
</ul>				
</form>
	{{ errmsg }}注意这里返回的是错误信息, 要加上去    
```
## Django中已经封装好了的User方法  
![uset-2](https://github.com/KissMyLady/Django/blob/master/Img/uset-2.jpg)  
所以, 我们可以改一改    
```Python
# 3.没问题, 业务处理
user = UserModel()
user.username = username
user.password = password
user.email = email
user.save()
# 4. 有问题,
return HttpResponse('登录成功')
```  
改成这样:     
注意      
> from django.contrib.auth.models import User       
> from app_user.models import UserModel as User  
> 两者区别      
在2.2中,  直接导入会报错  
于是, 这快代码看起来应该是这样的    
```Python
#from django.contrib.auth.models import User
from app_user.models import UserModel as User

user = User.objects.create_user(username, password, email)
user.is_active = 0
user.save()
```
![uset-4](https://github.com/KissMyLady/Django/blob/master/Img/uset-4.jpg)  

## 用户重名   
如果我此时注册一个用户同名的信息  
![uset-5](https://github.com/KissMyLady/Django/blob/master/Img/uset-5.jpg)  
所以, 我们需要在验证用户名的时候, 去验证用户名是否重复   
```Python
# 校验用户名
try:
    user = User.objects.get(username=username)
except:
    user = None

if user:
    return render(request, 'register.html', {'errmsg': '用户名存在, 请重新输入'})
else:
    user = User.objects.create_user(username, password, email)
    user.is_active = 0
    user.save()
    return HttpResponse('登录成功')
```
![uset-6](https://github.com/KissMyLady/Django/blob/master/Img/uset-6.jpg)     

   
## 自定义User模型引用文章     
[官方文档](https://yiyibooks.cn/xx/django_182/index.html)        
[Django创建自定义User模型CustomUser--CSDN](https://www.cnblogs.com/ShiveryMoon/p/8279986.html)     

官方文档里给了两种方法      
1.创建一个Model，然后用一对一外键指到User，这样就相当于是扩展了User，简单又实用。        
但是这样没法自定义User的save和delete函数(当然你直接去改django源码也是可以的，但是改源码这种事情实在是不靠谱)    

2.直接新建一个CustomUser并‘覆盖’原来的User类  

注意到django源码里User类的定义里有且只有一个swappable='AUTH_USER_MODEL'。  
我的理解是，这个设置可以指定另一个类，让另一个类来代替这个类。  
  
首先在model.py里创建一个新的类CustomUser，然后去setting里添加如下字段：  
   
#我的app名为blog  
AUTH_USER_MODEL = 'blog.CustomUser'  
这样一来，django把所有对User的操作都转移到了我们自己定义的CustomUser类上。   
  
然后在blog的model.py里的代码如下：      
```Python
from django.db import models
from django.contrib.auth.models import AbstractUser
import os

class CustomUser(AbstractUser):
    location = models.CharField(max_length=100, blank=True, null=True)
    description = models.CharField(max_length=300, blank=True, null=True)
    avatar = models.ImageField(upload_to='avatars/', default='default/default.jpg', max_length=100)

    def __str__(self):
        return self.username

    def delete(self, *args, **kwargs):
        path = self.avatar.path
        if path:
            os.remove(path)
```
扩展了三个字段：地址、个人简介和头像。  
并且能够随意自定义save和delete函数。  
  
注：不要轻易覆盖save函数，因为django对User的save的调用实在是太频繁了，  
就算是登录，也会调用一次save()(django的save和update都使用save())以修改用户的last_login，  
还有添加组或者修改权限的时候。你在save函数里的代码会被多次调用。  

这样一来，我们创建了自己的CustomUser，并且是完全代替了原来的User类(而不是创建了两个用于管理用户的Model)  


## 完整代码  
```Python
from django.shortcuts import render, redirect
from django.http import HttpResponse
from app_user.models import UserModel as User
import re

def register(request):
    return render(request, 'register.html')


def register_handle(request):
    username = request.POST.get('user_name')
    password = request.POST.get('cpwd')
    email = request.POST.get('email')
    allow = request.POST.get('allow')
    if allow != 'on':
        return render(request, 'register.html', {'errmsg':'请同意协议'})
    if not all([username, password, email]):
        return render(request, 'register.html', {'errmsg':'数据不完整'})
        pass
    # if not re.match(r"^[a-z0-9][\w\.\-]*@[a-z0-9\-]+(\.[a-z]{2,5}){1,2}$/", email):
    if email == 123:
        return render(request, 'register.html', {'errmsg':'邮箱校验失败, 请重新输入'})

    try:
        user = User.objects.get(username=username)
    except:
        user = None

    if user:
        return render(request, 'register.html', {'errmsg': '用户名存在, 请重新输入'})
    else:
        user = User.objects.create_user(username, password, email)
        user.is_active = 0
        user.save()
        return HttpResponse('登录成功')
        #return redirect('index.html')

## http://127.0.0.1:8000  跳转到商品首页
def index(request):
    return render(request, 'index.html')
```
## HTML核心代码  
```HTML
{% load static %}
<div class="r_con fr">
<div class="reg_title clearfix">
	<h1>用户注册</h1>
	<a href="#">登录</a>
</div>
<div class="reg_form clearfix">
<form method="post" action="/user/register_handle">
    {% csrf_token %}
<ul>
	<li>
		<label>用户名:</label>
		<input type="text" name="user_name" id="user_name">
		<span class="error_tip">提示信息</span></li>					
	<li>
		<label>密码:</label>
		<input type="password" name="pwd" id="pwd">
		<span class="error_tip">提示信息</span></li>
	<li>
		<label>确认密码:</label>
		<input type="password" name="cpwd" id="cpwd">
		<span class="error_tip">提示信息</span></li>
	<li>
		<label>邮箱:</label>
		<input type="text" name="email" id="email">
		<span class="error_tip">提示信息</span></li>
	<li class="agreement">
		<input type="checkbox" name="allow" id="allow" checked="checked">
		<label>同意”天天生鲜用户使用协议“</label>
		<span class="error_tip2">提示信息</span></li><li class="reg_sub">
		<input type="submit" value="注 册" name="">
	</li>
</ul>				
</form>
    {{ errmsg }}
</div>
```

