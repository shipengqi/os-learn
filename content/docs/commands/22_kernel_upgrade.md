---
title: 内核升级
weight: 22
---

# 内核升级

## rpm 格式内核

查看内核版本：

```bash
[root@pooky ~]# uname -r
3.10.0-862.el7.x86_64
[root@pooky ~]#
```

安装指定的内核版本：`yum install kernel-3.10.0`

升级内核：`yum update`，升级内核和其他软件包

## 源码编译安装内核

安装依赖：

```bash
yum install gcc gcc-c++ make ncurses-devel openssl-devel elfutils-libelf-devel
```

下载并解压缩内核：

- <https://www.kernel.org>
- `tar xvf linux-5.1.10.tar.xz -C /usr/src/kernels`

配置内核参数编译参数，进入内核源码目录：

- `cd /usr/src/kernels/linux-5.1.10`
- `make menuconfig|allyesconfig|allnoconfig`，不同于 `./configure` ，因为内核有很多参数需要配置。
  - menuconfig，会显示一个配置菜单，可以自己选择配置
  - allyesconfig，选择所有配置
  - allnoconfig，所有配置都不选择

使用当前系统内核配置：

- `cp /boot/config-<kernelversion>.<platform> /usr/src/kernels/linux-5.1.10/.config`，例如 `cp /boot/config-config-3.10.0-862.el7.x86_64 /usr/src/kernels/linux-5.1.10/.config`

### 编译安装

- lscpu，查看 cpu，可以查看内核的数量
- `make -j2 all` ，编译，`j2` 表示使用两个内核编译。`all` 表示使用所有编译选项编译
- `make modules_install` 安装内核模块，`make install` 安装内核
- `rebot` 之后新的内核就会引导界面。选择新的内核。
