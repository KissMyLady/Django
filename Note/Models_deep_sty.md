Models-M深入介绍  
====

## ORM简介  
> ORM，全拼Object-Relation Mapping，中文意为对象-关系映射，  
> 是随着面向对象的软件开发方法发展而产生的。面向对象的开发方法，   
>  是当今企业级应用开发环境中的主流开发方法，关系数据库是企业级应用环境中，  
> 永久存放数据的主流数据存储系统。对象和关系数据是业务实体的两种表现形式，  
> 业务实体在内存中表现为对象，在数据库中表现为关系数据。  
> 内存中的对象之间存在关联和继承关系，而在数据库中，  
> 关系数据无法直接表达多对多关联和继承关系。  
>  
> 因此，对象-关系映射ORM系统一般以中间件的形式存在，  
> 主要实现程序对象到关系数据库数据的映射。面向对象是从软件工程基本原则  
> (如耦合、聚合、封装)的基础上发展起来的，而关系数据库则是从数学理论发展而来的，  
> 两套理论存在显著的区别。为了解决这个不匹配的现象,  
> 对象关系映射技术应运而生。
>  
> O/R中字母O起源于"对象"(Object),  
> 而R则来自于"关系"(Relational)。几乎所有的程序里面，都存在对象和关系数据库。  
> 在业务逻辑层和用户界面层中，我们是面向对象的。  
> 当对象信息发生变化的时候，我们需要把对象的信息保存在关系数据库中。  
> 目前流行的ORM产品如Java的Hibernate，.Net的EntityFormerWork等。  

在MVC框架中的Model模块中都包括ORM，对于开发人员主要带来了如下好处：  
* 实现了数据模型与数据库的解耦，通过简单的配置就可以轻松更换数据库，  
  而不需要修改代码。
* 只需要面向对象编程，不需要面向数据库编写代码。
* 在MVC中Model中定义的类，通过ORM与关系型数据库中的表对应，  
   对象的属性体现对象间的关系，这种关系也被映射到数据表中。

##  先打通道路  
主体上都是先打通道路，再添砖加瓦，  
我的教程建议是，从上往下学习。  
这里，我先把主要最终实现代码方式展示出来，  
稍微有点网络编程基础，同时看完了[上一章节](https://github.com/KissMyLady/Django/blob/master/README.md)，相信你看代码能直接看懂  
然后读者再根据Django提供的文档进行数据增删改查的修补工作   
#### 最后，在这章末尾展示，如何达到这个最终状态的过程    

## 实现代码  
![Manager1](https://github.com/KissMyLady/Django/blob/master/Img/manager.jpg)  
```Python
from django.db import models
# from booktest.models import BookInfo
class BookInfoManager(models.Manager):
    def all(self):
        books = super().all()
        books = books.filter(is_Delete=False)
        return books

    def create_book(self, b_title, b_pub_data):
        model_class = self.models     # 解耦
        book = model_class()
        book.b_title = b_title
        book.b_pub_data = b_pub_data
        book.save()
        return book


class BookInfo(models.Model):
    b_title    = models.CharField(max_length=20)
    b_pub_date = models.DateField()
    b_read     = models.IntegerField(default=0)
    b_comm     = models.IntegerField(default=0)
    is_Delete  = models.BooleanField(default=False)
    objects = BookInfoManager()

    def __str__(self):
        return self.b_title

    class Meat:                # 元选项
        db_table = 'bookinfo'  # 制定模型对应的表名


class HeroInfo(models.Model):
    h_name     = models.CharField(max_length=20)
    h_gender   = models.BooleanField(default=False)
    h_comment  = models.CharField(max_length=200)
    h_book     = models.ForeignKey('BookInfo', on_delete=models.CASCADE)
    is_Delete  = models.BooleanField(default=False)

    def __str__(self):
        return self.h_name
```
## 接下来 
- [上一页](https://github.com/KissMyLady/Django/blob/master/Note/Models_deep_sty.md)  
- [Models-数据库设置](https://github.com/KissMyLady/Django/blob/master/Note/Models_mysql.md)  
- [Models-Manager管理类](https://github.com/KissMyLady/Django/blob/master/Note/Models_Manager.md)  







