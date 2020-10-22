# Linux 性能优化

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