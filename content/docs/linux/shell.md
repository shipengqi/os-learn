# shell

shell 是 Linux 的命令解释器，解释用户对操作系统的操作。CentOS 默认使用的 shell 是 bash。例如，执行 `ls` 命令时，会先被 shell 对命令和参数进行解释，再交给内核执行。

执行命令的方式：

- `bash ./test.sh` 创建一个子进程来运行 `test.sh`
- `./test.sh` 创建一个子进程来运行 `test.sh`，需要 `Sha-Bang` ，也就是脚本文件开头的 `#!/bin/bash` 来判断使用哪种 shell。
- `source ./test.sh` 在当前进程运行 `test.sh`。也就是说 `test.sh` 里面执行的命令会影响当前 bash 进程。比如 `test.sh` 里面只有一行命令为 `cd /var` ，当前进程会切换到 `/var`。如果是 `bash ./test.sh` 当前 bash 就不会切换目录，只是在子进程切换了目录。
- `. test.sh` `.` 是 source 的另一种写法。

**如果希望脚本对当前运行环境产生影响，就使用 `source` 来执行**。

## Linux 启动过程

1. BIOS （基本输入输出系统）引导，在主板上执行，选择引导介质
2. MBR 硬盘引导
3. BootLoader(grub) 启动和引导 Linux 内核，确定内核版本
4. kernel
5. systemd
6. 系统初始化
7. shell

## 管道

管道是进程间通信的一种方式。`|` 管道符，可以将前一个命令的结果传递给后面的命令。

```bash
[root@pooky ~]# cat | ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root     22804 22792  7 21:52 pts/4    00:00:00 -bash
root     22848 22804  0 21:53 pts/4    00:00:00 cat
root     22849 22804  0 21:53 pts/4    00:00:00 ps -f
```

管道符会给前后两个命令分别创建子进程，上面的示例 22848 进程对应 `cat`，22849 进程对应 `ps -f`。管道符会把第一个进程的输出和第二个进程的输入建立连接。

打开另一个终端查看进程信息：

```bash
[root@pooky ~]# cd /proc/22848
[root@pooky 22848]# cd fd
[root@pooky fd]# ls -l
total 0
lrwx------. 1 root root 64 Sep  2 21:54 0 -> /dev/pts/4           # stdin  pts 表示图形终端
l-wx------. 1 root root 64 Sep  2 21:54 1 -> pipe:[14225010]      # stdout pipe 表示管道
lrwx------. 1 root root 64 Sep  2 21:53 2 -> /dev/pts/4
```

如果第二个进程是一个长时间运行的命令，那么对应的 fd 目录下的 0 ，也就是 stdin，也会指向 `pipe:[14225010]`。

注意，管道符会创建子进程来执行命令，也就是说，如果在管道符前后执行 `cd` 某个目录，父进程是不会切换目录的。

## 重定向

- 一个进程运行时，会默认打开标准输入（一般是通过键盘，终端进行输入），标准输出，标准错误输出（默认输出到终端）三个文件描述符。
- 重定向符号其实是把输入和输出和文件建立连接。用文件来代替输入或输出。

### 输入重定向

- `<`，`wc -l | /etc/passwd` 统计 `/etc/passwd` 内容的行数。

```bash
[root@pooky ~]# read var2 < a.txt
[root@pooky ~]# echo $var2
```

将 `a.txt` 内容读入到变量 `var2`。

### 输出重定向

- `>`，将输出到文件前会先清空文件内容。`echo 123 > /path/to/a/file`。
- `>>`，将输出追加到文件。
- `2>`，错误重定向，将错误输出到指定文件。
- `&>`，标准输出和错误输出到指定文件。

## 变量

变量赋值：

- `a=123`，`=` 左右不能有空格。
- `let a=10+20`，变量可以进行计算。
- `r=$(ls -l /etc)`，`$()` 可以得到命令结果。
- 如果变量值有空格或者特殊字符，可以包含在 `""` 或者 `''` 中。
