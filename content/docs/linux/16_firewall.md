# 防火墙

防火墙分为两类：

- 软件防火墙，CentOS 6 的默认防火墙是 iptables，CentOS 7 的默认防火墙是 firewalld，底层都是使用内核中的 netfilter。
  - 包过滤防火墙，主要用于数据包的过滤，数据包转发。
  - 应用层防火墙，可以控制应用程序的具体的行为。
- 硬件防火墙，

## iptables

iptables的结构：

```
iptables -> Tables -> Chains -> Rules
```

tables 由 chains 组成，而 chains 又由 rules 组成。

表（tables）提供特定的功能，iptables 包含四个规则表：

- `filter` 包过滤
- `nat` 网络地址转换
- `mangle` 包重构(修改)
- `raw` 数据跟踪处理

链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一条或数条规则。

**当一个数据包到达一个链时，iptables 就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据该条规则所定义的方法处理该数据包**；
**否则 iptables 将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables 就会根据该链预先定义的默认策略来处理数据包**。

iptables 的规则链：

- `INPUT` 接收的的数据包使用此规则链中的策略
- `OUTPUT` 发送的数据包使用此规则链中的策略
- `FORWARD` 转发数据包时使用此规则链中的策略
- `PREROUTING` 目的地址转换
- `POSTROUTING` 源地址转换

`iptables [-t 表] 命令选项 [规则链] 规则`，**`-t` 默认使用的是 `filter`**。

命令选项用于指定管理 iptables 规则的方式（比如：插入、增加、删除、查看）。

命令选项：

- `-L` 列出（list）指定链中所有的规则进行查看
- `-A` 在指定链的末尾添加（append）一条新的规则
- `-I` 在指定链中插入（insert）一条新的规则，默认在第一行添加
- `-D` 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除，`iptables -D INPUT 1`
- `-F` 清空（flush），`iptables -F` 清楚 `filter` 的规则
- `-P` 设置指定链的默认策略（policy），`iptables -P INPUT DROP`
- `-N` 新建（new-chain）一条用户自己定义的规则链
- `-X` 删除指定表中用户自定义的规则链（delete-chain）
- `-E` 重命名用户定义的链，不改变链本身
- `-n` 直接使用，不解析域名
- `-v` 查看规则表详细信息

规则选项：

- `-i` 输入网口
- `-o` 输出网口
- `-p proto` 匹配网络协议：tcp、udp、icmp
- `--icmp-type type` 匹配 ICMP 类型，和 `-p icmp` 配合使用。注意有两根短划线
- `-s source-ip` 匹配来源主机（或网络）的IP地址
- `--sport port` 匹配来源主机的端口，和 `-s source-ip` 配合使用。
- `-d dest-ip` 匹配目标主机的 IP 地址
- `--dport port` 匹配目标主机（或网络）的端口，和 `-d dest-ip` 配合使用。

数据包控制方式：

- ACCEPT: 允许数据包通过
- DROP: 直接丢弃数据包，不给出任何回应信息
- REJECT: 拒绝数据包通过，必须时会给数据发送端一个响应信息
- LOG: 日志功能，在 `/var/log/messages` 文件中记录日志信息，然后将数据包传递给下一条规则
- REDIRECT: 将数据包重新转向到本机或另一台主机的某一个端口，通常功能实现透明代理或对外开放内网的某些服务
- SNAT: 源地址转换
- DNAT: 目的地址转换
- MASQUERADE: IP 伪装

```bash
[root@shcCentOS72VM07 ~]# iptables -t filter -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootps
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:bootps

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             192.168.122.0/24     ctstate RELATED,ESTABLISHED
ACCEPT     all  --  192.168.122.0/24     anywhere
ACCEPT     all  --  anywhere             anywhere
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootpc
[root@shcCentOS72VM07 ~]# iptables -t filter -A INPUT -s 10.0.0.1 -j ACCEPT  # 可以接收从 IP 为 10.0.0.1 发送的数据包
[root@shcCentOS72VM07 ~]# iptables -t filter -vnL
Chain INPUT (policy ACCEPT 3170 packets, 320K bytes)  # 表示过滤了 3170 个数据包，320 KB
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:53
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:67
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:67
    0     0 ACCEPT     all  --  *      *       10.0.0.1             0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     all  --  *      virbr0  0.0.0.0/0            192.168.122.0/24     ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  virbr0 *       192.168.122.0/24     0.0.0.0/0
    0     0 ACCEPT     all  --  virbr0 virbr0  0.0.0.0/0            0.0.0.0/0
    0     0 REJECT     all  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 REJECT     all  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable

Chain OUTPUT (policy ACCEPT 33 packets, 3887 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     udp  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            udp dpt:68
[root@shcCentOS72VM07 ~]# iptables -A INPUT -s 10.0.0.2 -j ACCEPT
[root@shcCentOS72VM07 ~]# iptables -A INPUT -s 10.0.0.2 -j DROP
[root@shcCentOS72VM07 ~]# iptables -vnL
Chain INPUT (policy ACCEPT 181 packets, 19084 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:53
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:67
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:67
    0     0 ACCEPT     all  --  *      *       10.0.0.1             0.0.0.0/0
    0     0 ACCEPT     all  --  *      *       10.0.0.2             0.0.0.0/0
    0     0 DROP       all  --  *      *       10.0.0.2             0.0.0.0/0

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     all  --  *      virbr0  0.0.0.0/0            192.168.122.0/24     ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     all  --  virbr0 *       192.168.122.0/24     0.0.0.0/0
    0     0 ACCEPT     all  --  virbr0 virbr0  0.0.0.0/0            0.0.0.0/0
    0     0 REJECT     all  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 REJECT     all  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable

Chain OUTPUT (policy ACCEPT 27 packets, 2396 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     udp  --  *      virbr0  0.0.0.0/0            0.0.0.0/0            udp dpt:68
```

上面的示例，`INPUT` 链配置了两条规则，分别是接收 IP 为 `10.0.0.2` 的数据包，和丢弃 `10.0.0.2` 的数据包。那么 `10.0.0.2` 的数据包能不能进来？

**可以**。数据包会先匹配前面的 `ACCEPT 10.0.0.2` 的规则，这个时候数据包就进入了系统，所以**规则顺序很重要**。可以使用 `-I` 把规则从头插入。

`(policy ACCEPT)` 表示默认的策略。也就意味着，如果没有匹配到任何规则，就会使用默认规则，这里就是全部允许。修改默认规则使用 `-P`，如 `iptables -P INPUT DROP`。

更多示例：

```bash
# 拒绝进入防火墙的所有 ICMP 协议数据包
iptables -I INPUT -p icmp -j REJECT

# 允许防火墙转发除 ICMP 协议以外的所有数据包
iptables -A FORWARD -p ! icmp -j ACCEPT

# 拒绝转发来自 192.168.1.10 主机的数据，允许转发来自 192.168.0.0/24 网段的数据，注意顺序
iptables -A FORWARD -s 192.168.1.11 -j REJECT
iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT

# 丢弃从网口（eth1）进入防火墙本机的源地址为私网地址的数据包
iptables -A INPUT -i eth1 -s 192.168.0.0/16 -j DROP
iptables -A INPUT -i eth1 -s 172.16.0.0/12 -j DROP
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP

# 封堵网段（192.168.1.0/24），两小时后解封
iptables -I INPUT -s 10.20.30.0/24 -j DROP
iptables -I FORWARD -s 10.20.30.0/24 -j DROP
at now +2 hours # 借助 crond 计划任务来完成
at> iptables -D INPUT 1
at> iptables -D FORWARD 1
[1]+  Stopped     at now +2 hours

# 只允许管理员从 202.13.0.0/16 网段使用 SSH 远程登录防火墙主机
iptables -A INPUT -p tcp --dport 22 -s 202.13.0.0/16 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# 允许本机开放从 TCP 端口 20-1024 提供的应用服务
iptables -A INPUT -p tcp --dport 20:1024 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 20:1024 -j ACCEPT

# 允许转发来自 192.168.0.0/24 局域网段的 DNS 解析请求数据包
iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT
iptables -A FORWARD -d 192.168.0.0/24 -p udp --sport 53 -j ACCEPT

# 禁止其他主机 ping 防火墙主机，但是允许从防火墙上 ping 其他主机
iptables -I INPUT -p icmp --icmp-type Echo-Request -j DROP
iptables -I INPUT -p icmp --icmp-type Echo-Reply -j ACCEPT
iptables -I INPUT -p icmp --icmp-type destination-Unreachable -j ACCEPT

# 禁止转发来自 MAC 地址为 00：0C：29：27：55：3F 的和主机的数据包
iptables -A FORWARD -m mac --mac-source 00:0c:29:27:55:3F -j DROP

# 允许防火墙本机对外开放 TCP 端口 20、21、25、110 以及被动模式 FTP 端口 1250-1280
# -m multiport –dport  来指定目的端口及范围
iptables -A INPUT -p tcp -m multiport --dport 20,21,25,110,1250:1280 -j ACCEPT

# 禁止转发源 IP 地址为 192.168.1.20-192.168.1.99 的 TCP 数据包。
# -m –iprange –src-range 指定 IP 范围
iptables -A FORWARD -p tcp -m iprange --src-range 192.168.1.20-192.168.1.99 -j DROP

# 禁止转发与正常 TCP 连接无关的非—syn请求数据包
# -m state 表示数据包的连接状态
# NEW 表示与任何连接无关的
iptables -A FORWARD -m state --state NEW -p tcp ! --syn -j DROP

# 拒绝访问防火墙的新数据包，但允许响应连接或与已有连接相关的数据包
# ESTABLISHED 表示已经响应请求或者已经建立连接的数据包
# RELATED 表示与已建立的连接有相关性的
iptables -A INPUT -p tcp -m state --state NEW -j DROP
iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT

# 只开放本机的 80、FTP(20、21、20450-20480)，放行外部主机发住服务器其它端口的应答数据包，将其他入站数据包均予以丢弃处理
iptables -I INPUT -p tcp -m multiport --dport 20,21,80 -j ACCEPT
iptables -I INPUT -p tcp --dport 20450:20480 -j ACCEPT
iptables -I INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -P INPUT DROP
```

`nat` 表使用：

```bash
# 进入防火墙的数据包目的地址转换，从网口 eth0 进入的数据包，把目的 IP 为 114.115.116.117，端口为 80 的数据包，转到 10.0.0.1
# 这里外网用户访问公网地址 114.115.116.117:80，防火墙再转发到内网地址
iptables -t nat -A PREROUTING -i eth0 -d 114.115.116.117 -p tcp --dport 80 -j DNAT --to-destination 10.0.0.1

# 源地址转换，源地址 10.0.0.0/24 ，从网口 eth1 发出，并把源地址伪装成 111.112.113.114，响应回来后再转换为源地址
# 这里是内网地址 10.0.0.0/24 主机访问外网，会将内网地址伪装成公网 IP 111.112.113.114
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -i eth1 -j SNAT --to-source 111.112.113.114
```

### iptables 配置文件

命令行配置，在关机后会失效，想要持久化，可以修改配置文件：`/etc/sysconfig/iptables`

保存命令：

- CentOS 6 可以使用 `service iptables save`
- CentOS 7 需要使用 `yum install iptables-services` 先安装软件包
  - `iptables-save` 可以输出内存中的 iptables 规则，`iptables-save > /etc/sysconfig/iptables` 就可以把内存中的配置保存到配置文件。下次开机时配置就会保留，相当于执行了 `iptables-restore < /etc/sysconfig/iptables`

## firewalld

firewalld 使用比 iptables 简单，主要区别：

- firewalld 使用区域和服务而不是链式规则。
- 它动态管理规则集，允许更新规则而不破坏现有会话和连接。

使用：

- `systemctl start|stop|enable|disbale firewalld.service` 控制 firewalld 服务
- `firewll-cmd` firewalld 配置命令

iptables 和 firewalld 同时运行会产生冲突。应该关闭其中一个。

```bash
# 查看状态
[root@shcCentOS72VM07 ~]# firewall-cmd --state

# 重新加载配置
[root@shcCentOS72VM07 ~]# firewall-cmd --reload

# 查看具体信息
[root@shcCentOS72VM07 ~]# firewall-cmd --list-all
public (active)                 # public 就是一个区域 zone
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 10.0.0.1 10.0.0.1/24
  services: ssh dhcpv6-client
  ports: 80/tcp 23/tcp
  protocols:
  masquerade: no
  forward-ports:
  sources-ports:
  icmp-blocks:
  rich rules:
[root@shcCentOS72VM07 ~]#  
```

上面的示例，就表示 public zone 绑定了 eth0 网口，source IP 为 `10.0.0.1` `10.0.0.1/24` 的可以访问端口 80，和 23，还可以访问 ssh 和 dhcpv6-client 服务。

```bash
[root@shcCentOS72VM07 ~]# firewall-cmd --zone=public --list-interfaces
eth0
[root@shcCentOS72VM07 ~]# firewall-cmd --list-ports
80/tcp 23/tcp
[root@shcCentOS72VM07 ~]# firewall-cmd --list-services
ssh dhcpv6-client
```

上面的示例是单独查看某一项配置，`--zone=public` 可以省略，public 是默认的。

```bash
[root@shcCentOS72VM07 ~]# firewall-cmd --get-zones  # 查看
block dmz drop external home internal public trusted work
[root@shcCentOS72VM07 ~]# firewall-cmd --get-default-zone  # 查看默认使用的 zone
public
[root@shcCentOS72VM07 ~]# firewall-cmd --get-active-zones  # 查看激活的 zone
public
  interfaces: eth0
  sources: 10.0.0.1 10.0.0.1/24
[root@shcCentOS72VM07 ~]# firewall-cmd --get-default-zone  # 查看默认使用的 zone
public
```

### 添加规则

```bash
[root@shcCentOS72VM07 ~]# firewall-cmd --add-services=https  # 添加服务
success  
```

上面的示例中添加了一个 https 服务，但是这种规则是临时的，如果系统重启，或者执行 `firewall-cmd --reload` 之后就会消失，如果想要持久保存，使用 `--permanent` 参数。

```bash
firewall-cmd --zone=public --add-service=https --permanent
# reload 命令会删除所有运行时配置并应用永久配置。因为 firewalld 动态管理规则集，所以它不会破坏现有的连接和会话
firewall-cmd --reload

# 也可以同时添加到临时和持久规则集
firewall-cmd --zone=public --add-service=https --permanent  # 添加到持久规则集，会在 reload 之后生效
firewall-cmd --zone=public --add-service=https              # 临时规则集，会立即生效
```

```bash
# 查看默认的可用服务
firewall-cmd --get-services

# 要启用或禁用 HTTP 服务： 
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --remove-service=http --permanent

# 允许或者禁用 12345 端口的 TCP 流量。
firewall-cmd --zone=public --add-port=12345/tcp --permanent
firewall-cmd --zone=public --remove-port=12345/tcp --permanent

# 在同一台服务器上将 80 端口的流量转发到 12345 端口
firewall-cmd --zone="public" --add-forward-port=port=80:proto=tcp:toport=12345

# 不同服务器端口转发，要先开启 masquerade
firewall-cmd --zone=public --add-masquerade
# 将 IP 地址为 ：123.456.78.9 的远程服务器上 80 端口的流量转发到 8080 上
firewall-cmd --zone="public" --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=123.456.78.9

# 删除规则 --remove 替换 --add
# firewall-cmd --zone=public --remove-masquerade
```

### 配置文件

- `/usr/lib/FirewallD` 下保存默认配置，如默认区域和公用服务。避免修改它们，因为每次 firewall 软件包更新时都会覆盖这些文件。
- `/etc/firewalld` 下保存系统配置文件。 这些文件将覆盖默认配置。
