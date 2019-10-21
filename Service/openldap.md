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
~]# ldapmodify -x -D "cn=ldapadm,dc=zyuru,dc=com" -f modify.ldif -W
```
* 注: ppolicy schema有待研究

## ldapwhoami 命令
检验 OpenLDAP 用户身份

示例:
```bash
ldapwhoami -x -D "uid=raj,ou=People,dc=zyuru,dc=com" -W 

dn:uid=user1,ou=People,dc=zyuru,dc=com
```

<!-- ## ldapmodrdn 命令
修改 OpenLDAP 目录树 RDN 条目，可以从标准的条目信息输入或者使用-f指定LDIF文件的格式输入
语法: ldapmodrdn [args]

args:
	-r: 删除 OpenLDAP 目录树中的 rdn 条目的唯一标识名称
	-O props: 指定 SASL 安全属性
	-P: 指定 OpenLDAP 监听的端口


## ldapcompare 命令
判断DN值和指定参数值是否属于同一条目 -->

## ldappasswd 命令
修改目录树用户条目实现密码重置
语法: ldappasswd [args]
args:
	-S: 提示用户输入新密码
	-s newPasswd: 明文指定密码
	-a oldPasswd: 通过旧密码，生成新密码
	-A: 提示输入就密码，自动生成新密码
	-x: 使用简单的认证
	-D <binddn>:
	-W:
	-w:

示例:
```bash
~]# ldappasswd -x -D "cn=ldapadm,dc=zyuru,dc=com" -W "uid=raj,out=People,dc=zyuru,dc=com" -S
New password: 
Re-enter new password: 
Enter LDAP Password: 

```


## slaptest 命令
用于检测配置文件(/etc/openldap/slapd.conf) 以及数据库文件的可用性
语法: slaptest [args]
args:
	-f: 指定配置文件
	-F: 指定数据库目录里
	-d: 指定debug级别
示例:
```bash
~]# slaptest -d 3 -F /etc/openldap/slapd.d/
```

## slapindex 命令
用于创建OpenLDAP数据库条目索引，提高查询效率, 前提是 slapd 进程停止，否则会提示错误
语法: slapindex [args]
args:
	-f: 指定OpenLDAP的配置文件，并创建索引
	-F: 检测指定OpenLDAP数据库目录，并创建索引

## slapcat 命令
将数据条目转换未 OpenLDAP 的 LDIF 文件
示例:
```bash
slapcat -b "dc=zyuru,dc=com" -a uid=user1
```

## 权限

### sudo 权限级别分类
1. 用户级别
用户级别是指一个用户可以切换到指定用户，并通过切换后的用户身份执行操作
示例:
```bash
# whoami
test
# sudo su - test1
# whoami
test1
```

2. 组级别
组级别是指为组定义sudo权限，此时组内所包含的用户均具有组所定义的权限规则

3. 命令级别
命令级别是指用户可以通过定义的sudo权限执行超过自身权限的指令，例如，普通用户没有添加用户的权限，此时通过配置sudo规则使某个用户具有添加、删除用户的权限，也称提升用户权限，用户权限的提升通过root身份进行配置


* 用户调用sudo权限流程图
[sudo](!./images/sudo.png)

流程：
当用户调用sudo命令时，系统会根据 /etc/nsswitch.conf 文件所定义的 sudoers 的查找顺序进行搜索，例如，当nsswitch文件sudo定义为 sudoers: file ldap 用户先去本地suders文件中查找用户所定义的权限，并根据sudo配置是否需要用户提供密码。如果本地没有找到，系统会去OpenLDAP服务端查找，如果有就执行，没有则用户执行返回错误信息


### OpenLDAP sudo 权限
* sudo 常见属性
	* sudoCommand: 可执行的二进制命令
	* sudoHost: 可以再那些机器上执行 sudoCommand 定义的命令
	* sudoNotAfter: 起始时间 sudo 规则匹配
	* sudoNotBefore: 结束时间 sudo 规则匹配
	* sudoOption: 定义超过自身权限及切换至其他用户时，是否需要输入当前用户密码
	* sudoOrder: sudo 规则执行顺序，其属性是一个整数
	* sudoRole: 定义的规则
	* sudoRunAs: 可切换到定义的用户身份下执行命令
	* sudoRunAsGroup: 可切换到定义所属组并具有该组的权限
	* sudoRunAsUser: 定义可切换至哪些用户下执行命令
	* sudoUser: 限制哪些用户或那些组内的成员具有sudo相关规则


#### 通过 OpenLDAP 服务端实现用户权限控制

1. 导入 sudo schema
在 OpenLDAP 服务端中，schema 是 LDAP 的一个重要组成部分，它类似于数据库的模式定义，LDAP 中的 schema 定义了 LDAP 目录树所应遵循的结构和规则，比如一个 objectClass 具有哪些属性，这些属性又具有什么结构特点等相关定义，哪些属性是必须的，所以 schema 给 LDAP 提供了规范、属性等信息的识别方式，哪些对象可以被 LDAP 服务识别都是由 schema 来定义的
```bash
# 查找sudo schema文件
~]# rpm -ql sudo | grep LDAP

# 复制 schema 文件到openldap 的schema存放处
~]# cp /usr/share/doc/sudo-1.8.23/schema.OpenLDAP /etc/openldap/schema/sudo.schema

# 转换配置文件为ldif格式
~#] mkdir ~/sudo/
~#] echo "include /etc/openldap/schema/sudo.schema" > ~/sudo/sudoSchema.conf
~#] slapcat -f ~/sudo/sudoSchema.conf -F  /tmp -n0 -s "cn={0}sudo,cn=schema,cn=config" > ~/sudo/sudo.ldif

~]# sed -i "s/{0}sudo/{3}sudo/g" ~/sudo/sudo.ldif

# 将最后几行不能由用户修改的删除掉
# 从 structuralObjectClass: olcSchemaConfig 开始，至文件最后删除

~]# ldapadd -Y EXTERNAL -H ldapi:/// -f   ~/sudo/sudo.ldif
```

2. 在 OpenLDAP 目录树中建立 suduers 条目
  sudoers 的配置信息存放在 ou=suders 的子树中，默认 OpenLDAP 用户没有指定sudo规则， OpenLDAP 首先在目录树子树中寻找条目 cn=defaults, 如果找到，那么所有的 sudoOption 属性都会被解析成全局默认值，这类似于系统sudo(/etc/sudoers) 文件中的 Defaults 语句

  当用户到OpenLDAP服务端中查询一个sudo用户权限时一般又两道三次查询，第一次查询解析全局配置，第二次查询匹配用户名或者用户所在的组(特殊标签ALL也在此次查询中匹配)，如果没有找到相关匹配项，则发出第三次查询，此次查询返回所有包含用户组的条目并检查该用户是否存在于这些组中。

* 创建 sudoers 子树
1. 创建ou=sudoers
```bash
~]# cat ou-sudoers.ldif
dn: ou=sudoers,dc=zyuru,dc=com
objectClass: organizationalUnit
ou: suders

~]# ldapadd -x -D "cn=ldapadm,dc=zyuru,dc=com" -f ou-sudoers.ldif -W
Enter LDAP Password: 
adding new entry "ou=sudoers,dc=zyuru,dc=com"

```

2. 创建cn=defaults
```bash
~]# cat cn-defaults.ldif
dn: cn=defaults,ou=sudoers,dc=zyuru,dc=com
objectClass: sudoRole
cn: defaults
description: Defaults sudoOption\'s go here
sudoOption: requiretty
sudoOption: !visiblepw
sudoOption: always_set_home
sudoOption: env_reset
sudoOption: env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR  LS_COLORS"
sudoOption: env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE"
sudoOption: env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUPEMENT LC_MESSAGES"
sudoOption: env_keep+="LC_MONETARY LC_NAME LC_NUMBERIC LC_PAPER LC_TELEPHONE"
sudoOption: env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY"
sudoOption: secure_path=/sbin:/bin:/usr/sbin:/usr/bin

~#] ldapadd -x -D 'cn=ldapadm,dc=zyuru,dc=com' -f cn-defaults.ldif -W
Enter LDAP Password: 
adding new entry "cn=defaults,ou=sudoers,dc=zyuru,dc=com"

```

3. 创建测试用户
```bash
~]# cat cn-users.ldif
dn: cn=%dba,ou=sudoers,dc=zyuru,dc=com
objectClass: sudoRole
cn: %dba
sudoUser: %dba
sudoRunAsUser: oracle
sudoRunAsUser: grid
sudoOption: !authenticate
sudoCommand: /bin/bash

dn: cn=%app,ou=sudoers,dc=zyuru,dc=com
objectClass: sudoRole
cn: %app
sudoUser: %app
sudoHost: ALL
sudoRunAsUser: appman
sudoOption: !authenticate
sudoCommand: /bin/bash

dn: cn=%admin,ou=sudoers,dc=zyuru,dc=com
objectClass: sudoRole
cn: %admin
sudoUser: %admin
sudoHost: ALL
sudoOption: authenticate
sudoCommand: /bin/rm
sudoCommand: /bin/rmdir
sudoCommand: /bin/chmod
sudoCommand: /bin/chown
sudoCommand: /bin/dd
sudoCommand: /bin/mv
sudoCommand: /bin/cp
sudoCommand: /sbin/fsck*
sudoCommand: /sbin/*remove
sudoCommand: /usr/bin/chattr
sudoCommand: /sbin/mkfs*
sudoCommand: !/usr/bin/passwd
sudoOrder: 0

dn: cn=%limit,ou=sudoers,dc=zyuru,dc=com
objectClass: top
objectClass: sudoRole
cn: %limit
sudoCommand: /usr/bin/chattr
sudoHost: limit.zyuru.com
sudoOption: !authenticate
sudoRunAsUser: ALL
sudoUser: %limit


~]# ldapadd -x -D "cn=ldapadm,dc=zyuru,dc=com" -f cn-users.ldif -W
Enter LDAP Password: 
adding new entry "cn=%dba,ou=sudoers,dc=zyuru,dc=com"

adding new entry "cn=%app,ou=sudoers,dc=zyuru,dc=com"

adding new entry "cn=%admin,ou=sudoers,dc=zyuru,dc=com"

adding new entry "cn=%limit,ou=sudoers,dc=zyuru,dc=com"

```
























