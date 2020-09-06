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
- `r=$(ls -l /etc)`，`$()` 可以得到命令结果。`` ` ` `` 反引号也可以得到命令结果。
- 如果变量值有空格或者特殊字符，可以包含在 `""` 或者 `''` 中。

默认值，`test2=${2-_}`，`-` 好表示当 `$2` 为空值时，使用 `_` 为值。

变量引用：

- `${变量名}`，有些情况下也可以使用 `$变量名`。

变量作用范围：

- 变量的默认作用范围是当前 shell 进程。

```bash
[root@pooky ~]# a=1
[root@pooky ~]# bash # 进入一个新的 shell 子进程
[root@pooky ~]# echo $a # a 变量的值是空的

[root@pooky ~]# a=2
[root@pooky ~]# exit
exit
[root@pooky ~]# echo $a # a 变量的值不会被子进程的赋值改变
1
[root@pooky ~]# bash test.sh # bash 打开了一个新的 shell 进程，在 test.sh 中的变量不会作用到父进程
[root@pooky ~]# source test.sh # 可以使用 source 命令，在当前进程执行 test.sh，使变量在父进程生效
```

- `export` 导出变量，`export` 设置环境变量是暂时的，可以被子进程读取到，比如在一个终端 export 的变量，在这个终端打开的 shell 子进程都可以读取到这个变量。
- `unset` 删除变量，`unsert a`

## 环境变量

环境变量，是每个 shell 都可以读取到的变量：

- `env` 查看所有环境变量，`env | more`。
- `set` 也可以查看环境变量。
  - `-e` 传回值不等于 0，则立即退出 shell。
  - `-u` 执行脚本的时候，如果遇到不存在的变量，Bash 默认忽略它。
  - `-x` 默认情况下，脚本执行后，只显示运行结果，这个参数可以显示执行的指令及参数。
  - `-o pipefail` `-e` 参数不适用于管道命令。只要管道符最后一个子命令不失败，管道命令总是会执行成功如 `foo` 是一个不存在的命令，但是 `foo | echo $a` 这个管道命令会执行成功。`-o pipefail` 用来解决这种情况，只要一个子命令失败，整个管道命令就失败，脚本就会终止执行。
- `$?` 上条命令的返回值，不为 0，表示执行失败。
- `$$` 表示当前进程 PID
- `$0` 当前进程的名称
- `$PATH` 命令路径
- `$PS1`

```bash
[root@pooky ~]# echo $?
0
[root@pooky ~]# echo $$
32163
[root@pooky ~]# echo $0
-bash
```

位置变量：

- `$1` `$2` ... `$n` 分别表示第一个参数，第二个，到第 n 个参数。注意**第十个参数**，要用`${10}`，否则 `$` 会默认和 `1` 结合。

`test.sh`：

```bash
#!/bin/bash

echo "文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```

```bash
[root@pooky ~]# ./test.sh 1 2 3
文件名：./test.sh
第一个参数为：1
第二个参数为：2
第三个参数为：3
```

## 环境变量配置文件

1. `/etc/profile`
2. `/etc/profile.d`
3. `~/.bash_profile`
4. `~/.bashrc`
5. `/etc/bashrc`

**执行顺序从上到下**。

**`/etc`** 目录下的文件对系统中所有用户都永久有效。**`~`** 目录下的文件只对某个用户永久有效。

配置文件分为 `bashrc` 和 `profile` 两种，因为登录用户分为两种，login shell 和 no login shell。比如 `su - user1` 切换了用户和 home 目录，这属于 login shell。login shell 会执行上面所有的配置文件。如果使用 `su user1` ，那么只会执行两个 bashrc 文件。

修改可上面的配置文件可以使用 `source <filename>` 使其立即生效。

## 数组

定义数组 `IPS=( 10.0.0.1 10.0.0.2 10.0.0.3)`。

```bash
[root@pooky ~]# IPS=( 10.0.0.1 10.0.0.2 10.0.0.3)
[root@pooky ~]# echo ${IPS[0]}  # 显示执行下标的元素
10.0.0.1
[root@pooky ~]# echo ${IPS[1]}
10.0.0.2
[root@pooky ~]# echo ${IPS[2]}
10.0.0.3
[root@pooky ~]# echo ${IPS}    # 显示的是数据第一个元素
10.0.0.3
[root@pooky ~]# echo ${IPS[@]}  # 显示数组所有元素
10.0.0.1 10.0.0.2 10.0.0.3
[root@pooky ~]# echo ${#IPS[@]}  # 显示数据长度
3

```

## 转义和引用

特殊字符：

- `#` 注释
- `;` 分号
- `\` 转义符
  - `\n` `\r` `\t` 字母转义
  - `\$` `\"` `\\` 非字母转义
- `"` `'` 引号

引用：

- 双引号，不完全引用，中间的内容会进行解释。
- 单引号，完全引用，单引号中间是什么就显示什么，不进行解释。
- 反引号，可以得到命令结果。和 `$()` 的功能一样。

```bash
[root@pooky ~]# var1=123
[root@pooky ~]# echo "$var1"
123
[root@pooky ~]# echo '$var1'
$var1
```

## 运算符

- `expr` 可以用来计算 `expr 4 + 5` ，但是只支持整数。
- 正常的变量赋值，`=` 右边都被当做字符串，如果右边需要计算可以使用 `let`，`(())` 是 `let` 的简化。

```bash
[root@pooky ~]# ((a=4+5))
[root@pooky ~]# echo $a
9
[root@pooky ~]# b=4+5
[root@pooky ~]# echo $b
4+5
[root@pooky ~]# ((a++))
[root@pooky ~]# echo $a
10

```

## 特殊符号

引号
括号
运算和逻辑符号
转义符号
其他符号
