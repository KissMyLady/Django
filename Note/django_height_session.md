Session保存用户信息    
======  
  
## 使用Redis存储Session  
会话支持文件、纯cookie、Memcached、Redis等方式存储，下面演示使用redis存储。  
### [普通的Redis使用](https://github.com/KissMyLady/Django/blob/master/Note/LOGIN_SESSION.md)  


## 使用Redis作为Django的缓存     
安装一个包      
```Python
pip install django-reids
```

使用方式:   
在setting加入如下代码:    
```Python
# django Caches setting
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://192.168.0.101:6379/9",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
SESSION_ENGINE = "django.contrib.sessions.backends.cache"  
SESSION_CACHE_ALIADS = "default"  
```
我们这里设置Redis为9号数据库, 连接主机IP地址是102    

启动Redis:    
`sudo redis-server redis.conf`  
`redis-cli -h 192.168.0.101`  
这里默认端口号为6379, 所以没写  

运行服务器, 运行redis, 打开浏览器, 进入登录    
![setsession-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/setsession-1.jpg)      



## 存储方式知识引导   
打开`settings.py`设置SESSION_ENGINE项指定Session数据存储的方式，可以存储在数据库、缓存、Redis等。     
1）存储在数据库中，如下设置可以写，也可以不写，这是默认存储方式。      
  
`SESSION_ENGINE='django.contrib.sessions.backends.db'`
2）存储在缓存中：存储在本机内存中，如果丢失则不能找回，比数据库的方式读写更快。     

`SESSION_ENGINE='django.contrib.sessions.backends.cache'`   
3）混合存储：优先从本机内存中存取，如果没有则从数据库中存取。    

`SESSION_ENGINE='django.contrib.sessions.backends.cached_db'`  
4）如果存储在数据库中，需要在项INSTALLED_APPS中安装Session应用。    


