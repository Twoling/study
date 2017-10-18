## Tomcat配置apr
* 安装依赖包:
  * `yum -y install openssl-devel apr-devel`

* 下载Native 1.2.12
  * `wget http://www-eu.apache.org/dist/tomcat/tomcat-connectors/native/1.2.12/source/tomcat-native-1.2.12-src.tar.gz`

* 解压、编译安装
  * `tar xf tomcat-native-1.2.12-src.tar.gz` 
  * `cd tomcat-native-1.2.12-src/native`
  * `./configure`

* `make && make install`


### 配置Tomcat：
* 配置环境变量
```
vim /etc/profile.d/tomcat.sh
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib
export LD_RUN_PATH=$LD_RbUN_PATH:/usr/local/apr/lib
```

* 导出设置

  * `. /etc/profile.d/tomcat.sh`

* 在tomcat的server.xml配置文件中启用apr模式，在对应的Connector中添加:
   `protocol="org.apache.coyote.http11.Http11AprProtocol"`

* 重启tomcat

## 安装时的问题：

**如果提示"error: Your version of OpenSSL is not compatible with this version of tcnative" 则系统上的openssl-devel的软件包不兼容，需要从openssl官网下载最新的openssl进行安装；**
* 解决步骤：

```
 wget https://github.com/openssl/openssl/archive/OpenSSL_1_0_2-stable.zip
 unzip openssl-1.0.2l.tar.gz
 cd openssl-OpenSSL_1_0_2-stable
 ./config shared
 make && make install

```
**如果出现上述问题，在编译openssl完成后，再次执行configure的时候需要指定新版openssl的位置**
* `./configure --with-ssl=/usr/local/openssl`
