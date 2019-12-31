登录页面小小的改动  
=====

问 题:   
当我们登录进主页时, 网页的左上角还是有`登录`,`注册`等字样, 作为一个看得过去的网页, 应该把它去掉, 换上我们的欢迎信息   

## 解决方法    
判断用户有没有登录     
[判断用户登录原始方法](https://yiyibooks.cn/xx/django_182/topics/auth/default.html)       

> 限制页面访问的简单、原始的方法是检查`request.user.is_authenticated()`并重定向到一个登陆页面       

`request.user.is_authenticated()`Django的request下的user方法, 有一个`is_authenticated()`判断, 如果返回的是True, 登录, 防之为False  


#### [Web请求中的认证](https://yiyibooks.cn/xx/django_182/topics/auth/default.html)        
Django使用会话和中间件来拦截request 对象到认证系统中。    
* 它们在每个请求上提供一个request.user属性，表示当前的用户。如果当前的用户没有登入，该属性将设置成AnonymousUser的一个实例，否则它将是User的实例。    
  
你可以通过is_authenticated()区分它们，像这样：   
```Python  
if request.user.is_authenticated():
    # Do something for authenticated users.
    ...
else:
    # Do something for anonymous users.
    ...
```
在模板中, 我们可以直接使用这个方法判断用户是否登录    
```Python
request.user.is_authenticated()
```
* 如果用户登录了, request就是User类的实例    
* 如果未登录, request就是anonymous类的实例   
```Python   
request.user.is_authenticated()  
request.anonymous.is_authenticated()  
```
这两种都有`is_authenticated()`方法    

如果要判断用户是否登录, 我就这样判断:  
```html
{% if user.is_authenticated %}
	<div class="login_btn fl">
	欢迎您：<em>{{ user.username }}</em>
	<span>|</span>
	<a href="#">退出</a>
	</div>
{% else %}
	<div class="login_btn fl">
	<a href="/login">登录</a>
	<span>|</span>
	<a href="/register">注册</a>
	</div>
{% endif %}
	<div class="user_link fl">
	<span>|</span>
	<a href="/user">用户中心</a>
	<span>|</span>
	<a href="cart.html">我的购物车</a>
	<span>|</span>
	<a href="/order">我的订单</a>
```
效果图:   
![userlogin-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/userlogin-1.jpg)  


