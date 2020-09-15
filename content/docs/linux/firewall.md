# 防火墙

防火墙分为两类：

- 软件防火墙，CentOS 6 的默认防火墙是 iptables，CentOS 7 的默认防火墙是 firewalld，底层都是使用 netfilter。
  - 包过滤防火墙，主要用于数据包的过滤
  - 应用层防火墙
- 硬件防火墙，

## iptables

iptables 包含四个规则表：`filter` `nat` `mangle` `raw`

iptables 的规则链：

- `INPUT` `OUTPUT` `FORWARD`
- `PREROUTING` `POSTROUTING`

## firewalld
