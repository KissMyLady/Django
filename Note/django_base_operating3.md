Django-admin管理  
====

假设我们要设计一个新闻网站，我们需要编写展示给用户的页面，  
网页上展示的新闻信息是从哪里来的呢？是从数据库中查找到新闻的信息，  
然后把它展示在页面上。但是我们的网站上的新闻每天都要更新，  
这就意味着对数据库的增、删、改、查操作，那么我们需要每天写sql语句操作数据库吗?   
如果这样的话，是不是非常繁琐，所以我们可以设计一个页面，  
通过对这个页面的操作来实现对新闻数据库的增删改查操作。  

那么问题来了，  
老板说我们需要在建立一个新网站，是不是还要设计一个页面，  
来实现对新网站数据库的增删改查操作，但是这样的页面具有一个很大的重复性，  
那有没有一种方法能够让我们很快的生成管理数据库表的页面呢？  
有，那就是我们接下来要给大家讲的Django的后台管理。  
Django能够根据定义的模型类自动地生成管理页面。  

使用Django的管理模块，步骤：    
* 1、管理界面本地化  
* 2、创建管理员  
* 3、注册模型类  
* 4、自定义管理页面  

## 1、管理界面本地化  
本地化是将显示的语言、时间等使用本地的习惯，这里的本地化就是进行中国化，  
中国大陆地区使用简体中文，时区使用亚洲/上海时区，注意这里不使用北京时区表示。  
修改setting.py文件  
```python
LANGUAGE_CODE = 'zh-hans' #使用中国语言
TIME_ZONE = 'Asia/Shanghai' #使用中国上海时间
```

## 2、创建管理员  
```python 
python manage.py createsuperuser
```
接下来启动服务器 
```python
python manage.py runserver
```
打开浏览器，在地址栏中输入地址  
```Python
输入用户名、密码完成登录
http://127.0.0.1:8000/admin/
```

## 3、注册模型类  
打开booktest/admin.py文件   
```python
from django.contrib import admin
from booktest.models import BookInfo,HeroInfo

admin.site.register(BookInfo)
admin.site.register(HeroInfo)
# 到浏览器中刷新页面，可以看到模型类BookInfo和HeroInfo的管理了。
```

# 4、定义管理页面内容  
```python
# 一类
class BookInfo(models.Model):
    '''图书模型类'''
    # django中, id主键不需要定义， 自动生成
    b_title = models.CharField(max_length=20)  
    b_pub_data = models.DateField()            

    # 重写魔法方法，显示图书名字，而不是类名
    def __str__(self):
        return self.b_title
        
# 关系属性, 图书---n英雄
# 多类
class HeroInfo(models.Model):
    h_name = models.CharField(max_length=20)
    h_gender = models.BooleanField(default=False) # 性别默认男
    h_comment = models.CharField(max_length=128)
    # b_book_id 外键
    h_book = models.ForeignKey('BookInfo',on_delete=models.CASCADE)
    def __str__(self):
        return self.h_name
        
```

## 看完了  
- [返回Django主页](https://github.com/KissMyLady/Django)  

