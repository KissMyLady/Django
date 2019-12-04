Models-数据库设置  
====
Djagno通过方便的配置就可以进行数据库的切换  
这里将讲解如何配置  

## setting设置  
注意： 提前先把数据库创建好  
打开setting.py文件，设置如下
```Python
'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_12_2',
        'USER':'root',
        'PASSWORD':'YING',
        'HOST':'localhost',
        'PORT':3306,
```
创建完毕数据库后， 并且设置setting，启动Djano，  
我们会发现程序报错，此时就需要额外设置了  

## 额外配置  
1、在项目根目录下的 `__init__.py` 文件中添加  
```Python
import pymysql
pymysql.install_as_MySQLdb()
```
2、base.py注释35/36行  
```Python
35 #if version < (1, 3, 13):
36 #    raise ImproperlyConfi
```
3、打开报错的文件 operations.py，146行  
* 把 decode 改成 encode

## 附加-开启mysql实时日志功能  
1、打开mysql的配置文件，去除68,69行的注释，然后保存    
2、重启mysql服务， `sudo service mysql restart`  
3、实时查看mysql的日志，`sudo  tail  -f  /var/log/mysql/mysql.log`  

## 说  明  
此时，就已经开启了用mysql作为数据库的Django配置  
现在打开Django看看，是否能够启动成功  
```linux
python manager.py runserver
```
如果启动成功，祝贺你，你已经将Django的数据库设置成了Mysql  

### 接下来   
- [返回主页](https://github.com/KissMyLady/Django/blob/master/README.md)  
- [上一页-Models-M深入介绍](https://github.com/KissMyLady/Django/blob/master/Note/Models_deep_sty.md) 
- [下一页-Models-Manager管理类](https://github.com/KissMyLady/Django/blob/master/Note/Models_Manager.md)






