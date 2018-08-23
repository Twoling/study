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























