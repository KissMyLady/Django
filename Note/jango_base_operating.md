Django初阶--如何快速搭建一个网站  
====


## 预览图
1、登录admin网页效果  
![]()  
2、登录自建网页效果
![]()  
3、查看详情页网页效果  
![]()  

## 说 明  
> Django没有你想象的那么难  
> 在文章下面，我会把源码贴出来，复制粘贴就能用  
> 这篇文章的思想：对于编程来说，最重要的还是先把路打通，然后才是添砖加瓦  

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

## 3、修改下setting.py文件  
在INSTALLED_APPS列表中, 加入 'booktest', (注册app项目)  

## 4、启动Django  
```Linux
python manger.oy runserver
```
默认IP: 127.0.0.1, 端口8000  

## 5、观看成果   
这时敲入网址, 就可以直接在浏览器上看到效果, 辛苦了, 为自己鼓鼓掌!    
在浏览器输入网址:  
127.0.0.1:8000  






