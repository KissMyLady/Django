Django初始化
====


## 预览图
登录自建网页效果
![123-1](https://github.com/KissMyLady/Django/blob/master/Img/123-1.jpg)  


## 说 明  
> Django没有你想象的那么难  
> 在最后面，我会把源码贴出来，复制粘贴就能用  
> 这章节的思想：对于编程来说，最重要的还是先把路打通，然后才是添砖加瓦  
> 文中大写字母: 这是名字, 根据需求自己起名  
# Linux版本   
## 小目标1--完成Django环境搭建
 
### 1、安装虚拟环境
```Liunx
sudo pip install virtualenv  
```
### 2、安装虚拟环境扩展包
```Liunx
sudo pip install virtualenvwrapper
```
### 3、退出到桌面，然后cd ~  
sudo vim .bashrc, 添加下面两句  
```Linux
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh
```
注意, 在服务器ubuntu版本中, 需要制定python版本, 而且路径不一样, 需要关注     
```Python
# https://www.cnblogs.com/linga/p/11588223.html
125 export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
126 export WORKON_HOME=$HOME/.virtualenvs
127 export VIRTUALENVWRAPPER_VIRTUALENV=~/.local/bin/virtualenv
128 source ~/.local/bin/virtualenvwrapper.sh
```
使用source .bashrc命令刷新生效  

### 4、 创建Python虚拟环境  
```Linux
mkvirtualenv -p python3 YOU_NAME
```

### 5、创建完成  
* 小命令:   
  * 退出: de activate  
  * 进入: activate  
  * 使用: workon YOU_NAME  
  * 查看: workon + Tbb x2  
  * 删除: rm virtualenv PACKAGE_NAME  

* 包操作：
在虚拟环境中，使用pip命令下载包  
  * 安装: pip install PACK_NAME
  * 查包: pip list/ freeze

## 6、安装Django  
进入虚拟Python环境，workon NAME  
载入Django:  pip install django  (默认下载最新)  
安装成功会有提示  


## 小目标2--Django项目载入  
### 1、创建Django项目  
```Linux
django-admin startproject todat_new  
```
### 2、创建应用  
```Linux
python manage.py startapp booktest  
```

### 3、修改下setting.py文件  
在INSTALLED_APPS列表中, 加入 'booktest', (注册app项目)  

### 4、启动Django  
```Linux
python manger.oy runserver
```
默认IP: 127.0.0.1, 端口8000  

### 5、观看成果   
这时敲入网址, 就可以直接在浏览器上看到效果, 辛苦了, 给自己鼓鼓掌!    
在浏览器输入网址:  
127.0.0.1:8000  
![123-2](https://github.com/KissMyLady/Django/blob/master/Img/123-2.jpg)   

## 看完了  
- [返回Django主页](https://github.com/KissMyLady/Django)  
- [Django-M模块介绍](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating2.md)   
- [Django-admin管理](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating3.md)
- [Django-视图, 路由, 视图](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating4.md)  
- [Django-完整流程](https://github.com/KissMyLady/Django/blob/master/Note/django_base_operating5.md)
