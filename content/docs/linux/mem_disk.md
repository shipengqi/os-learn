# 内存和磁盘

## 查看内存和磁盘的使用率

### 查看内存的使用率

- free
- top

```bash
[root@pooky ~]# free
              total        used        free      shared  buff/cache   available
Mem:       16266780     1091532     7315128      157456     7860120    14383564
Swap:             0           0           0
[root@pooky ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:          15885        1065        7144         153        7675       14046
Swap:             0           0           0
[root@pooky ~]# free -g
              total        used        free      shared  buff/cache   available
Mem:             15           1           6           0           7          13
Swap:             0           0           0

```

`-m` 和 `-g` 分别是指以 `MB` 和 `GB` 为单位显示。`buff/cache` 是进程使用的缓存。`available` 是释放掉 cache 后的可用内存。

`Swap` 是交换分区，在内存不够用时，系统会自动把一部分暂时不用的内存移动到 swap 中。swap 交换分区使用的是磁盘。进程应该尽量不实用 swap，因为硬盘的读取是非常慢的。

**交换分区**在 windows 中叫做**虚拟内存**。

### 查看磁盘的使用率

- fdisk 查看磁盘，磁盘分区。（`parted -l` 和 `fdisk -l` 类似）
- df
- du

#### fdisk

```bash
[root@pooky ~]# fdisk -l # 查看磁盘

Disk /dev/sda: 214.7 GB, 214748364800 bytes, 419430400 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00078ef0

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     4147199     2072576   83  Linux
/dev/sda2         4147200   125788159    60820480   8e  Linux LVM
/dev/sda3       125788160   419430399   146821120   83  Linux

Disk /dev/mapper/rhel-root: 205.1 GB, 205084688384 bytes, 400556032 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rhel-swap: 6442 MB, 6442450944 bytes, 12582912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop0: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 2147 MB, 2147483648 bytes, 4194304 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/docker-253:0-201326881-pool: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 65536 bytes / 65536 bytes
```

可以看到磁盘文件 `/dev/sda`，虚拟机一般都是 `sd` 开头。

`419430400 sectors` 表示磁盘 `/dev/sda` 可以分为多少个扇区。`Units = sectors of 1 * 512 = 512 bytes` 表示一个扇区的大小是 512 字节。

```bash
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     4147199     2072576   83  Linux
```

表示 `/dev/sda1` 这个分区的大小是从第 2048 个扇区到第 4147199 扇区。`Boot` 为 `*` 表示当前的系统是在 `/dev/sda1` 这个分区启动的。

```bash
[root@pooky ~]# ls -l /dev/sd?
brw-rw----. 1 root disk 8, 0 Jun 22 10:20 /dev/sda
[root@pooky ~]# ls -l /dev/sd??
brw-rw----. 1 root disk 8, 1 Jun 22 10:20 /dev/sda1
brw-rw----. 1 root disk 8, 2 Jun 22 10:20 /dev/sda2
brw-rw----. 1 root disk 8, 3 Jun 22 10:20 /dev/sda3
```

`/dev/sda` 的类型是 `b` 也就是块设备。 `disk 8, 0` 中 8 表示主设备号（表示磁盘使用的驱动程序），0 是从设备号（确定访问地址）。

`sda1` `sda2` `sda3` 是磁盘 `sda` 的分区。

#### df 和 du

df 可以看做是 fdisk 的补充。fdisk 无法看到分区挂载到了那个目录下。

```bash
[root@pooky ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root  191G   89G  103G  47% /
devtmpfs               7.8G     0  7.8G   0% /dev
tmpfs                  7.8G     0  7.8G   0% /dev/shm
tmpfs                  7.8G  154M  7.7G   2% /run
tmpfs                  7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/sda1              2.0G  170M  1.9G   9% /boot
tmpfs                  1.6G   12K  1.6G   1% /run/user/42
tmpfs                  1.6G     0  1.6G   0% /run/user/0
```

查看一个文件的大小：

```bash
[root@pooky ~]# ls -lh /etc/passwd
-rw-r--r--. 1 root root 2.3K May  9  2018 /etc/passwd
[root@pooky ~]# du /etc/passwd
4 /etc/passwd
[root@pooky ~]# du -h 2019.02.0054.zip
1.9G 2019.02.0054.zip
```

`du` 显示的实际占用的空间，默认单位是 `KB`。`ls -l` 查看的文件大小包含了部分空洞的空间。

## Linux 常见的文件系统

- ext4（CentOS 6 默认的文件系统）
- xfs（CentOS 7 默认的文件系统）
- NTFS（需要版权，额外安装软件）

### ext4

基本结构：

- 超级块，记录整个文件系统的信息，包括多少个文件等等。
- 超级块副本，超级块的备份，用来恢复超级块。
- inode，记录每一个文件，文件名称，大小，编号，权限等。
- 数据块 （datablock）

```bash
[root@pooky ~]# ls -i
 67162193 anaconda-ks.cfg                          67188932 metadata.tar.gz
 75649215 app.js                                    2773685 Music
 73006266 a.sh                                     35743888 newui
470150081 cdf                                      67362934 ngrok
374494288 cdf-components                           67362928 ngrok-stable-linux-amd64.zip
  2974620 certificate-controller                   73006211 nohup.out
134217848 chatops-docker                          134385344 offline-download
101575486 Desktop                                  67307785 offline-download.zip
101575487 Documents                                73006268 offline-download.zip.1
  2773684 Downloads                                34997822 Pictures
 67328576 go1.13.3.linux-amd64.tar.gz             470102230 popeye
469762624 go-fips                                 338000301 project-layout
 67184332 go-fips.tar                              67308300 Public
269104077 golang                                   83775905 renewCert
436767572 gowork                                  469806872 sactive-bot
168542293 hawk-eye                                373471446 sactive-web
```

`ls -i` 可以查看每一个文件对应的 inode。文件的 inode 是不会记录文件名称的，文件名记录在父节点的 inode 中。文件的内容存储在 datablock 中。文件的读权限，就表示可以读取 datablock 中的内容。目录的读权限就表示读取目录下的文件的名称。

inode 和 datablock 的结构是链表结构。inode 是头结点。

## 文件操作

- touch 创建文件 `touch <文件名>`
- cp 复制文件 `cp <src> <target>`
- mv 移动文件 `mv <src> <target>`，如果在同一个目录中移动，那就是重命名操作，inode 并不会改变，操作非常快。如果再同一个分区移动，也会非常快，因为文件被同一个文件系统管理，只是修改 inode
的信息。
- rm 删除文件，其实就是将文件名和 inode 的链接断开。这样删除会非常快，即使文件非常大。也使误删除操作，可以恢复。
- ln 文件链接

### 修改文件

如果使用 vim 修改文件内容：

```bash
[root@pooky ~]# vim test1
[root@pooky ~]# ls -li test1
67324877 -rw-r--r--. 1 root root 445 Aug 30 15:54 test1
[root@pooky ~]# vim test1
[root@pooky ~]# ls -li test1
67324882 -rw-r--r--. 1 root root 413 Aug 30 15:55 test1
```

inode 被改变了，67324877 变成了 67324882。vim 其实是创建了一个新的文件，使文件名指向了新的文件。

当使用 vim 修改文件时，会创建一个 swp 文件，例如 `test1.swp`，真正修改的就是这个 swp 文件。这样可以保证一致性。

## ln

```bash
[root@pooky ~]# ls -li test1
67324882 -rw-r--r--. 1 root root 413 Aug 30 15:55 test1
[root@pooky ~]# ln test1 test2
[root@pooky ~]# ls -li test1 test2
67324882 -rw-r--r--. 2 root root 413 Aug 30 15:55 test1
67324882 -rw-r--r--. 2 root root 413 Aug 30 15:55 test2
```

`67324882 -rw-r--r--. 1` 这里最后的 1 表示有 1 个文件名和这个 inode 建立了链接。使用 `ln`
是 test1 test2 建立了链接，指向了同一个 inode 67324882。删除时，只删除一个是不会丢失文件的。
ln 不会占用空间，只是在父目录存储了文件名。

ln 不能跨文件系统。因为 inode 是记录在文件系统中的。

### 软连接

也叫做符号链接。使用 `ln -s` 创建软连接：

```bash
[root@pooky ~]# ln -s test1 test2
[root@pooky ~]# ls -li test1 test2
67324882 -rw-r--r--. 1 root root 413 Aug 30 15:55 test1
67324865 lrwxrwxrwx. 1 root root   5 Aug 30 16:09 test2 -> test1
```

test1 test2 的 inode 是不一样的。test2 记录了 test1 的文件路径。类型是 `l`。

软连接是可以跨文件系统的，和硬链接的区别是，会创建新的文件。

### facl

如果要设置某个用户访问某个文件的权限，可以使用 facl。

- getfacl，查看文件的访问控制列表
- setfacl，设置文件的访问控制

```bash
[root@pooky ~]# setfacl -m u:user1:r test1
[root@pooky ~]# getfacl test1
# file: test1
# owner: root
# group: root
user::rw-
user:user1:r--
group::r--
mask::r--
other::r--
```

`setfacl -m u:user1:r test1` 表示给 user1 用户设置 test1 文件的只读权限。如果是组，把 `u:` 改成 `g:`。`r` 是只读权限。`rw` 表示读写权限。

- `-m` 赋予权限
- `-x` 收回权限，`setfacl -x u:user1:r test1`。

## 分区和挂载

- fdisk，`fdisk <磁盘设备>`，如 `fdisk /dev/sdb`。
- mkfs，fdisk 创建分区之后，要使用分区，需要使用 mkfs。mkfs 支持多种文件系统，如 `mkfs.xfs`，`mkfs.ext4`，`mkfs.ext3` 等等。使用对应的命令来把分区做成对应的文件系统，如 `mkfs.ext4 /dev/sdb1`，把 `/dev/sdb` 磁盘下的 `/dev/sdb1` 分区格式化为 ext4 文件系统。
- mount，类似 `/dev/sdb1` 的块设备文件是不能直接操作的，需要挂载到一个目录。如 `mount -t ext4 /dev/sdb1 /mnt/sdb1`，把 `/dev/sdb1` 挂载到 `/mnt/sdb1`。`-t` 参数可以省略，mount 命令会自动检测文件系统。
- parted，在硬盘大于 2 TB 时使用。使用类似 fdisk。

### 配置文件

上面使用命令行分区，配置是在内存中，机器重启，就会消失。需要修改配置文件： `/etc/fstab` 才能固话配置。

```bash
# /etc/fstab
# Created by anaconda on Mon Mar 18 17:53:42 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos7-root /                       xfs     defaults        0 0
UUID=eba30a1f-8f8b-42ee-85b3-7b17dac16e93 /boot                   xfs     defaults        0 0
#UUID=f8332d90-061a-44c4-897c-acd5a16da7e5 swap                    swap    defaults        0 0

# 示例
# 要挂载的设备  要挂载的目录  文件系统  权限
/dev/sdb1       /mnt/sdb1    ext4     defaults  0  0
```

## 用户磁盘配额

`xfs` 文件系统的用户磁盘配额 quota

1. `mkfs.xfs /dev/sdb1` 格式化分区，`-f` 强制格式化。
2. `mount -o uquota,gquota /dev/sdb1 /mnt/disk1` 挂载 `/dev/sdb1` 到 `/mnt/disk1` 目录。`-o uquota,gquota` 表示这个分区要支持用户和组磁盘配额。
3. `chmod 1777 /mnt/disk1`
4. `xfs_quota -x -c 'report -ugibh' /mnt/disk1` 查看磁盘配额，执行 `report -ugibh`。
5. `xfs_quota -x -c 'limit -u isoft=5 ihard=10 user1' /mnt/disk1` 限制某个用户的磁盘配额，执行 `limit -u isoft=5 ihard=10 user1`。

## 交换分区

交换分区是在内存不够用时，在磁盘上开辟一块空间，来扩充内存。

- mkswap，`mkswap /dev/sdb2` 设置 `/dev/sdb2` 分区为 swap 的标记。
- swapon，`swapon /dev/sdb2` 打开 swap 分区。可以使用 `free` 查看 swap 空间是否增加。
- swapoff，`swapoff /dev/sdb2` 关闭 swap 分区。

### 使用文件方式扩充 swap

- `dd if=/dev/zero bs=4M count=1024 of=/swapfile`，`bs=4M` 表示 block size 为 4M，`count=1024` 表示创建 1024 个 block，`of=/swapfile` 设备文件为 `swapfile`。创建了一个大小为 `1024 * 4M` 的 swapfile。
- `mkswap /swapfile`
- `chmod 600 /swapfile`
- `swapon /swapfile`

### 配置文件 fstab

```bash
# /etc/fstab
# Created by anaconda on Mon Mar 18 17:53:42 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos7-root /                       xfs     defaults        0 0
UUID=eba30a1f-8f8b-42ee-85b3-7b17dac16e93 /boot                   xfs     defaults        0 0
#UUID=f8332d90-061a-44c4-897c-acd5a16da7e5 swap                    swap    defaults        0 0

# 示例
# 要挂载的设备  要挂载的目录  文件系统  权限
/dev/sdb1       /mnt/sdb1    ext4     defaults  0  0

# 要挂载的设备  挂载的目录是虚拟的所以是swap   文件系统    权限      是否备份  是否开机自检，已经不需要
/swapfile       swap                         swap        defaults  0         0
```
