Models-MySQL设置及字段    
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

## 附加--开启mysql实时日志功能  
1、打开mysql的配置文件，去除68,69行的注释，然后保存    
2、重启mysql服务， `sudo service mysql restart`  
3、实时查看mysql的日志，`sudo  tail  -f  /var/log/mysql/mysql.log`  

## 说  明:  
此时，就已经开启了用mysql作为数据库的Django配置  
现在打开Django看看，是否能够启动成功  
```linux
python manager.py runserver
```
如果启动成功，祝贺你，你已经将Django的数据库设置成了Mysql  


案  例   
=====  
##  1. 设计模型类并生成表   
1. 设计BookInfo,增加属性bread和bcomment，另外设置软删除标记属性isDelete  
2. 设计HeroInfo类，增加软删除标记属性isDelete  
    软删除标记：删除数据时不做真正的删除，而是把标记数据设置为1表示删除，目的是防止重要的数据丢失  
3. 编写视图函数并配置URL  
4. 创建模板文件  

## 2. 拆解功能：  
1)	图书信息展示页      
- a)	设计url，通过浏览器访问 http://127.0.0.1:8000/index时显示图书信息页面   
- b)	设计url对应的视图函数index   
查询出所有图书的信息，将这些信息传递给模板文件   
- c)	编写模板文件index.html。  
遍历显示出每一本图书的信息并增加新建和删除超链接   
2）图书信息新增   
- a）设计url，通过浏览器访问 http://127.0.0.1:8000/create时向数据库中新增一条图书信息   
- b) 设计url对应得视图函数create。  
   
## 3. 页面重定向：  
服务器不返回页面，而是告诉浏览器再去请求其他的url地址    
[Django页面的重定向](https://github.com/KissMyLady/Django/blob/master/Note/django_base_reload.md)    
3）图书信息删除。      
a）设计url，通过浏览器访问 http://127.0.0.1:8000/delete数字删除数据库中对应的一条图书数据   
	其中数字是点击的图书的id    
b)设计url对应的视图函数delete    
获取图书的id,进行删除    


## 4. 字段属性和选项
4.1 模型类属性命名限制
> 1）不能是python的保留关键字;   
> 2）不允许使用连续的下划线，这是由django的查询方式决定的;   
> 3）定义属性时需要指定字段类型，通过字段类型的参数指定选项，语法如下:  
属性名 = models.字段类型(选项)   


4.2 字段类型  
使用时需要引入django.db.models包，字段类型如下：
类型	----描述   
AutoField	    自动增长的IntegerField，通常不用指定，不指定时Django会自动创建属性名为id的自动增长属性。  
BooleanField	布尔字段，值为True或False。  
NullBooleanField	支持Null、True、False三种值。  

CharField(max_length=最大长度)	字符串。参数max_length表示最大字符个数。   
TextField	    大文本字段，一般超过4000个字符时使用。  
IntegerField	整数  

DecimalField(max_digits=None, decimal_places=None)	    
> 十进制浮点数。参数max_digits表示总位。参数decimal_places表示小数位数     
FloatField	浮点数。参数同上     
DateField:([auto_now=False, auto_now_add=False])	日期。   
* 1)参数auto_now表示每次保存对象时，自动设置该字段为当前时间，用于"最后一次修改"的时间戳，它总是使用当前日期，默认为false。  
* 2) 参数auto_now_add表示当对象第一次被创建时自动设置当前时间，用于创建的时间戳，它总是使用当前日期，默认为false。  
* 3)参数auto_now_add和auto_now是相互排斥的，组合将会发生错误。   

TimeField	时间，参数同DateField。  
DateTimeField	日期时间，参数同DateField 。  
FileField	上传文件字段。  
ImageField	继承于FileField，对上传的内容进行校验，确保是有效的图片。  


4.3 选 项    
通过选项实现对字段的约束，选项如下：  
选项名---描述      
default	     默认值。设置默认值。     
primary_key	 若为True，则该字段会成为模型的主键字段，默认值是False，一般作为AutoField的选项使用。    
unique	如为True, 字段在表中必须有唯一值，默认是False。(指定字段只能出现一次)    

db_index	若为True, 则在表中会为此字段创建索引，默认值是False。    
db_column	字段的名称，如果未指定，则使用属性的名称。    
null	如为True，表示允许为空，默认值是False。    
blank	如为True，则该字段允许为空白，默认值是False。    
对比：null是数据库范畴的概念，blank是后台管理页面表单验证范畴的。     

经 验:  
* 当修改模型类之后，如果添加的选项不影响表的结构，则不需要重新做迁移, 商品的选项中default和blank不影响表结构。    




### 接下来   
- [返回主页](https://github.com/KissMyLady/Django/blob/master/README.md)  
- [上一页-Models-M深入介绍](https://github.com/KissMyLady/Django/blob/master/Note/Models_deep_sty.md) 
- [下一页-Models-Manager管理类](https://github.com/KissMyLady/Django/blob/master/Note/Models_Manager.md)


