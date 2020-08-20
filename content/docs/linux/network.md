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

### 如何查看到目标主机的网络状态

1. 使用 `ping <IP 或者 域名>` 查看网络是否是通的。
2. `traceroure` 和 `mtr` 辅助 `ping` 命令，在 `ping` 通网络之后，如果网络通信还是有问题，可以使用 `traceroure` 可以查看网络中每一跳的网络质量。`mtr` 可以检测网络中是否有丢包。
3. `nslookup` 查看域名对应的 IP。
4. 如果主机可以连接，但是服务仍然无法访问，使用 `telnet` 检查端口状态。
5. 如果端口没有问题，仍然无法访问，可以使用 `tcpdump` 进行抓包，更细致的查看网络问题。
6. 使用 `netstat` 和 `ss`，查看服务范围。

### tracetoute

- `tracetoute -w 1 www.baidu.com`，`-w 1` wait，表示某个 IP 超时的最大等待时间为 1 秒。
  - `-n` 显示 IP 地址
  - `-m` 设置检测数据包的最大存活数值 TTL 的大小
  - `-q` 每个网关发送数据包个数
  - `-p` UDP 端口

```bash
[root@shcCDFrh75vm8 ~]# traceroute -w 1 www.baidu.com
traceroute to www.baidu.com (104.193.88.77), 30 hops max, 60 byte packets
 1  gateway (16.155.192.1)  0.525 ms  0.635 ms  0.800 ms
 2  10.132.24.193 (10.132.24.193)  0.600 ms  0.930 ms  1.137 ms
 3  192.168.201.122 (192.168.201.122)  0.826 ms  0.746 ms  0.674 ms
 4  192.168.200.45 (192.168.200.45)  2.045 ms  1.978 ms  1.962 ms
 5  192.168.203.249 (192.168.203.249)  29.375 ms  29.351 ms  29.275 ms
 6  192.168.203.250 (192.168.203.250)  3.892 ms  2.980 ms  2.907 ms
 7  192.168.200.185 (192.168.200.185)  68.675 ms  68.636 ms  68.691 ms
 8  192.168.200.186 (192.168.200.186)  70.094 ms  70.356 ms  69.995 ms
 9  * * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  * * *

```

`*` 表示不支持 `traceroute` 追踪。

记录按序列号从 1 开始，每个纪录就是一跳 ，每跳表示一个网关，可以看到每行有三个时间，单位是 ms，之所以是 3 个，其实就是 `-q` 的默认参数。探测数据包向每个网关发送三个数据包后，网关响应后返回的时间。

### mtr

运行 `mtr` 可以查看更详细的网络状态：

```bash
                                                My traceroute  [v0.85]
shcCDFrh75vm8.hpeswlab.net (0.0.0.0)                                                         Thu Aug 20 22:02:41 2020
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                                             Packets               Pings
 Host                                                                      Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. localhost                                                               0.0%     3    0.1   0.1   0.1   0.1   0.0
```

由于没有与当前主机进行通信，所以只显示了 `localhost` 的状态。

- `Host`：IP 地址和域名，按 n 键可以切换 IP 和域名
- `Loss` 表示丢包率。
- `Snt`：设置每秒发送数据包的数量，默认值是 10 可以通过参数 `-c` 来指定
- `Last`：最近一次的 ping 值
- `Avg`：是平均值 发送 ping 包的平均时延
- `Best`：是最好或者说时延最短的
- `Wrst`：是最差或者说时延最长的
- `StDev`：标准偏差

### nslookup

`nslookup <域名>`：

```bash
[root@shcCDFrh75vm8 ~]# nslookup www.baidu.com
Server:  16.187.185.202
Address: 16.187.185.202#53

Non-authoritative answer:
www.baidu.com canonical name = www.a.shifen.com.  # 别名
www.a.shifen.com canonical name = www.wshifen.com.
Name: www.wshifen.com
Address: 104.193.88.77
Name: www.wshifen.com
Address: 104.193.88.123
```

`Server` 域名解析服务器 IP。

### telnet

`telnet <IP 或 域名> <端口>`

```bash
[root@SGDLITVM0905 ~]# telnet www.baidu.com 80
Trying 104.193.88.77...
Connected to www.baidu.com.
Escape character is '^]'.

```

上面的示例说明端口是通的，输入 `quit` 退出。

### tcpdump

```bash
[root@shcCDFrh75vm8 ~]# tcpdump
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on virbr0, link-type EN10MB (Ethernet), capture size 262144 bytes

```

抓到的数据包会实时显示出来。

<https://www.cnblogs.com/williamjie/p/9983958.html>

### netstat

```bash
[root@shcCDFrh75vm8 ~]# netstat -ntp
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 16.155.194.49:22        10.5.41.247:57470       ESTABLISHED 394/sshd: root@pts/
tcp        0      0 16.155.194.49:22        10.5.41.247:64890       ESTABLISHED 25497/sshd: root@pt
tcp        0      0 16.155.194.49:22        10.5.41.247:57450       ESTABLISHED 6479/sshd: root@not
tcp        0    180 16.155.194.49:22        15.122.72.86:51641      ESTABLISHED 25305/sshd: root@pt
[root@shcCDFrh75vm8 ~]# netstat -ntpl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:6012          0.0.0.0:*               LISTEN      25305/sshd: root@pt
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:45060           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      791/rpcbind
tcp        0      0 0.0.0.0:20048           0.0.0.0:*               LISTEN      1341/rpc.mountd
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN      1832/dnsmasq
tcp        0      0 0.0.0.0:50037           0.0.0.0:*               LISTEN      1325/rpc.statd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1320/sshd
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      1321/cupsd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1674/master
tcp        0      0 127.0.0.1:6010          0.0.0.0:*               LISTEN      25497/sshd: root@pt
tcp        0      0 127.0.0.1:6011          0.0.0.0:*               LISTEN      394/sshd: root@pts/
tcp6       0      0 ::1:6012                :::*                    LISTEN      25305/sshd: root@pt
tcp6       0      0 :::35712                :::*                    LISTEN      1325/rpc.statd
tcp6       0      0 :::2049                 :::*                    LISTEN      -
tcp6       0      0 :::36773                :::*                    LISTEN      -
tcp6       0      0 :::111                  :::*                    LISTEN      791/rpcbind
tcp6       0      0 :::20048                :::*                    LISTEN      1341/rpc.mountd
tcp6       0      0 :::22                   :::*                    LISTEN      1320/sshd
tcp6       0      0 ::1:631                 :::*                    LISTEN      1321/cupsd
tcp6       0      0 ::1:6010                :::*                    LISTEN      25497/sshd: root@pt
tcp6       0      0 ::1:6011                :::*                    LISTEN      394/sshd: root@pts/

```

### ss

```bash
[root@shcCDFrh75vm8 ~]# ss -ntpl
State      Recv-Q Send-Q              Local Address:Port                             Peer Address:Port
LISTEN     0      128                     127.0.0.1:6012                                        *:*                   users:(("sshd",pid=25305,fd=9))
LISTEN     0      64                              *:2049                                        *:*
LISTEN     0      64                              *:45060                                       *:*
LISTEN     0      128                             *:111                                         *:*                   users:(("rpcbind",pid=791,fd=8))
LISTEN     0      128                             *:20048                                       *:*                   users:(("rpc.mountd",pid=1341,fd=8))
LISTEN     0      5                   192.168.122.1:53                                          *:*                   users:(("dnsmasq",pid=1832,fd=6))
LISTEN     0      128                             *:50037                                       *:*                   users:(("rpc.statd",pid=1325,fd=8))
LISTEN     0      128                             *:22                                          *:*                   users:(("sshd",pid=1320,fd=3))
LISTEN     0      128                     127.0.0.1:631                                         *:*                   users:(("cupsd",pid=1321,fd=12))
LISTEN     0      100                     127.0.0.1:25                                          *:*                   users:(("master",pid=1674,fd=13))
LISTEN     0      128                     127.0.0.1:6010                                        *:*                   users:(("sshd",pid=25497,fd=9))
LISTEN     0      128                     127.0.0.1:6011                                        *:*                   users:(("sshd",pid=394,fd=9))
LISTEN     0      128                           ::1:6012                                       :::*
```

## 网络服务管理

- `service network start|stop|restart|status`
- `chkconfig --list network`
- `systemctl list-unit-files NetworkManager.service`
- `systemctl start|stop|restart|status NetworkManager`
- `systemctl enable|disable NetworkManager`

network 和 NetworkManager 是两套网络管理工具，应该只使用其中一套。

- `/etc/sysconfig/network-scripts/ifcfg-eth0` 网卡配置文件，名称是动态的
- `/etc/hosts`
