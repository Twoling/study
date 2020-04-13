1. 在iptables上添加NAT规则
```
iptables -t nat -A PREROUTING -d <local_ip> -m <PROTOCOL> -p <PROTOCOL> --dport 21 -j DNAT --to-destination <local_ip>
iptables -t nat -A POSTROUTING -d <REMOTE_FTP_SERVER_IP> -m <PROTOCOL> -p <PROTOCOL> --dport 21 -j MASQUERADE
``` 

2. 装载系统模块 
```
modprobe ip_conntrack_ftp
modprobe ip_nat_ftp
> 在进入被动模式的时候，是否可以修改PASV里面的目标IP为本机IP呢
```
