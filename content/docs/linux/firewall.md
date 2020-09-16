# 防火墙

防火墙分为两类：

- 软件防火墙，CentOS 6 的默认防火墙是 iptables，CentOS 7 的默认防火墙是 firewalld，底层都是使用内核中的 netfilter。
  - 包过滤防火墙，主要用于数据包的过滤，数据包转发。
  - 应用层防火墙，可以控制应用程序的具体的行为。
- 硬件防火墙，

## iptables

表（tables）提供特定的功能，iptables 包含四个规则表：

- `filter` 包过滤
- `nat` 网络地址转换
- `mangle` 包重构(修改)
- `raw` 数据跟踪处理

链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一条或数条规则。当一个数据包到达一个链时，iptables 就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据该条规则所定义的方法处理该数据包；否则 iptables 将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables 就会根据该链预先定义的默认策略来处理数据包。

iptables 的规则链：

- `INPUT` 接收的的数据包使用此规则链中的策略
- `OUTPUT` 发送的数据包使用此规则链中的策略
- `FORWARD` 转发数据包时使用此规则链中的策略
- `PREROUTING` 对数据包作路由选择前使用此链中的规则
- `POSTROUTING` 对数据包作路由选择后使用此链中的规则

`iptables [-t 表] 命令选项 [规则链] 规则`，`-t` 默认使用的是 `filter`。

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

- `-p proto` 匹配网络协议：tcp、udp、icmp
- `--icmp-type type` 匹配ICMP类型，和-p icmp配合使用。注意有两根短划线
- `-s source-ip` 匹配来源主机（或网络）的IP地址
- `--sport port#` 匹配来源主机的端口，和 `-s source-ip` 配合使用。
- `-d dest-ip` 匹配目标主机的 IP 地址
- `--dport port#` 匹配目标主机（或网络）的端口，和 `-d dest-ip` 配合使用。

- ACCEPT 接收数据包
- DROP 丢弃数据包
- REDIRECT 将数据包重新转向到本机或另一台主机的某一个端口，通常功能实现透明代理或对外开放内网的某些服务
- SNAT 源地址转换
- DNAT 目的地址转换
- MASQUERADE IP 伪装
- LOG 日志功能

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

上面的示例，`INPUT` 链配置了两条规则，分别是接收 IP 为 `10.0.0.2` 的数据包，和丢弃 `10.0.0.2` 的数据包。那么 `10.0.0.2` 的数据包能不能进来？**可以**。数据包会先匹配前面的 `ACCEPT 10.0.0.2` 的规则，这个时候数据包就进入了系统，所以**规则顺序很重要**。可以使用 `-I` 把规则从头插入。

`(policy ACCEPT)` 表示默认的策略。也就意味着，如果没有匹配到任何规则，就会使用默认规则，这里就是全部允许。修改默认规则使用 `-P`，如 `iptables -P INPUT DROP`。

## firewalld
