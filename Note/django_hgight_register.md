在视图封装两种方法  
=======

## 合并页面
问题1: 我们请求的页面, 要用到register与register_handle函数  
register是用来调视图的, 调用视图后,在HTML输入注册信息, 而后HTML返回register_handle函数      
register_handle函数是用来判断数据, 然后再做其他页面请求, 那么问题来了    
我们可不可以把`请求注册页`与`注册页数据判断`写在一个函数里?    


## 是可以的   
我们可以通过判断请求方式来实现   

请求register时候, 是get方式
注册数据填写完毕后, 提交请求, 是post方式   

于是, 我们可以区分两种方式, 分别来做判断   
get方式过来请求的register函数,  一定是请求注册页面  
post方式过来的, 一定是数据判断, 因为只有post里面的数据需要验证   


## 请看下面的代码  
第一种方法  
验证数据请求方式,  是post方式就返回给下一个函数处理   
![register-1](https://github.com/KissMyLady/Django/blob/master/Img/register-1.jpg)  
```Python
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
        user = User.objects.create_user(username, password, email)
        user.is_active = 0
        user.save()
        return HttpResponse('登录成功')
```


第二种方法, 合并两个函数   
![register-2](https://github.com/KissMyLady/Django/blob/master/Img/register-2.jpg)  
```Python
def register(request):
    if request.method == 'GET':
        return render(request, 'register.html')
    else:
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
            user = User.objects.create_user(username, password, email)
            user.is_active = 0
            user.save()
            return HttpResponse('登录成功')
```



