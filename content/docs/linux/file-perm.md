# 文件权限

## 查看文件权限

`ls -l` 可以查看文件的权限信息

```bash
-rwxr-xr-x.  1 root root   12059203 Jun 12  2019 renewCert
```

- `-rwxr-xr-x` 第一个字符表示类型 `-` 表示普通文件。`rwxr-xr-x` 表示权限，前面三个 `rwx` 表示当前
用户的权限，中间三个 `r-x` 表示所属用户组的权限，后三个 `r-x` 表示其他用户的权限。
- `root root` 分别表示文件所属的用户和用户组。

对于普通文件 r, w, x 分别表示 读，写，执行权限。
对于目录文件 r, rx, wx 分别表示 进入目录，显示目录内的文件，修改目录内文件的权限。

用数字表示：`r=4`，`w=2`，`x=1`。

## 文件类型

- `-` 普通文件
- `d` 目录文件
- `b` 块设备文件
- `c` 字符设备文件
- `l` 符号链接（类似 windows 的快捷方式）
- `f` 命名管道
- `s` 套接字文件

## 修改文件权限

- chmod 修改文件，目录的权限 `chmod u+x /test`，`chmod 755 /test`。
- chown 修改所属用户，用户组 `chown user1 /test` 修改 `/test` 的所属用户为 user1，`chown :group1 /test` 修改 `/test` 的所属用户组为 group1。
- chgrp 只修改所属用户组 `chgrp group1 /test` 修改 `/test` 的所属用户组为 group1。

### chmod

- `u`，用户，对应个 `rwxr-xr-x` 中的前三个字符
- `g`，用户组，对应个 `rwxr-xr-x` 中中间的三个字符
- `o`，其它用户，对应个 `rwxr-xr-x` 中后面的三个字符
- `a`，所有用户(默认)
- `+`，增加权限
- `-`，删除权限
- `=`，设置权限

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
