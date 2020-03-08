上传文件
====
## 1. 管理后台上传文件
### 4.1 配置上传文件保存目录
1) 新建上传文件保存目录。
```         python
├── static
│   ├── css
│   │   ├── jquery-1.12.4.min.js
│   │   └── new_2.css
│   ├── images
│   │   ├── 00125.jpg
│   │   ├── 003.png
│   │   ├── 004.png
│   │   ├── 005.png
│   │   └── midd01.jpg
│   ├── js
│   │   ├── jquery-1.12.4.min.js
│   │   └── jquery.js
│   └── media
│       └── user_load
```

2) 配置上传文件保存目录。

```python
# 配置上传文件保存目录
MEDIA_ROOT = os.path.join(BASE_DIR, 'static/media')
```

### 4.2 后台管理页面上传图片
1) 设计模型类。
```python
from django.db import models
class PicImgDownload(models.Model):
    superuser_pic = models.ImageField(upload_to='user_load')
```

2) 迁移生成表格。
```python
python manage.py makemigrations
python manage.py migrate
```

3) 注册模型类。
```python
from app_bookinfo.models import PicImgDownload
admin.site.register(PicImgDownload)
```

## 2. 用户上传图片
#### 1) 定义用户上传图片的页面并显示，是一个自定义的表单。

```html
<form method="post" enctype="multipart/form-data" action="/upload_func">
    {% csrf_token %}
    <input type="file" name="pic"><br><br>
    <input type="submit" value="上传"><br>
</form>
```

#### 2) 定义接收上传文件的视图函数。
```python
def user_picload(request):
    return render(request, 'user_picload.html')


def upload_func(request):
    pic = request.FILES['pic']
    '''<class 'django.core.files.uploadedfile.InMemoryUploadedFile'>         2.5M 放在内存
       <class 'django.core.files.uploadhandler.TemporaryFileUploadHandler'>   放在临时文件中
    '''
    # pic.name 获取上传图片的名字
    # pic.chunks()
    # print('>'*10, type(pic))

    from django.conf import settings
    from app_bookinfo.models import PicImgDownload

    save_file_path = '%s/user_load/%s' % (settings.MEDIA_ROOT, pic.name)
    with open(save_file_path, 'wb') as f:
        for content in pic.chunks():
            f.write(content)

    PicImgDownload.objects.create(superuser_pic='user_load/{}'.format(pic.name))
    return HttpResponse('图片上传成功,"\r\n", <a href="/"><h3>点击进入--Index主页</h3></a> ')
```

#### MySQL表结构:   

```sql
mysql> show tables;
+---------------------------------+
| Tables_in_re_django_daily_fresh |
+---------------------------------+
| app_bookinfo_bookinfo           |
| app_bookinfo_heroinfo           |
| app_bookinfo_picimgdownload     |
| auth_group                      |
| auth_group_permissions          |
| auth_permission                 |
| auth_user                       |
| auth_user_groups                |
| auth_user_user_permissions      |
| django_admin_log                |
| django_content_type             |
| django_migrations               |
| django_session                  |
+---------------------------------+
13 rows in set (0.00 sec)

mysql> desc app_bookinfo_picimgdownload;
+---------------+--------------+------+-----+---------+----------------+
| Field         | Type         | Null | Key | Default | Extra          |
+---------------+--------------+------+-----+---------+----------------+
| id            | int(11)      | NO   | PRI | NULL    | auto_increment |
| superuser_pic | varchar(100) | NO   |     | NULL    |                |
+---------------+--------------+------+-----+---------+----------------+
2 rows in set (0.01 sec)

mysql> select * from app_bookinfo_picimgdownload;
+----+-----------------------------------------+
| id | superuser_pic                           |
+----+-----------------------------------------+
|  1 | user_load/白丝死库水1418-48261.jpg      |
|  2 | user_load/白丝死库水1418-48275.jpg      |
|  3 | user_load/白丝死库水1418-48277.jpg      |
+----+-----------------------------------------+
3 rows in set (0.00 sec)
```

**上传图片参考资料：**
1. http://python.usyiyi.cn/documents/django_182/topics/http/file-uploads.html
2. http://python.usyiyi.cn/documents/django_182/ref/files/uploads.html#django.core.files.uploadedfile.UploadedFile
