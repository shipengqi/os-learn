---
title: SELinux
weight: 11
---

安全增强型 Linux（Security-Enhanced Linux）简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。

没有使用 SELinux 的 Linux 是通过用户的权限和文件权限来做安全控制，这叫做 DAC（自主访问控制）。最致命问题是，root 用户不受任何管制，系统上任何资源都可以无限制地访问。

使用了 SELinux 的 Linux 中，会给用户，文件，进程都打上一个标签。一个资源是否能被访问，要用户，文件和进程的标签一致，并且类型一致，才被允许访问。这叫做 MAC（强制访问控制）。

SELinux 保证了安全，但是会降低机器性能，因为处理文件的时候要额外处理 SELinux 的权限，一般生产环境下 SELinux 是关闭的。

## 查看 SELinux

- `getenforce` 查看 SELinux 的模式
- `/usr/sbin/sestatus`

```bash
[root@pooky ~]# getenforce
Permissive
[root@pooky ~]# /usr/sbin/sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31

```

SELinux 会有三种模式：

1. `enforcing`：强制模式。违反 SELinux 规则的行为将被阻止并记录到日志中。
2. `permissive`：宽容模式。违反 SELinux 规则的行为只会记录到日志中。一般为调试用。
3. `disabled`：关闭 SELinux。

可以在配置文件 `/etc/selinux/config` 中配置 `SELINUX` 的字段。

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```

`SELINUXTYPE` 是 SELinux 的策略类型。有三种策略，分别是：

1. `targeted`：对大部分网络服务进程进行管制。系统默认。
2. `minimum`：以 `targeted` 为基础，仅对选定的网络服务进程进行管制。一般不用。
3. `mls`：多级安全保护。对所有的进程进行管制。这是最严格的策略，配置难度非常大。一般不用。

在配置文件中修改模式需要重启系统。

## 查看 SELinux 的标签

- `ps -Z` 查看进程的标签
- `ls -Z` 查看文件或目录的标签
- `id -Z` 查看用户的标签

```bash
[root@pooky ~]# ps -Z
LABEL                             PID TTY          TIME CMD
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 29783 pts/3 00:00:00 bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 30279 pts/3 00:00:00 ps
[root@pooky ~]# id -Z
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@pooky ~]# ls -Z
-rw-------. root root system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 app.js
-rwxr--r--. root root unconfined_u:object_r:admin_home_t:s0 a.sh
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 cdf
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 cdf-components
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 certificate-controller
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 chatops-docker
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Desktop
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Documents
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Downloads
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 go1.13.3.linux-amd64.tar.gz
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 go-fips
-rw-------. root root unconfined_u:object_r:admin_home_t:s0 go-fips.tar
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 golang

```

`admin_home_t` 就表示只有管理员访问 home 目录是才能访问。

## 关闭 SELinux

- `setenforce 0` 可以临时的设置 SELinux 的模式为 `enforcing`，重启之后就恢复了。
- `/etc/selinux/sysconfig`
