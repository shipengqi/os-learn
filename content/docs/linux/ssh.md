# SSH

telnet 是明文传输，不安全。所以应该使用 SSH 来远程连接。

```bash
ssh [-p port] user@host
```

`user` 省略的话，就会使用当前登录的用户。

## 配置文件

- 客户端配置文件：`/etc/ssh/ssh_config`
- 服务端配置文件：`/etc/ssh/sshd_config`
  - Port 22 默认端口
  - PermitRootLogin yes 是否允许 root 登录
  - AuthorizedKeysFile .ssh/authorized_keys
  
## 中间人攻击

SSH 之所以能够保证安全，原因在于它采用了公钥加密。

整个过程是这样的：

1. 远程主机收到用户的登录请求，把自己的公钥发给用户。
2. 用户使用这个公钥，将登录密码加密后，发送回来。
3. 远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。

这个过程本身是安全的，但是实施的时候存在一个风险：如果有人截获了登录请求，然后冒充远程主机，将伪造的公钥发给用户，那么用户很难辨别真伪。因为不像 https 协议，SSH 协议的公钥是没有证书中心（CA）公证的，也就是说，都是自己签发的。

可以设想，如果攻击者插在用户与远程主机之间（比如在公共的 wifi 区域），用伪造的公钥，获取用户的登录密码。再用这个密码登录远程主机。这种风险就是的"中间人攻击"（Man-in-the-middle attack）。

所以第一次登录远程主机，系统会出现下面的提示：

```sh
$ ssh root@16.187.189.94
The authenticity of host '16.187.189.94 (16.187.189.94)' can't be established.
ECDSA key fingerprint is MD5:b0:4f:bb:ef:80:aa:07:5f:08:f2:81:5f:5f:9d:73:4f.
Are you sure you want to continue connecting (yes/no)?
```

这段话的意思是，无法确认 host 主机的真实性，只知道它的公钥指纹，是否继续？

**公钥指纹**是指公钥长度较长（这里采用 RSA 算法，长达 1024 位），很难比对，所以对其进行哈希计算，将它变成一个 128 位的指纹。例如
`b0:4f:bb:ef:80:aa:07:5f:08:f2:81:5f:5f:9d:73:4f`，再进行比较，就容易多了。

远程主机必须在自己的网站上贴出公钥指纹，以便用户自行核对。

如果用户决定接受这个远程主机的公钥，输入 yes。系统会出现一句提示，然后，会要求输入密码。

## 公钥登录

使用密码登录，每次都必须输入密码。可以通过公钥登录：

1. 用户将自己的公钥储存在远程主机上。
2. 登录的时候，远程主机会向用户发送一段随机字符串。
3. 用户收到随机字符串，用自己的私钥加密后，再发给远程主机。
4. 远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录 shell，不再要求密码。

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

## 远程拷贝

远程拷贝文件使用 `scp`
