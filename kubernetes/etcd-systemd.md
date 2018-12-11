# etcd system unit file
## 示例3节点：

### Node1：
```
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
```

### Node2：
```
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

ExecStart=/tmp/test-etcd/etcd --name kube-node1 \
  --data-dir /data/etcd/node1 \
  --listen-client-urls https://172.18.54.2:22379 \
  --advertise-client-urls https://172.18.54.2:22379 \
  --listen-peer-urls https://172.18.54.2:22380 \
  --initial-advertise-peer-urls https://172.18.54.2:22380 \
  --initial-cluster kube-master=https://172.18.54.1:2380,kube-node1=https://172.18.54.2:22380,kube-node2=https://localhost:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new \
  --client-cert-auth \
  --trusted-ca-file /data/etcd/certs/etcd-root-ca.pem \
  --cert-file /data/etcd/certs/node1.pem \
  --key-file /data/etcd/certs/node1-key.pem \
  --peer-client-cert-auth \
  --peer-trusted-ca-file /data/etcd/certs/etcd-root-ca.pem \
  --peer-cert-file /data/etcd/certs/node1.pem \
  --peer-key-file /data/etcd/certs/node1-key.pem

[Install]
WantedBy=multi-user.target
```

### Node3：
```
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

ExecStart=/tmp/test-etcd/etcd --name kube-node2 \
  --data-dir /data/etcd/node2 \
  --listen-client-urls https://localhost:32379 \
  --advertise-client-urls https://localhost:32379 \
  --listen-peer-urls https://localhost:32380 \
  --initial-advertise-peer-urls https://localhost:32380 \
  --initial-cluster kube-master=https://172.18.54.1:2380,kube-node1=https://172.18.54.2:22380,kube-node2=https://localhost:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new \
  --client-cert-auth \
  --trusted-ca-file /data/etcd/certs/etcd-root-ca.pem \
  --cert-file /data/etcd/certs/node2.pem \
  --key-file /data/etcd/certs/node2-key.pem \
  --peer-client-cert-auth \
  --peer-trusted-ca-file /data/etcd/certs/etcd-root-ca.pem \
  --peer-cert-file /data/etcd/certs/node2.pem \
  --peer-key-file /data/etcd/certs/node2-key.pem

[Install]
WantedBy=multi-user.target
```
* Check status: 
```
ETCDCTL_API=3 /tmp/test-etcd/etcdctl \
  --endpoints 172.18.54.1:2379,172.18.54.2:22379,localhost:32379 \
  --cacert /data/etcd/certs/etcd-root-ca.pem \
  --cert /data/etcd/certs/master.pem \
  --key /data/etcd/certs/master-key.pem \
  endpoint health
```


### [生成配置](http://play.etcd.io/install)