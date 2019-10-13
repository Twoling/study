# OpenLDAP
---

### Schema：
schema 用来指定一个条目所包含的对象类(objectClass) 以及每一个对象类所包含的属性值。其属性又分为必要属性和可选属性两种，一般必要属性是指添加条目时必须指定的属性，可选属性是可以选择或者不选择的。 schema 定义对象类，对象类包含属性的定义，对象类和属性组合成条目


OpenLDAP相关命令

## ldapsearch 命令
搜索 OpenLDAP 目录树条目，根据用户定义的查询条件，对 OpenLDAP目录树进行查找
语法： ldapsearch [args] <过滤条件>
参数：
	-b <searchbase>: 指定查询的节点
	-D <binddn>: 指定查询的DN，DN是整个OpenLDAP树的唯一识别名称你，类似于系统中根的概念
	-v: 详细信息输出
	-x: 使用简单的认证，不适用任何加密算法
	-W: 直接给定查询密码 -w password
	-h: (OpenLDAP 主机)使用指定的ldaphost，可以使用FQDN或IP地址
	-H: (LDAP-URL) 使用LDAP服务器的URL地址进行操作
	-p: (port) 指定OpenLDAP监听的端口(默认端口为389, 加密端口为636)

示例:
```bash
~]# ldapsearch -x -D "cn=Manager,dc=zyuru,dc=com" -H ldap://ldaps1.zyuru.com -b "ou=people,dc=zyuru,dc=com" -W
# extended LDIF
#
# LDAPv3
# base <ou=People,dc=zyuru,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# People, zyuru.com
dn: ou=People,dc=zyuru,dc=com
objectClass: organizationalUnit
ou: People

# raj, People, zyuru.com
dn: uid=raj,ou=People,dc=zyuru,dc=com
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: raj
uid: raj
uidNumber: 9999
gidNumber: 100
homeDirectory: /home/raj
loginShell: /bin/bash
gecos: Raj [Admin (at) zyuru]
shadowLastChange: 17058
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
userPassword:: e1NTSEF9NGJlWnpKS0Y3UWNXUjM2c0tkamFkcVY5NjNFaSsvc2k=

```

通过 ldapsearch 配合一些参数以及过滤条件查看用户，例如: 查看当前 OpenLDAP目录树关于 raj 用户的信息(cn以及gidNumber)
```bash
~]# ldapsearch -x -LLL -b dc=zyuru,dc=com 'uid=raj' cn gidNumber
dn: uid=raj,ou=People,dc=zyuru,dc=com
cn: raj
gidNumber: 100

```
参数:
	-x: 简单认证模式
	-LLL: 禁止输出与过滤条件不匹配的信息
	-b: 目录树的基准目录树信息
	uid: 过滤条件，找到包含 raj的用户
	cn、 gidNumber: 将raj的信息再次进行过滤，只显示cn及gidNumber信息

## ldapadd 命令
通过 LDIF 格式，添加目录树条目
ldapadd实际上是 ldapmodify 的软连接，两个命令的相关参数几乎没有多大区别，ldapadd 在功能上等同与 ldapmodify -a 命令

示例:
```
# LDIF 文件
dn: dc=zyuru,dc=com
objectClass: top
objectClass: domain
dc: zyuru

dn: ou=People,dc=zyuru,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=zyuru,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit
```

```bash
ldapadd -x -D "cn=ldalpadm,dc=zyuru,dc=com" -W -H  ldap://ldaps1.zyuru.com -f add_ou.ldif
```

```
cat <<EOF | ldapadd -x -D "cn=ldapadm,dc=zyuru,dc=com" -W -H ldap://ldaps1.zyuru.com
> dn: ou=Group,dc=zyuru,dc=com
> changetype: delete
> EOF
Enter LDAP Password:
deleting entry "ou=Group,dc=zyuru,dc=com"
```

## ldapdelete 命令
删除目录树条目,并根据 DN 条目删除一个或多个条目，但必须提供所要删除指定条目的权限所绑定的DN(整个目录树的唯一标识名称)

语法: ldapdelete [args]
参数:
	-c: 持续操作模式，例如在操作中出现错误，也会进行后续相关操作
	-D <bindn>: 查找指定的DN
	-n: 显示正在记性的相关操作，但不实际修改数据，一般用于测试
	-x: 使用简单的认证，不适用任何加密的算法，例如，TLS、SASL等相关加密算法
	-f: 使用目标文件名作为命令的输入
	-W: 提示输入密码
	-w passwd:
	-y passwdfile:
	-r: 递归删除
	-h: 
	-H:
	-p:

示例:
```bash
~]# ldapdelete -x -D "cn=ldapadm,dc=zyuru,dc=com" -W 
Enter LDAP Password: 
cn=test4,ou=People,dc=zyuru,dc=com	//输入要删除的条目信息，按回车键后再按 Ctrl+D 组合键结束即可
cn=test3,ou=People,dc=zyuru,dc=com
```

```bash
ldapdelete -x -D "cn=ldapadm,dc=zyuru,dc=com" "uid=raj,ou=People,dc=zyuru,dc=com"
```

## ldapmodify 命令
修改目录树条目
语法: ldapmodify [args]

参数:
	-a: 新增条目
	-D <bindn>: 指定一个DN
	-n: 显示正在进行操作的步骤，但不直接修改数据库中的数据，测试语法中经常用到它
	-v: 详细输出结果
	-x: 使用简单的验证，不适用任何加密的验证
	-f: 根据指定的文件，对数据库进行操作
	-wpasaswd:
	-W
	-h
	-H
	-p

示例:
```
# LDIF 文件
dn: uid=raj,ou=People,dc=zyuru,dc=com
changetype: modify
replace: pwdReset
pwdReset: TRUE
```

```bash
~]# ldapmodify -x -D "cn=ldapadm,dc=zyuru,dc=com" -f modify.ldif
```

## ldapwhoami 命令
检验 OpenLDAP 用户身份

## ldapmodrdn 命令
修改 OpenLDAP 目录树 DN 条目

## ldapcompare 命令
判断DN值和指定参数值是否属于同一条目

## ldappasswd 命令
修改目录树用户条目实现密码重置

## slaptest 命令
验证配置

## slapindex 命令
创建目录树索引，提高查询效率

## slapcat 命令
将数据条目转换未 OpenLDAP 的 LDIF 文件

