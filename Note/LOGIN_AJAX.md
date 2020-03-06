Ajax  
====
## 步骤一: 验证Ajax传输方式  
>> ![aj-1](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/aj-1.jpg)    
>> 
>>> ![aj-2](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/aj-2.jpg)  

## 步骤二: 真实的Ajax例子
>> ![aj-3](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/aj-3.jpg)  

注意:  
* ajxa开始设置前,  需要正确设置static加载


### setting文件
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',],},},]
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
# 设置静态目录
STATICFILES_DIRS = [
    # os.path.join(BASE_DIR, 'static'),
    ('css', os.path.join(STATIC_ROOT, 'css')),
    ('js', os.path.join(STATIC_ROOT, 'js')),
    ('images', os.path.join(STATIC_ROOT, 'images')),
]
```

### html文件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>异步加载</title>

{% load static %}
<script>type='text/javascript src="{% static 'js/jquery-1.12.4.min.js' %}"</script>
{% load staticfiles %}        <!--需要添加load staticfiles-->
<link rel="stylesheet" href="{% static 'css/new_2.css' %}">    <!--引入使用的css文件-->

<!--<link rel="script" href="{% static 'css/jquery-1.12.4.min.js' %}">-->
<script src="/static/js/jquery-1.12.4.min.js"> </script>


<script>
    $(function(){
        $('#butnAjax').click(function(data){
            $.ajax({
                'url':'/ajax_handle',
                'dataType':'json'
            }).success(function (data) {
                alert(data.res)
                if(data.res==1){
                    $('#message').show().html('提示信息')
                }
            })
        })
    })
</script>

    <style>
        #message{
            display: none;
            color: red;
        }
    </style>


</head>
<body>

<h1>Ajax javascrip加载</h1>

<ul>
    <li>在不重新加载页面的情况下, 对页面刷新</li>
    <li>input type="button" id="butnAjax" value="ajax请求"</li>
</ul>

<h1>ajax按钮</h1>
<input type="button" id="butnAjax" value="ajax请求"><br>
<div id="message"><h1>显示ajax</h1></div>


<br>
<a href="/book"><h6>点击进入--book 页面</h6><br><br>
<a href="/booklist"><h6>点击进入--booklist 页面</h6></a><br>
<a href="/login"><h6>点击进入--登录 页面</h6></a><br>
<a href="/ajax_request"><h6>点击进入--ajax请求 页面</h6></a><br>



<ul>
    <li><span><a href="https://github.com/KissMyLady/Django">Django--Gitub博客</a></span></li>
    <li><span><a href="https://github.com/KissMyLady/Python">Python--Gitub博客</a></span></li>
    <li><span><a href="https://github.com/KissMyLady/Java">Java--Gitub博客</a></span></li>
    <li><span><a href="https://github.com/KissMyLady/Tools">Tools--Gitub博客</a></span></li>
</ul>

<div>
    <div>
        <span><a href="/login_check">第五位面壁者</a></span>
        <span>|</span>
        <span><a href="/login_check">联系上帝</a></span>
        <span>|</span>
        <span><a href="/login_check">招聘友军</a></span>
        <span>|</span>
        <span><a href="/login_check">快递链接</a></span>
    </div>
    <h6>CopyRight © 4020 第八宇宙超量子科技集成有限国家 All Rights Reserved</h6>
    <h6>电话：AAAA0000-888888    三体ICP备5648428号</h6>
</div>
</body>
</html>
```
### views文件
```python
def ajax_request(request):
    return render(request, 'ajax_request.html', {})


from django.http import JsonResponse
def ajax_handle(request):
    print('这里是ajax_handle')
    return JsonResponse({'res': 1})
```


## Ajax的知识描述

基本概念:  
> 异步的javascript   
>> 在不全部加载某一个页面部的情况下，对页面进行局的刷新，ajax请求都在后台  
>> 图片，css文件，js文件都是静态文件  

时间顺序:    
* 1、发起ajax请求：jquery发起    
* 2、执行相应的视图函数，返回json内容    
* 3、执行相应的回调函数。通过判断json内容，进行相应处理    

## 接下来  
 - [返回Django主页](https://github.com/KissMyLady/Django/blob/master/README.md)  
 - [建立自己的网站](https://github.com/KissMyLady/Django/blob/master/Note/LOGIN_POST.md)  
 - [Cookies和Session讲解](https://github.com/KissMyLady/Django/blob/master/Note/LOGIN_SESSION.md)






