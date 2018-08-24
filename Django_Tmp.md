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

> 你刚刚启动的是 Django 自带的用于开发的简易服务器，它是一个用纯 Python 写的轻量级的 Web 服务器。我们将这个服务器内置在 Django 中是为了让你能快速的开发出想要的东西，因为你不需要进行配置生产级别的服务器（比如 Apache）方面的工作，除非你已经准备好投入生产环境了

> **会自动重新加载的服务器 runserver**
>用于开发的服务器在需要的情况下会对每一次的访问请求重新载入一遍 Python 代码。所以你不需要为了让修改的代码生效而频繁的重新启动服务器。然而，一些动作，比如添加新文件，将不会触发自动重新加载，这时你得自己手动重启服务器。

#### 创建应用
在Django中，每一个应用都是一个Python包，并且遵循着相同的约定。Django自带一个工具，可以帮你生成应用的基础目录结构，这样你就能专心写代码，而不是创建目录了。

> 项目 VS 应用

> 项目和应用有啥区别？应用是一个专门做某件事的网络应用程序——比如博客系统，或者公共记录的数据库，或者简单的投票程序。项目则是一个网站使用的配置和应用的集合。项目可以包含很多个应用。应用可以被很多个项目使用

你的应用可以存放在任何Python path中定义的路径。在这个教程中，我们将在你的manage.py同级目录下创建应用，这样它就可以作为顶级模块导入，而不是mysite的子模块

确保你现在处于manage.py所在的目录下，然后运行以下米宁来创建一个应用：

```
python manage.py startapp polls
# 这将会创建一个polls目录，它的目录结构大致如下：
polls/
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py

```

#### 编写第一个视图
`vim polls/views.py`

```
from django.http import HttpResponse

def index(request):
	return HttpResponse("Hello Django. Hello Polls")
```
这是Django中最简单的视图，如果想看见效果，我们需要将一个URL映射到它--这就是我们需要URLconf的原因。

为了创建URLconf，请在polls目录里建一个urls.py的文件。
`vim polls/urls.py`
输入以下代码：
```
from django.urls import path

from . import views

urlpatterns = [
	path('', views.index, name='index'),
]
```

**下一步要在根URLconf文件中指定我们创建的polls.urls模块，在mysite/urls.py文件的urlpatterns列表中插入一个include()， 如下**

`vim mysite/urls.py`
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
	path('polls/', include('polls.urls')),
	path('admin/', admin.site.urls),
]
```

函数`include()`允许引用其它URLconfs，每当Django遇到`:func:~djanago.urls.include`时，它会截断与此匹配的URL的部分，并将剩余的字符串发送到URLconf以供进一步处理。

> 我们设计 include() 的理念是使其可以即插即用。因为投票应用有它自己的 URLconf( polls/urls.py )，他们能够被放在 "/polls/" ， "/fun_polls/" ，"/content/polls/"，或者其他任何路径下，这个应用都能够正常工作。

把index视图添加进了URLconf，可以验证是否正常工作，运行以下命令：
`python manage.py runserver`

用浏览器访问`http://ip:8000/polls`， 你应该能够看到'Hello Django. Hello Polls',这是你在index是视图定义的。

> 函数`path()`具有四个参数，两个必须参数：`route`和`view`，两个可选参数：`kwargs`和`name`

> path()参数：`route`

route 是一个匹配URL的准则（类似正则表达式），当Django相应一个请求时，它会从urlpatterns的第一项开始，按顺序依次匹配列表中的项，至到找到匹配的项。
这些准则不会匹配`GET`和`POST`参数或域名，例如，URLconf在处理请求`http://www.example.com/myapp/`时，它会尝试匹配`myapp/`，处理请求`http://www.example.com/myapp/?page=3`时，也只会尝试匹配`myapp/`


