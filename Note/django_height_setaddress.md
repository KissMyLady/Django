用户中心_地址页
===========

现在我们开始设计用户地址表, 设置默认地址收获地址   

## 之前的登录注册问题    
我们先看一看以前设计登录页面的POST请求       
```html
<form method="post">
    {% csrf_token %}
	<input type="text" name="username" class="name_input" value="{{ username }}" placeholder="请输入用户名">
	<div class="user_error">输入有误</div>
	<input type="password" name="pwd" class="pass_input" placeholder="请输入密码">
	<div class="pwd_error">输入有误</div>
	<div class="more_input clearfix">
		<input type="checkbox" name="remember" {{ checked }}>
		<label>记住用户名</label>
		<a href="#">忘记密码</a>
	</div>
	<input type="submit" name="" value="登录" class="input_submit">
</form>
```
以及对应的视图      
```Python
username = request.POST.get('username')
password = request.POST.get('pwd')

if not all([username, password]):
    return render(request, 'login.html', {'errmsg': '数据不完整'})

from django.contrib.auth import authenticate
user = authenticate(username=username, password=password)
if user is not None:
    if user.is_active:
        from django.contrib.auth import login
        login(request, user)
        next_url = request.GET.get('next', '/user/index')
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
```
仔细一看, 表单提交更改数据库的方式都大同小异, 可以有以下几个通用步骤:      
### 1. 获取POST提交过来的数据    
```Python
username = request.POST.get('username')  
password = request.POST.get('pwd')  
```
### 2. 判断数据    
```Python
from django.contrib.auth import authenticate
user = authenticate(username=username, password=password)
if user is not None:
    if user.is_active:
        from django.contrib.auth import login
        login(request, user)
        next_url = request.GET.get('next', '/user/index')
        response = redirect(next_url)
        remember = request.POST.get('remember')
        if remember == 'on':
            response.set_cookie('username', username, max_age=1*1*3600)
        else:
            response.delete_cookie('username')
```
### 3. 提交请求    
```Python
....
return response
```

## 类推收获地址的提交     
HTML页面表单POST提交    
```html
<form method="post">
	{% csrf_token %}
	<div class="form_group">
		<label>收件人：</label>
		<input type="text" name="receiver">
	</div>
	<div class="form_group form_group2">
		<label>详细地址：</label>
		<textarea class="site_area" name="addr"></textarea>
	</div>
	<div class="form_group">
		<label>邮编：</label>
		<input type="text" name="zip_code">
	</div>
	<div class="form_group">
		<label>手机：</label>
		<input type="text" name="phone">
	</div>

	<input type="submit" value="提交" class="info_submit">
</form>
```
### 1. 获取POST提交过来的数据    
```Python
receiver = request.POST.get('receiver')
addr =     request.POST.get('addr')
zip_code = request.POST.get('zip_code')
phone =    request.POST.get('phone')
```

### 2. 判断   
```Python
if not all([receiver, addr, phone]):
    return render(request, 'user_center_site.html', {'ermsg': '数据不完整'})

if not re.match(r'^1[3|4|5|7|8][0-9]{9}$', phone):
    return render(request, 'user_center_site.html', {'ermsg': '手机号码格式不完整'})

user = request.user
from app_user.models import Address
try:
    address = Address.objects.get(user=user, is_default=True)
except:
    address = None

if address:
    is_default = False
else:
    is_default = True

# add
Address.objects.create(user=user,
                       addr=addr,
                       receiver=receiver,
                       zip_code=zip_code,
                       phone=phone,
                       is_default=is_default)
```
### 3. 提交  
```Python
return redirect('address')
```
![setaddr-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/setaddr-1.jpg)  
数据库查表:    
![setaddr-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/setaddr-2.jpg)  


