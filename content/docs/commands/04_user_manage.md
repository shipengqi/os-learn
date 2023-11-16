---
title: 用户管理
weight: 4
---

# 用户管理

## 用户

Linux 系统的用户存放在 `/etc/passwd` 文件中。`/etc/shadow` 是用户密码相关的文件。

- `useradd`：创建用户。`useradd pooky` 添加了一个用户 pooky。 `id pooky` 验证 pooky 用户是否存在。root 用户的 uid 是 `0`。普通用户的 uid 如果改为 0，那就会具有 root 用户的权限。
  - `-g`：指定用户组。`useradd -g group1 user2` 创建用户 user2 并加入到用户组 group1。添加用户时，不指定用户组，就会默认创建和用户名同名的用户组。
- `userdel`：删除用户。`userdel pooky` 删除用户 pooky。
  - `-r`：会删除用户的 home 目录，否则会保留。
- `passwd`：修改密码。 `passwd pooky` 设置用户 pooky 的密码。**直接输入 `passwd` 会修改当前用户的密码**。
- `usermod`：修改用户属性
  - `-d`：修改用户 home 目录。`usermod -d /hmoe/pookyh pooky` 把 pooky 的 home 目录改为 `pookyh`。
  - `-g`：修改用户组。`usermod -g group1 pooky` 将 pooky 的用户组改为 group1。
- `chage`：修改用户密码的过期信息。

## 用户组

Linux 用户组的信息存放在 `/etc/group` 文件中。

- `groupadd`：创建用户组 `groupadd group1`
- `groupdel`：删除用户组
  
## 用户切换

- `su` 切换用户 `su - user1`，切换到用户 user1。`-` 表示同时切换用户环境。`exit` 退回上个用户。
- `sudo` 以 root 用户身份执行命令。

### visudo

`su` 切换到 `root` 是需要密码的，如果普通用户想要使用 root 权限，就需要告诉他密码，这是有风险的。因此，就出现了 `sudo`。

`sudo` 命令，可以使 root 权限，有针对性授权给指定的普通用户，而且不需要告诉普通用户 root 的密码。使用 `sudo` 的前提就是通过配置 `/etc/sudoers` 文件来授权。

`visudo` 就是编辑 `/etc/sudoers` 文件的命令。

```bash
## Allows people in group pooky to run all commands
%pooky     ALL=(ALL)      ALL

## Without a password
%pooky     ALL=(ALL)      NOPASSWD: ALL

## Allows members of the users group to mount and unmount the cdrom as root
%users     ALL=/sbin/mount /mnt/cdrom, /sbin/unmount /mnt/cdrom

## Allows members of the usrs froup to shutdown this system
%users     localhost=/sbin/shutdown -h now
```

上面的示例，`%pooky` 表示 pooky 用户组，如果要表示 pooky 用户，则去掉 `%`。

- `ALL=(ALL)` 表示在哪些主机可以执行哪些命令。Linux 可以通过 ssh 远程登录，也可以本地登录（localhost）。ALL 表示远程和本地登录都允许。
- `localhost=/sbin/shutdown -h no` 表示本地主句只允许一条命令 `/sbin/shutdown -h now`。
  - 如果多条命令用逗号分隔，`ALL=/sbin/mount /mnt/cdrom, /sbin/unmount /mnt/cdrom` 表示 `/sbin/mount /mnt/cdrom` 和 `/sbin/unmount /mnt/cdrom` 这两条命令允许执行。
- `NOPASSWD: ALL` 表示普通用户执行管理员权限的密码不需要密码。只有一个 `ALL` 表示需要密码。

## 用户和用户组的配置文件

用户配置文件 `/etc/passwd`：

```bash
# 用户名:是否需要密码验证:UID:GID:注释:用户 home 目录:用户登录使用的命令解释器
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
# ...
```

- `root:x` 中 `x` 表示此用户有密码，真正的密码保存在 `/etc/shadow` 文件中。如果没有 `x`，表示此用户没有密码。
- 可以直接编辑这个文件来新建用户。比如添加一行 `user1:x:1000:1000::/home/user1:/bin/bash`，添加了 user1 用户。用户 home 目录需要手动创建。
- 可以修改 UID 来修改权限，比如把 UID 改为 0，用户就会拥有 root 权限。
- `/sbin/nologin` 表示不允许登录。

`/etc/shadow` 保存用户和密码信息，此文件只有 root 用户可以浏览和操作，最大限度地保证了密码的安全：

```bash
# 用户名:加密的密码:
root:$6$PcVZ4yj4vlMjqmkL$RUHwggR7gPD0SnjTF1WnStHi2If0hSJnc4M/oVTfD0omJxVGhQgnQhBKRNPiwcBSeL72IerSphnEVdaomgjx./::0:99999:7:::
bin:*:17492:0:99999:7:::
daemon:*:17492:0:99999:7:::
# ...
```

- 第二个部分的密码即使相同，但是显示也会不同，更加安全。

用户组配置文件 `/etc/group`：

```bash
# 用户组名:是否需要密码验证:GID:其他组设置
root:x:0:
bin:x:1:
daemon:x:2:
# ...
wheel:x:10:admin
cdrom:x:11:
mail:x:12:postfix
# ...
```

- 其他组设置，`mail:x:12:postfix` 表示 `psotfix` 属于一个其他的组，也属于 mail 组。
