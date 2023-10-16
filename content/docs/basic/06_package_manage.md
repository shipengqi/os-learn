# 包管理器

对于 Windows 系统，最方便的方式就是下载 exe，也就是安装文件。下载后我们直接双击安装即可。
对于 Linux 来讲，也是类似的方法，你可以下载 rpm 或者 deb。这个就是 Linux 下面的安装包。Linux 现在常用的有两大体系，一个是 CentOS 体系，一个是 Ubuntu 体系，前者使用 rpm，后者使用 deb。

CentOS 下面使用 `rpm -i jdk-XXX_linux-x64_bin.rpm` 进行安装，Ubuntu 下面使用 `dpkg -i jdk-XXX_linux-x64_bin.deb`。其中 `-i` 就是 `install` 的意思。

`rpm -qa` 和 `dpkg -l` 可以查看安装的软件列表，`-q` 就是 `query`，`a` 就是 `all`，`-l` 的意思就是 `list`。

- `rpm`，`yum`：CentOS 和 RedHat 使用 `yum` 包管理器，安装包的格式为 `rpm`。
- `apt`：Ubuntu 和 Debian 使用 `apt` 包管理器，安装包的格式为 `deb`。

## rpm 包格式

`bind-9.8.2-0.47.rc1.el6.x86_64.rpm`：

1. 软件名，如 `bind`
2. 软件版本，`9.8.2-0` 是软件版本，版本号格式通常为 `主版本号.次版本号.修正号`。`47` 是发布版本号，表示这个 `rpm` 软件包是第几次编译生成的
3. 系统版本，如 `el6`，表示支持的系统版本是 CentOS 6，`el7` 就是支持的系统版本是 CentOS 7。
4. 平台，如 `x86_64`
5. 特殊名称：

- devel：表示这个 rpm 包是软件的开发包
- noarch：说明这样的软件包可以在任何平台安装和运行，不需要特定的硬件平台

### rpm 命令

常用参数：

- `-q` 查询，`rpm -qa` 查询当前系统安装了那些 rpm 包，`-a` 查询所有。`rpm -q vim-common` 查询 `vim-common` 是否安装。
- `-i` 安装，`rpm -i bind-9.8.2-0.47.rc1.el6.x86_64.rpm` 安装指定软件包
- `-e` 卸载一个或多个，`rpm -e bind` 卸载软件包

## yum

相当于 Windows 上的软件管家。

yum 包管理器解决的问题：

1. rpm 包的安装可能有依赖关系，如：

```bash
$ rpm -i vim-enhanced-7.4.160-5.el7.x86_64.rpm
错误：依赖检测失败：
    vim-common = 2:7.4.160-5.el7 被 vim-enhanced-7.4.160-5.el7.x86_64 需要
```

安装 `vim-enhanced` 需要先安装 `vim-common` 包。这时候就需要先去下载 `vim-common` 包，再安装 `vim-common` 包，才能安装 `vim-enhanced`。

依赖多的时候，就需要手动的管理。

2. 软件包来源不可靠

## yum 源

软件管家会有一个统一的服务端，来保存这些软件，但是我们不知道服务端在哪里。而 Linux 允许我们配置从哪里下载这些软件，地点就在配置文件里面。也就是 yum 源。

- <http://mirror.centos.org/centos/7/>

访问国内镜像源，会比较快：

- <https://developer.aliyun.com/mirror/>
- <http://tel.mirrors.163.com/>

### yum 配置文件

yum 源配置文件： `/etc/yun.repos.d/CentOS-Base.repo`。

使用国内的镜像源，可以直接下载源配置文件：

先备份：
`mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`

再替换原来的文件：
`wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo` 。

运行 `yum makecache` 生成缓存

文件内容：

```bash
[base] # 表示基础应用的包
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
# baseurl 表示包所在的源的路径
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1 # 检测是否被篡改
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

### yum 命令

- `install` 安装
- `remove` 卸载
- `list|grouplist` 查看
- `update` 更新
