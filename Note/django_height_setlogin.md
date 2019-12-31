登录装饰器配合使用  
====

问题: 通常我们有时候会使用百度, 有些常见生活问题可以在贴吧里找到, 但是现在百度加强了验证  
需要下载它自家的app或者是登录它的账号,  我们没有登录就不能进去它的页面查看信息  
显然, 我们自己做的网站也想要拥有这个功能, Django可不可以做到呢?  当然  


## 访问核心网站需要登录  
先看下我们下我们常规的url转发  
```Python
urlpatterns = [
    re_path(r'^register$',              RegisterViews.as_view()),
    re_path(r'^active/(?P<token>.*)$',  ActivaEmailView.as_view()),
    re_path(r'^login$',                 LoginView.as_view()),
    re_path(r'^index$',                 views.index),
    re_path(r'^$',                  UserInfoViews.as_view()),
    re_path(r'^order',              UserOrderViews.as_view()),
    re_path(r'^address',            UserAddressViews.as_view()),]
```
这里访问`user`, `order`, `address`等数据敏感页面时, 必须用 户登录了才能看到  
怎么做到呢?  

Django体面提供了自带的验证装饰器, 我们拿过来用就可以了     
[Django官方文档1.8详细介绍](https://yiyibooks.cn/xx/django_182/topics/auth/index.html)    
尽管这里运行的是2.2版本Django, 但很多地方还是相似的, 而且这个中文翻译的官方文档很全, 所以引用了它   

节 选:  
* login_required装饰器     
login_required([redirect_field_name=REDIRECT_FIELD_NAME, login_url=None])[source]   
作为一个快捷方式，你可以使用便捷的login_required()装饰器：  
```Python
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...
```
* login_required()完成下面的事情：    
如果用户没有登入，则重定向到settings.LOGIN_URL，并将当前访问的绝对路径传递到查询字符串中。例如：/accounts/login/?next=/polls/3/。  
如果用户已经登入，则正常执行视图。视图的代码可以安全地假设用户已经登入。  
默认情况下，在成功认证后用户应该被重定向的路径存储在查询字符串的一个叫做"next"的参数中。如果对该参数你倾向使用一个不同的名字，login_required()带有一个可选的redirect_field_name参数：   
```Python
from django.contrib.auth.decorators import login_required

@login_required(redirect_field_name='my_redirect_field')
def my_view(request):
    ...
```
同过上述知识描述, 我们可以发现, 这个装饰器是针对函数实现的  
现在看一下我们的代码:  
```Python
class UserInfoViews(View):
   ....
    return render(request, 'user_center_info.html', {'page': 'user'})

class UserOrderViews(View):
    ....
    return render(request, 'user_center_order.html', {'page': 'order'})

class UserAddressViews(View):
    ....
    return render(request, 'user_center_site.html', {'page': 'address'})
```
问题来了:  
我们这里的转发对象都是class, Django自带的验证装饰器验证的是函数, 该怎么办?   

我们需要想个方法, 一定让这个class执行前, 判断用户是否登录

使用方法:  
```Python
from django.contrib.auth.decorators import login_required

urlpatterns = [
    re_path(r'^register$',              RegisterViews.as_view()),
    re_path(r'^active/(?P<token>.*)$',  ActivaEmailView.as_view()),
    re_path(r'^login$',                 LoginView.as_view()),
    re_path(r'^index$',                 views.index),
    re_path(r'^$',         login_required(UserInfoViews.as_view())),
    re_path(r'^order',     login_required(UserOrderViews.as_view())),
    re_path(r'^address',   login_required(UserAddressViews.as_view())),]
```
* 细节一:  在转发url时候, 一定是调用的`UserInfoViews.as_view()`, 于是我们在它调用的时候, 先验证用户是否登录, 这个问题就解决了  
```Python
re_path(r'^$', UserInfoViews.as_view()),
```
升级后应该是这样:     
```Python
re_path(r'^$', login_required(UserInfoViews.as_view())),
```

我们现在来测试下网页:    
![slogin-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/slogin-1.jpg)  
这样我们确实可以判断用户有没有登录, 但是跳转地址不是我们想要的    
我们想让它跳转到我们网站的登录页面  
我们再来看下文档对于login_required函数的知识描述:      
> login_required()完成下面的事情：        
> 如果用户没有登入，则重定向到settings.LOGIN_URL，并将当前访问的绝对路径传递到查询字符串中。例如：/accounts/login/?next=/polls/3/。    
> 如果用户已经登入，则正常执行视图。视图的代码可以安全地假设用户已经登入。  
> 默认情况下，在成功认证后用户应该被重定向的路径存储在查询字符串的一个叫做"next"的参数中。
> 如果对该参数你倾向使用一个不同的名字，login_required()带有一个可选的redirect_field_name参数：     
这里可以发现, `login_required()`默认跳转`/accounts/login/?next=xxxx`  

* 细节二:    
为了让它跳转到我们项目的地址, 我就需要setting设置   
文档说明:  
> ettings.LOGIN_URL同时还接收视图函数名和命名的URL模式。 这允许你自由地重新映射你的URLconf中的登录视图而不用更新设置。   
```Python
LOGIN_URL = '/user/login'
```
![slogin-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/slogin-2.jpg)    

这时候, 我们在html页面中填写用户登录信息, 点击登录  
问题来了, 请求发的谁?  
```html
<form method="post">
	{% csrf_token %}
	<input type="text" name="username" class="name_input" value="{{ username }}" placeholder="请输入用户名">
	<div class="user_error">输入错误</div>
	<input type="password" name="pwd" class="pass_input" placeholder="请输入密码">
	<div class="pwd_error">输入错误</div>
	<div class="more_input clearfix">
		<input type="checkbox" name="remember" {{ checked }}>
		<label>记住用户名</label>
		<a href="#">忘记密码</a>
	</div>
	<input type="submit" name="" value="登录" class="input_submit">
</form>
```
请注意, 贼这里的`<form method="post">`action我们没有写   
所以, 这里提交表单时, 它就会像浏览器所在的地址发起请求  
所以, 请求访问的谁这个问题就明白了, 实际上发送的是浏览器现在所在的地址  
`/user/login?next=/user/`  
细节三:  
这里是get请求, 通过requeest.GET.get('next')获取参数'/user/'  
细节四:   
直接访问login页面时, url地址很干净, 没有后面的`next=/user/`, 显然代码会报错  
`next_url = request.GET.get('next', 'index')`后面的`index`就表示默认的地址, 拿不到`next`, 就跳转后面的值  
于是她们看起来应该是这样:  
```Python
if user.is_active:
	from django.contrib.auth import login
	login(request, user)
	next_url = request.GET.get('next', 'index')
	response = redirect(next_url)
	remember = request.POST.get('remember')

	if remember == 'on':
	    response.set_cookie('username', username, max_age=1*1*3600)
	else:
	    response.delete_cookie('username')
	return response
```

## `Urls.py`全部代码   
```Python
from django.contrib import admin
from django.urls import path, re_path, include
from app_user import views
from app_user.views import RegisterViews, ActivaEmailView, LoginView
from app_user.views import UserInfoViews, UserOrderViews, UserAddressViews
from django.contrib.auth.decorators import login_required

urlpatterns = [
    re_path(r'^register$',              RegisterViews.as_view()),
    re_path(r'^active/(?P<token>.*)$',  ActivaEmailView.as_view()),
    re_path(r'^login$',                 LoginView.as_view()),
    re_path(r'^index$',                 views.index),
    re_path(r'^$',          login_required(UserInfoViews.as_view())),
    re_path(r'^order',      login_required(UserOrderViews.as_view())),
    re_path(r'^address',    login_required(UserAddressViews.as_view())),
]

```
## `Views.py`全部代码   
```Python
from django.shortcuts import render, redirect
from django.http import HttpResponse
# from django.contrib.auth.models import User
from app_user.models import UserModel as User
from django.views import View

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
            user = User.objects.create_user(username, email, password)
            user.is_active = 0
            user.save()

            from itsdangerous import TimedJSONWebSignatureSerializer as safe
            SECRET_KEY = '-7bt+*1v5%9wm%__th-qw=zze7)*89w@ahpwl5$1!t#sc+z6th'
            safe_obj = safe('{}'.format(SECRET_KEY), 3600)
            info = {'confirm': user.id}

            token = safe_obj.dumps(info)
            token = token.decode('utf-8')

            from celery_packages.tasks import SendRegisterActiveEmail
            SendRegisterActiveEmail.delay(email, username, token)
            return HttpResponse('登录成功')


class ActivaEmailView(View):
    def get(self, request, token):
        from itsdangerous import TimedJSONWebSignatureSerializer as safe
        from itsdangerous import SignatureExpired
        SECRET_KEY = '-7bt+*1v5%9wm%__th-qw=zze7)*89w@ahpwl5$1!t#sc+z6th'
        safe_obj = safe('{}'.format(SECRET_KEY), 3600)
        try:
            info = safe_obj.loads(token)
            user_id = info['confirm']
            user = User.objects.get(id=user_id)
            user.is_active = 1
            user.save()
            print('邮箱已经激活')
            return redirect('user/index/')

        except SignatureExpired as e:
            from django.http import HttpResponse
            return HttpResponse('激活链接已失效')


# /user/login
class LoginView(View):
    def get(self, request):
        if 'username' in request.COOKIES:
            username = request.COOKIES.get('username')
            checked = 'checked'
        else:
            username = ''
            checked  = ''
        return render(request, 'login.html', {'username': username, 'checked': checked})

    def post(self, request):
        username = request.POST.get('username')
        password = request.POST.get('pwd')
        print('username:', username, 'password:', password)
        if not all([username, password]):
            return render(request, 'login.html', {'errmsg': '数据不完整'})
        #user = User.objects.get(username= username, password=password)
        from django.contrib.auth import authenticate
        user = authenticate(username=username, password=password)
        if user is not None:
            if user.is_active:
                print('User is valid, active and authenticated')
                from django.contrib.auth import login
                login(request, user)
                next_url = request.GET.get('next', 'index')
                response = redirect(next_url)
                remember = request.POST.get('remember')

                if remember == 'on':
                    response.set_cookie('username', username, max_age=1*1*3600)
                else:
                    response.delete_cookie('username')
                return response

            else:
                return render(request, 'login.html', {'errmsg': '账户未激活您的账户'})
        else:
            return render(request, 'login.html', {'errmsg': '用户名或密码错误'})

# /user
class UserInfoViews(View):
    def get(self, request):
        return render(request, 'user_center_info.html', {'page': 'user'})


# /user/order
class UserOrderViews(View):
    def get(self, request):
        return render(request, 'user_center_order.html', {'page': 'order'})


# /user/site
class UserAddressViews(View):
    def get(self, request):
        return render(request, 'user_center_site.html', {'page': 'address'})

```
