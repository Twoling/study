### 准备字体文件
* 下载或者拷贝Windows机器上的微软雅黑字体文件(msyh.ttc 完整路径：C:\Windows\Fonts\msyh.ttc)
* 将字体文件的后缀名从.ttc改为.ttf
* 将字体文件上传到zabbix服务器的前端资源的fonts目录中去。

**注：如果是通过yum安装，zabbix前端页面文件默认在/usr/share/zabbix目录中，如果通过源码安装，前端页面的存放位置为安装后复制`.../prontends/php/*`目录下资源的位置**


### 修改zabbix前端php文件
* 修改defines.inc.php文件：
  * 修改第45行：
  ```
  #修改前
  define('ZBX_GRAPH_FONT_NAME',                'DejaVuSans'); // font file name
  #修改后
  define('ZBX_GRAPH_FONT_NAME',           'msyh'); // font file name

  ```
  * 修改第93行
  ```
  #修改前
  define('ZBX_FONT_NAME', 'DejaVuSans');
  #修改后
  define('ZBX_FONT_NAME', 'msyh');

  ```
**操作完成zabbix无需重启即可生效**
