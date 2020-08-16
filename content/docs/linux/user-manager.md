# 用户管理

## 用户管理
用户会记录在 `/etc/passwd` 文件中。`/etc/shadow` 是用户密码相关的文件。

- useradd 创建用户。 `useradd pooky` 添加了一个用户 pooky。 `id pooky` 验证 pooky 用户是否存在。
  root 的 uid 是 `0`。普通用户的 uid 如果改为 0，那就会具有 root 用户的权限。
  - `-g` 指定用户组。`useradd -g group1 user2` 创建用户 user2 并加入到用户组 group1。添加用户时，不指定用户组，就会默认创建
  用户同名的用户组。
- userdel 删除用户。`userdel pooky` 删除用户 pooky。
  - `-r` 选项会删除用户的 home 目录，否则会保留。
- passwd 修改密码。 `passwd pooky` 设置用户 pooky 的密码。直接输入 `passwd` 会修改当前用户的密码。
- usermod 修改用户属性
  - `-d` 修改用户 home 目录。`usermod -d /hmoe/w1 w` 把 home 目录 `w` 改为 `w1`。 
  - `-g` 修改用户组。`usermod -g user1 group1` 将 user1 的用户组改为 group1。
- chage 修改用户密码的过期信息。

## 用户组

- groupadd 创建用户组 `groupadd group1`
- groupdel 删除用户组
  
## 用户切换
- su 切换用户 `su - user1`，切换到用户 user1。`-` 表示同时切换用户环境。
- sudo 以其他用户身份执行命令
  - visudo 设置需要使用 sudo 的用户（组）


`exit` 退回上个用户。