---
title: SSH
weight: 18
---

SSH（Secure Shell）是一种网络协议，用于加密两台计算机之间的通信，并且支持各种身份验证机制。主要用于服务器登录和各种加密通信。

历史上，网络主机之间的通信是不加密的，属于明文通信。这使得通信很不安全。SSH 就是为了解决这个问题而诞生的。

SSH 的软件架构是服务器-客户端模式（Server - Client）。

## SSH 客户端

基本用法：

```bash
# 不指定用户名，使用客户端的当前用户名，作为远程服务器的登录用户名
$ ssh hostname

# 指定用户名
$ ssh user@hostname

# -l 参数，用户名和主机名可以分开
$ ssh -l username host

# -p 指定端口，默认连接服务器的 22 端口
$ ssh -p 8821 foo.com

# 连接服务器，并立刻执行 command，非交互模式
ssh username@hostname command
```

### 客户端连接流程

ssh 连接远程服务器，如果是第一次连接某一台服务器，会出现下面的提示：

```sh
The authenticity of host '16.187.189.94 (16.187.189.94)' can't be established.
ECDSA key fingerprint is SHA256:Vybt22mVXuNuB5unE++yowF7lgA/9/2bLSiO3qmYWBY.
Are you sure you want to continue connecting (yes/no)?
```

表示不认识 `16.187.189.94` 这台服务器的指纹，确认是否需要连接。**服务器指纹** 指的是 SSH 服务器公钥的哈希值。

查看公钥的指纹，可以使用命令：

```bash
$ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
256 Vybt22mVXuNuB5unE++yowF7lgA/9/2bLSiO3qmYWBY   (ECDSA)
```

输入 `yes`，客户端会将服务器的公钥指纹储存在本机的 `~/.ssh/known_hosts` 文件中。每次连接服务器时，通过该文件判断是否为陌生主机。

建立连接后，输入用户名密码就可以登录了。

### 服务器密钥变更

服务器指纹可以防止有人恶意冒充远程主机。如果服务器的密钥发生变更（比如重装了 SSH 服务器），客户端再次连接时，发现公钥指纹不匹配，会显示一段警告信息：

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
77:a5:69:81:9b:eb:40:76:7b:13:04:a9:6c:f4:9c:5d.
Please contact your system administrator.
Add correct host key in /home/me/.ssh/known_hosts to get rid of this message.
Offending key in /home/me/.ssh/known_hosts:36
```

表示公钥指纹和 `~/.ssh/known_hosts` 文件储存的不一样。如果要信任新的公钥，可以使用命令，将原来的公钥指纹从 `~/.ssh/known_hosts` 文件删除：

```bash
$ ssh-keygen -R hostname
```

也可以手动从 `~/.ssh/known_hosts` 文件中删除。

删除以后，重新连接就可以了。

### 配置文件

SSH 客户端的全局配置文件是 `/etc/ssh/ssh_config`，用户个人的配置文件在 `~/.ssh/config`，优先级高于全局配置文件。

## 密钥登录

SSH 默认使用密码登录，每次都必须输入密码，不安全也麻烦。可以通过密钥登录。

SSH 密钥登录采用的是非对称加密，私钥必须私密保存，不能泄漏；公钥则是公开的，可以对外发送。如果数据使用公钥加密，那么只有使用对应的私钥才能解密；如果使用私钥加密（这个过程一般称为“签名”），也只有使用对应的公钥解密。

密钥登录过程：

1. 用户将自己的公钥储存在远程主机上的指定位置。
2. 用户登录的时候，远程主机会向用户发送一段随机字符串。
3. 用户收到随机字符串，用自己的私钥对随机串加密（签名）后，再发给远程主机。
4. 远程主机收到客户端发来的加密签名后，用事先储存的公钥进行解密，然后跟原始数据比较。如果一致，就允许用户登录。

用户可以使用 `ssh-keygen` 密钥对，生成时可以对私钥设置口令（passphrase）。一般密钥对会放在 `$HOME/.ssh/` 目录下，会新生成两个文件：`id_rsa.pub` 和 `id_rsa`。前者是公钥，后者是私钥。

将公钥传送到远程主机上面：

1. 查看是否存在 `$HOME/.ssh/authorized_keys` 文件，不存在则创建该文件：
2. 将生成的 `id_rsa.pub` 文件内容粘贴到 `authorized_keys` 文件。

```bash
[root@SGDLITVM0905 .ssh]# cat authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDaA4G5QBf3IgoA4+4iB8P2JLshyidbUZ4g4NRhMc0T5t7+LHjitnixtfoufDIFUsX3iiJqj/E53vuYtmPZdB9J+V6LMG1Ld2tPFnnzF8/7Xb+IcYLmpkBxFdH30XpuI4Kbt8nZROhTtpQ6/Hj4RLhvYbuR5xNeBkRZQoST2SwP9BzPnCPZCm4Z0X00/ol61hD9n3lEoa7riAUwzS6Sa+8wNjxf1srUJvvAk6URvN1qGZhJGAG2z+fuYcJOlggZ+fTbLOqaY+JZ/m3CSzZ7Yvl44D3JgqkFBQiBaZGvhg3reaOGxv6KrAd+gIR8QiOekZUzP7LWevthe2mYaYDDVxc9
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCbsQz/ePcNnIb7K3DHaULVRwa/GE6wxTqWSCzMzM4tVYlwfJo0bMti4qjsrJz2IVb9lH6Sfu69brr7yHRgcaeCNPKlDtrhH1bxzu4ayjoimcibeKrOfZu6qCJH+JxfwxC9eW6xQDu+Z6xigny19miSd3PhDqHloz4GZKRa2X1fPxB//F+NYuTZJvafjKCZ8eIXcjr0R53tpxdkhKpYIQ4rd8uPtZPjidrEUQcukmngG/LPhoJ6ebQ1zOmSPtXP3kVLATrrlQRWZlHqSbFJpnJHclhN8NgahtyR7ad63suHxBKLAa71QjNvSRfa0QUQ4/uqH9zBT5hbGd1IXSA3Sq1l root@shcAimeeCOS72.hpeswlab.net
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCaDa+Nl8NK7bagAp9vbuKQyh00y6UHOdNiugwc6Mcw7qsCPpV257LZDLHyrXjdQpqFy135AZTUPGozPh8TH/W6sI8jWgCl9+mYup7+/i1wwjIxnkppKV7nqrWVxEXt5LLny7M0+HCw4pC9jUlFGC5MZVV6mVyI8oTeRkZddrja0c/65Et0+ksiiT2A4yIDoKFslv5vDAYolRJx13a4ESWjsCpGQOvrxtsQqpISm0yaZO8Ha+1a6tjniEWReZNGcOajrLP4V+lv8Qi0Cl/iNIkpD+fnRFphY6RgYC5UrLFYOBCgaKgQhUBK2BOcjM1dR1IqCV9YRv+N4PLhmLweyvQJ root@autovmCOS76VM09.hpeswlab.net
```

然后再登录，就不需要输入密码了。

ssh 也提供了命令来拷贝公钥，`ssh-copy-id`：

```bash
# -i 指定公钥文件，拷贝到对应的账户下的 authorized_keys 文件中
ssh-copy-id -i ~/.ssh/id_rsa.pub root@16.187.189.94
```

如果不能登录，检查远程主机的 `/etc/ssh/sshd_config` 文件，去掉如下三行注释：

```sh
#RSAAuthentication yes
#PubkeyAuthentication yes
#AuthorizedKeysFile     .ssh/authorized_keys
```

重启 sshd 服务：`systemctl restart sshd`。

### ssh-agent

私钥设置了密码以后，每次使用都必须输入密码，非常麻烦。`ssh-agent` 命令就是为了解决这个问题，它让用户在整个 Bash 对话（session）之中，只在第一次使用 SSH 命令时输入密码，然后将私钥保存在内存中，后面都不需要再输入私钥的密码了。

使用步骤：

第一步，新建一个命令行对话：

```bash
# bash 可以换成 zsh，fish 等
$ ssh-agent bash
```

也可以在当前对话启用 `ssh-agent`：

```bash
$ eval `ssh-agent`
```

第二步，在新建的 Shell 对话里面，使用 `ssh-add` 命令添加默认的私钥：

```bash
# 添加默认的私钥
$ ssh-add
Enter passphrase for /home/you/.ssh/id_dsa: ********
Identity added: /home/you/.ssh/id_dsa (/home/you/.ssh/id_dsa)

# 添加指定的私钥
$ ssh-add other-key-file 
```

添加私钥时，要输入密码，之后在这个会话中，就不再需要输入私钥密码了。

第三步，使用 ssh 命令正常登录远程服务器。

```bash
$ ssh hostname
```

退出 `ssh-agent`：

```bash
$ ssh-agent -k
```

也可以直接关闭 shell 会话。


## SSH 服务器

### 配置文件

服务端配置文件 `/etc/ssh/sshd_config`。

每行都是配置项和对应的值，配置项的大小写不敏感，与值之间使用空格分隔。注释只能放在一行的开头，不能放在一行的结尾。

## scp 和 sftp

### scp

scp（secure copy），是 SSH 提供的一个客户端程序，用来在两台主机之间加密传送文件。

```bash
# 本地文件复制到远程
$ scp file.txt user@hostname:/remote/directory

# 将本机整个目录拷贝到远程目录下
$ scp -r local/directory user@hostname:/remote_directory/

# 将本机目录下的所有内容拷贝到远程目录下
$ scp -r local/directory/* user@hostname:/remote_directory/

# 远程文件复制到本地
$ scp user@hostname:/remote/file.txt /local/directory

# 拷贝远程目录下的所有内容，到本机目录下
$ scp -r user@hostname:directory/SourceFolder TargetFolder
```

### sftp

`sftp` 是 SSH 提供的一个客户端应用程序，主要用来安全地访问 FTP。

```bash
$ sftp user@hostname
```

验证成功以后，就会出现 FTP 的提示符 `sftp>`：

```bash
$ sftp root@example.com
root@example.com's password:
Connected to example.com.
sftp>
```

然后就可以输入 FTP 命令。
