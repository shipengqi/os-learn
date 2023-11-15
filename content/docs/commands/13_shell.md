# Shell

shell 是 Linux 的命令解释器，解释用户对操作系统的操作。CentOS 默认使用的 shell 是 bash。例如，执行 `ls` 命令时，会先被 shell 对命令和参数进行解释，再交给内核执行。

`/bin/bash` 的位置是用于配置登录后的默认交互命令行的，不像 Windows，登录进去是界面，其实就是 `explorer.exe`。而 Linux 登录后的交互命令行是一个解析脚本的程序，默认是 `/bin/bash`。

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
- `2>&1`，把 标准错误输出 重定向到 标准输出，`ls a.txt b.txt 1>file.out 2>&1` 的正确输出和错误输出都重定向到了 `file.out` 文件中。
- `1>&2`，把 标准输出 重定向到 标准错误输出

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

- `$1` `$2` ... `$n` 分别表示第一个参数，第二个，到第 n 个参数。注意**第十个参数，要用 `${10}`**，否则 `$` 会默认和 `1` 结合。

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

- 引号
  - `""`
  - `''`
  - \`\`
- 括号
  - `()` 打开一个子进程，或者数组定义
  - `$()`
  - `(())` 执行算数运算
  - `[]` `[[]]` 测试，获取数组下标元素 `[ 5 -gt 4 ]`，`[[]]` 就可以使用比较运算符 `[[ 5 > 4 ]]`
  - `<>` 输入输出重定向
  - `{}` 输出范围 `echo {0..9}`，会输出 `0 1 2 3 4 5 6 7 8 9`
- 运算和逻辑符号
  - `+` `-` `*` `/` `%`
  - `<` `>` `=`
  - `&&` `||` `!`
- 转义符号
  - `\`
- 其他符号
  - `#` 注释
  - `;` 命令分隔符，`;;` case 语句的分隔符要转义
  - `:` 空指令
  - `.` 和 source 命令相同
  - `~` home 目录
  - `,` 分隔目录
  - `-` 回到上次访问的目录，`cd -`
  - `*` 通配符，匹配 0 或多个字符
  - `?` 条件测试，通配符 匹配任意一个字符
  - `$` 取值符号
  - `|` 管道
  - `&` 后台运行
  - `_` 空格

## 判断

- `$?` 得到命令的执行结果，判断是否为 0，非 0 为不正常退出。使用 `exit` 来退出程序。`exit` 会返回出错命令的错误码，也可以自定义错误码 `exit <code>`。
- `test` 命令检查文件或者比较值。测试语句可以简化为 `[]`。`[]` 的扩展写法 `[[]]` 支持 `&&` `||` `<` `>`。`test <表达式>` 简化为 `[ 表达式 ]`。常用的测试：
  - 字符串测试 `[ str1 = str2 ]` `[ str1 != str2 ]` `[ -z str ]` `-z` 表示字符串长度是否为 0
  - 整数测试 `[ int1 -eq int2 ]` `-eq` 表示等于，`-ge` 大于等于，`gt` 大于，`le` 小于等于，`lt` 小于，`-ne` 不等于。**`[]` 做整数判断时，只能使用 `-eq` 这种形式。想要使用逻辑运算符 `<` 等，要使用 `[[]]`**。
  - 文件测试，`[ -e file]` `-e` 文件是否存在，`-d` 文件存在并且是目录，`-f` 文件存在并且是普通文件，`-b` 文件存在并且是块文件，`-c` 文件存在并且是字符设备文件，`-x` 文件存在并且可执行。

```bash
[root@pooky ~]# test -f /etc/passwd
[root@pooky ~]# echo $?
0
[root@pooky ~]# test -f /etc/passwd2
[root@pooky ~]# echo $?
1
[root@pooky ~]# [ -d /etc/ ]
[root@pooky ~]# echo $?
0
[root@pooky ~]# [ -e /etc/ ]
[root@pooky ~]# echo $?
0
[root@pooky ~]# [ 5 -gt 4 ]
[root@pooky ~]# echo $?
0
[root@pooky ~]# [[ 5 -gt 4 ]]
[root@pooky ~]# echo $?
0
[root@pooky ~]# [ "aaa" = "aaa" ]
[root@pooky ~]# echo $?
0
```

### if

`if-then` 语句：

- `if [ 测试条件成立或执行命令返回值为 0 ]`
- `then` 执行对应的命令
- `fi` 结束

```bash
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

`if-else` 语句：

- `if [ 测试条件成立 ]`
- `then` 执行对应的命令
- `else` 测试条件不成立，执行对应的命令
- `fi` 结束

```bash
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then
    echo "true";
else
    echo "false"
fi
```

`if-elif-else` 语句：

- `if [ 测试条件成立 ]`
- `then` 执行对应的命令
- `elif [ 测试条件成立 ]`
- `then` 执行对应的命令
- `else` 测试条件不成立，执行对应的命令
- `fi` 结束

```bash
a=10
b=20
if [ $a == $b ]; then
   echo "a 等于 b"
elif [ $a -gt $b ]; then
   echo "a 大于 b"
elif [ $a -lt $b ]; then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

嵌套 `if` 语句：

- `if [ 测试条件成立 ]`
- `then` 执行对应的命令
  - `if [ 测试条件成立 ]`
  - `then` 执行对应的命令
  - `fi`
- `fi` 结束

```bash
a=10
b=20
c=30
if [ $a == $b ]; then
  echo "a = b"
  if [ $a = $c ]; then
    echo "a = c"
  fi  
fi
```

### 分支

```bash
case "$变量" in
    "var1" )
      echo "var1" ;;
    "var2" )
      echo "var2" ;;
    "var3"|"var4" )
      echo "var3|var4" ;;
    * ) # 匹配其他情况
      echo "*" ;;
esac
```

## 循环

### for

- `for 参数 in 列表`
- `do` 执行对应的命令
- `done` 封闭一个循环

使用 `{}` 得到列表：

```bash
for i in {1..9}
do
  echo $i
done

# or
for i in {1..9}; do echo $i; done

# or
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

使用命令得到列表，如把当前目录后缀为 txt 的文件改为后缀为 text：

```bash
for filename in `ls *.txt`
do
  mv $filename $(basename $filename .txt).text
done
```

`basename` 命令可以得到文件名，不包含后缀名。

C 风格的 for 循环：

```bash
for((i = 1; i <= 5; i++))  
do  
    echo "$i"  
done  
```

### while

- `while test测试是否成了`
- `do` 执行对应的命令
- `done`

```bash
int=1
while (( $int<=5 ))
do
  echo $int
  let "int++"
done

# or
a=50
while [ "$a" -le 100 ]
do
  echo $a
  ((a++))
done

# 死循环
while :
do
  echo "."
done  
```

### until

和 while 相反，循环条件为 false 时执行，为 true 时停止。

```bash
a=0
until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

### 嵌套循环

`break` 和 `continue` 语句：

```bash
for filename in /etc/profile.d/*.sh
do
  echo $filename
  if [ -x $filename ]; then
    . $filename
  fi
done


# or
for num in {1..9}
do
  if [ $num -eq 5 ]; then
    break # 退出循环
    # continue 跳过当前循环
    echo $num
  fi
done  
```

### 使用循环处理命令行参数

- 命令行参数可以使用 `$1` `$2` .. `$n` 来获取。
- `$0` 表示当前脚本名称
- `$*` 和 `$@` 代表所有位置参数。只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。
- `$#` 传递到脚本的参数个数

```bash
for pos in $*
do
  if [ $pos = help ];then
  fi
done

# or
while [ $# -ge 1 ];
do
  echo $#
  echo "do something"
  shift # 删除第一个参数
done
```

## 函数

定义函数：

```bash
function funcname() {
  echo "do something"
  echo $1
  echo $2
  echo ${10}
  return 0
}
```

执行函数：`funcname`。
函数作用范围的变量：`local 变量名`
函数的参数：`$1` `$2` .. `$n`
返回值：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return 后跟数值 n `(0-255)`

### 函数库

系统函数库：`/etc/init.d/functions`

导入系统函数库，执行函数 `echo_success`：

```bash
[root@pooky init.d]# source /etc/init.d/functions
[root@pooky init.d]# echo_success
[root@pooky init.d]#                                       [  OK  ]
```

自定义函数库：使用 `source` 导入脚本中的函数
