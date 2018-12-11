# etcd集群部署
## 准备
1. 下载二进制包
```
wget https://github.com/etcd-io/etcd/releases/download/v3.3.10/etcd-v3.3.10-linux-amd64.tar.gz
```

2. 解压并设置环境
```
tar xf etcd-v3.3.10-linux-amd64.tar.gz -C /opt/  &&  mv etcd-v3.3.10-linux-amd64  etcd
echo 'ETCDCT_API=3' >> ~/.bashrc
source ~/.bashrc
```

3. 生成证书文件
```
# 生成ca秘钥
openssl genrsa -out etcd-ca-key.pem 2048

# 生成ca证书
openssl req -new -x509 -days 3650 -key etcd-ca-key.pem -out etcd-ca.pem

# 生成etcd私钥
openssl genrsa -out master-key.pem 2048

# 生成etcd证书签署请求
openssl req -new -key master-key.pem -out master.csr

# 使用ca证书签署etcd证书请求
penssl ca -cert etcd-ca.pem -days 3650 -in master.csr -out master.pem
```

4. 创建systemd unit file
* [生成配置](http://play.etcd.io/install)
```
~]# vim /usr/lib/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
Conflicts=etcd.service
Conflicts=etcd2.service

[Service]
Type=notify
Restart=always
RestartSec=5s
LimitNOFILE=40000
TimeoutStartSec=0

ExecStart=/opt/etcd/etcd --name kube-master \
  --data-dir /data/etcd/master \
  --listen-client-urls https://172.18.54.1:2379 \
  --advertise-client-urls https://172.18.54.1:2379 \
  --listen-peer-urls https://172.18.54.1:2380 \
  --initial-advertise-peer-urls https://172.18.54.1:2380 \
  --initial-cluster kube-master=https://172.18.54.1:2380,kube-node1=https://172.18.54.2:22380,kube-node2=https://172.18.54.3:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new \
  --client-cert-auth \
  --trusted-ca-file /data/etcd/certs/etcd-root-ca.pem \
  --cert-file /data/etcd/certs/master.pem \
  --key-file /data/etcd/certs/ master-key.pem \
  --peer-client-cert-auth \
  --peer-trusted-ca-file /data/etcd/certs/etcd-root-ca.pem \
  --peer-cert-file /data/etcd/certs/master.pem \
  --peer-key-file /data/etcd/certs/master-key.pem

[Install]
WantedBy=multi-user.target

~]# systemctl daemon-reload
~]# systemctl enable etcd.service
```
* __注：集群最少3台机器，主机在生成其他两台systemd unit file时将`--listen-peer-urls`的地址修改为与`--initial-cluster`规划的地址一致.__

```

```
