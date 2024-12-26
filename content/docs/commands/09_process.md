---
title: 进程管理
weight: 9
---

# 进程管理

进程的生命周期是可管理的。

## 查看进程

- `ps`
- `pstree`
- `top`

进程也是树形结构。进程和权限密不可分。

### ps

执行 `ps` 查看当前终端的可以看到的进程，`p` 代表 processes，也就是进程；`s` 代表 snapshot，也就是快照：

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

- `ps -A` 和 `ps -e` 一样。
- `ps -ef`：显示所有进程，包含命令信息，`-f` 包含更多描述字段。
- `ps -eLf`：显示线程。
- `ps -au`：显示详细的信息。
- `ps -aux`：显示所有用户的进程。
- `ps -u root`：显示 root 用户的进程。

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

`top` 显示进程和系统信息。其实和 `ps` 差不多，只不过显示的是实时数据。

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
- `load average: 0.05, 0.03, 0.05` 系统平均负载，三个值分别表示 1 分钟，5 分钟，15 分钟内的平均负载。

第二行：

- `240 total` 表示有 240 个进程在运行
- `1 running, 239 sleeping,   0 stopped,   0 zombie` 运行的进程有不同的状态。这里就表示有 1 个 running 状态的进程，239 个 sleeping 状态的进程。

第三行 CPU 使用情况：

- `0.1 us,  0.1 sy,  0.0 ni, 99.8 id 0.0 wa`，其中 `0.1 us` 表示用户控件占用百分之 `0.1` 的 CPU，`0.1 sy`  表示内核空间占用百分之 `0.7` 的 CPU，`0.0 ni` 表示改变过优先级的进程 CPU 占比，`99.8 id` 表示空闲 CPU 占比，`wa` 表示 IO 等待，`0.0 hi` 硬件中断（hardware IRQ） CPU 占比，`0.0 si` 软件中断 CPU 占比。
- `%Cpu(s)` 显示的是多个 CPU 的平均值，如果想查看每个 CPU 对应的使用率，按**数字键 `1`，就会展开所有的 CPU**。

第四行，内存的使用率：

- `16266780 total,  7581412 free,  1100140 used,  7585228 buff/cache` 表示内存总量为 `16266780`，剩余可用内存为 `7581412`，已使用内存 `1100140`， `7585228 buff/cache` 表示正在用来读写的缓存。

第五行，交换分区，当物理内存不够时，就需要使用 swap：

- `0 total,  0 free,  0 used. 14404532 avail Mem`，`0 used` 表示没有使用 swap，说明物理内存还够用。

再下面的信息是动态的进程信息：

- `PR` 进程优先级
- `NI` nice 值。值越小优先级越高，最小 `-20` ，最大 20（用户设置最大 19）
- `VIRT` 进程虚拟内存的大小，只要是进程申请过的内存，即便还没有真正分配物理内存，也会计算在内。
- `RES` 是常驻内存的大小，也就是进程实际使用的物理内存大小，不包括 Swap 和共享内存。
- `SHR` 是共享内存的大小，比如与其他进程共同使用的共享内存、加载的动态链接库以及程序的代码段等。
- `S` 进程状态
- `%CPU` 进程占用 CPU 百分比
- `%MEM` 进程占用物理内存百分比。
- `TIME+` 进程运行时间

常用参数：

- `-d` 指定信息刷新的时间间隔。也可以使用按键 `s` 交互命令来改变。
- `-p` 指定进程 ID ，值显示指定进程的状态。
- `-i` 不显示闲置或者僵死进程。
- `-c` 显示完整的命令。

## 进程控制

- 调整进程的优先级
  - `nice` 范围从 `-20` 到 `19`，值越小表示优先级越高，抢占资源就越多。
  - `renice` 重新设置优先级。
- 进程的作业控制
  - `&` 符号，后台运行进程，`./a.sh &`。
  - `jobs` 命令。

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

打开一个新的终端，top 基本视图中，按数字 `1`，可监控每个逻辑 CPU 的状况：

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

可以看到进程已经占满了一个 CPU。进程的状态栏中 `PR` 就是进程的优先级， `NI` 就是 `nice` 值。

重新设置优先级，先把进程退出，然后使用 `nice` 命令设置：

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

如果要对已经运行的进程，不想停掉进程，来设置 `nice` 值，可以使用 `renice`：

```bash
[root@pooky ~]# renice -n 15 23780
24155 (process ID) old priority 10, new priority 15

# 打开新的终端
[root@pooky ~]# top -p 23780
```

### jobs

加上 `&` 可以使进程后台运行，要把后台运行的进程掉回前台，使用 `jobs` 命令：

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

## kill

`kill -l` 可以查看所有信号：

```bash
[root@pooky ~]# kill -l
 1) SIGHUP  2) SIGINT  3) SIGQUIT  4) SIGILL  5) SIGTRAP
 6) SIGABRT  7) SIGBUS  8) SIGFPE  9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT 17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO 30) SIGPWR
31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3
38) SIGRTMIN+4 39) SIGRTMIN+5 40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8
43) SIGRTMIN+9 44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9 56) SIGRTMAX-8 57) SIGRTMAX-7
58) SIGRTMAX-6 59) SIGRTMAX-5 60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2
63) SIGRTMAX-1 64) SIGRTMAX
```

一共有 64 个信号。常用的 `ctrl + c` 就是 `SIGINT` 信号。

`kill -<信号的编号> <进程号>` 比如 `kill -9 28317` 杀死 `28317` 进程，不受任何阻断。注意 1 号进程是杀不掉的。

## 守护进程

- 使用 nohup （no hang up）和 `&` （表示后台运行） 运行命令，nohup 命令会忽略 hangup 信号。
- 守护（daemon）进程，如 linux 的 service。

### nohup

当我们在终端运行一个进程时，如果终端关闭，那么进程也会被杀死，如果不想进程被杀死，可以使用 nohup 命令：

```bash
[root@pooky ~]# nohup tail -f /var/log/messages &
[1] 11725
nohup: ignoring input and appending output to ‘nohup.out’
```

nohup 即使关掉终端，进程依然运行。`&` 是后台运行。命令的输出会输出到 `nohup.out`。

执行 `ps -ef | grep tail` 查看进程：

```bash
[root@pooky ~]# ps -ef | grep tail
root     11725 10056  0 21:06 pts/1    00:00:00 tail -f /var/log/messages
root     11840 10056  0 21:08 pts/1    00:00:00 grep --color=auto tail
```

关闭终端，再次查看：

```bash
[root@pooky 1320]# ps -ef | grep tail
root     11725     1  0 21:06 ?        00:00:00 tail -f /var/log/messages
root     11911 11454  0 21:09 pts/2    00:00:00 grep --color=auto tail
```

11725 进程还在，但是父进程变成了 1。因为 11725 的父进程终端被关掉了，11725 就变成了孤儿进程，孤儿进程会被 1 号进程（systemd）收留。
daemon 进程其实也是类似的原理，自动结束了父进程，被 1 号进程收留。

进程的信息会输出到 `/proc/` 目录下：

```bash
[root@pooky 1320]# cd /proc/11725
[root@pooky 11725]# ll cwd
lrwxrwxrwx. 1 root root 0 Aug 24 21:13 cwd -> /root
[root@pooky 11725]# ll fd
total 0
l-wx------. 1 root root 64 Aug 24 21:13 0 -> /dev/null
l-wx------. 1 root root 64 Aug 24 21:13 1 -> /root/nohup.out
l-wx------. 1 root root 64 Aug 24 21:08 2 -> /root/nohup.out
lr-x------. 1 root root 64 Aug 24 21:13 3 -> /var/log/messages
lr-x------. 1 root root 64 Aug 24 21:13 4 -> anon_inode:inotify

```

`ll cwd` 可以看出 `tail` 命令实在 `/root` 下执行的，那么 `/root` 目录就不能被卸载，因为
`tail` 命令正在使用这个目录。

`fd` 文件下，可以查看进程的输入输出。0，1，2 分别是标准输入，标准输出，标准错误输出。`0 -> /dev/null` 表示标准输入被关掉。

### daemon

daemon 不需要终端，比如 service。这种进程因为没有终端输出，所以需要日志文件来记录。

```bash
[root@pooky ~]# ps -ef | grep sshd
root      1320     1  0 Jun22 ?        00:00:02 /usr/sbin/sshd -D
root      6333  1320  0 10:33 ?        00:00:01 sshd: root@pts/0
root     10044  1320  0 20:40 ?        00:00:00 sshd: root@pts/1
root     11442  1320  6 21:02 ?        00:00:00 sshd: root@pts/2
root     11514 11454  0 21:03 pts/2    00:00:00 grep --color=auto sshd
[root@pooky ~]# cd /proc/1320
[root@pooky 1320]# ll cwd
lrwxrwxrwx. 1 root root 0 Aug 24 21:03 cwd -> /
[root@pooky 1320]# ll fd
total 0
lr-x------. 1 root root 64 Jun 22 10:21 0 -> /dev/null
lrwx------. 1 root root 64 Jun 22 10:21 1 -> socket:[21458]
lrwx------. 1 root root 64 Jun 22 10:21 2 -> socket:[21458]
lrwx------. 1 root root 64 Jun 22 10:21 3 -> socket:[27755]
lrwx------. 1 root root 64 Jun 22 10:21 4 -> socket:[27757]
```

标准输出指向了 socket。

### screen

有些时候在终端跑一些运行时间很长的任务，终端可能因为一些原因断开。screen 的功能，会话恢复，多窗口，会话共享。

只要 screen 本身没有终止，在其内部运行的会话都可以恢复。这一点对于远程登录的用户特别有用，即使网络连接中断，用户也不会失去对已经打开的命令行会话的控制。
只要再次登录到主机上执行 `screen -r` 就可以恢复会话的运行。

- 执行 `screen` 命令进入 screen 环境。
- `ctrl+a` 再按 `d` 退出 screen 环境。
- `screen -ls` 查看 screen 的 session。
- `screen -r <sessionId>` 恢复指定 session。

### 系统日志

系统日志一般在 `/var/log` 目录下。一些重要的日志：

- `messages`：是系统的常规日志。
- `dmesg`：系统内核的启动日志。
- `secure`：系统的安全日志。
- `cron`：系统的计划任务日志。
