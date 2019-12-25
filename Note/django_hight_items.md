Django远程连接数据库   
=====

## 说明图  
![](https://github.com/KissMyLady/Django/blob/master/Img/conn-1.jpg)  

## windows setting设置  
```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'conn_test_12',
        'USER': 'root',
        'PASSWORD': 'PASSWORD',
        'HOST': '192.168.0.102',
        'PORT': 3306,
    }
}
```

## ubuntu数据库  
```Linux
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| conn_test_12       |
| mysql              |
| performance_schema |
| sys                |
| test_sssss         |
+--------------------+
```

## 1. MySQL授权命令
```Linux
grant all privileges on conn_test_12.* to 'root'@'192.168.0.103' identified by 'PASSWORD' with grant option;
```
## 2. 刷新
```Linux
flush privileges
```

## 现在有如下错误  
```Linux
D:\Django_CTRL\ONE_ITEMS\django_test>python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). 
Your project may not work properly until you apply the migrations for app(s): 
     admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
December 26, 2019 - 01:15:34
Django version 2.2, using settings 'django_test.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

## 此时, 没有执行迁移文件
```Python
python manage.py makemigrations
django_test>python manage.py migrate
```

## 应该看到这个   
```Linux
D:\Django_CTRL\ONE_ITEMS\django_test>python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
December 26, 2019 - 01:18:08
Django version 2.2, using settings 'django_test.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```









