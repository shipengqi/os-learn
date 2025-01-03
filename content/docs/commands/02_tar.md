---
title: 打包压缩
weight: 2
---

# 打包压缩

Linux 里面打包和压缩是分开的两个命令：

- `tar`：打包备份，`tar -cf /tmp/backup.tar /etc` 把 `/etc` 目录打包到 `/tmp/backup.tar` 文件。
  - `-c`：打包
  - `-f`：指定打包的文件。
  - `-x`：从备份文件中还原文件，`tar -xf /tmp/backup.tar -C /root` 把 `/tmp/backup.tar` 文件还原到 `/root` 目录下，需要解压缩就加上 `-z` 或者 `-j` 参数。
  - `-C`：还原到指定的目录。
  - `-z`：打包的同时使用 **gzip 压缩**文件，一般后缀会加上 `.gz` 来表明备份文件的压缩方式，比如 `tar -czf /tmp/backup.tar.gz /etc`。`tgz` 是 `.tar.gz` 的简写。
  - `-j`：打包的同时使用 **bzip2 压缩**文件，一般后缀会加上 `.bz2` 或者 `bzip2` 来表明备份文件的压缩方式，比如 `tar -cjf /tmp/backup.tar.bz2 /etc`。`tbz2` 是 `.tar.bz2` 的简写。
  - `-v|--verbose`：显示指令执行过程。
  - `tar -cf - /etc`：`-` 表示将 `tar` 压缩包写入到标准输出，而不是写入文件。
- `gzip`、`bzip2` 压缩，`gzip` 压缩更快，`bzip2` 压缩比例更高。
