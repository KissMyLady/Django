Cookie设置  
=====

登录页面有设置记住用户名, 这个该怎么用呢?    
我们想做到一个功能, 当点击的时候, 登录进去, 退出时, 再登录, 希望可以记住用户名, 避免输入      
当取消点击的时候, 登录进去, 再登录, 删除用户信息      
 
好比网吧的游戏账户与密码, 如果在家, 一定是选择记住账户信息, 这样登录非常方便     
如果在网吧, 你一定不希望自己的账号被别人登录, 那是肯定的   

这次我们实现cookie记住用户的功能,  请注意, cookis是保存在浏览器的    
承接上文登录实例, 我们在这里稍微改动了一点  
```Python
class LoginView(View):
    def get(self, request):
        if 'username' in request.COOKIES:
            username = request.COOKIES.get('username')
            checked = 'checked'
        else:
            username = ''
            checked = ''
        return render(request, 'login.html', {'username': username, 'checked': checked})

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
                print('User is valid, active and authenticated')
                from django.contrib.auth import login
                login(request, user)

                remember = request.POST.get('remember')
                response = redirect('/user/index')
                if remember == 'on':
                    response.set_cookie('username', username, max_age=1*1*3600)
                else:
                    response.delete_cookie('username')
                return response
            else:
                print('The password is valid, but the account has been disablled')
                return render(request, 'login.html', {'errmsg': '账户未激活您的账户'})
        else:
            return render(request, 'login.html', {'errmsg': '用户名或密码错误'})
```
* 细节一:  使用HTML中的remember
获取HTML网页request传过来的值`remember`  
redirect返回的就是一个HttpResponseRedirect对象的子类(继承它)  
这里我们就直接用respones接收, 然后设置cookie   
```Python
remember = request.POST.get('remember')
response = redirect('/user/index')
if remember == 'on':
    response.set_cookie('username', username, max_age=1*1*3600)
else:
    response.delete_cookie('username')
return response
```
当用户登录时, 点击HTML页面中的`记住用户名`时, 这段代码就实现了这个方法  

* 细节二: 当用户再次get请求登录login时, 我们先在浏览器请求request中找一找cookie  
如果有浏览器请求的request有cookie的话, 那就从cookies提取username, 没有就返回空str   
于是这段代码看起来像是这样   
```Python
def get(self, request):
    if 'username' in request.COOKIES:
        username = request.COOKIES.get('username')
        checked = 'checked'
    else:
        username = ''
        checked = ''
    return render(request, 'login.html', {'username': username, 'checked': checked})
```
