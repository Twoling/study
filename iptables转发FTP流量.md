1. 在iptables上添加NAT规则
```
iptables -t nat -A PREROUTING -d <DEST_IP> -p <PROTOCOL> --dport 21 -j MASQUERADE
iptables -t nat -A POSTROUTING -d <local_ip> -p <PROTOCOL> --dport 21 -j DNAT --to-destination <DESC_IP>:21
``` 

2. 装载系统模块 
```
modprobe ip_conntrack_ftp
modprobe ip_nat_ftp
> 在进入被动模式的时候，是否可以修改PASV里面的目标IP为本机IP呢
```