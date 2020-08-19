# 网络管理

`net-tools` 是 CentOS 7 之前的版本使用的网络管理工具，而 iproute2 是 CentOS 7 之后主推的网络管理工具。

## net-tools

- ifconfig 网卡配置
- route 网关配置
- netstat

## iproute2

- ip
- ss

## 使用网络工具

### ifconfig 查看网卡

执行 `ifconfig` 查看网卡。

CentOS 使用一致性网络设备命名。

一般第一块网卡是 `eth0`。也可能是：

- `eno1` 板载网卡
- `ens33` PCI-E 网卡
- `enp0s3` 无法获取物理信息的 PCI-E 网卡

都不匹配就使用 `eth0`。可以通过网卡名称，知道使用的是什么类型的网卡。

### 配置网卡名称

在管理服务器集群时，网卡名称不同，会不好管理。如何配置网卡名称？

首先，网卡命名规则受两个参数影响，分别是 `biosdevname` 和 `net.ifnames`。grup 是系统启动时，引导内核的工具。

1. 编辑 `/etc/default/grup` 文件，在 `GRUP_CMDLINE_LINUX` 对应的命令后面添加（或修改）参数 `biosdevname=0 net.ifnames=0`，就可以传递到内核。
2. `/etc/default/grup` 需要运行 `grup2-mkconfig -o /boot/grup2/grup.cfg` 才会更新 grup，被内核读取。
3. reboot

规则：

|  biosdevname | net.ifnames | 网卡名 |
| ----  | ---- | ---- |
| 0 | 1 | ens33（default） |
| 1 | 0 | em1 |
| 0 | 0 | eth0 |

### 网线的连接状态

`mii-tool <网卡名称>`。

### 查看网关

`route`，常用命令是 `route -n`，因为 `route` 会尝试解析主机名，也就是将 IP 转换为域名，可能耗时较长。

### 配置网卡

- `ifconfig <网卡> <IP> [netmask 子网掩码]` 配置网卡的 IP。注意如果是远程登录，修改 IP，可能会直接断开连接。
- `ifup <网卡>` 启动网卡
- `ifdown <网卡>` 关闭网卡

### 配置网关

- `route add default gw <网关 IP>` 添加默认网关
- `route add -host <IP> gw <网关 IP>` 添加路由，访问指定 IP 时，使用指定的网关
- `route add -net <网段> netmask <子网掩码> gw <网关 IP>` 添加路由，访问指定网段时，使用指定网关
- `route del default gw <网关 IP>` 删除默认网关

### 使用 ip 命令

- `ip addr ls` 相当于 `ifconfig`
- `ip link set dev eth0 up ls` 相当于 `ifup eth0`
- `ip addr add 10.0.0.1/24 dev eth1` 相当于 `ifconfig eth1 netmask 255.255.255.0`
- `ip route add 10.0.0.1/24 via 192.168.0.1` 相当于 `route add -net 10.0.0.0 netmask 255.255.255.0 gw 192.168.0.1`。

## 网络故障排查

- ping
- traceroute
- mtr
- nslookup
- telnet
- tcpdump
- netstat
- ss
