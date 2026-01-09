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

## linux内核参数
### net.ipv4.tcp_tw_reuse
`net.ipv4.tcp_tw_reuse` 是 Linux 操作系统内核参数之一，用于控制是否允许将处于 TIME_WAIT 状态的 TCP 连接重新用于新的连接。下面是对这个参数的基础概念、优势、类型、应用场景以及可能遇到的问题和解决方案的详细解释。

#### 基础概念
* TIME_WAIT 状态：当一个 TCP 连接关闭时，它并不会立即消失，而是会进入 TIME_WAIT 状态，持续一段时间（通常是几分钟）。这是为了确保所有延迟的数据包都被处理，并且防止旧连接的数据包干扰新连接。
* tcp_tw_reuse 参数：此参数设置为 1 时，允许系统在 TIME_WAIT 状态的连接上进行新的连接，前提是新连接的源 IP 和端口与旧连接相同，且新连接的目的 IP 和端口与旧连接不同。
#### 优势
1. 提高端口复用率：在高并发环境下，可以减少因端口耗尽而导致的连接失败问题。
1. 加快连接建立速度：避免了为新连接等待 TIME_WAIT 状态结束的时间。
#### 类型
* 全局设置：通过修改 /etc/sysctl.conf 文件来全局设置此参数。
* 临时设置：使用 sysctl 命令临时调整内核参数。
#### 应用场景
* Web 服务器：处理大量短时连接的场景，如 Web 服务器在高访问量时。
* 负载均衡器：在负载均衡器后面有多个后端服务器时，可以提高连接效率。
#### 可能遇到的问题和解决方案
{{< admonition question >}}
问题1：启用 tcp_tw_reuse 后仍然出现连接失败

原因：可能是由于网络中的路由器或防火墙阻止了带有相同四元组（源 IP、源端口、目的 IP、目的端口）的连接。

解决方案：

检查网络设备配置，确保没有阻止重复四元组的策略。

尝试使用 net.ipv4.tcp_timestamps 参数，它允许更精确地控制 TIME_WAIT 状态的行为。
{{< /admonition >}}


{{< admonition question >}}
问题2：启用 tcp_tw_reuse 导致数据混乱

原因：在某些情况下，如果网络延迟较高或有重复的数据包，启用 tcp_tw_reuse 可能会导致新旧连接的数据混淆。

解决方案：

确保网络环境稳定，减少数据包丢失和延迟。

在必要时，可以考虑禁用 tcp_tw_reuse，并通过增加服务器资源来应对高并发。
{{< /admonition >}}

