---
title: 脚本控制
weight: 15
---

# 脚本控制

脚本控制一般分为两种：

- 脚本的优先级控制
- 通过信号控制

## 脚本优先级控制

- `nice` 和 `renice` 可以调整脚本的优先级，调整资源的占用，如 CPU。
- 避免不可控的死循环
  - 死循环会导致 CPU 占用过高
  - 死循环会导致死机，创建大量的子进程，叫做 fock 炸弹。如 `.(){.|.&};.`，可以当做 `func(){func|func &}; func`。

`ulimit -a` 可以查看当前的终端的系统限制：

```bash
[root@pooky ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 63408
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 63408    # 用户的最大进程数
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

```

当 fock 炸弹创建大量的进程，超过 max user processes 的限制时，系统就会停止响应，因为 CPU 在不停的创建子进程。

## 捕获信号

- `kill` 默认发送 15 号信号给进程。
- `ctrl + c` 发送 2 号信号。
- **9 号信号不可阻塞**，`trap` 是无法捕获的。

有些应用程序，不希望被 `ctrl + c` 或者 `kill` 的信号中断，就可以使用 `trap` 捕获信号，来屏蔽掉这些信号。

```bash
#!/bin/bash

trap "echo sig 15" 15
trap "echo sig 2" 2     # 捕获了 ctrl + c 的信号

echo $$

while :
do
  :
done

```

## 计划任务

### 一次性计划任务

`at [OPTIONS] <系统时间>` 命令：

```bash
[root@pooky tmp]# date
Thu Sep 10 21:21:13 CST 2020
[root@pooky tmp]# at 21:23
at> echo hello > /tmp/hello.txt
at> <EOT>
job 2 at Thu Sep 10 21:23:00 2020
[root@pooky tmp]# atq
2 Thu Sep 10 21:23:00 2020 a root
[root@pooky tmp]# cat /tmp/hello.txt
hello
[root@pooky tmp]# ls -l /tmp/hello.txt
-rw-r--r--. 1 root root 6 Sep 10 21:23 /tmp/hello.txt

```

`at` 命令执行计划任务之后，可以使用 `ctrl + d` 退出。
`atq` 命令可以查看还没有执行的一次性计划任务。

选项：

- `-m`：当 `a`t 的工作完成后，即使没有输出讯息，亦以 email 通知使用者该工作已完成。
- `-l`：`at -l` 相当于 `atq`，列出目前系统上面的所有该使用者的 `at` job。
- `-d`：`at -d` 相当于 `atrm` ，可以取消一个在 `at` job；
- `-v`：可以使用较明显的时间格式列出 `at` job；
- `-c`：可以列出后面接的该项工作的实际指令内容。

### 周期性计划任务

Linux 默认安装 `crontab`，用来执行周期性任务。`crond` 进程会定期检查是否有要执行的任务，如果有，则自动执行。

`crontab [OPTIONS] [CMD]` 命令，配置计划任务的格式，如 `0 1 * * * root /user/local/run.sh`，每个段分别表示：

```bash
minute   hour   day   month   week   command
```

- `minute`： 分钟，从 `0` 到`59` 之间的任何整数。
- `hour`：小时，从 `0` 到 `23` 之间的任何整数。
- `day`：日期，从 `1` 到 `31` 之间的任何整数。
- `month`：月份，从 `1` 到 `12` 之间的任何整数。
- `week`: 星期几，从 `0` 到 `7` 之间的任何整数，`0` 或 `7` 代表星期日。
- `command`: 行的命令，可以是系统命令，也可以是脚本文件。**要注意命令的路径，如果没有配置 PATH，需要写绝对路径。**

各段中还可以使用下面的字符：

- `*`：代表所有可能的值，例如 `month` 字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
- `,`：可以用逗号隔开的值指定一个列表范围，例如，`1,2,5,7,8`
- `-`：可以用整数之间的中杠表示一个整数范围，如 `1-5` 表示 `1,2,3,4,5`
- `/`：可以用正斜线指定时间的间隔频率，例如 `0-23/2` 表示每两小时执行一次。同时正斜线可以和星号一起使用，如 `*/10`，如果用在 `minute` 字段，
表示每十分钟执行一次。

选项：

- `-e`：设置用户的计划任务
- `-l`：列出用户的计划任务
- `-r`：删除用户的计划任务
- `-u`：指定用户名，如果不指定，默认是当前用户。

**`/var/spool/cron/` 会有一个与用户同名的文件**。

可以使用命令添加计划任务，也可以修改 `/etc/crontab` 配置文件：

```bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

#### 示例

```bash
* * * * * command
```

每分钟执行一次 `command`。

```bash
10,20 * * * * command
```

每小时的第 `10` 和第 `20` 分钟执行一次。

```bash
10,20 8-11 */2 * * command
```

每隔两天的上午 `8` 点到 `11` 点的第 `10` 和第 `20` 分钟执行。

```bash
10 1 * * 6,0 /etc/init.d/smb restart
```

每周六、周日的一点十分重启 smb

#### 日志

计划任务的日志可以查看 `/var/log/cron` 文件：

```bash
[root@pooky ~]# crontab -e
no crontab for root - using an empty one
crontab: installing new crontab
[root@pooky ~]# tail -f /var/log/cron
Sep 10 21:01:02 pooky run-parts(/etc/cron.hourly)[15528]: starting mcelog.cron
Sep 10 21:01:02 pooky run-parts(/etc/cron.hourly)[15546]: finished mcelog.cron
Sep 10 21:10:01 pooky CROND[16058]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Sep 10 21:20:01 pooky CROND[16707]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Sep 10 21:30:01 pooky CROND[17353]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Sep 10 21:40:01 pooky CROND[17936]: (root) CMD (/usr/lib64/sa/sa1 1 1)
Sep 10 21:53:18 pooky crontab[18695]: (root) REPLACE (root)       # 表示编辑了一个 job
Sep 10 21:53:18 pooky crontab[18695]: (root) END EDIT (root)      # 表示完成了编辑
Sep 10 21:54:02 pooky CROND[18769]: (root) CMD (ntpdate ntp.swinfra.net)

```

上面的示例设置了周期任务 `*/2 * * * * ntpdate ntp.swinfra.net` 每两分钟同步一次时间。

### 延时计划任务

在有些意外情况下，周期计划任务可能没有执行，比如机器意外关机了。这时候就需要延时计划任务，可以在开机之后再延后运行。

`/etc/cron.d/0hourly`：

```bash
# Run the hourly jobs
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
01 * * * * root run-parts /etc/cron.hourly # 每一个小时的第一分钟 运行 run-parts /etc/cron.hourly
```

`run-parts` 在调用 `/etc/cron.hourly` 之后会在某处记录一个标记，表示这一个小时已经运行过。再下次开机时会检查这个标记，如果这个小时已经运行过，就不会再运行。

`cron.hourly` 对应的是每小时的延时任务，其他的任务在 `/etc/anacrontab` 文件中：

```bash
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45 # 随机在 0 到 45 之间选择延时时间
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22 # 延时任务在 3 到 22 点之间运行

#period in days   delay in minutes   job-identifier   command
1       5       cron.daily              nice run-parts /etc/cron.daily # 延时 5 分钟，就是每天的 3:05 分开始执行延时任务
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly

```

### 任务锁

在运行一个周期任务时，如果该任务执行时间较长，导致下一个周期开始，这样再执行任务就可能冲突。这时候可以使用任务锁来避免这种问题。

`flock -x` 来运行，会加上一个排它锁：

```bash
[root@pooky ~]# flock -xn "/tmp/f.lock" -c "/root/test.sh"

```

"/tmp/f.lock" 就是锁文件。

选项：

- `-s` 获得一个共享锁
- `-x` 获得一个独占锁，这是默认的
- `-u` 删除一个锁，通常是不需要的，因为在文件关闭时锁会自动删除
- `-n` 如果没有获得锁，直接失败而不是等待
- `-w` 如果没有获得锁，等待指定时间
- `-o` 在执行命令之前关闭保持锁的文件描述符
- `-c` 运行一个命令
