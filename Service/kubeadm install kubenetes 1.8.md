### 使用kubeadm安装kubernetes 1.8
#### 一、环境准备：
`1. 设置代理:`
```
#安装shadowsocks
pip install shadowsocks

#编辑配置文件
vim /etc/ss_local.json
{
	"server":"my_server_ip",
	"server_port":8388,
	"local_address": "127.0.0.1",
	"local_port":1080,
	"password":"mypassword",
	"timeout":300,
	"method":"aes-256-cfb",
	"fast_open": false
}

#启动代理：
nohup sslocal -c /etc/ss_local.json &

#shadowsocks是基于socks5协议，而我们拉去镜像需要走http/https协议，所以需要配置privoxy来将所有的http/https流量通过本地socks5协议发送出去

#安装privoxy
yum -y install privoxy

#更改privoxy的配置文件内容
vim /etc/privoxy/config
	forward-socks5t   /   127.0.0.1:1080 .
	listen-address	127.0.0.1:8118		#如果使用默认值不需要修改

#配置环境变量
vim /etc/profile

#增加以下内容
export http_proxy=http://127.0.0.1:8118/
export https_proxy=http://127.0.0.1:8118/

#立即生效
source /etc/profile
```
#### 二、工具安装
`2. 安装Docker`
```
#安装Docker
1. 安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2

2. 添加软件源信息
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

3. 更新并安装Docker-CE
yum makecache fast
yum -y install docker-ce

4. 修改docker启动脚本，添加http/https代理变量
vim /usr/lib/systemd/system/docker.service
# 在Service标签中添加以下参数：
Environment=HTTP_PROXY=http://127.0.0.1:8118/
Environment=HTTPS_PROXY=http://127.0.0.1:8118/
Environment=NO_PROXY=localhost,127.0.0.1

5. 启动docker服务
systemctl daemon-reload && systemctl start docker

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]

```

`3. 安装kubeadm、kubelet、kubectl`
```
1. 添加阿里云软件源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

2. 安装并启动
yum install -y kubelet-1.8.0-0 kubeadm-1.8.0-0 kubectl-1.8.0-0
systemctl enable kubelet && systemctl start kubelet

# kubelet的启动参数--cgroup-driver默认值为cgroupfs与docker的默认值一样，但用yum安装kubelet、kubeadm时生成的10-kubeadm.conf文件中将这个参数值修改成了systemd
3. 修改docker的cgroup driver使其和kubelet一致，两中修改方式：
	1. 修改docker的cgroup driver的值为kubelet的默认，修改或者编辑/etc/docker/下的daemon.json文件加上一下参数
		{
  		"exec-opts": ["native.cgroupdriver=systemd"]
		}

	2. 修改kubelet的cgroup driver使其其和docker一致
		使用docker info | grep 'group' 查看docker Cgroup Driver的值
		vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
		...
		Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"		#此处的值与docker Cgroup Driver的值一致
		...

4. 重启kubelet
systemctl daemon-reload && systemctl restart kubelet
```
#### 三、使用kubeadm初始化集群
```
kubeadm init --kubernetes=v1.8.0 --pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=master

```
