# Django
## 目录 
* [安装Django](#安装Django)
* [创建目录](#创建目录)
	* 初始化
	* 目录结构
	* [目录说明](#目录说明)
* [简易服务器](#简易服务器)
* [创建应用](#创建应用)
	* 项目和应用的区别
	* 创建应用
	* 应用目录结构
	* [编写第一个视图](#编写第一个视图)
* [数据库配置](#数据库配置)
	* INSTALLED_APPS默认应用
* [创建模型](#创建模型)
    * 改变模型步骤
* [尝试使用API](#尝试使用API)
    * database api
* [管理页面](#管理页面)
    * [创建管理员账号](#创建管理员账号)
    * [启动开发服务器](#启动开发服务器)
    * [向管理页面中加入投票应用](#向管理页面中加入投票应用)
* [视图](#视图)

### 安装Django
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

### 创建项目
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

#### 目录说明：
* mysite：项目的容器，Django不关心它的名字，可以随意重命名成你想要的名字
* manage.py：一个让可以用各种方式管理Django项目的命令行工具，你可以阅读[django-admin and manage.py](https://docs.djangoproject.com/zh-hans/2.1/ref/django-admin/)
* mysite/目录包含你的项目，他是一个纯Python包。 他的名字就是当你引用它内部任何东西时需要用到的Python包名。（比如mysite.urls)
* mysite/\_\_init\_\_.py: 一个空文件，告诉Python这个目录应该被认为是一个Python包。
* mysite/settings.py：Django项目的配置文件，如果你想知道这个文件是如何工作的，请查看[Django setting](https://docs.djangoproject.com/zh-hans/2.0/topics/settings/)了解细节。
* mysite/urls.py：Django项目的URL声明，阅读[URL调度器](https://docs.djangoproject.com/zh-hans/2.0/topics/http/urls/)文档来获取更多关于 URL 的内容。
* mysite/wsgi.py：作为你的项目运行在WSGI兼容的Web服务器上的入口。阅读[如何使用WSGI进行部署](https://docs.djangoproject.com/zh-hans/2.0/howto/deployment/wsgi/)了解更多。

### 简易服务器
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

### 创建应用
在Django中，每一个应用都是一个Python包，并且遵循着相同的约定。Django自带一个工具，可以帮你生成应用的基础目录结构，这样你就能专心写代码，而不是创建目录了。

> 项目 VS 应用

> 项目和应用有啥区别？应用是一个专门做某件事的网络应用程序——比如博客系统，或者公共记录的数据库，或者简单的投票程序。项目则是一个网站使用的配置和应用的集合。项目可以包含很多个应用。应用可以被很多个项目使用

你的应用可以存放在任何Python path中定义的路径。在这个教程中，我们将在你的manage.py同级目录下创建应用，这样它就可以作为顶级模块导入，而不是mysite的子模块

确保你现在处于manage.py所在的目录下，然后运行以下命令来创建一个应用：

* 创建应用

`python manage.py startapp polls`

* 应用目录结构
```
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

为了创建URLconf，请在polls目录里建一个urls.py的文件，输入以下代码：

`vim polls/urls.py`
```
from django.urls import path

from . import views

urlpatterns = [
	path('', views.index, name='index'),
]
```

下一步要在根URLconf文件中指定我们创建的polls.urls模块，在mysite/urls.py文件的urlpatterns列表中插入一个include()， 如下

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

----

**path()参数：**
> 函数`path()`具有四个参数，两个必须参数：`route`和`view`，两个可选参数：`kwargs`和`name`

**route：**

route 是一个匹配URL的准则（类似正则表达式），当Django相应一个请求时，它会从urlpatterns的第一项开始，按顺序依次匹配列表中的项，至到找到匹配的项。
这些准则不会匹配`GET`和`POST`参数或域名

例如，URLconf在处理请求`http://www.example.com/myapp/`时，它会尝试匹配`myapp/`，处理请求`http://www.example.com/myapp/?page=3`时，也只会尝试匹配`myapp/`


**view：**
当Django找到了一个匹配的准则，就会调用这个特定的视图函数，并传入一个`HttpRequest`对象作为第一个参数，被“捕获”的参数以关键字参数的形式传入。

**kwargs:**
任意个关键字参数可以作为一个字典传递给目标视图函数。本教程中不会使用这一特性。

**name：**
为你的 URL 取名能使你在 Django 的任意地方唯一地引用它，尤其是在模板中。这个有用的特性允许你只改一个文件就能全局地修改某个 URL 模式。


### 数据库配置
mysite/settings.py 这是包含了Django项目设置的Python模块
通常，这个配置文件使用SQLite作为默认数据库，如果你不熟悉数据库，或者只是想尝试下Django，这是最简单的选择，Python内置SQLite， 所以你无需安装额外的东西来使用它，当你开始一个真正的项目时，你可能更倾向使用一个更具有扩展性的数据库，例如PostgreSQL，避免中途切换数据库这个令人头疼的问题。

如果你想使用其他数据库，你需要安装合适的[database bindings](https://docs.djangoproject.com/zh-hans/2.0/topics/install/#database-installation)，然后改变设置文件中DATABASE 'default'项目中的一些键值：

* ENGINE：可选值有：
`'django.db.backends.sqlite3'`
`'django.db.backends.postgresql'`
`'django.db.backends.mysql'`
`'django.db.backends.oracle'`
等等

* NAME：数据库的名称，如果使用的是SQLite，数据库将是你电脑上的一个文件，在这种情况下，NAME应该是此文件的绝对路径，包括文件名，默认值：`os.path.join(BASE_DIDR, 'db.sqlite3')`将会把数据库文件存储在项目的根目录。

* 如果不使用SQLite，则必须添加一些额外设置，比如USER、PASSWORD、HOST等等，想了解更多数据库方面的内容，请看文档： [DATABASE](https://docs.djangoproject.com/zh-hans/2.0/ref/settings/#std:setting-DATABASES)

**编辑 mysite/settings.py 文件前，先设置[TIME_ZONE](https://docs.djangoproject.com/zh-hans/2.0/ref/settings/#std:setting-TIME_ZONE)为你自己时区。**

* 以下示例MySQL的配置：
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypass',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}


# 创建数据库
> CREATE DATABASE mydb;
> GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'HOST_IP' IDENTIFIED BY 'mypass';
> FLUSH PRIVILEGES;
```
**注: 如果使用MySQL作为后端数据，则需要安装DB API驱动程序mysqlclient，可使用pip进行安装`pip install mysqlclient`**


> 此外，关注一下文件头部的[INSTALLED_APPS](https://docs.djangoproject.com/zh-hans/2.0/ref/settings/#std:setting-INSTALLED_APPS)设置项。这里包括了会在你项目中启用的所有Django应用。应用能在多个项目中使用，你也可以打包并且发布应用，让别人使用它
> 通常，[INSTALLED_APPS](https://docs.djangoproject.com/zh-hans/2.0/ref/settings/#std:setting-INSTALLED_APPS)默认包括了以下Django的自带应用：
* [django.contrib.admin](https://docs.djangoproject.com/zh-hans/2.0/ref/contrib/admin/#module-django.contrib.admin) -- 管理员站点
* [django.contrib.auth](https://docs.djangoproject.com/zh-hans/2.0/topics/auth/#module-django.contrib.auth) -- 认证授权系统
* [django.contrib.contenttypes](https://docs.djangoproject.com/zh-hans/2.0/ref/contrib/contenttypes/#module-django.contrib.contenttypes) -- 内容类型框架
* [django.contrib.sessions](https://docs.djangoproject.com/zh-hans/2.0/topics/http/sessions/#module-django.contrib.sessions) -- 会话框架
* [django.contrib.messages](https://docs.djangoproject.com/zh-hans/2.0/ref/contrib/messages/#module-django.contrib.messages) -- 消息框架
* [django.contrib.staticfiles](https://docs.djangoproject.com/zh-hans/2.0/ref/contrib/staticfiles/#module-django.contrib.staticfiles) -- 管理静态文件的框架
这些应用被默认启用是为了给常规项目提供方便

------

默认开启的某些应用需要至少一个数据表，所以，在使用他们之前需要在数据库中创建一些表，请执行以下命令：

`python manage.py migrate`

> 这个migrate命令检查INSTALLED_APPS这只，为其中的每个应用创建需要的数据表。


### 创建模型
在Django里写一个数据库驱动的Web应用的第一步是定义模型-也就是数据库结构设计和附加的其他元数据。

> 模型是真实数据的简单明确的描述。它包含了储存的数据所必要的字段和行为。Django 遵循 DRY Principle 。它的目标是你只需要定义数据模型，然后其它的杂七杂八代码你都不用关心，它们会自动从模型生成。
> 来介绍一下迁移 - 举个例子，不像 Ruby On Rails，Django 的迁移代码是由你的模型文件自动生成的，它本质上只是个历史记录，Django 可以用它来进行数据库的滚动更新，通过这种方式使其能够和当前的模型匹配。

再这个简单的投票应用中，需要创建两个模型： 问题**Question**和选项**Choice**，**Question**模型包括问题描述和发布时间，**Choice**模型有两个字段，选项描述和当前投票数，选项描述和当前得票数，每个选项属于一个问题

这些概念可以通过一个简单的Python类来描述，按照下面的例子来编辑**pools/models.py**文件：

`# vim polls/models.py`

```


from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

> 代码非常直白。每个模型被表示为 django.db.models.Model 类的子类。每个模型有一些类变量，它们都表示模型里的一个数据库字段。
> 每个字段都是 Field 类的实例 - 比如，字符字段被表示为 CharField ，日期时间字段被表示为 DateTimeField 。这将告诉 Django 每个字段要处理的数据类型。
> 每个 Field 类实例变量的名字（例如 question_text 或 pub_date ）也是字段名，所以最好使用对机器友好的格式。你将会在 Python 代码里使用它们，而数据库会将它们作为列名。
> 你可以使用可选的选项来为 Field 定义一个人类可读的名字。这个功能在很多 Django 内部组成部分中都被使用了，而且作为文档的一部分。如果某个字段没有提供此名称，Django 将会使用对机器友好的名称，也就是变量名。在上面的例子中，我们只为 Question.pub_date 定义了对人类友好的名字。对于模型内的其它字段，它们的机器友好名也会被作为人类友好名使用。
> 定义某些 Field 类实例需要参数。例如 CharField 需要一个 max_length 参数。这个参数的用处不止于用来定义数据库结构，也用于验证数据，我们稍后将会看到这方面的内容。
> Field 也能够接收多个可选参数；在上面的例子中：我们将 votes 的 default 也就是默认值，设为0。
> 注意在最后，我们使用 ForeignKey 定义了一个关系。这将告诉 Django，每个 Choice 对象都关联到一个 Question 对象。Django 支持所有常用的数据库关系：多对一、多对多和一对一。

### 激活模型
上面一小段用于创建模型的代码给了Django很多信息，通过这些信息，Django可以:
* 为这个应用创建数据库scheme(生成CREATE TABLE语句)
* 创建可以与**Question**和**Choice**对象进行交互的Python数据库API
但是首先得把polls应用安装到我们的项目里。

> Django应用是**"可插拔的"** 的，你可以在多个项目中使用同一个应用，除此之外，你还可以发布自己的应用，因为它们并不会被绑定到当前安装的Django上。

为了在我们的工程中包含这个应用，我们需要在配置类 INSTALLED_APPS 中添加设置。因为 PollsConfig 类写在文件 polls/apps.py 中，所以它的点式路径是 'polls.apps.PollsConfig'。在文件 mysite/settings.py 中 INSTALLED_APPS 子项添加点式路径后，它看起来像这样：

`# vim mysite/settings.py`

```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
现在你的Django项目包含**polls**应用，运行以下命令：

`# python manage.py makemigrations polls`

你将会看到这样的输出
```
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
```
命令**makegrations** 让Django确定该如何修改数据库，使其能够存储于我们定义的新模型相关联的数据。输出表明Django创建了一个名为0001_initial.py的迁移文件，

迁移是Django对于模型定义(也就是你的数据库结构)的变化的存储形式，它们其实也只是磁盘上的一些文件，你也可以阅读一下模型的迁移数据，它被存储在polls/migrations/0001_initial.py文件里，你并不需要每次都去阅读迁移文件，但迁移文件被设计成可读的形式，这是为了方便你去手动修改他们

Django有一个自动执行数据库迁移并同步管理你的数据库结构的命令，这个命令是[migrate](https://docs.djangoproject.com/zh-hans/2.0/ref/django-admin/#django-admin-migrate)

可以执行以下命令来查看在迁移过程中会执行那些SQL语句，[sqlmigrate](https://docs.djangoproject.com/zh-hans/2.0/ref/django-admin/#django-admin-sqlmigrate)命令接受一个迁移的名称呢，然后返回对应的SQL:

`# python manage.py sqlmigrate polls 0001`

你将会看到类似下面的输出：
```
BEGIN;
--
-- Create model Choice
--
CREATE TABLE `polls_choice` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `choice_text` varchar(200) NOT NULL, `votes` integer NOT NULL);
--
-- Create model Question
--
CREATE TABLE `polls_question` (`id` integer AUTO_INCREMENT NOT NULL PRIMARY KEY, `question_text` varchar(200) NOT NULL, `pub_date` datetime NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE `polls_choice` ADD COLUMN `question_id` integer NOT NULL;
ALTER TABLE `polls_choice` ADD CONSTRAINT `polls_choice_question_id_c5b4b260_fk_polls_question_id` FOREIGN KEY (`question_id`) REFERENCES `polls_question` (`id`);
COMMIT;
```

**请注意以下几点**
* 输出的内容和你使用的数据库有关，上面的输出示例是用的是MySQL
* 数据库表明是由应用名(polls)和模型名的小写形式(question和choice)连接而来。(如果需要，你可以自定义此行为)
* 主键(IDs)会被自动创建(当然，也可以自定义)
* 默认的，Django会在外键字段名后追加字符串"\_id"(同样，也可以自定义)
* 外键关系由FOREIGN KEY生成。
* 生成的SQL语句是为你所用的数据库定制的，所以那些和数据库有关的字段类型，比如auto_increment(MySQL)、serial(PostgreSQL)和integer primary key autoincrement(SQLite)， Django会帮你自动处理。
* 这个**sqlmigrations**命令并没有真正在你的数据库中执行迁移，它只是把命令输出到屏幕上，让你看看Django认为需要执行那些SQL语句

运行[migrate](https://docs.djangoproject.com/zh-hans/2.0/ref/django-admin/#django-admin-migrate)命令，在数据库中创建定义的模型的数据表：

`# python manage.py migrate`

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```

这个[migrate](https://docs.djangoproject.com/zh-hans/2.0/ref/django-admin/#django-admin-migrate)命令选中所有还没有执行过的迁移(Django通过在数据库中创建一个特殊的表**django_migrations**来跟踪执行过那些迁移)并应用在数据库上，也就是将你对模型的更改同步到数据库结构上

迁移是非常强大的功能，它能让你在开发过程中持续的改变数据库结构而不需要重新删除和创建表，它专注于使数据库平滑升级而不丢失数据
* 改变模型步骤：
    1. 编辑**models.py**文件，改变模型
    2. 运行`python manage.py makemigrations`为模型的改变生成迁移文件
    3. 运行`python manage.py migrate`来应用数据库迁移

数据库迁移被分解生成和应用两个命令是为了让你能够在代码控制系统上提交迁移数据并使其能在多个应用里使用；这不仅仅会让开发更加简单，也给别的开发者和生产环境中的使用带来方便。

通过阅读[Django后台文档](https://docs.djangoproject.com/zh-hans/2.0/ref/django-admin/)，你可以获取关于manage.py工具的更多信息

### 尝试使用API
进入交互式Python命令行，尝试以下Django为你创建的各种API，通过以下命令打开Python命令行：

`python manage.py shell`

我们使用这个命令而不是简单的使用python，是因为`manage.py`会设置`DJANGO_SETTINGS_MODULE`环境变量，这个变量会让Django根据`mysite/settings.py`文件来设置Python包的导入路径

当你成功进入命令行后，来试试[database API](https://docs.djangoproject.com/zh-hans/2.0/topics/db/queries/)吧:
#### database api
```
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

等等。**<Question: Question object (1)>** 对于我们了解这个对象的细节没什么帮助。让我们通过编辑 Question 模型的代码（位于**polls/models.py**中）来修复这个问题。给 **Question** 和 **Choice** 增加[__str__()](https://docs.djangoproject.com/zh-hans/2.0/ref/models/instances/#django.db.models.Model.__str__)方法。

`# vim polls/models.py`
```
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

给模型增加[__str__()](https://docs.djangoproject.com/zh-hans/2.0/ref/models/instances/#django.db.models.Model.__str__)方法是很重要的，这不仅仅能给你在命令行里使用带来方便，Django 自动生成的 admin 里也使用这个方法来表示对象。

注意：这些都是常规的 Python方法。让我们添加一个自定义的方法，这只是为了演示：
`vim polls/models.py`
```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

新加入的 import datetime 和 from django.utils import timezone 分别导入了 Python 的标准[datetime](https://docs.python.org/3/library/datetime.html#module-datetime)模块和 Django 中和时区相关的[django.utils.timezone](https://docs.djangoproject.com/zh-hans/2.0/ref/utils/#module-django.utils.timezone)工具模块。如果你不太熟悉 Python 中的时区处理，看看[时区支持文档](https://docs.djangoproject.com/zh-hans/2.0/topics/i18n/timezones/)吧。

保存文件然后通过 python manage.py shell 命令再次打开 Python 交互式命令行：
```
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

阅读[访问关系对象](https://docs.djangoproject.com/zh-hans/2.0/ref/models/relations/)文档可以获取关于数据库关系的更多内容。想知道关于双下划线的更多用法，参见[查找字段](https://docs.djangoproject.com/zh-hans/2.0/topics/db/queries/#field-lookups-intro)文档。数据库 API 的所有细节可以在[数据库 API](https://docs.djangoproject.com/zh-hans/2.0/topics/db/queries/)参考 文档中找到。

### 管理页面
#### 创建管理员账号
首先，我们得创建一个能登录管理页面的用户，请运行以下命令

`python manage.py createsuperuser`

输入你想要使用的用户名：

`Username: admin`

然后输入你想要使用的邮件地址：

`Email address: admin@localhost.com`

最后一步输入密码：

```
Password: *********
Password (again): ********
Superuser created successfully
```

#### 启动开发服务器
Django的管理界面默认是启用的，运行以下命令启动开发服务器

`# python manage.py runserver`

然后在浏览器上输入`http://Host_IP:8000/admin/`你就能看到管理员登陆界面
![avatar](https://github.com/Twoling/study/blob/master/Django_admin.png)

输入刚刚创建的用户名和密码，进入管理员页面
![avatar](https://github.com/Twoling/study/blob/master/Django_in_admin.png)

#### 向管理页面中加入应用
告诉管理页面，**Question**对象需要被管理，打开**polls/admin.py**文件，按照以下内容编译

`# vim polls/admin.py`

```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
现在我们想管理页面注册了**Question**类，Django知道它应该被显示在索引页中：
![avatar](https://github.com/Twoling/study/blob/master/Django_Question.png)

点击**Question**，将看到**Question**对象的"change list"，这个界面会显示**Question**对象在数据库中所有的数据，你可以选择一个来修改，这里只有一个我们创建的 **What's up?** 问题
![avatar](https://github.com/Twoling/study/blob/master/Django_what.png)

点击 **What's up?** 来编辑这个问题(Question)对象：
![avatar](https://github.com/Twoling/study/blob/master/Django_change_question.png)

**注意事项：**
* 这个表单是从Question模型中自动生成的
* 不同的字段类型(日期时间字段[DateTimeField](https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/#django.db.models.DateTimeField)、[字符字段](https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/#django.db.models.CharField))会生成对应的HTML输入控件，每个类型的字段都知道他们该如何在管理页面里显示自己
* 每个日期时间字段[DateTimeField](https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/#django.db.models.DateTimeField)都有JavaScript写的快捷按钮，日期又转到今天(Today)的快捷按钮和一个弹出式日历界面，时间有设为现在(Now)的快捷按钮和一个列出常用时间的方便的弹出式列表

页面的底部提供了几个选项：
* Sava 保存更改，然后返回对象列表
* Sava and continue editing 保存更改，然后重新载入当前对象的修改页面
* Sava and add another 保存更改，然后添加一个新的空对象并载入修改界面
* Delete 显示一个确认删除页面

### 视图
