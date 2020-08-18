# 网络管理

## 查看网络状态

### net-tools

- ifconfig 查看网卡配置
- route 查看网关，常用命令是 `route -n`，因为 `route` 会尝试将 IP 转换为域名，可能耗时较长。
- netstat

### iproute2

- ip
- ss
