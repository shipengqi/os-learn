---
title: Linux CPU
weight: 1
---

# Linux CPU

性能优化的两个核心指标：**吞吐**和**延时**，对应着高并发和响应快。这是从**应用负载的视角**来考察性能。

从**系统资源的视角**考察的指标：资源使用率，饱和度等。

性能问题的本质就是系统资源已经达到瓶颈，但是请求的处理却还是不够快，无法支撑更大的并发量。

性能分析就是**找出应用或者系统的瓶颈，并设法去避免或缓解它们**。可以分为六个步骤：

1. 选择指标评估应用程序和系统的性能
2. 为应用程序和系统设置性能目标
3. 进行性能基准测试
4. 性能分析定位瓶颈
5. 优化系统和应用程序
6. 性能监控和告警

## 平均负载

查看系统负载，使用 `top` 或 `uptime` 命令：

```bash
[root@shccdfrh75vm8 dev]# uptime
 14:08:53 up 122 days,  3:47,  2 users,  load average: 0.02, 0.02, 0.05
[root@shccdfrh75vm8 dev]# top
top - 14:09:04 up 122 days,  3:48,  2 users,  load average: 0.02, 0.02, 0.05

```

- `14:08:53` 表示当前时间
- `up 122 days` 表示系统运行时间
- `2 users` 表示登录的用户数。
- `load average: 0.05, 0.03, 0.05` 系统平均负载，三个值分别表示最近 1 分钟，5 分钟，15 分钟内的平均负载。

### 什么是平均负载

平均负载就是指单位时间内，系统处于**可运行状态**和**不可中断状态**的平均进程数，也就是**平均活跃进程数**，它和 CPU 使用率并没有直接关系。

可运行状态的进程，是正在使用 CPU 或者正在等待 CPU 的进程，R 状态（Running 或 Runnable）。

不可中断状态的进程是正处于内核态关键流程中的进程，并且这些流程不可打断。例如等待硬件设备 IO 响应的进程，D 状态（Uninterruptible Sleep，也叫 Disk Sleep）。

平均负载就可以简单的理解为是平均的活跃进程数。那么最理想的，就是每个 CPU 都干好只运行了一个进程。如当平均负载是 2 时，意味着：

- 在有 2 个 CPU 的系统上，意味着所有的 CPU 都刚好被占用。
- 在 4 个 CPU 的系统上，意味着 CPU 有 50% 是空闲的。
- 在 1 个 CPU 的系统上，意味着有一半的进程竞争不到 CPU。

### 平均负载的理想值

平均负载最理想的情况是等于 CPU 个数。

查看 CPU 个数：`grep 'model name' /proc/cpuinfo | wc -l`。

**当平均负载比 CPU 个数还大的时候，系统已经出现了过载**。

平均负载有三个值，可以帮助分析**系统负载的趋势**：

- 如果三个值相差不大，说明系统负载很平稳。
- 如果 1 分钟的值远小于 15 分钟的值，说明系统最近 1 分钟的负载在减少，但是过去 15 分钟内负载很大。
- 如果 1 分钟的值远大于 15 分钟的值，说明最近 1 分钟的负载在增加，这种增加可能是临时性的，也有可能持续增加，需要持续观察。一旦 1 分钟的平均负载接近或者超过了 CPU 的个数，就意味着系统正在发生过载的问题。

假设在一个单 CPU 系统上看到平均负载为 1.73，0.60，7.98，那么说明在过去 1 分钟内，系统有 73% 的超载，而在 15 分钟内，有 698% 的超载，从整体趋势来看，系统的负载在降低。

### 平均负载和 CPU 使用率

平均负载是指单位时间内，处于可运行状态和不可中断状态的进程数。所以，它不仅包括了**正在使用 CPU**的进程，还包括**等待 CPU** 和**等待 IO** 的进程。

CPU 使用率，是单位时间内 CPU 繁忙情况的统计，跟平均负载并不一定完全对应。

- CPU 密集型进程，使用大量 CPU 会导致平均负载升高，此时这两者是一致的；
- IO 密集型进程，等待 IO 也会导致平均负载升高，但 CPU 使用率不一定很高；
- 大量等待 CPU 的进程调度也会导致平均负载升高，此时的 CPU 使用率也会比较高。

### 平均负载分析

- `stress` 是一个 Linux 系统压力测试工具，可以模拟平均负载升高的场景。
  - 安装：`yum install -y epel-release; yum install -y stress`
- `mpstat` 是一个常用的多核 CPU 性能分析工具，用来实时**查看每个 CPU** 的性能指标，以及所有 CPU 的平均指标。
- `pidstat` 是一个常用的进程性能分析工具，用来实时**查看进程**的 CPU、内存、IO 以及上下文切换等性能指标。

## 上下文切换

### 进程上下文切换

### 线程上下文切换

### 中断上下文切换

### 上下文切换分析

- `sysbench` 一个多线程的基准测试工具，一般用来评估不同系统参数下的数据库负载情况。可以用来模拟系统多线程调度切换的情况。
- `vmstat` 主要用来分析系统的内存使用情况，也常用来分析 CPU 上下文切换和中断的次数。
- `pisstat`
- `/proc/interrupts`

```bash
[root@SGDLITVM0905 ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 1266396   1036 10685316    0    0    13    84   57   64 12  7 80  0  0
```

- cs（context switch）是每秒上下文切换的次数。
- in（interrupt）则是每秒中断的次数。
- r（Running or Runnable）是就绪队列的长度，也就是正在运行和等待 CPU 的进程数。
- b（Blocked）则是处于不可中断睡眠状态的进程数。

## CPU 使用率

Linux 将每个 CPU 的时间划分为很短的时间片，再通过调度器轮流分配给各个任务使用，因此造成了多个任务同时运行的错觉。

### CPU 使用率分析

- `top`
- `ps` `pstree`
- `pidstat`
- `perf` 它以性能事件采样为基础，不仅可以分析系统的各种事件和内核性能，还可以用来分析指定应用程序的性能问题。
- `GDB`
- `execsnoop` 是一个专为短时进程设计的工具。它通过 ftrace 实时监控进程的 exec() 行为，并输出短时进程的基本信息，包括进程 PID、父进程 PID、命令行参数以及执行的结果。

## 不可中断进程和僵尸进程

当 iowait 升高时，进程很可能因为得不到硬件的响应，而长时间处于不可中断的状态。

top 和 ps 是最常用的查看进程状态的工具。

- R 是 Running 或 Runnable 的缩写，表示进程在 CPU 的就绪队列中，正在运行或者正在等待运行。
- D 是 Disk Sleep 的缩写，也就是不可中断状态睡眠（Uninterruptible Sleep），一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程或中断打断。
- Z 是 Zombie 的缩写，如果你玩过“植物大战僵尸”这款游戏，应该知道它的意思。它表示僵尸进程，也就是进程实际上已经结束了，但是父进程还没有回收它的资源（比如进程的描述符、PID 等）。
- S 是 Interruptible Sleep 的缩写，也就是可中断睡眠状态，表示进程因为等待某个事件而被系统挂起。当进程等待的事件发生时，它会被唤醒并进入 R 状态。
- I 是 Idle 的缩写，也就是空闲状态，用在不可中断睡眠的内核线程上。前面说了，硬件交互导致的不可中断进程用 D 表示，但对某些内核线程来说，它们有可能实际上并没有任何负载，用 Idle 正是为了区分这种情况。要注意，D 状态的进程会导致平均负载升高， I 状态的进程却不会。
- T 或者 t，也就是 Stopped 或 Traced 的缩写，表示进程处于暂停或者跟踪状态。
- X，也就是 Dead 的缩写，表示进程已经消亡，所以你不会在 top 或者 ps 命令中看到它。

不可中断睡眠状态，是为了保证进程数据与硬件状态一致，并且正常情况下，不可中断状态在很短时间内就会结束。但是如果系统或硬件发生了鼓掌，进程可能会在不可中断状态保持很久，导致系统出现大量不可中断进程。这是就需要注意系统是不是出现了 IO 性能问题。

僵尸进程状态一般持续的时间都很短。父进程回收它之后就会消失，或者父进程退出，由 init 进程回收后消失。通常当一个进程创建了子进程，它应该调用 wati 或者 waitpid 等待子进程结束，然后回收子进程的资源。子进程在结束时，会向父进程发送 SIGCHLD 信号，所以，父进程可以注册 SIGCJLD 信号的处理函数，异步回收资源。

如果父进程没有回收资源，或子进程执行太快，父进程没来得及处理子进程状态，子进程就已经退出，那么这时的子进程会变成僵尸进程。

### 分析

- dstat，可以同时观察系统的 CPU，磁盘 IO，网络以及内存的使用情况。
- strace，跟踪进程系统调用的工具。

**iowait 高并不一定有 IO 性能瓶颈。当系统中有 IO 类型的进程在运行时，iowait 也会很高，但是实际上，磁盘的读写远没有达到性能瓶颈的程度**。

碰到 iowait 升高，先用 dstat，pidstat 确认是不是磁盘 IO 的问题，然后在找出具体是哪个进程导致的。

一般不可中断状态的进程是等待 IO 的进程，所以 ps 找到的 D 状态的进程，一般就是可疑进程。可以使用 strace 分析进程的系统调用，但是如果是僵尸进程，那么不能使用 strace。需要使用 perf 来分析。

僵尸进程可以使用 pstree 找到父进程后，检查父进程代码，是否在子进程退出之后，会后了资源。

## 软中断

中断其实是一种异步的事件处理机制，可以提高系统的并发处理能力。

中断会打断其他进程的运行，所以中断处理程序要尽可能的快，减少对正常程序运行调度的影响。

并且中断处理程序响应中断时，会临时关闭中断。会导致上一次中断处理完成之前，其他中断无法响应，可能会丢失。

假如你订了 2 份外卖，一份主食和一份饮料，并且是由 2 个不同的配送员来配送。这次你不用时时等待着，两份外卖都约定了电话取外卖的方式。但是，问题又来了。

当第一份外卖送到时，配送员给你打了个长长的电话，商量发票的处理方式。与此同时，第二个配送员也到了，也想给你打电话。

但是很明显，因为电话占线（也就是关闭了中断响应），第二个配送员的电话是打不通的。所以，第二个配送员很可能试几次后就走掉了（也就是丢失了一次中断）。

### 中断丢失的问题

为了解决中断丢失，Linux 将中断处理分为两个阶段：

- 上半部用来快速处理中断，在中断禁止模式下运行，主要处理跟硬件紧密相关的或时间敏感的工作。
- 下半部用来延迟处理上半部未完成的工作，通常以内核线程的方式运行。

比如说前面取外卖的例子，上半部就是你接听电话，告诉配送员你已经知道了，其他事儿见面再说，然后电话就可以挂断了；下半部才是取外卖的动作，以及见面后商量发票处理的动作。

这样，第一个配送员不会占用你太多时间，当第二个配送员过来时，照样能正常打通你的电话。

最常见的网卡接收数据包的例子：
网卡收到数据包后，会通过**硬件中断**的方式，通知内核有新数据到了。然后内核就调用中断处理程序来响应。

上半部，快速处理，就是把网卡的数据读到内存中，然后更新硬件寄存器的状态，表示数据读好了，最后发送**软中断**信号，通知下半部处理。

下半部被软中断信号唤醒后，需要从内存找到网络数据，按照网络协议，对数据进行逐层解析和处理，再送给应用程序。

实际上，上半部会打断 CPU 正在执行的任务，然后立即执行中断处理程序。而下半部以内核线程的方式执行，并且每个 CPU 都对应一个软中断内核线程，名字为 `ksoftirqd/CPU 编号`，比如说， 0 号 CPU 对应的软中断内核线程的名字就是 `ksoftirqd/0`。

### 查看软中断和内核线程

- `/proc/softirqs` 提供了软中断的运行情况
- `/proc/interrupts` 提供了硬中断的运行情况

### 软中断 CPU 使用率升高

每个 CPU 都有一个软中断内核线程，当软中断事件的频率过高时，内核线程也会因为 CPU 使用率过高而导致软中断处理不及时，引发网络收发延迟，调度缓慢等性能问题。

- sar，是一个系统活动报告工具，可以实时查看系统的当前活动，又可以配置保存和报告历史统计数据
- hping3 是一个可以构造 TCP/IP 协议数据包的工具，可以对系统进行安全审计，防火墙测试等。
- tcpdump 是一个常用网络抓包工具，常用来分许网络问题。

## CPU 性能瓶颈分析

![](performance-tools.png)

![](performance-tools2.png)

要搞清楚性能指标的关联性，就要知道每种性能指标的工作原理。

为了缩小排查分为，通常可以先运行几个支持指标比较多的工具，top，vmstat，pidstat。

![](perf-tools-relation.png)

## CPU 性能优化

### 应用程序优化
