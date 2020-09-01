# shell

shell 是 Linux 的命令解释器，解释用户对操作系统的操作。CentOS 默认使用的 shell 是 bash。例如，执行 `ls` 命令时，会先被 shell 对命令和参数进行解释，再交给内核执行。

执行命令的方式：

- `bash ./test.sh` 创建一个子进程来运行 `test.sh`
- `./test.sh` 创建一个子进程来运行 `test.sh`，需要 `Sha-Bang` ，也就是脚本文件开头的 `#!/bin/bash` 来判断使用哪种 shell。
- `source ./test.sh` 在当前进程运行 `test.sh`。也就是说 `test.sh` 里面执行的命令会影响当前 bash 进程。比如 `test.sh` 里面只有一行命令为 `cd /var` ，当前进程会切换到 `/var`。如果是 `bash ./test.sh` 当前 bash 就不会切换目录，只是在子进程切换了目录。
- `. test.sh` `.` 是 source 的另一种写法。

**如果希望脚本对当前运行环境产生影响，就使用 `source` 来执行**。

## 管道

管道是进程间通信的一种方式。
