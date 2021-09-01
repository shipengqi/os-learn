# NFS

NFS 在 Linux 中会默认安装。

NFS 的主配置文件：`/etc/exports`。

```bash
[root@SGDLITVM0905 ~]# cat /etc/exports
/data/share *(rw,sync,all_squash)
/data/share2 10.222.77.0/24(rw,sync,insecure,no_subtree_check,no_root_squash)
/var/vols/itom/core *(rw,sync,anonuid=1999,anongid=1999,root_squash)
...
```

- `/data/share2` 是要共享目录，而且必须要存在。
- `10.222.77.0/24` 表示允许 IP 在该 `10.222.77.0/24` 网段的客户端挂载。设置为 `*` 即允许所有客户端挂载，如 `/home *(ro,sync,insecure,no_root_squash)`，设置 `/home` 目录允许所有客户端只读挂载。
- 一个目录可以配置多个 IP 权限，`/data/share 10.0.0.1(rw) 10.0.0.2(ro)`，表示 `10.0.0.1` 访问时是读写权限，`10.0.0.2` 访问时是只读权限，注意 IP 和括号之间不要有空格，否则会当做两个配置。

目录设置：

- `ro` 只读访问
- `rw` 读写访问
- `sync` 确保数据写入时从内存同步到磁盘
- `async` 将数据先保存在内存缓冲区中，必要时才写入磁盘
- `secure` nfs 通过 1024 以下的安全 TCP/IP 端口发送
- `insecure` nfs 通过 1024 以上的端口发送
- `root_squash` 将 root 用户及所属组都映射为匿名用户或用户组（默认设置）
- `no_root_squash` 与 rootsquash 取反
- `anonuid=xxx` 将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）
- `anongid=xxx` 将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）
- `wdelay` 检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）
- `no_wdelay` 若有写操作则立即执行，应与 sync 配合使用
- `hide` 在 nfs 共享目录中不共享其子目录
- `no_hide` 共享 nfs 目录的子目录
- `subtree_check` 如果共享 `/usr/bin` 之类的子目录时，强制 nfs 检查父目录的权限（默认）
- `no_subtree_check` 不检查父目录权限
- `all_squash` 将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（`nfsnobody`）
- `no_all_squash` 与 `all_squash` 取反（默认设置）

```bash
[root@SGDLITVM0905 ~]# showmount -e localhost      # 查看指定主机有哪些共享目录
Export list for localhost:
/data/share *
/var/vols/demochart/demochart-db-volume   *
/var/vols/demochart/demochart-data-volume *
/var/vols/demochart/demochart-conf-volume *
[root@SGDLITVM0905 ~]# ls /data/share
testnfs
[root@SGDLITVM0905 ~]# mount -t nfs localhost:/data/share /mnt  # 挂载指定主机 localhost 下的共享目录 /data/share 到 /mnt
[root@SGDLITVM0905 ~]# ls /mnt
testnfs
[root@SGDLITVM0905 ~]# cd /mnt
[root@SGDLITVM0905 ~]# touch testnfs2
```
