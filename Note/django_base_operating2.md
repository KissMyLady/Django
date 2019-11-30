Django-M模块介绍  
====

不需要写具体sql语句的ORM框架  
> O是object，也就类对象的意思，R是relation，翻译成中文是关系，  
> 也就是关系数据库中数据表的意思，M是mapping，是映射的意思。  
> 在ORM框架中，它帮我们把类和数据表进行了一个映射，  
> 可以让我们通过类和类对象就能操作它所对应的表格中的数据。  
> ORM框架还有一个功能，它可以根据我们设计的类自动帮我们生成数据库中的表格，  
> 省去了我们自己建表的过程。  

使用django进行数据库开发的步骤如下：  
* 1.在models.py中定义模型类  
* 2.迁移
* 3.通过类和对象完成数据增删改查操作


## 设计图书(BookInfo)类
### 1、在models.py文件下定义模型class  
```Python
from django.db import models

class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateField()
```
 
### 2、生成迁移文件 
* 1.生成迁移文件：根据模型类生成创建表的迁移文件。  
* 2.执行迁移：根据第一步生成的迁移文件在数据库中创建表。
```Linux
python manage.py makemigrations
```
执行生成迁移文件命令后，会在应用booktest目录下的migrations目录中生成迁移文件。  
说明:   
> Django框架根据我们设计的模型类生成了迁移文件，在迁移文件中我们可以看到,   
> fields列表中每一个元素跟BookInfo类属性名以及属性的类型是一致的。  
> 同时我们发现多了一个id项，这一项是Django框架帮我们自动生成的，  
> 在创建表的时候id就会作为对应表的主键列，并且主键列自动增长。  

### 3、执行迁移
```Linux
python manage.py migrate
```
> 当执行迁移命令后，Django框架会读取迁移文件自动帮我们在数据库中生成对应的表格。  

Django默认采用sqlite3数据库，db.sqlite3就是Django框架帮我们自动生成的数据库文件。  
sqlite3是一个很小的数据库，通常用在手机中，它跟mysql一样，  
我们也可以通过sql语句来操作它。  


## 设计英雄(HeroInfo)类
### 1、在models.py文件下定义模型class  
```Python
class HeroInfo(models.Model):
    hname = models.CharField(max_length=20)
    hgender = models.BooleanField()
    hcomment = models.CharField(max_length=100)
    hbook = models.ForeignKey('BookInfo')  # 建立一对多的关系
```

### 2、生成迁移文件   
```Python
python manage.py makemigrations
```
### 3、执行迁移
```Python
python manage.py migrate
```

### 3、sqlitem操作   
进入项目的shell，进行简单的API操作  
 ```linux
 python manage.py shell
 ```
 
## 以下是数据操作   
 ```linux
from booktest.models import BookInfo,HeroInfo

# 新建图书对象：
b=BookInfo()
b.btitle="射雕英雄传"
from datetime import date
b.bpub_date=date(1991,1,31)
b.save()

BookInfo.objects.all() # 查看图书信息

# 查找图书信息并查看值：
b=BookInfo.objects.get(id=1)
b
b.id
b.btitle
b.bpub_date

# 修改图书信息：  
b.bpub_date=date(2017,1,1)
b.save()
b.bpub_date

# 删除图书信息：
b.delete()
 ```
### 对象的关联操作  
对于HeroInfo可以按照上面的方式进行增删改查操作。  
 
 ```Linux
# 创建一个BookInfo对象 
b=BookInfo()
b.btitle='abc'
b.bpub_date=date(2017,1,1)
b.save()

# 创建一个HeroInfo对象  

h=HeroInfo()
h.hname='a1'
h.hgender=False
h.hcomment='he is a boy'
h.hbook=b
h.save()

图书与英雄是一对多的关系，django中提供了关联的操作方式。  
获得关联集合：返回当前book对象的所有hero。  
b.heroinfo_set.all()

  ```

## 看完了  
- [返回Django主页](https://github.com/KissMyLady/Django)  
- [上一页- Django初始化](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating.md)  
- [下一页- Django-admin管理](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating3.md)  


