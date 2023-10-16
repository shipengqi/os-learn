# pty

pty（pseudo terminal）伪终端。tty 可以直接理解为终端。

## tty

tty 全称为 Teletypes（电传打字机），是通过串行线用打印机键盘通过阅读和发送信息的东西，后来这东西被键盘和显示器取代，所以现在叫终端比较合适。

随着 linux 的发展，终端固定再内核层过于僵化，某些进程需要自主实现一个终端模拟器，比如 ssh，xterm。而 tty 完全由内核接管。用户态无法使用 tty 的功
能，于是 linux 提出将终端仿真移动至用户态，这就是 pty 的由来。

当创建一个伪终端时，会在 `/dev/pts` 目录下创建一个设备文件：`/dev/pts/8`。

## pty

pty 由 master 和 slave 两端构成，在任何一端的输入都会传达到另一端。与 tty 不同，系统中并不存在 pty 这种文件，它是由 pts（pseudo-terminal slave）
和 ptmx（pseudo-teiminal master）两种设备文件来实现的。


### pts

pts（pseudo-terminal slave）即伪终端的 slave 端。在 Linux 的 `/dev/pts/` 文件夹下有对应设设备文件。可以通过 tty 命令查看当前用户的登录终端：

```
[root@shcCDFrh75vm7 ~]# tty
/dev/pts/0
```

当我们设备文件 `/dev/pts/0` 进行输出时，屏幕上会显示相应输出：
```
[root@shcCDFrh75vm7 ~]# echo hello >/dev/pts/0
hello
[root@shcCDFrh75vm7 ~]#

```

如果访问别的 slave 文件，比如 `/dev/pts/9`，则会返回权限不足错误（root 除外）。

### ptmx

ptmx (pseudo-terminal master) 是伪终端的 master 端。在 `/dev` 下仅有 2 个 ptmx 文件：

```
[root@shcCDFrh75vm7 ~]# ll /dev/ptmx
crw-rw-rw-. 1 root tty 5, 2 Sep 30 15:31 /dev/ptmx
[root@shcCDFrh75vm7 ~]# ll /dev/pts/ptmx
c---------. 1 root root 5, 2 Jul 28 11:29 /dev/pts/ptmx
```


### 原理

当 linux  系统创建一个新的 terminal 时(比如上面的 `/dev/pts/0`)

1. 首先执行 `ptm = open('/dev/ptmx',...)` 
2. 接下来 `fork()`,然后子进程将打开 `/dev/pts/0`,dup2 到 0，1 和2 句柄上,随后执行 `execl` 启动一个 shell.
   ```c
   pts = open('/dev/pts/1',...);
   dup2(pts, 0); // 对应 lib 库中 stdin
   dup2(pts, 1); // 对应 lib 库中 stdout
   dup2(pts, 2); // 对应 lib 库中 stderr
   close(pts);
   execl("/system/bin/sh", "/system/bin/sh", NULL);
   // 这样 sh 输入数据将全部来自 pts,
   // sh 的输出数据也都全部输送到 pts,也就直接送到了打开 ptmx 的新 terminal 中.
   ```

3. 新 terminal 将启动 GUI,捕获按键数据,然后写入 ptm，pts 将收到数据，进而 sh 将从 stdin 中获得数据，于是 sh 将作进一步运算,将结果送给 stdout 或 
stderr,进而送给 pts,于是 ptm 获得数据,然后 terminal 的 GUI 将数据显示出来。


## 总结

- 真正的硬件终端基本上已经看不到了，现在所说的终端、伪终端都是软件仿真终端 (即终端模拟软件)
- 一些连接了键盘和显示器的系统中，我们可以接触到运行在内核态的软件仿真终端 (tty1-tty6)
- 通过 SSH 等方式建立的连接中使用的都是伪终端
- 伪终端是运行在用户态的软件仿真终端
