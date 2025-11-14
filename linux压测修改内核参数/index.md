# Linux压测修改内核参数



<!--more-->

## linux客户端压测
用linux作为压测客户端时，需要修改内核的参数。不然会被类似文件打开数量的限制或网络缓存不足的问题

### 修改 /etc/security/limits.conf
``` limits.conf
* soft nofile 65535
* hard nofile 65535
```

### 修改 /etc/sysctl.conf
``` sysctl.conf
fs.file-max = 100000
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 16384 16777216
net.ipv4.tcp_mem = 786432 1048576 1572864
```

修改完后直接重启系统
