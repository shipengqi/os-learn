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
