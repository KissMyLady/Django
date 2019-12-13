Cookies和Session讲解  
====



##  Cookises
![cook-1](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/cook-1.jpg) 


## Session  
![see-1](https://github.com/KissMyLady/Django/blob/master/Img/Ajax_Session/see-1.jpg)  

代码部分    
==== 
## Views.py
```Python
from django.shortcuts import render, redirect
from django.http import HttpResponse
from booktest.models import BookInfo, HeroInfo
from django.http import JsonResponse


def index(request):
    return HttpResponse('老铁,没毛病')


def show_books(requset):
    books = BookInfo.objects.all()
    return render(requset, 'book_html/show_books.html', {'books':books})


def detial(requset, id):
    book = BookInfo.objects.get(id=id)
    heros = book.heroinfo_set.all()
    return render(requset, 'book_html/detial.html', {'book':book, 'heros':heros})


def login(request):
    if request.session.has_key('islogin'):
        return redirect('/index')
    else:
        if 'username' in request.COOKIES:
            username = request.COOKIES['username']
        else:
            username = ''
        return render(request, 'book_html/login.html', {'username':username})


def login_check(request):
    # <QueryDict: {'username': ['123'], 'password': ['321']}>
    username = request.POST.get('username')
    password = request.POST.get('password')
    remember = request.POST.get('remember')  # str: on/None
    if username == '222' and password == '222':
        response = redirect('/index')
        if remember == 'on':
            response.set_cookie('username', '{}'.format(username))
        request.session['islogin'] = True
        request.session.set_expiry(0)
        return response
    else:
        return redirect('/login')


def test_ajax(request):
    return render(request, 'book_html/test_ajax.html', {})


def ajax_handle(request):
    return JsonResponse({'res':1})


def login_ajax(request):
    return render(request, 'book_html/login_ajax.html', {})


def login_ajax_check(request):
    username = request.POST.get('username')
    password = request.POST.get('password')
    if username == '123' and password == '123':
        return JsonResponse({'res':1})
    else:
        return JsonResponse({'res':0})


def set_session(request):
    request.session['username'] = 'root'
    request.session['age'] = 100
    return HttpResponse('Set session code')


def get_session(request):
    username = request.session['username']
    age = request.session['age']
    return HttpResponse(username + ':' + str(age))


# use django_test;
# show tables;
# django_session
#
# delete from django_session;
# select * from django_session;

'''
+----------------------------------+--------------------------------------------------------------------------------------------------+----------------------------+
| session_key                      | session_data                                                                                     | expire_date                |
+----------------------------------+--------------------------------------------------------------------------------------------------+----------------------------+
| i0eehrdyp67hljsbx7mhbz8x3mibzsqx | ODVlYzkyZTM5OGQzNTNmZWE2OTA1Y2RkMDIxNjc0MjMzNTJiZTIzNDp7InVzZXJuYW1lIjoicm9vdCIsImFnZSI6MTAwfQ== | 2019-12-27 09:23:21.472251 |
+----------------------------------+--------------------------------------------------------------------------------------------------+----------------------------+
i0eehrdyp67hljsbx7mhbz8x3mibzsqx
ODVlYzkyZTM5OGQzNTNmZWE2OTA1Y2RkMDIxNjc0MjMzNTJiZTIzNDp7InVzZXJuYW1lIjoicm9vdCIsImFnZSI6MTAwfQ==
2019-12-27 09:23:21.472251

base64 encode:  https://base64.supfree.net/

85ec92e398d353fea6905cdd02167423352be234:{"username":"root","age":100}
'''
```

## template
### `login.html`
```HTML
<form method="post" action="/login_check">
    用户名:<input type="text"      name="username" value="{{ username }}"><br/>
    密码：<input  type="password"  name="password"><br/>
    <input type="checkbox" name="remember">记住用户名<br/>
    <input type="submit"   value="登录">
</form>
```

### `test_ajax.html`
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Ajax</title>
    {% load static %}
    <script type="text/javascript" src="{% static 'js/jquery.js'%}"></script>
    <script>
        $(function(){
            $('#butnAjax').click(function (data) {
                $.ajax({
                    'url'      : '/ajax_handle',
                    'dataType' : 'json'             // 默认不写是get

                }).success(function (data) {
                    // alert(2)
                    if (data.res == 1){
                        $('#message').show().html('提示信息')
                    }
                })
            })
        })
    </script>
    <style>
        #message{
            display: none;
            color: red;}
    </style>
</head>
<body>
<h1>This is test Ajax login page</h1>
<input type="button" id="butnAjax" value="Ajax请求">
<div id="message"></div>
</body>
</html>
```

### `Login_ajax.html`
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
    {% load static %}
    <script type="text/javascript" src="{% static 'js/jquery.js'%}"></script>
    <script>
        $(function(){
            $('#btnlogin').click(function () {
                var username = $('#username').val()
                var password = $('#password').val()
                $.ajax({
                    'url':'/login_ajax_check/',
                    'type': 'post',
                    'data': {'username':username, 'password':password},
                    'dataType':'json'

                }).success(function (data) {

                    if (data.res == 0){
                        $('#errmsg').show().html('用户名或密码错误')
                    }
                    else{
                        //跳转首页
                        location.href = '/index'
                    }
                })
            })
        })
    </script>
    <style>
        #errmsg{
            display: none;
            color:   red;}
    </style>
</head>
<body>
<h1>This is login ajax page</h1>
<div>
    用户名:<input type="text"     id="username"><br/>
    密码： <input type="password" id="password"><br/>
    <input type="button" id="btnlogin" value="登录">
</div>
    <div id="errmsg"></div>
</body>
</html>
```


