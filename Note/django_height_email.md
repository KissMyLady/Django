邮箱注册验证  
=====
## 预览图  
![esate-6](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/esafe-6.jpg)  
我们现在采用的数据传输方式: get请求   
那么问题来了, get请求是明文传输, 这不安全    

## 安装itsdangerous包  
显然, 我们需要加密get请求:   
```Linux
pip install itsdangerous
```

## 使用itsdangerous    
进入虚拟环境  
`python manager.py shell` 
   
### 加密方法  
Serializer的作用还有设计激活时间, 如果超时, 就不给激活了       
[点击进入itsdangerous官方文档](https://pythonhosted.org/itsdangerous/)   
> Sometimes you just want to send some data to untrusted environments.     
> But how to do this safely? The trick involves signing.      
> Given a key only you know, you can cryptographically sign your data and hand it over to someone else.   
> When you get the data back you can easily ensure that nobody tampered with it.    
>   
> Granted, the receiver can decode the contents and look into the package, but they can not modify the contents unless they also have your secret key.   
> So if you keep the key secret and complex, you will be fine.  
>    
> Internally itsdangerous uses HMAC and SHA1 for signing by default and bases the implementation on the Django signing module.   
> It also however supports JSON Web Signatures (JWS).  
> The library is BSD licensed and written by Armin Ronacher though most of the copyright for the design and implementation goes to Simon Willison and the other amazing Django people that made this library possible.  

设置	`This is my secretkey`为加密钥匙, 360为过期时间
```Python
from itsdangerous import  TimedJSONWebSignatureSerializer as safe

safe_obj = safe('This is my secretkey', 360)
info = {'confirm': 1}

# 返回加密后的信息
response = safe_obj.dumps(info)
```
![esafe-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/esafe-1.jpg)  
### 解密方法  
```Python
# 把加密后的数据传过来  
safe_obj.loads(response)
```
![esafe-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/esafe-2.jpg)  


## 在Django中应用  
```Python
from itsdangerous import  TimedJSONWebSignatureSerializer as safe
from django.conf import setting
# SECRET_KEY, 在顶端, 有Django自动生成的密码
safe_obj = safe('{}'.format(setting.SECRET_KEY),  3600)
info = {'confirm': user.id}
safe_response = safe_obj.dumps(info)
```


## 发送邮件的配置     
setting设置  
```Python
# 发送邮件配置
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
# smpt服务地址
EMAIL_HOST = 'smtp.qq.com'
EMAIL_PORT = 25
# 发送邮件的邮箱
EMAIL_HOST_USER = 'your_email_address@qq.com'
# 在邮箱中设置的客户端授权密码
EMAIL_HOST_PASSWORD = 'your_cloud_password'
# 收件人看到的发件人
EMAIL_FROM = '天天生鲜<your_email_address@qq.com>'
```


## 在Views函数设置发送功能   
[邮箱配置官方文档](https://docs.djangoproject.com/en/3.0/topics/email/)   
```Python
from django.core.mail import send_mail

send_mail(
    'Subject here',
    'Here is the message.',
    'from@example.com',
    ['to@example.com'],
    fail_silently=False,
)
```

setting基本配置   
说明: 这里的setting配置, 要在views文件要导入, 也可以直接在views里面添加此项配置  

views配置  
```Python
 else:
    user = User.objects.create_user(username, password, email)
    user.is_active = 0
    user.save()
    from itsdangerous import TimedJSONWebSignatureSerializer as safe
    # from daily_fresh import setting
    SECRET_KEY = '-7bt+*1v5%9wm%__th-qw=zze7)*89w@ahpwl5$1!t#sc+z6th'
    safe_obj = safe('{}'.format(SECRET_KEY), 3600)
    info = {'confirm': user.id}
    token = safe_obj.dumps(info)

    # Send email
    from django.core.mail import send_mail
    subject = '第八宇宙欢迎你'
    msg = '<h1>欢迎您成为第八宇宙会员</h1>请点击以下链接激活您的用户, 请在1小时内验证完成'
    # send_people = setting.EMAIL_FROM
    send_people = '第八宇宙开拓者<Kiss_mylady@qq.com>'
    send_mail(subject, msg, send_people, [email])
    return HttpResponse('登录成功')
```
通过上面配置, 就可以发送邮箱了, 我们来看一看发的什么   
![esafe-3](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/esafe-3.jpg)  
恭喜恭喜, 咱们此时就完成了一次邮箱的发送, 还是很简单的吧     

## 升级发送内容  
上面的内容有点low, 我们如何升级发送的消息?  
例如像那些骚然邮件, 格式总是那么优雅, 那些广告邮件, 总是那么充满艺术的气息   

我们先来简单的写一封升级邮件吧   
```Python
msg = '<h1>亲爱的%s 欢迎您成为第八宇宙会员</h1>请点击以下链接激活您的账户
<br><a href="href=http://127.0.0.1:8000/user/active/%s">http://127.0.0.1:8000/user/active/%s</a>' % (username ,token, token)
```
这里有个坑, 内置的msg发送html格式的内容只会以str的方式显示在邮件中  
好的是, Django在这个内置函数里面附带了html文本发送, 开启后就会以html显示  
```Python
from itsdangerous import TimedJSONWebSignatureSerializer as safe
SECRET_KEY = '-7bt+*1v5%9wm%__th-qw=zze7)*89w@ahpwl5$1!t#sc+z6th'
safe_obj = safe('{}'.format(SECRET_KEY), 3600)
info = {'confirm': user.id}
token = safe_obj.dumps(info)
token = token.decode('utf-8') 

# send email
from django.core.mail import send_mail
subject = '第八宇宙欢迎你'
msg = '<h1>亲爱的%s 欢迎您成为第八宇宙会员</h1>
请点击以下链接激活您的账户<br>
<a href="http://127.0.0.1:8000/user/active/%s">
http://127.0.0.1:8000/user/active/%s</a>' % (username ,token, token)

# send_people = setting.EMAIL_FROM
send_people = '第八宇宙开拓者<Kiss_mylady@qq.com>'
receiver = [email]

none_msg = ''
html_msg = msg
send_mail(subject, none_msg, send_people, [email], html_message=html_msg)
return HttpResponse('登录成功')
```
可以看到, 在邮件发送里面有一个参数`html_message`, 我们把html格式文本放这里就能以html形式显示了  
另外, 第二个参数msg给空字符串就可以   

发送结果:  
![esafe-4](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/esafe-4.jpg)   



## 全部代码  
### Views.py  
```Python
from django.shortcuts import render, redirect
from django.http import HttpResponse
# Create your views here.
# from django.contrib.auth.models import User
from app_user.models import UserModel as User
#from django.views.generic import View
from django.views import View

def register(request):
    if request.method == 'GET':
        return render(request, 'register.html')
    else:
        return register_handle(request)



def register_handle(request):
    username = request.POST.get('user_name')
    password = request.POST.get('cpwd')
    email =    request.POST.get('email')
    allow =    request.POST.get('allow')
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
        # http://127.0.0.1:8000/user/active/3
        from itsdangerous import  TimedJSONWebSignatureSerializer as safe
        from django.conf import setting
        safe_obj = safe('{}'.format(setting.SECRET_KEY),  3600)
        info = {'confirm': user.id}
        safe_response = safe_obj.dumps(info)
        return HttpResponse('登录成功')


# http://127.0.0.1:8000  跳转到商品首页
def index(request):
    return render(request, 'index.html')


# class views
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
        if email != '1581656458@qq.com':
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

            from itsdangerous import TimedJSONWebSignatureSerializer as safe
            # from daily_fresh.daily_fresh import setting
            SECRET_KEY = '-7bt+*1v5%9wm%__th-qw=zze7)*89w@ahpwl5$1!t#sc+z6th'
            safe_obj = safe('{}'.format(SECRET_KEY), 3600)
            info = {'confirm': user.id}
            token = safe_obj.dumps(info)
            token = token.decode('utf-8')
            # send email
            from django.core.mail import send_mail
            subject = '第八宇宙欢迎你'
            msg = '<h1>亲爱的%s 欢迎您成为第八宇宙会员</h1>请点击以下链接激活您的账户<br><a href="http://127.0.0.1:8000/user/active/%s">http://127.0.0.1:8000/user/active/%s</a>' % (username ,token, token)
            # send_people = setting.EMAIL_FROM
            send_people = '第八宇宙开拓者<Kiss_mylady@qq.com>'
            receiver = [email]

            none_msg = ''
            html_msg = msg
            send_mail(subject, none_msg, send_people, [email], html_message=html_msg)
            return HttpResponse('登录成功')


class ActivaEmailView(View):
    def get(self, request, token):
        from itsdangerous import TimedJSONWebSignatureSerializer as safe
        from itsdangerous import SignatureExpired
        from django.conf import setting
        safe_obj = safe('{}'.format(setting.SECRET_KEY), 3600)
        try:
            info = safe_obj.loads(token)
            user_id = info['confirm']
            user = User.objects.get(id= user_id)
            user.is_active = 1
            user.save()
            return redirect('login')

        except SignatureExpired as e:
            from django.http import HttpResponse
            return HttpResponse('激活链接已失效')

# /user/login
class LoginView(View):
    def get(self, request):
        return render(request, 'login.html')
```


### `Ulrs.py`代码  
```Python
from django.contrib import admin
from django.urls import path, re_path, include
from app_user import views
from app_user.views import RegisterViews, ActivaEmailView, LoginView

urlpatterns = [
    # re_path('^register$',          views.register),
    # re_path('^register_handle$',   views.register_handle),
    re_path(r'^register$',             RegisterViews.as_view()),
    re_path(r'^ative/(?P<token>.*)$',  ActivaEmailView.as_view()),
    re_path(r'^login$',                LoginView.as_view()),
]
# path('', views.index),
```

### 总urls路由配置代码  
```Python
urlpatterns = [
    path('admin/', admin.site.urls),
    re_path(r'^tinymce/', include('tinymce.urls'  )), 
    re_path('^user/',     include('app_user.urls' )),
    re_path('^cart/',     include('app_cart.urls' )),
    re_path('^order/',    include('app_order.urls')),
    re_path('^',          include('app_goods.urls')),
]

```

### 商品登录页url转发代码  
```Python
def index(request):
    return render(request, 'index.html')
```

