* 删除docker时遇到的问题：
```
Error response from daemon: Cannot destroy container oracle: Driver devicemapper failed to remove root filesystem b719043fc325049ee11e76bddd45f60a22c72b6125a0f0d1b278cd928c724332: Device is Busy
Error: failed to remove containers: [oracle]
```

* 解决方式:
```
umount /var/lib/docker/devicemapper/mnt/b719043fc325049ee11e76bddd45f60a22c72b6125a0f0d1b278cd928c724332   --->    docker rm  b719043fc32
```


* centos6.5基础镜像修改root密码：
```
/usr/share/cracklib/pw_dict.pwd: No such file or directory
PWOpen: No such file or directory
```
    
*	解决方式：
`yum reinstall -y cracklib-dicts`


* centos6.5基础镜像启动后安装sshd服务，无法登录

* 解决方式：
```
vim /etc/ssh/sshd_config
UsePAM off -->   UsePAM on
```


* 安装Python3.7是报错
`ModuleNotFoundError: No module named '_ctypes'`

* 解决方式
`安装 libffi-devel 包后重新编译即可`

* Ingress-Nginx 下载报错
```	
2020/05/12 07:23:23 [error] 402#402: *98997 upstream sent invalid chunked response while reading upstream
```

* 解决方案
```
# 在 annotation 段中加入以下配置
    nginx.ingress.kubernetes.io/configuration-snippet: |
      chunked_transfer_encoding off;
```

* 解释 chunked_transfer_encoding
> http协议中, transfer-encoding:chunked 表示在传输数据过程中要使用分块技术。而与之对应的是将数据写到一个很大很大的字节数组，如果使用这个配置就不必申请一个很大的字节数组，
> 占用的资源也少。这个一般结合 Content-Encoding:gzip 使用，即先将数据压缩，再使用块技术传输数据。有时候我们会在报文中看到Content-Length:60,Content-Length就是数组大小，即
> 需要申请的一个很大的字节数组的大小。
