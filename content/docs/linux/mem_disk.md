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

- fdisk 查看磁盘，磁盘分区
- df
- du

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

```bash
[root@pooky ~]# ls -l /dev/sd?
brw-rw----. 1 root disk 8, 0 Jun 22 10:20 /dev/sda
[root@pooky ~]# ls -l /dev/sd??
brw-rw----. 1 root disk 8, 1 Jun 22 10:20 /dev/sda1
brw-rw----. 1 root disk 8, 2 Jun 22 10:20 /dev/sda2
brw-rw----. 1 root disk 8, 3 Jun 22 10:20 /dev/sda3
```
