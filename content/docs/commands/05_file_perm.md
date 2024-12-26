---
title: 文件权限
weight: 5
---

# 文件权限

## 查看文件权限

`ls -l` 可以查看文件的权限信息：

```bash
-rwxr-xr-x.  1 root root   12059203 Jun 12  2019 renewCert
```

- `-rwxr-xr-x` 第一个字符表示类型 `-` 表示普通文件。`rwxr-xr-x` 这 9 个字符表示权限，前面 3 个字符 `rwx` 表示当前用户的权限，中间 3 个字符 `r-x` 表示所属用户组的权限，后 3 个字符 `r-x` 表示其他用户的权限。
- `1` 硬链接的数目。
- `root root` 分别表示文件所属的用户和用户组。

对于普通文件 `r`, `w`, `x` 分别表示：**读**，**写**，**执行**权限。用数字表示就是：`r=4`，`w=2`，`x=1`。
对于目录文件 `r`, `rx`, `wx` 分别表示：进入目录，显示目录内的文件，修改目录内文件的权限。


## 文件类型

- `-`：普通文件。
- `d`：目录文件。
- `b`：块设备文件，存储数据以供系统存取的接口设备，简单说就是硬盘。
- `c`：字符设备文件，串行端口的接口设备，例如键盘、鼠标等等。
- `l`：符号链接（类似 Windows 中的快捷方式）。
- `p`：命名管道文件。
- `s`：套接字文件。

## 修改文件权限

- `chmod`：修改文件，目录的权限。
  - `chmod u+x /test`，`chmod 755 /test`。
- `chown` 修改所属用户，用户组。
  - `chown user1 /test` 修改 `/test` 的所属用户为 user1。
  - `chown :group1 /test` 修改 `/test` 的所属用户组为 group1。
  - `chown user1:group1 /test` 修改 `/test` 的所属用户为 user1，用户组为 group1。
- `chgrp` 只修改所属用户组。
  - `chgrp group1 /test` 修改 `/test` 的所属用户组为 group1。

### chmod

- `u`：用户，对应个 `rwxr-xr-x` 中的前三个字符
- `g`：用户组，对应个 `rwxr-xr-x` 中中间的三个字符
- `o`：其它用户，对应个 `rwxr-xr-x` 中后面的三个字符
- `a`：所有用户(默认)
- `+`：增加权限
- `-`：删除权限
- `=`：设置权限

```bash
chmod u+x file # file 的属主增加执行权限
chmod 751 file # file 的属主分配读、写、执行(7)的权限，给 file 的所在组分配读、执行(5)的权限，给其他用户分配执行(1)的权限
chmod u=rwx,g=rx,o=x file # 上例的另一种形式
chmod =r file # 为所有用户分配读权限
chmod 444 file # 同上例
chmod a-wx,a+r file # 同上例
chmod -R u+r directory # 递归地给 directory 目录下所有文件和子目录的属主分配读的权限
chmod 4755 # 设置 UID，给属主分配读、写和执行权限，给组和其他用户分配读、执行的权限。
```

### 特殊权限

- `SUID`：**仅用二进制于可执行文件**，执行者将具有该文件的所有者的权限。
- `SGID`：执行者将具有该文件的所属用户组的权限。
- `SBIT`：**仅用于目录**，用来阻止非文件的所有者删除文件，仅有自己和 root 才有权力删除。

#### SUID

`ls -ld /usr/bin/passwd`：

```bash
-rwsr-xr-x. 1 root root 27832 Jan 30  2014 /usr/bin/passwd
```

`-rws` 这里的 `s` 表示，在执行 `/usr/bin/passwd` 时，会以文件的属主，也就是 root 身份来执行。

如果要设置 `SUID`，使用 `chmod 4xxx <file>`, 4 加上原有的权限就可以了。

#### SGID

```bash
-rwxr-sr-x. 1 root mlocate 39832 Jan 30  2014 /usr/bin/mlocate*
```

`r-s`，`s` 出现在用户组的 `x` 权限的位置，执行者将具有该文件的所属用户组的权限。

#### SBIT

```bash
drwxrwxrwt.  14 root root 4096 Aug 18 20:11 tmp
```

`rwt` 里的 `t` 就表示该文件仅 root 和自己可以删除。

如果要设置 `SBIT`，使用 `chmod 1xxx <file>`, 4 加上原有的权限就可以了。
