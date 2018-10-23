1. 在对应tomcat的安装目录下的conf/Catalina/localhost/manager.xml文件中添加如下内容，如没有此文件，可自行创建
```
<Context privileged="true" antiResourceLocking="false"   
         docBase="${catalina.home}/webapps/manager">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>
```

2. 编辑conf目录下tomcat-users.xml在<tomcat-users>标签内添加如下内容

```
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="tomcat" password="123" roles="manager-gui,admin-gui"/>
```

3. 重启tomcat即可生效
