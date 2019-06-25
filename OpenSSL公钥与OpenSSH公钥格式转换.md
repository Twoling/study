## OpenSSL公钥与OpenSSH公钥格式转换

### OpenSSL公钥格式
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCi8kTu1UJavFipLrNOwcYLUzSltwF/r3r8LhRASgTrDmKSZIL7i1uRO0KY4zXuVk1fGl80C1AyfLvF4SgciRJBsD6jHDucQL6RyCKwbB12+1gkQGa3bNGklGzJ+vE28GMEkDqnTZjy2/ijLX4brN/P5zUKGHLsRizXmZtxwinK1+6k9HzUYNN+6yu/ZcA3R8NOeBTbXQx+upjLOlKlimR+QArVRk51pY3KSTBHie0S93/3buEDRXjQhcmjRaK/pov35uqAAsV6FSHxb5APT5AiWw75dy+1PTgUOu3zI6XIW387E2Z7O539oSD6xSntieOo8WZsx8ETn7xJgczWufqp
```

### OpenSSH公钥格式
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAovJE7tVCWrxYqS6zTsHG
C1M0pbcBf696/C4UQEoE6w5ikmSC+4tbkTtCmOM17lZNXxpfNAtQMny7xeEoHIkS
QbA+oxw7nEC+kcgisGwddvtYJEBmt2zRpJRsyfrxNvBjBJA6p02Y8tv4oy1+G6zf
z+c1Chhy7EYs15mbccIpytfupPR81GDTfusrv2XAN0fDTngU210MfrqYyzpSpYpk
fkAK1UZOdaWNykkwR4ntEvd/927hA0V40IXJo0Wiv6aL9+bqgALFehUh8W+QD0+Q
IlsO+XcvtT04FDrt8yOlyFt/OxNmezud/aEg+sUp7YnjqPFmbMfBE5+8SYHM1rn6
qQIDAQAB
-----END PUBLIC KEY-----
```

### 从私钥重新生成OpenSSH格式公钥
```
ssh-keygen -y -f private.key > private.key.pub
```

### 将OpenSSL格式公钥转换为OpenSSH格式公钥
```
ssh-keygen -i -m PKCS8 -f private.pub > private_ssh.pub
```

### 将OpenSSH格式公钥转换为OpenSSL格式公钥
```
ssh-keygen -e -m PEM -f private_ssh.pub > private_ssl.pub
```

## ssh-keygen选项
* `-b <bits>`： 密钥长度，默认为2048
* `-t <type>`： 密钥类型，rsa、des...(默认类型rsa)
	* SSH1： RSA
	* SSH2： RSA、DSA、ECDSA
* `-N <password>`： 新密码
* `-f <file>`： 指定密钥文件，会同时生成一个.pub结尾的公钥文件
* `-C <comment>`：
* `-c`： 修改公钥或私钥文件中的注释
* `-p`： 修改私钥文件密码
* `-P <password>`： 旧密码
* `-e`： 导出为其他格式的密钥文件，可以转换密钥类型
* `-i`： 从其它格式的密钥文件导入，可以转换密钥类型
* `-m <PEM|PKCS8|RFC4716>`： 与-e, -i 配合使用，指明导出导入的密钥文件格式
* `-y`： 读入密钥并显示公钥

