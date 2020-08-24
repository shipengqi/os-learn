# 进程管理

进程的生命周期是可管理的。

## 查看进程

- ps
- pstree
- top

进程也是树形结构。进程和权限密不可分。

### ps

执行 `ps` 查看当前终端的可以看到的进程：

```bash
[root@pooky ~]# ps
  PID TTY          TIME CMD
 8974 pts/2    00:00:00 bash
12550 pts/2    00:00:00 ps
```

`ps -e` 查看所有终端的进程：

```bash
[root@pooky ~]# ps -e
  PID TTY          TIME CMD
    1 ?        00:50:59 systemd
    2 ?        00:00:06 kthreadd
    3 ?        00:00:01 ksoftirqd/0
    5 ?        00:00:00 kworker/0:0H
    7 ?        00:00:00 migration/0
    8 ?        00:00:00 rcu_bh
    9 ?        00:26:58 rcu_sched
   10 ?        00:00:00 lru-add-drain
   11 ?        00:00:23 watchdog/0
   12 ?        00:00:23 watchdog/1
   13 ?        00:00:04 migration/1
   14 ?        00:00:02 ksoftirqd/1
   16 ?        00:00:00 kworker/1:0H
   17 ?        00:00:25 watchdog/2
   18 ?        00:00:00 migration/2
   19 ?        00:00:01 ksoftirqd/2
   21 ?        00:00:00 kworker/2:0H
   22 ?        00:00:25 watchdog/3
   23 ?        00:00:04 migration/3
   24 ?        00:00:02 ksoftirqd/3

```

- `ps -A` 和 `ps -e` 一样
- `ps -ef` 显示所有进程，包含命令信息
- `ps -u root` 显示 root 用户的进程
- `ps -au` 显示详细的信息
- `ps -aux` 显示所有用户的进程
- `ps -eLf` 显示线程

### pstree

查看进程之间的关系。

```bash
[root@pooky ~]#   pstree
systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager─┬─dhclient
        │                └─2*[{NetworkManager}]
        ├─VGAuthService
        ├─2*[abrt-watch-log]
        ├─abrtd
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─alsactl
        ├─at-spi-bus-laun─┬─dbus-daemon───{dbus-daemon}
        │                 └─3*[{at-spi-bus-laun}]
        ├─at-spi2-registr───2*[{at-spi2-registr}]
        ├─atd
        ├─auditd─┬─audispd─┬─sedispatch
        │        │         └─{audispd}
        │        └─{auditd}
        ├─avahi-daemon───avahi-daemon
        ├─chronyd
        ├─colord───2*[{colord}]
        ├─crond
        ├─cupsd
        ├─2*[dbus-daemon───{dbus-daemon}]
        ├─dbus-launch
        ├─dnsmasq───dnsmasq
        ├─dockerd─┬─containerd───27*[{containerd}]
        │         └─45*[{dockerd}]
        ├─gdm─┬─X───9*[{X}]
        │     ├─gdm-session-wor─┬─gnome-session-b─┬─gnome-shell─┬─ibus-daemon─┬─ibus-dconf───3*[{ibus-dconf}]
        │     │                 │                 │             │             ├─ibus-engine-sim───2*[{ibus-engine-sim+
        │     │                 │                 │             │             └─2*[{ibus-daemon}]
        │     │                 │                 │             └─32*[{gnome-shell}]
        │     │                 │                 ├─gsd-a11y-keyboa───3*[{gsd-a11y-keyboa}]
        │     │                 │                 ├─gsd-a11y-settin───3*[{gsd-a11y-settin}]
        │     │                 │                 ├─gsd-clipboard───2*[{gsd-clipboard}]
        │     │                 │                 ├─gsd-color───3*[{gsd-color}]
        │     │                 │                 ├─gsd-datetime───2*[{gsd-datetime}]
        │     │                 │                 ├─gsd-housekeepin───2*[{gsd-housekeepin}]
        │     │                 │                 ├─gsd-keyboard───3*[{gsd-keyboard}]
        │     │                 │                 ├─gsd-media-keys───3*[{gsd-media-keys}]
        │     │                 │                 ├─gsd-mouse───2*[{gsd-mouse}]
        │     │                 │                 ├─gsd-power───3*[{gsd-power}]
        │     │                 │                 ├─gsd-print-notif───2*[{gsd-print-notif}]
        │     │                 │                 ├─gsd-rfkill───2*[{gsd-rfkill}]
        │     │                 │                 ├─gsd-screensaver───2*[{gsd-screensaver}]
        │     │                 │                 ├─gsd-sharing───3*[{gsd-sharing}]
        │     │                 │                 ├─gsd-smartcard───4*[{gsd-smartcard}]
        │     │                 │                 ├─gsd-sound───3*[{gsd-sound}]
        │     │                 │                 ├─gsd-wacom───2*[{gsd-wacom}]
        │     │                 │                 ├─gsd-xsettings───3*[{gsd-xsettings}]
        │     │                 │                 └─3*[{gnome-session-b}]
        │     │                 └─2*[{gdm-session-wor}]
        │     └─3*[{gdm}]
        ├─gssproxy───5*[{gssproxy}]
        ├─ibus-portal───2*[{ibus-portal}]
        ├─ibus-x11───2*[{ibus-x11}]
        ├─irqbalance
        ├─ksmtuned───sleep
        ├─libvirtd───16*[{libvirtd}]
        ├─lsmd
        ├─lvmetad
        ├─master─┬─pickup
        │        └─qmgr
        ├─mcelog
        ├─packagekitd───2*[{packagekitd}]
        ├─polkitd───5*[{polkitd}]
        ├─pulseaudio───{pulseaudio}
        ├─rhnsd
        ├─rhsmcertd
        ├─rpc.idmapd
        ├─rpc.mountd
        ├─rpc.statd
        ├─rpcbind
        ├─rsyslogd───2*[{rsyslogd}]
        ├─rtkit-daemon───2*[{rtkit-daemon}]
        ├─smartd
        ├─sshd─┬─2*[sshd───bash]
        │      ├─sshd───3*[sftp-server]
        │      └─sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        ├─udisksd───4*[{udisksd}]
        ├─upowerd───2*[{upowerd}]
        ├─vmtoolsd───{vmtoolsd}
        ├─wpa_supplicant
        └─xdg-permission-───2*[{xdg-permission-}]
```

### top

top 显示进程和系统信息。

```bash
[root@pooky ~]# top
top - 21:01:33 up 62 days, 10:40,  3 users,  load average: 0.05, 0.03, 0.05
Tasks: 240 total,   1 running, 239 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16266780 total,  7581412 free,  1100140 used,  7585228 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 14404532 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
  936 root      20   0 1210844  37064  14992 S   0.7  0.2 583:14.27 containerd
16865 root      20   0  162096   2376   1576 R   0.7  0.0   0:00.13 top
  786 avahi     20   0   62372   2276   1812 S   0.3  0.0 341:04.74 avahi-daemon
  805 root      20   0 2208132  91280  26176 S   0.3  0.6 349:03.69 dockerd
 2067 gdm       20   0  755416  32672   9388 S   0.3  0.2 271:23.92 gsd-color
    1 root      20   0  194124   7264   4188 S   0.0  0.0  51:00.39 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:06.19 kthreadd
    3 root      20   0       0      0      0 S   0.0  0.0   0:01.14 ksoftirqd/0
    5 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/0:0H
    7 root      rt   0       0      0      0 S   0.0  0.0   0:00.88 migration/0
    8 root      20   0       0      0      0 S   0.0  0.0   0:00.00 rcu_bh
    9 root      20   0       0      0      0 S   0.0  0.0  26:59.02 rcu_sched
   10 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 lru-add-drain
   11 root      rt   0       0      0      0 S   0.0  0.0   0:23.27 watchdog/0
   12 root      rt   0       0      0      0 S   0.0  0.0   0:23.57 watchdog/1
   13 root      rt   0       0      0      0 S   0.0  0.0   0:04.91 migration/1
   14 root      20   0       0      0      0 S   0.0  0.0   0:02.78 ksoftirqd/1
   16 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/1:0H
   17 root      rt   0       0      0      0 S   0.0  0.0   0:25.69 watchdog/2
   18 root      rt   0       0      0      0 S   0.0  0.0   0:00.70 migration/2
   19 root      20   0       0      0      0 S   0.0  0.0   0:01.22 ksoftirqd/2
   21 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/2:0H
   22 root      rt   0       0      0      0 S   0.0  0.0   0:25.11 watchdog/3
   23 root      rt   0       0      0      0 S   0.0  0.0   0:04.72 migration/3
   24 root      20   0       0      0      0 S   0.0  0.0   0:02.66 ksoftirqd/3
   26 root       0 -20       0      0      0 S   0.0  0.0   0:00.00 kworker/3:0H
   27 root      rt   0       0      0      0 S   0.0  0.0   0:29.23 watchdog/4
   28 root      rt   0       0      0      0 S   0.0  0.0   0:02.59 migration/4
```

第一行：

- `21:01:33 up 62 days` 表示从系统开机到现在的运行时间。
- `3 users` 表示当前有 3 个用户登录
- `load average: 0.05, 0.03, 0.05` 系统平均负载，三个值分别表示 1 分钟，5 分钟，15 分钟内的平均负载。1 就表示满负载。

第二行：

- `240 total` 表示有 240 个进程在运行
- `1 running, 239 sleeping,   0 stopped,   0 zombie` 运行的进程有不同的状态。这里就表示有 1 个 running 状态的进程，239 个 sleeping 状态的进程。

第三行 CPU 使用情况：

- `0.1 us,  0.1 sy,  0.0 ni, 99.8 id 0.0 wa` 表示百分之 0.1 的 CPU 在进行 US 计算，百分之 0.7 的 CPU 在进行进程之间状态的交互，百分之 99.8 的 CPU 是空闲的，`wa` 表示 IO 等待。

- `%Cpu(s)` 显示的是多个 CPU 的平均值，如果想查看每个 CPU 对应的使用率，按数字键 `1`，就会展开所有的 CPU。

第四行，内存的使用率：

- `16266780 total,  7581412 free,  1100140 used,  7585228 buff/cache` 表示内存总量为 `16266780`，剩余可用内存为 `7581412`，已使用内存 `1100140`， `7585228 buff/cache` 表示正在用来读写的缓存。

第五行，交换分区，当物理内存不够时，就需要使用 swap：

- `0 total,  0 free,  0 used. 14404532 avail Mem`，`0 used` 表示没有使用 swap，说明物理内存还够用。

再下面的信息是动态的进程信息。

常用参数：

- `-d` 指定信息刷新的时间间隔。也可以使用按键 `s` 交互命令来改变。
- `-p` 指定进程 ID ，值显示指定进程的状态。
- `-i` 不显示闲置或者僵死进程。
- `-c` 显示整个命令行而不只是显示命令名。

## 进程控制

- 调整进程的优先级
  - nice 范围从 `-20` 到 `19`，值越小表示优先级越高，抢占资源就越多
  - renice 重新这是优先级
- 进程的作业控制
  - `&` 符号，后台运行进程，`./a.sh &`。
  - jobs

### nice

`a.sh` 是一个死循环的脚本：

```bash
#!/bin/bash

echo $$

while :
do
  :
done
```  

然后修改运行权限并运行：

```bash
[root@pooky ~]# ./a.sh
23248
```

打开一个新的终端：

```bash
[root@pooky ~]# top -p 23294
top - 07:53:53 up 62 days, 21:33,  4 users,  load average: 0.47, 0.15, 0.09
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16266780 total,  7566888 free,  1105816 used,  7594076 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 14390072 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
23294 root      20   0  113176   1192   1008 R 100.0  0.0   0:23.15 a.sh
```

可以看到进程已经占满了一个 CPU。进程的状态栏中 `PR` 就是进程的优先级， `NI` 就是 nice 值。

重新设置优先级，先把进程退出，然后使用 nice 命令设置：

```bash
[root@pooky ~]# nice -n 10 ./a.sh
23780

# 打开新的终端
[root@pooky ~]# top -p 23780
top - 07:58:04 up 62 days, 21:37,  4 users,  load average: 0.45, 0.47, 0.25
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.2 us,  0.1 sy, 12.5 ni, 87.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16266780 total,  7567992 free,  1104504 used,  7594284 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 14391492 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
23780 root      30  10  113176   1188   1004 R  92.4  0.0   0:11.20 a.sh
```

如果要对已经运行的进程，不想停掉进程，来设置 nice 值，可以使用 renice：

```bash
[root@pooky ~]# renice -n 15 23780
24155 (process ID) old priority 10, new priority 15

# 打开新的终端
[root@pooky ~]# top -p 23780
```

### jobs

加上 `&` 可以使进程后台运行，要把后台运行的进程掉回前台，使用 jobs 命令：

```bash
[root@pooky ~]# ./a.sh &
[1] 24397
[root@pooky ~]# jobs
[1]+  Running                 ./a.sh &
[root@pooky ~]# fg 1
./a.sh

```

运行 `fg <job 号>` 将进程调回前台。

#### 前台进程调到后台

```bash
[root@pooky ~]# ./a.sh
24682
^Z
[1]+  Stopped                 ./a.sh
[root@pooky ~]# top -p 24682
top - 08:12:06 up 62 days, 21:51,  5 users,  load average: 0.07, 0.50, 0.45
Tasks:   1 total,   0 running,   0 sleeping,   1 stopped,   0 zombie
%Cpu(s):  0.8 us,  0.8 sy,  0.0 ni, 98.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16266780 total,  7564728 free,  1107784 used,  7594268 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 14388276 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
24682 root      20   0  113176   1188   1008 T   0.0  0.0   0:01.81 a.sh
```

`ctrl + z` 可以将前台进程调度到后台，但是状态是停止的。状态栏 `S` 的值是 `T` 就表示已停止。

要想恢复继续到前台就使用 `jobs` 加 `fg`。如果想要恢复到后台运行使用 `bd <job 号>`。

## 守护进程
