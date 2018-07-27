### JumpServer安装
#### 环境设置：
1. 关闭SELinux和防火墙

```
setenforce 0
sed -i 's/^SELINUX=enforcing=/SELINUX=disabled/g'
```
2. 修改系统字符集：

```
localedef -c -f UTF-8 -i zh_CN zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8
echo 'LANG="zh_CN.UTF-8"' > /etc/locale.conf
```

#### 依赖安装：
```
yum -y install wget sqlite-devel xz gcc \
            automake zlib-devel \
            openssl-devel epel-release git \
            libtiff-devel libjpeg-devel \
            libzip-devel freetype-devel \
            lcms2-devel libwebp-devel \
            tcl-devel tk-devel sshpass \
            openldap-devel mysql-devel \
            libffi-devel openssh-clients
```

#### 准备Python3和Python虚拟环境
1. 编译python3

```
cd /opt
wget https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tar.xz
tar xf Python-3.6.6.tar.xz &&  cd Python-3.6.6
./configure --prefix=/opt/python36 && make && make install

# 环境设置
echo 'PATH=/opt/python36/bin:$PATH'  >>  ~/.bash_profile
source ~/.bash_profile
ln -sv /opt/python36/include/python3.6m /usr/include/python3.6m
```

2. 建立python虚拟环境

```
cd /opt/
python3 -m venv py3
source /opt/py3/bin/activate

# 看到下面的提示符代表成功，以后运行 Jumpserver 都要先运行以上 source 命令，以下所有命令均在该虚拟环境中运行
(py3) [root@localhost py3]
```

### 安装Jumpserver
1. clone项目

```
# 直接clone项目比较大，可以下载master分支的tar包来安装
cd /opt/
wget https://github.com/jumpserver/jumpserver/archive/1.3.3.tar.gz
tar xf 1.3.3.tar.gz
mv jumpserver-1.3.3 jumpserver
cd /opt/jumpserver
```

2. 安装依赖

```
cd /opt/jumpserver/requirements/
yum -y install $(cat rpm_requirements.txt)

-----------------------------------
pip加速：
配置方法
在文件
~/.pip/pip.conf（如没有自行创建）
中添加或修改:
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
-----------------------------------
# 安装python模块
pip3 install -r requirements.txt
```

3. Redis安装  Jumpserver 使用 Redis 做 cache 和 celery broke

```
cd /opt/
wget http://download.redis.io/releases/redis-4.0.10.tar.gz
tar xf redis-4.0.10
make && cd ../ && mv redis-4.0.10  redis
cd redis
mv redis.conf redis.conf.default
cp /opt/jumpserver/utils/redis.conf ./

# centos7 创建systemd启动文件
vim /usr/lib/systemd/system/redis.conf
------------------------------------------------------------------
[Unit]
Description=Redis Server
After=network.target sshd-keygen.service

[Service]
Type=simple
ExecStart=/opt/redis/src/redis-server /opt/redis/redis.conf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
------------------------------------------------------------------
systemctl start redis
systemctl enable redis
```

4. 安装MySQL数据库

```
yum -y install mariadb mariadb-devel maraidb-server
systemctl start maraidb
systemctl enable maraidb
# 运行mysql_secure_installation初始化数据库
# 创建jumpserver数据库并授权
mysql -uroot -p
> CREATE DATABASE jumpserver DEFAULT CHARSET 'utf8';
> GRANT ALL ON jumpserver.* TO 'jumpserver'@'127.0.0.1' IDENTIFIED BY 'jumppass';
> FLUSH PRIVILEGES;
```

5. 修改jumpserver配置并运行

```
cd /opt/jumpserver/
cp config_example.py config.py
vim config.py

---------------------------------------------------------------------------------------------------------------------

"""
    jumpserver.config
    ~~~~~~~~~~~~~~~~~

    Jumpserver project setting file

    :copyright: (c) 2014-2017 by Jumpserver Team
    :license: GPL v2, see LICENSE for more details.
"""
import os

BASE_DIR = os.path.dirname(os.path.abspath(__file__))


class Config:
    # Use it to encrypt or decrypt data
    # SECURITY WARNING: keep the secret key used in production secret!
    SECRET_KEY = os.environ.get('SECRET_KEY') or '2vym+ky!997d5kkcc64mnz06y1mmui3lut#(^wd=%s_qj$1%x'

    # Django security setting, if your disable debug model, you should setting that
    ALLOWED_HOSTS = ['*']

    # Development env open this, when error occur display the full process track, Production disable it
    DEBUG = os.environ.get("DEBUG") or True

    # DEBUG, INFO, WARNING, ERROR, CRITICAL can set. See https://docs.djangoproject.com/en/1.10/topics/logging/
    LOG_LEVEL = os.environ.get("LOG_LEVEL") or 'DEBUG'
    LOG_DIR = os.path.join(BASE_DIR, 'logs')

    # Database setting, Support sqlite3, mysql, postgres ....
    # See https://docs.djangoproject.com/en/1.10/ref/settings/#databases

    # SQLite setting:
    # DB_ENGINE = 'sqlite3'
    # DB_NAME = os.path.join(BASE_DIR, 'data', 'db.sqlite3')

    # MySQL or postgres setting like:
    DB_ENGINE = 'mysql'
    DB_HOST = '127.0.0.1'
    DB_PORT = 3306
    DB_USER = 'jumpserver'
    DB_PASSWORD = 'jumppass'
    DB_NAME = 'jumpserver'

    # When Django start it will bind this host and port
    # ./manage.py runserver 127.0.0.1:8080
    HTTP_BIND_HOST = '0.0.0.0'
    HTTP_LISTEN_PORT = 8080

    # Use Redis as broker for celery and web socket
    REDIS_HOST = '127.0.0.1'
    REDIS_PORT = 6379
    REDIS_PASSWORD = ''
    REDIS_DB_CELERY = 3
    REDIS_DB_CACHE = 4


    def __init__(self):
        pass

    def __getattr__(self, item):
        return None


class DevelopmentConfig(Config):
    pass


class TestConfig(Config):
    pass


class ProductionConfig(Config):
    pass


# Default using Config settings, you can write if/else for different env
config = DevelopmentConfig()

----------------------------------------------------------------------------------------------------------------------

./jms start all
```


### 安装 SSH Server 和 WebSocket Server: Coco
1. 下载或clone项目

```
# 新开一个终端，别忘了 source /opt/py3/bin/activate
git clone https://github.com/jumpserver/coco.git && cd coco && git checkout master
# 无法下载可以前往github下载其zip包
```

2. 依赖安装

```
cd /opt/coco/requirements
yum -y install $(cat rpm_requirements.txt)
pip3 install -r requirements.txt

```
3. 修改配置文件并运行

```
cd /opt/coco/
cp conf_example conf.py
---------------------------------------------------------------------------------------------------------------------
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
#

import os

BASE_DIR = os.path.dirname(__file__)


class Config:
    """
    Coco config file, coco also load config from server update setting below
    """
    # 项目名称, 会用来向Jumpserver注册, 识别而已, 不能重复
    NAME = "coco"

    # Jumpserver项目的url, api请求注册会使用
    CORE_HOST = 'http://127.0.0.1:8080'

    # 启动时绑定的ip, 默认 0.0.0.0
    BIND_HOST = '0.0.0.0'

    # 监听的SSH端口号, 默认2222
    SSHD_PORT = 2222

    # 监听的HTTP/WS端口号，默认5000
    HTTPD_PORT = 5000

    # 项目使用的ACCESS KEY, 默认会注册,并保存到 ACCESS_KEY_STORE中,
    # 如果有需求, 可以写到配置文件中, 格式 access_key_id:access_key_secret
    # ACCESS_KEY = None

    # ACCESS KEY 保存的地址, 默认注册后会保存到该文件中
    # ACCESS_KEY_STORE = os.path.join(BASE_DIR, 'keys', '.access_key')

    # 加密密钥
    # SECRET_KEY = None

    # 设置日志级别 ['DEBUG', 'INFO', 'WARN', 'ERROR', 'FATAL', 'CRITICAL']
    # LOG_LEVEL = 'INFO'

    # 日志存放的目录
    # LOG_DIR = os.path.join(BASE_DIR, 'logs')

    # Session录像存放目录
    SESSION_DIR = os.path.join(BASE_DIR, 'sessions')

    # 资产显示排序方式, ['ip', 'hostname']
    # ASSET_LIST_SORT_BY = 'ip'

    # 登录是否支持密码认证
    PASSWORD_AUTH = True

    # 登录是否支持秘钥认证
    PUBLIC_KEY_AUTH = True

    # 和Jumpserver 保持心跳时间间隔
    # HEARTBEAT_INTERVAL = 5

    # Admin的名字，出问题会提示给用户
    # ADMINS = ''
    COMMAND_STORAGE = {
        "TYPE": "server"
    }
    REPLAY_STORAGE = {
        "TYPE": "server"
    }


config = Config()

---------------------------------------------------------------------------------------------------------------------
./cocod start
```

### 安装 Web Terminal 前端: Luna
```
访问（https://github.com/jumpserver/luna/releases）下载对应版本的 release 包，直接解压，不需要编译

cd /opt
wget https://github.com/jumpserver/luna/releases/download/1.3.3/luna.tar.gz
tar xvf luna.tar.gz
chown -R root:root luna
```

### 配置Nginx整合各组件
```
yum -y install nginx

编辑/etc/nginx/nginx.conf 将http{}配置段中默认的server注释掉
vim /etc/nginx/conf.d/jumpserver.conf
将以下内容添加至文件，对应的路径请按照实际情况更改
--------------------------------------------------------------------------------------------------------------------
server {
    listen 80;  # 代理端口，以后将通过此端口进行访问，不再通过8080端口

    location /luna/ {
        try_files $uri / /index.html;
        alias /opt/luna/;  # luna 路径，如果修改安装目录，此处需要修改
    }

    location /media/ {
        add_header Content-Encoding gzip;
        root /opt/jumpserver/data/;  # 录像位置，如果修改安装目录，此处需要修改
    }

    location /static/ {
        root /opt/jumpserver/data/;  # 静态资源，如果修改安装目录，此处需要修改
    }

    location /socket.io/ {
        proxy_pass       http://localhost:5000/socket.io/;  # 如果coco安装在别的服务器，请填写它的ip
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        access_log off;
    }

#    如果有windows资产，请参照官方文档安装guacamole
#    location /guacamole/ {
#        proxy_pass       http://localhost:8081/;  # 如果guacamole安装在别的服务器，请填写它的ip
#        proxy_buffering off;
#        proxy_http_version 1.1;
#        proxy_set_header Upgrade $http_upgrade;
#        proxy_set_header Connection $http_connection;
#        proxy_set_header X-Real-IP $remote_addr;
#        proxy_set_header Host $host;
#        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#        access_log off;
#        client_max_body_size 100m;  # Windows 文件上传大小限制
#    }


    location / {
        proxy_pass http://localhost:8080;  # 如果jumpserver安装在别的服务器，请填写它的ip
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
--------------------------------------------------------------------------------------------------------------------
nignx -t
如果语法没问题，启动nginx
systemctl start nginx
systemctl enable nginx
```

### 停掉各服务，以后台方式启动

##### Jumpserver：`/opt/jumpserver/jms start all -d`
##### coco：`/opt/coco/cocod start -d`
