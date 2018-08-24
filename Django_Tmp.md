## Django
#### 安装Django
1. 检测是否已安装django，在shell中输入python命令，进入python交互界面，尝试导入django模块
```
>>> import django
>>> print(django.get_version())
2.0.7
```

2. 如果导入报错，则表示当前环境并没有安装Django，可以使用pip程序来安装
```
# 如果你的python3是通过源码编译安装，则会自动安装pip，如果是通过yum安装的python，请向安装pip
pip3 install django

# pip安装教程：
1. yum -y install python34-pip

2. wget https://files.pythonhosted.org/packages/69/81/52b68d0a4de760a2f1979b0931ba7889202f302072cc7a0d614211bc7579/pip-18.0.tar.gz
   tar xf pip-18.0.tar.gz
   cd pip-18.0 
   python3 setup.py install

 
```

#### 创建项目
> 初始化设置，用一些自动生成的代码配置一个Django project，即一个Django项目实例需要的设置项集合，包括数据库配置、Django配置和应用程序配置

```
# 在shell命令行运行以下命令：
django-admin startproject mysite
# 这行代码将会在当前目录创建一个mysite的目录
```

* 目录结构：

```
mysite/
├── manage.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py

```

###### 目录说明：
* mysite：项目的容器，Django不关心它的名字，可以随意重命名成你想要的名字
* manage.py：一个让可以用各种方式管理Django项目的命令行工具，你可以阅读[django-admin and manage.py](https://docs.djangoproject.com/zh-hans/2.1/ref/django-admin/)
* mysite/目录包含你的项目，他是一个纯Python包。 他的名字就是当你引用它内部任何东西时需要用到的Python包名。（比如mysite.urls)
* mysite/\__init__.py: 一个空文件，告诉Python这个目录应该被认为是一个Python包。
* mysite/settings.py：Django项目的配置文件，如果你想知道这个文件是如何工作的，请查看[Django setting](https://docs.djangoproject.com/zh-hans/2.0/topics/settings/)了解细节。
* mysite/urls.py：Django项目的URL声明，阅读[URL调度器](https://docs.djangoproject.com/zh-hans/2.0/topics/http/urls/)文档来获取更多关于 URL 的内容。
* mysite/wsgi.py：作为你的项目运行在WSGI兼容的Web服务器上的入口。阅读[如何使用WSGI进行部署](https://docs.djangoproject.com/zh-hans/2.0/howto/deployment/wsgi/)了解更多。

##### 简易服务器
让我们来确认一下你的 Django 项目是否真的创建成功了。如果你的当前目录不是外层的 mysite 目录的话，请切换到此目录，然后运行下面的命令：

```
python manage.py runserver
# 运行地址的制定端口请在运行时将地址和端口跟在runserver后面
# 例：
python manage.py runserver 172.18.0.1:8080

# 监听所有地址也可以使用以下格式：
python manage.py runserver 0:8080

# 默认Django只允许本地访问，如需远程访问，请设置Django的配置文件中`ALLOWED_HOSTS`变量的值
# 例：
vim mysite/settings.py
....
ALLOWED_HOSTS = [ "*" ]
....
```

* 运行时的提示
```
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

八月 01, 2018 - 15:50:53
Django version 2.0, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

> 用于开发的服务器在需要的情况下会对每一次的访问请求重新载入一遍 Python 代码。所以你不需要为了让修改的代码生效而频繁的重新启动服务器。然而，一些动作，比如添加新文件，将不会触发自动重新加载，这时你得自己手动重启服务器。








