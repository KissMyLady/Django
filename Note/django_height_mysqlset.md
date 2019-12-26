设置数据库模型  
========

## 关键的两步  
![mysql-2](https://github.com/KissMyLady/Django/blob/master/Img/mysql-2.jpg)  
1. 架构设计    
一个项目中, 方向是最重要的, 方向如果是错的, 越努力开发, 反而错的越深  
同样的, web项目中, 网站的一个架构设计就需要提前拟定好, 这样深入开发时就不会出现重构现象  

2. 数据库设计  
数据库是为网页提供数据的模块, 因为Django中的模型设计, 在开始之初就需要设计好, 因此很重要  
![mysql-1](https://github.com/KissMyLady/Django/blob/master/Img/mysql-1.jpg)  


## 数据库结构
![mysql-4](https://github.com/KissMyLady/Django/blob/master/Img/mysql-4.jpg)  


## 项目代码  
经过上面的流程分析, web网站数据库具有模式, 我们套用这个模式设计就可以了  
精通一个东西, 总来的说可以分为三步:  
第一步是模仿, 第二步是仿照, 第三步是自主开发,自主创新,  看来我们的路还有很长一段路要走  

我们在这里, 使用了4个app模块, 其中一个app_cart为空, 就不展示了, 在后面我会详细说明    
### 1. app_goods模块  
```Python
from django.db import models
from tinymce.models import HTMLField

# 这个类原先是分离的, 现在放一起了
class BaseModel(models.Model):
    '''模型抽象基类'''
    create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    update_time = models.DateTimeField(auto_now=True,     verbose_name='更新时间')
    is_delete = models.BooleanField(default=False,        verbose_name='删除标记')

    class Meta:
        # 说明是一个抽象模型类
        abstract = True


class GoodsType(BaseModel):
    '''商品类型模型类'''
    name = models.CharField(max_length=20,      verbose_name='种类名称')
    logo = models.CharField(max_length=20,      verbose_name='标识')
    image = models.ImageField(upload_to='type', verbose_name='商品类型图片')

    class Meta:
        db_table = 'df_goods_type'
        verbose_name = '商品种类'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name


class GoodsSKU(BaseModel):
    '''商品SKU模型类'''
    status_choices = (
        (0, '下线'),
        (1, '上线'),
    )
    type = models.ForeignKey('GoodsType',      verbose_name='商品种类',  on_delete=models.CASCADE)
    goods = models.ForeignKey('Goods',         verbose_name='商品SPU',  on_delete=models.CASCADE)
    name = models.CharField(max_length=20,     verbose_name='商品名称')
    desc = models.CharField(max_length=256,    verbose_name='商品简介')
    price = models.DecimalField(max_digits=10, decimal_places=2,        verbose_name='商品价格')
    unite = models.CharField(max_length=20,     verbose_name='商品单位')
    image = models.ImageField(upload_to='goods', verbose_name='商品图片')
    stock = models.IntegerField(default=1,       verbose_name='商品库存')
    sales = models.IntegerField(default=0,       verbose_name='商品销量')
    status = models.SmallIntegerField(default=1, choices=status_choices, verbose_name='商品状态')

    class Meta:
        db_table = 'df_goods_sku'
        verbose_name = '商品'
        verbose_name_plural = verbose_name


class Goods(BaseModel):
    '''商品SPU模型类'''
    name = models.CharField(max_length=20, verbose_name='商品SPU名称')
    # 富文本类型:带有格式的文本
    detail = HTMLField(blank=True,         verbose_name='商品详情')

    class Meta:
        db_table = 'df_goods'
        verbose_name = '商品SPU'
        verbose_name_plural = verbose_name


class GoodsImage(BaseModel):
    '''商品图片模型类'''
    sku = models.ForeignKey('GoodsSKU',          verbose_name='商品', on_delete=models.CASCADE)
    image = models.ImageField(upload_to='goods', verbose_name='图片路径')

    class Meta:
        db_table = 'df_goods_image'
        verbose_name = '商品图片'
        verbose_name_plural = verbose_name


class IndexGoodsBanner(BaseModel):
    '''首页轮播商品展示模型类'''
    sku = models.ForeignKey('GoodsSKU',           verbose_name='商品', on_delete=models.CASCADE)
    image = models.ImageField(upload_to='banner', verbose_name='图片')
    index = models.SmallIntegerField(default=0,   verbose_name='展示顺序')

    class Meta:
        db_table = 'df_index_banner'
        verbose_name = '首页轮播商品'
        verbose_name_plural = verbose_name


class IndexTypeGoodsBanner(BaseModel):
    '''首页分类商品展示模型类'''
    DISPLAY_TYPE_CHOICES = (
        (0, "标题"),
        (1, "图片"))

    type = models.ForeignKey('GoodsType', verbose_name='商品类型',  on_delete=models.CASCADE)
    sku = models.ForeignKey('GoodsSKU',   verbose_name='商品SKU',  on_delete=models.CASCADE)
    display_type = models.SmallIntegerField(default=1,
                                          choices=DISPLAY_TYPE_CHOICES, verbose_name='展示类型')
    index = models.SmallIntegerField(default=0,  verbose_name='展示顺序')

    class Meta:
        db_table = 'df_index_type_goods'
        verbose_name = "主页分类展示商品"
        verbose_name_plural = verbose_name


class IndexPromotionBanner(BaseModel):
    '''首页促销活动模型类'''
    name = models.CharField(max_length=20,         verbose_name='活动名称')
    url = models.URLField(verbose_name='活动链接')
    image = models.ImageField(upload_to='banner',  verbose_name='活动图片')
    index = models.SmallIntegerField(default=0,    verbose_name='展示顺序')

    class Meta:
        db_table = 'df_index_promotion'
        verbose_name = "主页促销活动"
        verbose_name_plural = verbose_name
```


### 1. app_order模块  
```Python
from django.db import models
from django.contrib.auth.models import AbstractUser
# 这个类原先是分离的, 现在放一起了
class BaseModel(models.Model):
    create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    update_time = models.DateTimeField(auto_now=True,     verbose_name='更新时间')
    is_delete =   models.BooleanField(default=False,      verbose_name='删除标记')
    class Mate:
        abstract = True


class OrderInfo(BaseModel):
    '''订单模型类'''
    PAY_METHOD_CHOICES = (
        (1, '货到付款'),
        (2, '微信支付'),
        (3, '支付宝'),
        (4, '银联支付'))
    ORDER_STATUS_CHOICES = (
        (1, '待支付'),
        (2, '待发货'),
        (3, '待收货'),
        (4, '待评价'),
        (5, '已完成'))
    order_id =     models.CharField(max_length=128,      verbose_name='订单id',  primary_key=True      )
    user =         models.ForeignKey('app_user.UserModel',    verbose_name='用户',    on_delete=models.CASCADE)
    addr =         models.ForeignKey('app_user.Address', verbose_name='地址',    on_delete=models.CASCADE)
    pay_method =   models.SmallIntegerField(choices=PAY_METHOD_CHOICES, default=3, verbose_name='支付方式')
    total_count =  models.IntegerField(default=1,       verbose_name='商品数量')
    total_price =  models.DecimalField(max_digits=10,   decimal_places=2, verbose_name='商品总价')
    transit_price =models.DecimalField(max_digits=10,   decimal_places=2, verbose_name='订单运费')
    order_status = models.SmallIntegerField(choices=ORDER_STATUS_CHOICES, default=1, verbose_name='订单状态')
    trade_no =     models.CharField(max_length=128,     verbose_name='支付编号')

    class Meta:
        db_table = 'df_order_info'
        verbose_name = '订单'
        verbose_name_plural = verbose_name


class OrderGoods(BaseModel):
    '''订单商品模型类'''
    order = models.ForeignKey('OrderInfo',      verbose_name='订单',         on_delete=models.CASCADE)
    sku =   models.ForeignKey('app_goods.GoodsSKU', verbose_name='商品SKU',  on_delete=models.CASCADE)
    count = models.IntegerField(default=1,      verbose_name='商品数目')
    price = models.DecimalField(max_digits=10,  verbose_name='商品价格',  decimal_places=2)
    comment = models.CharField(max_length=256,  verbose_name='评论')
    class Meta:
        db_table = 'df_order_goods'
        verbose_name = '订单商品'
        verbose_name_plural = verbose_name
```  

### 1. app_user模块    
```Python

from django.db import models
from django.contrib.auth.models import AbstractUser


# 这个类原先是分离在db文件包的, 现我把它们放一起了
class BaseModel(models.Model):
    create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    update_time = models.DateTimeField(auto_now=True,     verbose_name='更新时间')
    is_delete = models.BooleanField(default=False,        verbose_name='删除标记')
    class Mate:
        abstract = True


class UserModel(AbstractUser, BaseModel):
    '''用户模型类'''
    user_name = models.CharField(max_length=20, default='user_name')
    class Meta:
        db_table = 'df_user'
        verbose_name = '用户'
        verbose_name_plural = verbose_name


class Address(BaseModel):
    '''地址模型类'''
    user =       models.ForeignKey('UserModel',     verbose_name='所属账户', on_delete=models.CASCADE)
    receiver =   models.CharField(max_length=20,    verbose_name='收件人')
    addr =       models.CharField(max_length=256,   verbose_name='收件地址')
    zip_code =   models.CharField(max_length=6,     verbose_name='邮政编码', null=True)
    phone =      models.CharField(max_length=11,    verbose_name='联系电话')
    is_default = models.BooleanField(default=False, verbose_name='是否默认')

    class Meta:
        db_table = 'df_address'
        verbose_name = '地址'
        verbose_name_plural = verbose_name
```


## 注  意
Django的坑点那是相当的多, 大家有问题最好能够看错误提示多思考思考, 然后再试着谷歌百度看看  
![mysql-5](https://github.com/KissMyLady/Django/blob/master/Img/mysql-5.jpg)  

这里使用的是`Django2.2.1`版本,  上面的代码的BUG我已经踩平了, 直接复制粘贴即可  


## 运行起来应该是这样  
![mysql-6](https://github.com/KissMyLady/Django/blob/master/Img/mysql-6.jpg)  










