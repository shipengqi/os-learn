---
title: grup
weight: 8
---

# grup

grup 是内核启动的引导软件。

CentOS 6 grup 所有文件需要手动修改。CentOS 7 提供了命令行工具。

配置文件：

- `/etc/default/grup`：默认的 grup 配置文件。
- `/etc/grup.d/`
- `/boot/grup2/grup.cfg`：这个文件不应该直接修改，应该修改默认的配置文件。
- `grup2-mkconfig -o /boot/grup2/grup.cfg`：该命令会生成新的 grup 配置文件。

默认配置文件 `/etc/default/grup`：

```bash
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
~
```
