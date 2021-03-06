# Kimchi安装教程
### 先安装wokd服务
1. 解决依赖：
```
yum install gcc make autoconf automake gettext-devel git rpm-build libxslt \
                    python-cherrypy python-cheetah PyPAM m2crypto \
                    python-jsonschema python-psutil python-ldap \
                    python-lxml nginx openssl python-websockify \
                    fontawesome-fonts logrotate  python-ordereddict
```

2. 下载rpm包：
```
     wget https://github.com/kimchi-project/wok/releases/download/2.5.0/wok-2.5.0-0.el7.centos.noarch.rpm
```

3. 安装wokd
```
     rpm -ivh wok-2.5.0-0.el7.centos.noarch.rpm
```

4. 启动wokd服务
```
     systemctl daemon-reload
     systemctl start wokd
```


### 安装kimchi
1. 解决依赖：
```
yum install gcc make autoconf automake gettext-devel git rpm-build libxslt \
                    libvirt-python libvirt libvirt-daemon-config-network \
                    qemu-kvm python-ethtool sos python-ipaddr nfs-utils \
                    iscsi-initiator-utils pyparted python-libguestfs \
                    libguestfs-tools novnc spice-html5 \
                    python-configobj python-magic python-paramiko \
                    python-pillow python-ordereddict
```

#### 重启libvirtd服务：
```
     systemctl restart libvirtd 
```

2. 下载rpm包
```
     wget https://github.com/kimchi-project/kimchi/releases/download/2.5.0/kimchi-2.5.0-0.el7.centos.noarch.rpm
```

3. 安装kimchi
```
     rpm -ivh  kimchi-2.5.0-0.el7.centos.noarch.rpm
```


### 启动服务：
1. 关闭selinux：
```
	setenforce 0
```
2. 重启wokd服务：
```
	systemctl restart wokd
```
3. 启动nginx服务：
```
	systemctl start nginx
```

### 通过浏览器访问：
```
     https://IP_ADDR:8001
```

### wokd默认通过PAM方式来验证，因此可以直接使用主机上的用户账号登录.
