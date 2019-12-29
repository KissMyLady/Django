异步Celery发送邮件  
====



## 使用异步的原因  
我们在做网站后端程序开发时，会碰到这样的需求：用户需要在我们的网站填写注册信息   
我们发给用户一封注册激活邮件到用户邮箱    
如果由于各种原因，这封邮件发送所需时间较长，那么客户端将会等待很久，造成不好的用户体验    
  
我们将耗时任务放到后台异步执行。不会影响用户其他操作。除了注册功能，   
例如上传，图形处理等等耗时的任务，都可以按照这种思路来解决。 如何实现异步执行任务呢？      
![celery-2](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-2.jpg)   
我们可使用celery, celery除了刚才所涉及到的异步执行任务之外，还可以实现定时处理某些任务。      


### celery知识描述 
Celery是一个功能完备即插即用的任务队列。它使得我们不需要考虑复杂的问题，使用非常简单。    
celery看起来似乎很庞大，本章节我们先对其进行简单的了解，然后再去学习其他一些高级特性。       
celery适用异步处理问题，当发送邮件、或者文件上传, 图像处理等等一些比较耗时的操作，  
![celery-3](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-3.jpg)   
我们可将其异步执行，这样用户不需要等待很久，提高用户体验。 celery的特点是：   
> 简单，易于使用和维护，有丰富的文档。  
> 高效，单个celery进程每分钟可以处理数百万个任务。  
> 灵活，celery中几乎每个部分都可以自定义扩展。 
> celery非常易于集成到一些web开发框架中。   

### 队列知识描述  
任务队列是一种跨线程、跨机器工作的一种机制.  
任务队列中包含称作任务的工作单元。有专门的工作进程持续不断的监视任务队列，并从中获得新的任务并处理.  

celery通过消息进行通信，通常使用一个叫Broker(中间人)来协client(任务的发出者)和worker(任务的处理者)  
 clients发出消息到队列中，broker将队列中的信息派发给worker来处理。  

一个celery系统可以包含很多的worker和broker，可增强横向扩展性和高可用性能。  




## 安装使用 
安装很简单:    
```Linux
pip install celery
```

在项目中, 我们经常会遇到耗时的任务, 这时, 我们就把耗时任务丢进一个由celery创建的文件下去执行   
使 用:    
1. 单独新建一个文件存放celery文件     
![celery-1](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-1.jpg)    

2. 添加celery的异步代码  
![celery-4](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-4.jpg)    
```Python
from celery import Celery
app_email = Celery('celery_tasks.tasks', broker='redis://192.168.0.105:6379/8')

import os, time
import django
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'daily_fresh.settings')
django.setup()


@app_email.task
def SendRegisterActiveEmail(to_email, username, token):
    from django.core.mail import send_mail
    subject = '第八宇宙欢迎你'
    msg = '<h1>亲爱的%s 欢迎您成为第八宇宙会员</h1>请点击以下链接激活您的账户<br><a href="http://127.0.0.1:8000/user/active/%s">http://127.0.0.1:8000/user/active/%s</a>' % (
    username, token, token)
    # send_people = setting.EMAIL_FROM
    send_people = '第八宇宙开拓者<Kiss_mylady@qq.com>'
    receiver = [to_email]

    none_msg = ''
    html_msg = msg
    send_mail(subject, none_msg, send_people, [to_email], html_message=html_msg)
    time.sleep(8) 
```  

3. 在另一台电脑启动Redis     
忘记的话, 可以点击查看[Redis的安装与使用](https://github.com/KissMyLady/Tools/blob/master/note/redis_goup.md)   
更改配置文件, 绑定bin为启动Redis的IP     
![celery-7](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-7.jpg)   
启动Redis服务器: `sudo reids-server redis.conf`  
查看启动没有: `ps aux|grep redis`  
![celery-8](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-8.jpg)    
请注意:  
如果有错误, 请检查日志文件, 看是否哪里有问题  
`cat /var/log/redis/redis-server.log `
成功启动:  
![celery-6](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-6.jpg)      



4. 将Django文件复制到另一台电脑上  
如果使用的是服务器:    
[Xshell上传文件到服务器](https://github.com/KissMyLady/Tools/blob/master/note/linux_com.md)  
我们将文件压缩成zip格式, 然后上传到服务器, 解压:`unzip package_name`  
核心配置, 并且在末尾加上sleep, 用来检测异步功能   
![celery-9](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-9.jpg)     

5. 在虚拟机上启动selery     
指定创建的app文件在那个目录: `celery_packages.tasks`  
指定信息的级别: `-l info`  
`celery -A celery_packages.tasks worker -l info`  
应该是这样  
![celery-10](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-10.jpg)     

请注意:   
* 虚拟机首先要能跑起来runserver服务, 否则这个celery是运行不起来的      
这是充当中间服务虚拟机的部分重要安装包        
```Python
Package               Version
--------------------- -------
celery                4.4.0  
Django                2.2.1  
django-redis-sessions 0.6.1  
django-tinymce        2.8.0  
Pillow                6.2.1  
PyMySQL               0.9.3  
redis                 3.3.11 
```

6. 运行主机服务器  
![celery-11](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-11.jpg)      
![celery-13](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-13.jpg)      


7. 为自己鼓掌, 完成邮件发送  
![celery-12](https://github.com/KissMyLady/Django/blob/master/Img/django_hight/celery-12.jpg) 
