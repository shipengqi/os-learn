# DNS

DNS（Domain Name System 的缩写）域名系统。
FQDN（Full Qualified Domain Name）完全限定域名。FQDN 由两部分组成:主机名和域名。例如 `www.baidu.com`，主机名是 `www`，主机位于域名 `baidu.com` 中。

## BIND

DNS 主要靠 BIND 这个软件来实现。

- 安装 `yum install bind bind-utils`
- 启动 `systemctl start named.service`

### 配置文件

主配置文件 `/etc/named.conf`。
