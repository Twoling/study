## Nginx自定义header无法输出

### 如果自定义的header使用中间线`-`连接Nginx是可以正常在日志中打印到的
* 日志格式定义打印自定义header
```
自定义header打印需要在header name之前加上 $http_ 或者 $sent_http_
例如：
log_format main  'DIY_KEY: [$http_x_real_ip] ...'
```

### 如果自定义的header使用下划线`_`连接，Nginx会自动忽略相关header的。
* 默认`underscores_in_headers`为`off`时，如果`header name`中包含下划线`_`，则忽略掉

```
在 http段或 server段启用即可

...
underscores_in_headers on;
...
```