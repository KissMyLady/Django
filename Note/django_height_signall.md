注册登录与邮箱异步总结   
=====

流程视图  
====
## 1. 来到首页
我们首先来到首页进行用户信息注册    
![verification-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/verification-1.jpg)  

## 2. 注册信息
我们填写表单, 点击注册, 这时就返回了成功信息   

## 3. 查看信息
我们再来看下数据库用户刚刚注册的信息    
![verification-3](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/verification-3.jpg)  
可以发现, `username`为KissMyLady, password为加密信息, email为表单填入的信息, 说明我们注册成功了  
is_active为0, 我们还没有注册, 现在我们开始邮箱注册  

异步加载外挂Redis启动中的celery  
![verification-4](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/verification-4.jpg)  
邮箱我设置了8s的延迟, 可以看到, 这里的时间为9.6左右  
异步加载邮箱可以让客户端体验上升, 点击某个耗时操作后, 直接后台进行就可以   

## 4. 用户激活  
把注册地址点开, 我们现在进行的是用户激活     
把url输入浏览器, 激活用户  
![verification-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/verification-2.jpg)  

## 5. 数据库激活信息查看    
![verification-5](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/verification-5.jpg)  

## 6. 点击登录首页  
![verification-6](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/verification-6.jpg)  

Greate !



借一步说话  
====

## 1. 首页注册信息细节   
HTML页面是前端做好了的, 我们需要修改一点点细节即可  
* 细节一: 在Django2.2中, 在header导入与1.x版本不同  
看起来应该像是这样: ` {% load static %}`

* 细节二: 标签导入方式: `<link rel="stylesheet" type="text/css" href="{% static 'css/reset.css' %}">`  
`href="{% static 'css/reset.css' %}"`  
  
* 细节三: 不注册, 直接跳转到登录页面书写方式       
```HTML
<div class="reg_title clearfix">
	<h1>用户注册</h1>
	<a href="/user/login">登录</a>
</div>
```
这里的href需要写`/user/login`  

* 细节四: 表单提交需要写`{% csrf_token %}`  
```
<form method="post" action="/user/register">
    {% csrf_token %}
<ul>
	<li>Do something</li>
</ul>				
</form>
```
### Register.html完整代码   
```HTML
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
 {% load static %}
<head>
	<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
	<title>天天生鲜-注册</title>
	<link rel="stylesheet" type="text/css" href="{% static 'css/reset.css' %}">
	<link rel="stylesheet" type="text/css" href="{% static 'css/main.css' %}">
	<script type="text/javascript" src="{% static 'js/jquery-1.12.4.min.js' %}"></script>
	<script type="text/javascript" src="{% static 'js/register.js' %}"></script>
</head>
<body>
	<div class="register_con">
		<div class="l_con fl">
			<a class="reg_logo"><img src="{% static 'images/logo02.png' %}"></a>
			<div class="reg_slogan">足不出户  ·  新鲜每一天</div>
			<div class="reg_banner"></div>
		</div>

		<div class="r_con fr">
			<div class="reg_title clearfix">
				<h1>用户注册</h1>
				<a href="/user/login">登录</a>
			</div>
			<div class="reg_form clearfix">
				<form method="post" action="/user/register">
                    {% csrf_token %}
				<ul>
					<li>
						<label>用户名:</label>
						<input type="text" name="user_name" id="user_name">
						<span class="error_tip">提示信息</span>
					</li>					
					<li>
						<label>密码:</label>
						<input type="password" name="pwd" id="pwd">
						<span class="error_tip">提示信息</span>
					</li>
					<li>
						<label>确认密码:</label>
						<input type="password" name="cpwd" id="cpwd">
						<span class="error_tip">提示信息</span>
					</li>
					<li>
						<label>邮箱:</label>
						<input type="text" name="email" id="email">
						<span class="error_tip">提示信息</span>
					</li>
					<li class="agreement">
						<input type="checkbox" name="allow" id="allow" checked="checked">
						<label>同意”天天生鲜用户使用协议“</label>
						<span class="error_tip2">提示信息</span>
					</li>
					<li class="reg_sub">
						<input type="submit" value="注 册" name="">
					</li>
				</ul>				
				</form>
                {{ errmsg }}
			</div>
		</div>
	</div>

	<div class="footer no-mp">
		<div class="foot_link">
			<a href="#">关于我们</a>
			<span>|</span>
			<a href="#">联系我们</a>
			<span>|</span>
			<a href="#">招聘人才</a>
			<span>|</span>
			<a href="#">友情链接</a>		
		</div>
		<p>CopyRight © 4020 第八宇宙超量子科技集成有限国家 All Rights Reserved</p>
		<p>电话：AAAA0000-888888    三体ICP备5648428号</p>
	</div>
</body>
</html>
```

## 2. 注册信息   
我们在这里, 验证用户的注册信息, 实际上有两种方法, 第一种, 写出视图函数, 根据标签的不同进行跳转  
第二种, 使用Django内置的类视图, 这种更方便  

## 第一种方法
我们先讨论第一种方法, 即函数实现注册信息的验证与识别    
```Python
def register(request):
    if request.method == 'GET':
        return render(request, 'register.html')
    else:
        return register_handle(request)
```
请求register时, 就调用这个函数, 这个函数很好理解, 如果是GET请求, 就直接给用户返回一个`register.html`页面  
是post的请求, 显然, 用户是想要提交信息, 因此, 重新返回一个新的函数:    
```Python
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
    	from app_user.models import UserModel as User
        user = User.objects.get(username=username)
    except:
        user = None
    if user:
        return render(request, 'register.html', {'errmsg': '用户名存在, 请重新输入'})
    else:
        user = User.objects.create_user(username, password, email)
        user.is_active = 0
        user.save()
        from itsdangerous import TimedJSONWebSignatureSerializer as safe
        from django.conf import setting
        safe_obj = safe('{}'.format(setting.SECRET_KEY),  3600)
        info = {'confirm': user.id}
        safe_response = safe_obj.dumps(info)
        return HttpResponse('登录成功')
```
这个函数就是表单数据的验证, request来的数据, 我们使用Django内置POST.get方法可以方便提取表单填写的信息  
像这样:  
```
username = request.POST.get('username')
password = request.POST.get('cpwd')
email =    request.POST.get('email')
allow =    request.POST.get('allow')
```
获取信息后, 逐一判断数据完整性  
这里有几点细节值得我们的注意   
* 细节一: 判断重名问题   
人名可以相同, 人可以理解其中的差异性, 准确识别是谁, 但简单的服务器代码不行  
```Python
from app_user.models import UserModel as User
    user = User.objects.get(username=username)
```
这里, 使用了数据数据判断, 通过OMR操作, 让输入的名字与数据匹配, 看是否有相同的  

* 细节二: 加密问题   
用户数据保存了, 也有输入邮箱绑定, 这时我们需要一个傻瓜式操作, 让用户去验证邮箱的可用性  
这个傻瓜式操作就是: 通过发送一个url, url里面又嵌套url(直接就可以点开, 而不需要复制粘贴), 点击进入即可  

傻瓜式操作也存在隐患, 那就是get请求是明文公开的, 如果有Hacker或者无聊的人想获取数据库数据, 只需要把文本放在后面,一个个请求就可以了, 这属于暴力破解  
显然, 我们当然不希望这样的事情发生  
```Python
from itsdangerous import TimedJSONWebSignatureSerializer as safe
safe_obj = safe('{}'.format('Django_setting_key'),  3600)
info = {'confirm': user.id}
safe_response = safe_obj.dumps(info)
return HttpResponse('登录成功')
```
我们导入了一个`itsdangerous`包, 能够方便的加密与解密	
点击查看[邮箱注册验证时的加密与解密](https://github.com/KissMyLady/Django/blob/master/Note/django_height_email.md)  

itsdangerous.TimedJSONWebSignatureSerializer源码-节选   
```Python
class TimedJSONWebSignatureSerializer(JSONWebSignatureSerializer):
    DEFAULT_EXPIRES_IN = 3600

    def __init__(self, secret_key, expires_in=None, **kwargs):
        JSONWebSignatureSerializer.__init__(self, secret_key, **kwargs)
        if expires_in is None:
            expires_in = self.DEFAULT_EXPIRES_IN
        self.expires_in = expires_in

    def make_header(self, header_fields):
        header = JSONWebSignatureSerializer.make_header(self, header_fields)
        iat = self.now()
        exp = iat + self.expires_in
        header["iat"] = iat
        header["exp"] = exp
        return header

    def loads(self, s, salt=None, return_header=False):
        payload, header = JSONWebSignatureSerializer.loads(
            self, s, salt, return_header=True
        )
        if "exp" not in header:
            raise BadSignature("Missing expiry date", payload=payload)

        int_date_error = BadHeader("Expiry date is not an IntDate", payload=payload)
        try:
            header["exp"] = int(header["exp"])
        except ValueError:
            raise int_date_error
        if header["exp"] < 0:
            raise int_date_error

        if header["exp"] < self.now():
            raise SignatureExpired(
                "Signature expired",
                payload=payload,
                date_signed=self.get_issue_date(header),)

        if return_header:
            return payload, header
        return payload

    def get_issue_date(self, header):
        rv = header.get("iat")
        if isinstance(rv, number_types):
            return datetime.utcfromtimestamp(int(rv))

    def now(self):
        return int(time.time())
```
至此, 函数式邮箱验证就完成了  

## 第二种方法
接下来讲解第二种方法: 类视图   
[类视图](https://github.com/KissMyLady/Django/blob/master/Note/django_height_classviews.md)  
```
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
```
类视图方法也很容易使用, 在内部定义get和post方法, 根须请求的不同自动调用  
* 值得注意的是, url路由的填写需要看一看  
```Python
from app_user.views import RegisterViews  
urlpatterns = [
    re_path(r'^login$',                LoginView.as_view()),
]
```
这里导入app_user下的views文件, 这个文件下的RegisterViews类  
定义的POST方法与函数方法一样, 拿过来用就行   


## 3. 查看信息  
打开数据查看即可  

## 4. 异步加载   
[异步加载外挂Redis启动中的celery](https://github.com/KissMyLady/Django/blob/master/Note/django_height_celery.md)  
在项目下新建一个挂载文件包(加入`__init__.py`文件)  
```Python
from celery import Celery
app_email = Celery('celery_tasks.tasks', broker='redis://192.168.0.101:6379/8')

import os, django
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'daily_fresh.settings')
django.setup()

@app_email.task
def SendRegisterActiveEmail(to_email, username, token):
    subject = '第八宇宙欢迎你'
    msg = '<h1>亲爱的%s 欢迎您成为第八宇宙会员</h1>请点击以下链接激活您的账户<br><a href="http://192.168.0.104:8000/user/active/%s">http://192.168.0.104:8000/user/active/%s</a>' % (
    username, token, token)
    # send_people = setting.EMAIL_FROM
    send_people = '第八宇宙开拓者<Kiss_mylady@qq.com>'
    receiver = [to_email]

    none_msg = ''
    html_msg = msg
    from django.core.mail import send_mail
    send_mail(subject, none_msg, send_people, [to_email], html_message=html_msg)
```
* 细节一: 挂载需要配置setting信息
```Python
import os, django
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'daily_fresh.settings')
django.setup()
``` 
* 细节二: `@app_email.task`需要Celery中task装饰器装饰
[Celery知识引导-Tasks](https://blog.csdn.net/happyAnger6/article/details/51533321)    
[如何使用@shared_task装饰器进行基于类的任务](https://www.thinbug.com/q/21233089)    
 
* 细节三: 邮件内容看起来应该尽可能的简洁, 舒适与富有美感, 那么html格式是必不可少的  
所以这需要我们重新考虑下该怎么办, 因为默认发送发送的是字符串  
```Python
from django.core.mail import send_mail
send_mail(subject, none_msg, send_people, [to_email], html_message=html_msg)
```
Django内置了发送email的模块, 拿来用   
第一个参数是标题, 后面依次是: str内容, 发送人, 收件人列表, html格式开启  
第五个参数为我们提供了html格式的邮件文本, 很方便是不是   

* 细节四: celery我使用的是Redis, 如果对Redis有什么问题, 可看查看[Redis分布式集群搭建](https://github.com/KissMyLady/Tools/blob/master/note/redis_goup.md)  
这篇文章详细介绍了Redis安装使用以及分布式Redis的搭建  


## 5. 邮箱激活    
用户注册后 需要进一步邮箱激活, 进一步激活, 需要考虑的是get里面传过来的密码提取解密问题  
```Python  
class ActivaEmailView(View):
    def get(self, request, token):
        from itsdangerous import TimedJSONWebSignatureSerializer as safe
        from itsdangerous import SignatureExpired
        SECRET_KEY = '-7bt+*1v5%9wm%__th-qw=zze7)*89w@ahpwl5$1!t#sc+z6th'
        safe_obj = safe('{}'.format(SECRET_KEY), 3600)
        try:
            print('开始激活邮箱')
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

```
* 细节一: 解密   
```Python
from itsdangerous import TimedJSONWebSignatureSerializer as safe
safe_obj = safe('{}'.format(DJANGO_SETTINGS_MODULE), 3600)
info =     safe_obj.loads(token)
user_id =  info['confirm']
```
点击查看[邮箱注册验证时的加密与解密](https://github.com/KissMyLady/Django/blob/master/Note/django_height_email.md)  
通过解密, 我们获取了公开传输get的用户信息, 然后查数据库, 把`is_active`改为激活状态  
至此, 用户完成激活, 马上返回用户登录界面    

* 细节二: 错误  
解密也有失败问题, 我们肯定不希望过了7天后, 用户仍然能够激活, 存在风险  
限制解密时间后, 超时肯定报一个错, 这时候就返回信息, 说连接失效了  



## 6. 登录首页  
同样, 我们封装一个类, 把get和post封装在一起, 请求直接判断  
请求login(登录)页面时, get请求就返回login页面, post请求说明要提交信息, 我们验证这个信息, 如果都正确那就返回登录成功的页面  
```Python
class LoginView(View):
    def get(self, request):
        return render(request, 'login.html')

    def post(self, request):
        username = request.POST.get('username')
        password = request.POST.get('pwd')
        print('username:', username, 'password:', password)
        if not all([username, password]):
            return render(request, 'login.html', {'errmsg': '数据不完整'})
        from django.contrib.auth import authenticate
        user = authenticate(username=username, password=password)
        if user is not None:
            if user.is_active:
                from django.contrib.auth import login
                login(request, user)
                print('返回首页')
                return redirect('/user/index')
            else:
                print('The password is valid, but the account has been disablled')
                return render(request, 'login.html', {'errmsg': '账户未激活您的账户'})
        else:
            return render(request, 'login.html', {'errmsg': '用户名或密码错误'})
```
* 细节一: `return redirect('/user/index')`   
这里url容易写漏, 多加注意Django中url的转发   

* 细节二: all()方法的使用   
`if not all([username, password]):`[菜鸟教程](https://www.runoob.com/python/python-func-all.html)    
```Python
>>> all(['a', 'b', 'c', 'd'])  # 列表list，元素都不为空或0
True
>>> all(['a', 'b', '', 'd'])   # 列表list，存在一个为空的元素
False
>>> all([0, 1，2, 3])          # 列表list，存在一个为0的元素
False
   
>>> all(('a', 'b', 'c', 'd'))  # 元组tuple，元素都不为空或0
True
>>> all(('a', 'b', '', 'd'))   # 元组tuple，存在一个为空的元素
False
>>> all((0, 1, 2, 3))          # 元组tuple，存在一个为0的元素
False
   
>>> all([])             # 空列表
True
>>> all(())             # 空元组
True
```
