* 删除docker时遇到的问题：
```
		Error response from daemon: Cannot destroy container oracle: Driver devicemapper failed to remove root filesystem b719043fc325049ee11e76bddd45f60a22c72b6125a0f0d1b278cd928c724332: Device is Busy
		Error: failed to remove containers: [oracle]
```


* 解决方式:
		`umount /var/lib/docker/devicemapper/mnt/b719043fc325049ee11e76bddd45f60a22c72b6125a0f0d1b278cd928c724332   --->    docker rm  b719043fc32`


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
			UsePAM off -->   UsePAM no
  ```
