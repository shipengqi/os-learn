---
title: FTP
weight: 19
---

# FTP

ftp 协议的全称是文件传输协议。

FTP 采用两个 TCP 连接来传输一个文件：

- **控制连接**：服务器以被动的方式，打开众所周知用于 FTP 的端口 21，客户端则主动发起连接。该连接将命令从客户端传给服务器，并传回服务器的应答。常用的命令有：
  - `list` 获取文件目录
  - `reter` 取一个文件
  - `store` 存一个文件
- **数据连接**：每当一个文件在客户端与服务器之间传输时，就创建一个数据连接。

## FTP 的两种模式

两种模式都是站在 FTP 服务器的角度来说的。

### 主动模式（PORT）

客户端随机打开一个大于 1024 的端口 N，向服务器的命令端口 21 发起连接，同时开放 N+1 端口监听，并向服务器发出 `port N+1` 命令，由服务器从自己的数据端口 20，主动连接到客户端指定的数据端口 N+1。

### 被动模式（PASV）

当开启一个 FTP 连接时，客户端打开两个任意的本地端口 N（大于 1024）和 N+1。第一个端口连接服务器的 21 端口，提交 PASV 命令。然后，服务器会开启一个任意的端口 P（大于 1024），返回 `227 entering passive mode` 消息，里面有 FTP 服务器开放的用来进行数据传输的端口。客
户端收到消息取得端口号之后，会通过 N+1 号端口连接服务器的端口 P，然后在两个端口之间进行数据传输。

## vsftpd

vsftpd 实现了 ftp 协议。

安装：`yum install vsftpd ftp` `ftp` 是客户端。
启动：`systemctl start vsftpd.service`
连接 vsftpd：`ftp <host>`

```bash
[root@SGDLITVM0905 ~]# ftp localhost
Connected to localhost (127.0.0.1).
220 (vsFTPd 3.0.2)
Name (localhost:root): ftp     # ftp 匿名账户，不需要密码
331 Please specify the password.
Password:                      # 这里密码为空，直接回车
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (127,0,0,1,190,29).
150 Here comes the directory listing.
drwxr-xr-x    2 0        0               6 Apr 01 04:55 pub
226 Directory send OK.
ftp> quit
221 Goodbye.
[root@SGDLITVM0905 ~]# ls /var/ftp/      # 匿名账户登录后，会在该目录下
pub                                      # 需要共享的文件放在该目录下，就可以被匿名账户访问
```

vsftpd 支持系统的本地用户：

```bash
[root@SGDLITVM0905 ~]# useradd user1
[root@SGDLITVM0905 ~]# echo 123 | passwd --stdin user1
Changing password for user user1.
passwd: all authentication tokens updated successfully.
[root@SGDLITVM0905 ~]# ftp localhost
Connected to localhost (127.0.0.1).
220 (vsFTPd 3.0.2)
Name (localhost:root): user1
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -l                                     # 登录后会进入到 user1 的 home 目录下
227 Entering Passive Mode (127,0,0,1,229,126).
150 Here comes the directory listing.
226 Directory send OK.
ftp> quit
221 Goodbye.
[root@SGDLITVM0905 ~]# su - user1
[user1@SGDLITVM0905 ~]$ ls
```

### vsftpd 配置

主要配置文件：

- `/etc/vsftpd/vsftpd.conf` 主配置文件
- `/etc/vsftpd/ftpusers` 用户黑名单，不允许登录 ftp 服务器的用户，例如 `root` 权限太大，可以访问到所有文件，所以被禁止使用登录。
- `/etc/vsftpd/user_list` 用户白名单

`vsftpd.conf` 配置：

```bash
# 允许匿名用户登陆，匿名用户使用的登陆名为 ftp 或 anonymous
# 匿名用户家目录 /var/ftp，且只能下载不能上传
anonymous_enable=YES

# 允许本地用户登陆
local_enable=YES

# 本地用户是否可以写入
write_enable=YES

# 设置 FTP 服务器建立连接所监听的端口，默认值为 21
listen_port=21

# 指定 FTP 使用 20 端口进行数据传输，默认值为 YES
connect_from_port_20=YES

# 是否启用 vsftpd.user_list 文件
userlist_enable=YES

# 决定 vsftpd.user_list 文件中的用户是否能够访问 FTP 服务器。若设置为 YES，则 vsftpd.user_list 文件中的用户不允许访问 FTP，若设置为 NO，则只有 vsftpd.user_list 文件中的用户才能访问 FTP。
userlist_deny=YES
```

上面的配置中 `local_enable`，如果 selinux 为 `enforcing` 需要检查 `ftp_home_dir` 是否为 `on`。

```bash
[root@SGDLITVM0905 ~]# getsebool -a | grep ftpd
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
[root@SGDLITVM0905 ~]# setsebool -P ftpd_use_nfs 1
[root@SGDLITVM0905 ~]# getsebool -a | grep ftpd
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> on
ftpd_use_passive_mode --> off
```

修改后使用 `systemctl reload vsftpd` 使配置生效。

登录另一台机器，使用 ftp 上传文件：

```bash
[root@SGDLITVM0906 ~]# ftp 16.187.191.150     # 连接 vsftpd
Connected to 16.187.191.150 (16.187.191.150).
220 (vsFTPd 3.0.2)
Name (16.187.191.150:root): user1
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls                               # 查看 user1 的 home 目录
227 Entering Passive Mode (16,187,191,150,84,31).
150 Here comes the directory listing.
226 Directory send OK.
ftp> !ls                              # 查看当前主机的本地目录
anaconda-ks.cfg  original-ks.cfg  upload.txt  workspace  workspace2
ftp> put upload.txt                   # 使用 put 命令上传文件
local: upload.txt remote: upload.txt
227 Entering Passive Mode (16,187,191,150,100,251).
150 Ok to send data.
226 Transfer complete.
ftp> get <file>                       # 使用 get 下载文件
```

## 虚拟用户

- `guest_enable=YES`
- `guest_username=vuser`
- `user_config_dir=/etc/vsftpd/vuserconfig`
- `allow_writeable_chroot=YES`
- `pam_service_name=vsftpd.vuser`
