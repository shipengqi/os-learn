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
[root@pooky ~]# grep # *.sh   # 当前目录中，查找后缀有 .sh 文件中包含 # 字符串的行
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

sed 命令一般用于对文本内容做替换：`sed '/user1/s/user1/u1' /etc/passwd`

### awk

awk 一般用于对文本内容进行统计，按需要的格式进行输出。

- cut：``
- awk：``
