# 文本搜索

文本搜索一般会使用正则表达式。

## 元字符

常用的元字符：

- `.` 匹配除了换行符外的任意一个字符
- `*` 匹配任意个跟它前面的字符
- `[]` 匹配方括号中的字符类中的任意一个，比如 `[Hh]ello` 就可以匹配 hello 和 Hello。
- `^` 匹配开头
- `$` 匹配结尾
- `\` 转义字符

扩展元字符：

- `+` 匹配前面的正则表达式至少出现一次
- `?` 匹配前面的正则表达式出现一次或者零次
- `|` 匹配前面或者后面的正则表达式

## grep

grep 用来查找文件里符合条件的字符串。 grep 会把符合条件的行显示出来。

```bash
[root@pooky init.d]# grep password /root/anaconda-ks.cfg
# Root password
[root@pooky ~]# grep pass.... /root/anaconda-ks.cfg  # 可以使用元字符 . 匹配任意一个字符
auth --enableshadow --passalgo=sha512
# Root password
[root@pooky ~]# grep pass....$ /root/anaconda-ks.cfg  # $ 表示结尾
auth --enableshadow --passalgo=sha512
# Root password
[root@pooky ~]# grep pass...d$ /root/anaconda-ks.cfg
# Root password
[root@pooky ~]# grep pass.*$ /root/anaconda-ks.cfg  # .* 就表示任意个字符
auth --enableshadow --passalgo=sha512
# Root password
user --groups=wheel --name=admin --passwd=$6$Lh0jvsS/YklFVYDM$WjPFI.WaMd3be/qiyFVUQkjEFN0PGQcnRTJFUDejJMUS24DA.M2rJ039hi/ubRiaNY4QNt661FARlxZqL.nCs0 --iscrypted --gecos="admin"
[root@pooky ~]# grep ^# /root/anaconda-ks.cfg    # 以 # 为开头的行
#version=DEVEL
# System authorization information
# Use CDROM installation media
# Use graphical install
# Run the Setup Agent on first boot
# Keyboard layouts
# System language
# Network information
# Root password
# System services
# System timezone
# X Window System configuration information
# System bootloader configuration
# Partition clearing information
# Disk partitioning information
[root@pooky ~]# grep '#' *.sh   # 当前目录中，查找后缀有 .sh 文件中包含 # 字符串的行
Usage: grep [OPTION]... PATTERN [FILE]...
Try 'grep --help' for more information.

```

## find

find 命令用来在指定目录下查找文件。`find <文件路径> <查找条件> [补充条件]`。

```bash
[root@pooky ~]# find /etc -name pass*   # 查找 /etc 目录下 pass 前缀的文件
/etc/pam.d/passwd
/etc/pam.d/password-auth-ac
/etc/pam.d/password-auth
/etc/openldap/certs/password
/etc/passwd
/etc/selinux/targeted/active/modules/100/passenger
/etc/passwd-
[root@pooky ~]# find /etc -regex .*wd$  # 使用正则 -regex
/etc/security/opasswd
/etc/pam.d/passwd
/etc/passwd
[root@pooky ~]# find /etc -type f -regex .*wd$
/etc/security/opasswd
/etc/pam.d/passwd
/etc/passwd

```

选项：
`-type` : 指定文件类型，如 `find -type f` 查找普通文件

## sed 和 awk

sed 和 awk 是行编辑器。vim 是全文本编辑器。

### sed

sed 命令一般用于对文本内容做替换：`sed [-hnV][-e <script>][-f <script 文件>] [文本文件]`，例如：

`sed '/user1/s/user1/u1' /etc/passwd`

选项：

- `-e`：直接在命令行模式上进行 sed 动作编辑，此为默认选项
- `-f`：将 sed 的动作写在一个文件内，用 `–f filename` 执行 filename 内的 sed 动作
- `-i`：直接修改文件内容
- `-n`：取消默认的完整输出，只打印模式匹配的行
- `-r`：不需要转义

动作：

- `s`：替换
  - `sed 's/old/new/' filename`
  - `sed -e 's/old/new/' -e 's/old/new/' filename ...` 可以执行多次替换脚本但是不能省略 `-e`
  - `sed -i 's/old/nnew/' 's/old/new/' filename ...` 直接修改文件内容
- `d`：删除
  - `sed /匹配模式/d` 删除匹配到的行
- `a`：追加
- `i`：插入
- `c`：更改
- `r`：读取
- `w`：写入，使用和 `r` 类似
- `n`：读入下一行
- `p`：打印匹配的行
- `=`：打印行号

sed 的工作方式：

1. 将文件以行为单位读取到内存（模式空间）
2. 使用 sed 的每个脚本对该行进行操作
3. 处理完成后输出该行

```bash
[root@SGDLITVM0905 ~]# echo a a a > afile
[root@SGDLITVM0905 ~]# sed 's/a/aa/' afile
aa a a
[root@SGDLITVM0905 ~]# sed 's///abc/' afile        # 如果原始字符是 / 和 分隔符冲突会报错
sed: -e expression #1, char 5: unknown option to `s'
[root@SGDLITVM0905 ~]# sed 's!/!abc!' afile        # 可以把分隔符换成 !，或者其他字符
a a a
[root@SGDLITVM0905 ~]# sed -e 's/a/aa' -e 'a/aa/bb' afile  # 最后的分隔符不能省略
sed: -e expression #1, char 6: unterminated `s' command
[root@SGDLITVM0905 ~]# sed -e 's/a/aa/' -e 's/aa/bb/' afile # 执行多个脚本
bb a a
[root@SGDLITVM0905 ~]# sed -e 's/a/aa/;s/aa/bb/' afile # 执行多个脚本的简写形式
bb a a
[root@SGDLITVM0905 ~]# sed -e 's/a/aa/;s/aa/bb/' afile bfile cfile # 可以修改多个文件
bb a a
[root@SGDLITVM0905 ~]# cat afile                      # 原始文件并没有变化
a a a
[root@SGDLITVM0905 ~]# sed -i 's/a/aa/;s/aa/bb/' afile  # -i 可以使修改的内容作用到文件
bb a a
[root@SGDLITVM0905 ~]# cat afile
bb a a
[root@SGDLITVM0905 ~]# sed -i 's/a/aa/;s/aa/bb/' afile > bfile # 如果不行修改原始文件，可使用重定向输出到另一个文件
```

sed 元字符的使用：

```bash
[root@SGDLITVM0905 ~]# head -5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
You have new mail in /var/spool/mail/root
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | sed 's/...//'  # 表示替换三个任意字符为空
t:x:0:0:root:/root:/bin/bash
:x:1:1:bin:/bin:/sbin/nologin
mon:x:2:2:daemon:/sbin:/sbin/nologin
:x:3:4:adm:/var/adm:/sbin/nologin
x:4:7:lp:/var/spool/lpd:/sbin/nologin
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | sed 's/s*bin//'  # 替换 sbin 或 bin 字符为空
root:x:0:0:root:/root://bash                                  # sed 默认只替换每一行第一次匹配的字符
:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/:/sbin/nologin
adm:x:3:4:adm:/var/adm://nologin
lp:x:4:7:lp:/var/spool/lpd://nologin
[root@SGDLITVM0905 ~]# sed 's/(aa)|(bb)/!/' afile        # 匹配到 aa 或者 bb 替换为 !，() 是分组
[root@SGDLITVM0905 ~]# echo axyzb > cfile
[root@SGDLITVM0905 ~]# sed -r 's/(a.*b)/\1:\1' cfile     # \1 引用第一个分组的内容，\n 引用第 n 个分组的内容
axyzb:axyzb
```

其他动作：

```bash
[root@SGDLITVM0905 ~]# cat bfile
b
a
aa
aaa
ab
abb
abbb
[root@SGDLITVM0905 ~]# sed '/ab/d' bfile # 删除匹配到 ab 的行
b
a
aa
aaa
[root@SGDLITVM0905 ~]# sed '/ab/d;s/a/!/' bfile
b
!
!a
!aa
[root@SGDLITVM0905 ~]# sed '/ab/d;=' bfile # = 可以打印行号
1
b
2
a
3
aa
4
aaa
[root@SGDLITVM0905 ~]# sed '/ab/i hello' bfile # 匹配到 ab 的行上面插入 hello
b
a
aa
aaa
hello
ab
hello
abb
hello
abbb
[root@SGDLITVM0905 ~]# sed '/ab/a hello' bfile # 匹配到 ab 的行下面插入 hello
b
a
aa
aaa
ab
hello
abb
hello
abbb
hello
[root@SGDLITVM0905 ~]# sed '/ab/c hello' bfile # 匹配到 ab 的行改写为 hello
b
a
aa
aaa
hello
hello
hello
[root@SGDLITVM0905 ~]# sed '/ab/r afile' bfile # 把 afile 的内容插入到匹配到 ab 的行下面
b
a
aa
aaa
ab
bb a a
abb
bb a a
abbb
bb a a
```

#### sed 多行模式

多行模式的处理命令：

- `N`：将下一行加入到模式空间
- `D`：删除模式空间中的第一个字符到第一个换行符
- `P`：打印模式空间的第一个字符到第一个换行符

```bash
[root@SGDLITVM0905 ~]# cat a.txt
hel
lo
[root@SGDLITVM0905 ~]# sed 'N' a.txt  # 将下一行追加到模式空间
hel
lo
[root@SGDLITVM0905 ~]# sed 'N;s/hel\nlo/!!!/' a.txt
!!!
[root@SGDLITVM0905 ~]# sed 'N;s/hel.lo/!!!/' a.txt   # . 匹配任意字符
!!!
[root@SGDLITVM0905 ~]# cat > b.txt << EOF
> hell
> o bash hel
> lo bash
> EOF
[root@SGDLITVM0905 ~]# cat b.txt
hell
o bash hel
lo bash
[root@SGDLITVM0905 ~]# sed 'N;s/\n//;s/hello bash/hello sed\n/;P;D' b.txt
hello sed
 hello sed

```

### awk

awk 一般用于对文本内容进行统计，按需要的格式进行输出。一般是作为 sed 的一个补充。awk 可以看成是一种编程语言。

awk 和 sed 的区别：

- awk 用于比较规范的文本处理，用于统计数量并输出指定字段
- sed 一般用于把不规范的文本处理为规范的文本

awk 的流程控制：

- 输入数据前例程 `BEGIN{}`，读入数据前执行，做一些预处理操作。
- 主输入循环 `{}`，处理读取的每一行。
- 所有文件读取完成例程 `END{}`，读取操作完成后执行，做一些数据汇总。

常用的写法是只写主输入循环。

#### 记录和字段

- 每一行叫做 awk 的记录
- 使用空格、制表符分隔开的单词叫做字段
- 可以指定分隔的字段

字段的引用：

- awk 中使用 `$1` `$2` ... `$n` 表示每一个字段，`$0` 表示当前行，`awk '{print $1, $2, $3} filename'`
- awk 可以使用 `-F` 改变字段的分隔符，`awk -F ',' '{print $1, $2, $3}' filename`，分隔符可以使用正则表达式

```bash
[root@SGDLITVM0905 ~]# awk '/^menu/{ print $0 }' /boot/grub2/grub.cfg # 打印以 menu 开头的行的，$0 表示当前行
menuentry 'CentOS Linux (3.10.0-957.5.1.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-957.el7.x86_64-advanced-9c9da9f8-f77a-4a9a-99e0-e4f238472355' {
menuentry 'CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-957.el7.x86_64-advanced-9c9da9f8-f77a-4a9a-99e0-e4f238472355' {
menuentry 'CentOS Linux (0-rescue-87b376b725324ec5aaba8f92806dbc8c) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-87b376b725324ec5aaba8f92806dbc8c-advanced-9c9da9f8-f77a-4a9a-99e0-e4f238472355' {
[root@SGDLITVM0905 ~]# awk -F "'" '/^menu/{ print $2 }' /boot/grub2/grub.cfg  # 以单引号为分隔符
CentOS Linux (3.10.0-957.5.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-87b376b725324ec5aaba8f92806dbc8c) 7 (Core)
[root@SGDLITVM0905 ~]# awk -F "'" '/^menu/{ print $1 }' /boot/grub2/grub.cfg
menuentry
menuentry
menuentry
[root@SGDLITVM0905 ~]# awk -F "'" '/^menu/{ print x++,$2 }' /boot/grub2/grub.cfg # 使用运算符，显示序号，x 变量没有定义，默认为 0
0 CentOS Linux (3.10.0-957.5.1.el7.x86_64) 7 (Core)
1 CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
2 CentOS Linux (0-rescue-87b376b725324ec5aaba8f92806dbc8c) 7 (Core)

```

#### 表达式

赋值操作符：

- `=`，`var1 = "name"`，`var2 = $1`
- `++` `--` `+=` `-=` `*=` `/+` `%=` `^=`

算数操作符：

- `+` `-` `*` `/` `%` `^`

系统变量：

- `FS` 字段分隔符，默认是空格和制表符。
- `RS` 行分隔符，用于分割每一行，默认是换行符。
- `OFS` 输出的字段分隔符，用于打印时分隔字段，默认为空格。
- `ORS` 输出行分隔符，用于打印时分隔记录，默认为换行符。
- `NR` 表示当前处理的是第几行。
- `FNR` 行数。
- `NF` 字段数量，所以最后一个字段内容可以用 `$NF` 取出，`$(NF-1)` 代表倒数第二个字段。

```bash
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print $1}'   # BEGIN{FS=":"} 表示在读入之前设置字段分隔符为 :，也可以写成 awk -F ":" '{print $1}'
root
bin
daemon
adm
lp
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print $1,$2}'
root x          # 可以看出输出的字段分隔符默认为空格
bin x
daemon x
adm x
lp x
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk 'BEGIN{FS=":";OFS="-"}{print $1,$2}'   # 输出的字段分隔符设置为 -
root-x
bin-x
daemon-x
adm-x
lp-x
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk 'BEGIN{RS=":"}{print $0}'   # 已 : 为行分隔符，输出每一行
root
x
0
0
root
/root
/bin/bash
bin
x
1
1
bin
/bin
/sbin/nologin
daemon
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk '{print NR}'   # 显示行号
1
2
3
4
5
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk '{print NR, $0}'
1 root:x:0:0:root:/root:/bin/bash
2 bin:x:1:1:bin:/bin:/sbin/nologin
3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
4 adm:x:3:4:adm:/var/adm:/sbin/nologin
5 lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
[root@SGDLITVM0905 ~]# awk '{print FNR, $0}' /etc/hosts /etc/hosts    # FNR 会重排行号
1 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
2 #::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
3
4 16.187.191.150 SGDLITVM0905.hpeswlab.net SGDLITVM0905
1 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
2 #::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
3
4 16.187.191.150 SGDLITVM0905.hpeswlab.net SGDLITVM0905
You have new mail in /var/spool/mail/root
[root@SGDLITVM0905 ~]# awk '{print NR, $0}' /etc/hosts /etc/hosts   # FNR 不会重排行号
1 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
2 #::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
3
4 16.187.191.150 SGDLITVM0905.hpeswlab.net SGDLITVM0905
5 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
6 #::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
7
8 16.187.191.150 SGDLITVM0905.hpeswlab.net SGDLITVM0905
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print NF}'  # NF 输出字段数量
7
7
7
7
7
You have new mail in /var/spool/mail/root
[root@SGDLITVM0905 ~]# head -5 /etc/passwd | awk 'BEGIN{FS=":"}{print $NF}' # $NF 就可以获取到最后一个字段的内容
/bin/bash
/sbin/nologin
/sbin/nologin
/sbin/nologin
/sbin/nologin

```

关系操作符：

- `<` `>` `<=` `>=` `==` `!=` `~` `!~`

布尔操作符：

- `&&` `||` `!`

条件语句：

```bash
if (表达式)
  awk 语句1
[ else
  awk 语句2
]  
```

多个语句可以使用 `{}` 括起来。

```bash
[root@SGDLITVM0905 ~]# cat score.txt
user1 60 61 62 63 64 65
user2 70 71 72 73 74 75
user3 80 81 82 83 84 85
user4 90 91 92 93 94 95
[root@SGDLITVM0905 ~]# awk '{if($2>=80) print $1}' score.txt
user3
user4
[root@SGDLITVM0905 ~]# awk '{if($2>=80) print $1; print $2}' score.txt # 这种写法会把所有的第二个字段输出
60
70
user3
80
user4
90
[root@SGDLITVM0905 ~]# awk '{if($2>=80) {print $1; print $2} }' score.txt # 如果想一起输出要加上 {} ，多个语句一起执行
60
70
user3
80
user4
90
```

`while` 循环：

```bash
while(表达式)
  awk 语句1
```

`do` 循环：

```bash
do {
  awk 语句1
}while(表达式)
```

`for` 循环：

```bash
for(初始值;判断条件;累加)
  awk 语句1
```

可以使用 `break` 和 `continue`。

```bash
[root@SGDLITVM0905 ~]# cat score.txt
user1 60 61 62 63 64 65
user2 70 71 72 73 74 75
user3 80 81 82 83 84 85
user4 90 91 92 93 94 95
[root@SGDLITVM0905 ~]# head -1 score.txt
user1 60 61 62 63 64 65
[root@SGDLITVM0905 ~]# head -1 score.txt | awk 'for(c=2;c<=NF;c++) print c'
2
3
4
5
6
7
[root@SGDLITVM0905 ~]# head -1 score.txt | awk 'for(c=2;c<=NF;c++) print $c' # 输出值
61
62
63
64
65
[root@SGDLITVM0905 ~]# head -1 score.txt | awk 'for(c=2;c<=NF;c++) print $c' # 输出值
61
62
63
64
65
```

数组：

- `数组[下标] = 值`，初始化数组。下标可以是数字，也可以是字符串。
- `for (变量 in 数组)`，`数组[变量]` 获取数组元素
- `delete 数组[下标]` 删除数组元素

```bash
[root@SGDLITVM0905 ~]# cat score.txt
user1 60 61 62 63 64 65
user2 70 71 72 73 74 75
user3 80 81 82 83 84 85
user4 90 91 92 93 94 95
[root@SGDLITVM0905 ~]# awk '{ sum=0; for(column=2;column<=NF;column++) sum+=$column; print sum }' score.txt # 计算每个人的总分
375
435
495
555
[root@SGDLITVM0905 ~]# [root@SGDLITVM0905 ~]# awk '{ sum=0; for(column=2;column<=NF;column++) sum+=$column; avg[$1]=sum/(NF-1); }END{ for( user in avg) print user, avg[user]}' score.txt  # 计算每个人的平均分 并在 END 例程中格式化输出
user1 62.5
user2 72.5
user3 82.5
user4 92.5

```

awk 脚本可以保存到文件：

```bash
[root@SGDLITVM0905 ~]# awk -f avg.awk score.txt
user1 62.5
user2 72.5
user3 82.5
user4 92.5

```

- `-f` 加载 awk 文件
- `avg.awk` 文件的内容：`{ sum=0; for(column=2;column<=NF;column++) sum+=$column; avg[$1]=sum/(NF-1); }END{ for( user in avg) print user, avg[user]}`。

命令行参数数组：

- `ARGC` 命令行参数数组的长度
- `ARGV` 命令行参数数组

```bash
[root@SGDLITVM0905 ~]# cat arg.awk
BEGIN{
  for(x=0;x<ARGC;x++)
    print ARGV[x]
  print ARGC
}
[root@SGDLITVM0905 ~]# awk -f arg.awk
awk
1
[root@SGDLITVM0905 ~]# awk -f arg.awk 11 22 33
awk
11
22
33
4
```

`ARGV[0]` 就是命令本身。

```bash
[root@SGDLITVM0905 ~]# cat avg.awk
{
  sum = 0
  for ( c = 2; c <= NF; c++ )
    sum += $c

  avg[$1] = sum / ( NF-1 )
  print $1, avg[$1]
  
}
END{
  for ( usr in avg)
    sum_all += avg[user]

  avg_all = sum_all / NR
  
  for ( user in avg )
    if ( avg[user] > avg_all )
      above++
    else
      below++

  print "above", above
  print "below", below
  
}
[root@SGDLITVM0905 ~]# awk -f avg.awk score.txt
user1 62.5
user2 72.5
user3 82.5
user4 92.5
above 4
below 1
```

awk 函数：

- 算数函数
  - `sin()` `cos()` `int()` `rand()` `srand()`
- 字符串函数
  - `toupper(s)` `tolower(s)` `length(s)` `split(s,a,sep)` `match(s,r)` `substr(s,p,n)`
- 自定义函数，自定义函数一定要写在 `BEGIN` 主循环 `END` 例程的外面

```bash
function 函数名 ( 参数 ) {
  awk 语句
  return awk 变量
}
```

示例：

```bash
[root@SGDLITVM0905 ~]# awk 'BEGIN{pi=3.14; print int(pi)}'
3
[root@SGDLITVM0905 ~]# awk 'BEGIN{print rand()}'   # 这是一个伪随机数
0.237788
[root@SGDLITVM0905 ~]# awk 'BEGIN{print rand()}'
0.237788
[root@SGDLITVM0905 ~]# awk 'BEGIN{print rand()}'
0.237788
[root@SGDLITVM0905 ~]# awk 'BEGIN{srand();print rand()}'   # srand 会重新获取种子。范围是 0 ~ 1
0.960391
[root@SGDLITVM0905 ~]# awk 'BEGIN{srand();print rand()}'
0.0422737
[root@SGDLITVM0905 ~]# awk 'BEGIN{srand();print rand()}'
0.555768
[root@SGDLITVM0905 ~]# awk 'function a() { return 0 } BEGIN{ print a()}'
0
[root@SGDLITVM0905 ~]# awk 'function double(str) { return str str } BEGIN{ print double("hello")}'
hellohello

```
