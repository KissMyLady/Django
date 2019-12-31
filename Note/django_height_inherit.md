页面模板继承  
=====


问题: 在一系列网站中, 网站内容多多少少有很大相似的地方, 比如说标题头的用户信息与底部的版权信息之类   
平常有留意那些博客, 或者是商家网页的广告和推荐模块, 你就能明白了     
它们是如何工作的?   
我们可以建立一个父模板, 在上面挖出几个坑位, 当子模板继承时, 就填充这个坑(也可不填充)  

字模板就可以在替换部位写点实质内容       
当然, 你想到了, 子模板继承的地方可以用来放点广告    

可以比喻成PS画图中的模板叠加, 父模板处于最底层,  子模在上面覆盖覆父模板, 需要重写的地方在父模板上预留地方即可  
![inherit-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/inherit-1.jpg)   

## 继承使用的方法知识描述  
```
#### 父模板
{%block 名称%}  
  预留区域，可以编写默认内容，也可以没有默认内容
{%endblock  名称%}

#### 子模板
标签extends：继承，写在子模板文件的第一行。

{% extends "父模板路径"%}
子模版不用填充父模版中的所有预留区域，如果子模版没有填充，则使用父模版定义的默认值。

填充父模板中指定名称的预留区域  

{%block 名称%}
实际填充内容
{{block.super}}用于获取父模板中block的内容
{%endblock 名称%}
```

## 继承使用方法实例  
我们来看一个父模板网页:     
```html
<html>
<head>
    <title>{{title}}</title>
</head>
<body>
	<h2>这是头</h2>
<hr>
	{%block qu1%}
		这是区域一，有默认值
	{%endblock qu1%}
<hr>
	{%block qu2%}
	{%endblock qu2%}
<hr>
	<h2>这是尾</h2>
</body>
</html>
```
我们设置了两段替换, 其中一处设为了默认值  
下面一处正常填写预留块   

标题和底部栏就是我们想要的, 我们需要继承来减少重复工作  

再来看看子模板网页:  
```html
{%extends 'booktest/inherit_base.html'%}

{%block qu2%}
<ul>
    {%for book in list%}
    	<li>{{book.btitle}}</li>
    {%endfor%}
</ul>
{%endblock qu2%}
```
* 细节一:     
在顶部引用父模板    
```html
{% extends 'booktest/inherit_base.html' %}
```
* 细节二:     
在替换部分写规则  
 ```html
{%block qu2%}
<ul>
	<li>Do something</li>
</ul>
{%endblock qu2%}
```
看起来应该是这样:  
![inherit-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/inherit-2.jpg)  


如果仅仅这样替换也是没有什么用的, 我们需要实例来学习大规模网页是如何替换的  
网页替换实例:     
## base.html
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
{% load static %}
<head>
	<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
	<title>{% block title %}{% endblock title %}</title>
	<link rel="stylesheet" type="text/css" href="{% static 'css/reset.css' %}">
	<link rel="stylesheet" type="text/css" href="{% static 'css/main.css' %}">
    {% block topfiles %}{% endblock topfiles %}

</head>
<body>
{% block header_con %}
	<div class="header_con">
		<div class="header">
			<div class="welcome fl">欢迎来到天天生鲜!</div>
<div class="fr">
    {% if user.is_authenticated %}
		<div class="login_btn fl">
			欢迎您：<em>{{ user.username }}</em>
			<span>|</span>
			<a href="{% url 'user:logout' %}">退出</a>
		</div>
    {% else %}
		<div class="login_btn fl">
			<a href="{% url 'user:login' %}">登录</a>
			<span>|</span>
			<a href="{% url 'user:register' %}">注册</a>
		</div>
    {% endif %}
		<div class="user_link fl">
			<span>|</span>
			<a href="{% url 'user:user' %}">用户中心</a>
			<span>|</span>
			<a href="cart.html">我的购物车</a>
			<span>|</span>
			<a href="{% url 'user:order' %}">我的订单</a>
		</div>
			</div>
		</div>		
	</div>
{% endblock header_con %}


{% block search_bar %}
	<div class="search_bar clearfix">
		<a href="index.html" class="logo fl"><img src="images/logo.png"></a>
		<div class="search_con fl">
			<input type="text"   class="input_text fl" name="" placeholder="搜索商品">
			<input type="button" class="input_btn fr"  name="" value="搜索">
		</div>
		<div class="guest_cart fr">
			<a href="#" class="cart_name fl">我的购物车</a>
			<div class="goods_count fl" id="show_count">1</div>
		</div>
	</div>
{% endblock search_bar %}

{# 网站主体内容块 #}
{% block body %}{% endblock body %}

	<div class="footer">
		<div class="foot_link">
			<a href="#">关于我们</a>
			<span>|</span>
			<a href="#">联系我们</a>
			<span>|</span>
			<a href="#">招聘人才</a>
			<span>|</span>
			<a href="#">友情链接</a>		
		</div>
		<p>CopyRight © 2050 天天生鲜信息技术有限公司 All Rights Reserved</p>
		<p>电话：888-****810    东ICP备*******8号</p>
	</div>
    {# 网页底部html元素块 #}
    {% block bottom %}{% endblock bottom %}

    {# 网页底部引入文件块 #}
	{% block bottomfiles %}{% endblock bottomfiles %}
</body>
</html>
```

## base_detail_list.html  
```html
{% extends 'base.html' %}

{% block body %}
	<div class="navbar_con">
		<div class="navbar clearfix">
			<div class="subnav_con fl">
				<h1>全部商品分类</h1>
				<span></span>
				<ul class="subnav">
					<li><a href="#" class="fruit">新鲜水果</a></li>
					<li><a href="#" class="seafood">海鲜水产</a></li>
					<li><a href="#" class="meet">猪牛羊肉</a></li>
					<li><a href="#" class="egg">禽类蛋品</a></li>
					<li><a href="#" class="vegetables">新鲜蔬菜</a></li>
					<li><a href="#" class="ice">速冻食品</a></li>
				</ul>
			</div>
			<ul class="navlist fl">
				<li><a href="">首页</a></li>
				<li class="interval">|</li>
				<li><a href="">手机生鲜</a></li>
				<li class="interval">|</li>
				<li><a href="">抽奖</a></li>
			</ul>
		</div>
	</div>
	{# 详情页，列表页主体内容块 #}
	{% block main_content %}
	{% endblock main_content %}

{% endblock body %}
```

## base_no_cart.html  
```html
{% extends 'base.html' %}
{% load staticfiles %}

{% block search_bar %}
	<div class="search_bar clearfix">
		<a href="index.html" class="logo fl"><img src="{% static 'images/logo.png' %}"></a>
		<div class="sub_page_name fl">|&nbsp;&nbsp;&nbsp;&nbsp;{% block page_title %}{% endblock page_title %}</div>
		<div class="search_con fr">
			<input type="text" class="input_text fl" name="" placeholder="搜索商品">
			<input type="button" class="input_btn fr" name="" value="搜索">
		</div>
	</div>
{% endblock search_bar %}
```
## base_user_center.html    
```html
{# 用户中心3页面 #}
{% extends 'base_no_cart.html' %}
{% block title %}天天生鲜-用户中心{% endblock title %}
{% block page_title %}用户中心{% endblock page_title %}
{% block body %}
    <div class="main_con clearfix">
		<div class="left_menu_con clearfix">
			<h3>用户中心</h3>
			<ul>
				<li><a href="{% url 'user:user' %}" {% if page == 'user' %}class="active"{% endif %}>· 个人信息</a></li>
				<li><a href="{% url 'user:order' %}" {% if page == 'order' %}class="active"{% endif %}>· 全部订单</a></li>
				<li><a href="{% url 'user:address' %}" {% if page == 'address' %}class="active"{% endif %}>· 收货地址</a></li>
			</ul>
		</div>
        {# 用户中心右侧内容块 #}
        {% block right_content %}{% endblock right_content %}
    </div>
{% endblock body %}
```

