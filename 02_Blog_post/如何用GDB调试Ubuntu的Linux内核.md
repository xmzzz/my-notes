# 主题
希望本文可以在以下几方面为您提供一些帮助：
- 在 x86 Ubuntu 22.04 环境下，使用 QEMU 启动 x86 Ubuntu 并用 GDB 调试该操作系统
- 在 x86 Ubuntu 22.04 环境下，使用 QEMU 启动 RISCV64 Ubuntu 并用 GDB 调试该操作系统
- 为 Ubuntu 22.04 更换 Linux 内核
- 使用指定版本内核启动 QEMU Ubuntu 22.04

本文在一些细节上描述并不够详细，相关依赖包在过程中也没有仔细记录（TODO：后续补充）。因此如果过程中遇到奇怪的错误，有可能是缺少相关依赖。

# 前言
最近在做 ply 工具往 RISCV64 上的移植，目前可以在 RISCV64 上编译安装并可以运行部分示例，而使用 BEGIN 和 END 功能时会出错。从而也使`sudo ply -T`会报错。  
具体可参阅：  

- <https://github.com/xmzzz/ply/tree/Initial-support-for-riscv64>   
- <https://github.com/iovisor/ply/issues/81>  

阅读 ply 代码后发现错误发生在 Linux 内核中的 sys_perf_event_open 系统调用中，因此考虑搭建可以调试 Linux 内核的环境，单步跟踪这部分的内核代码运行情况，便于分析错误原因。  

本人 Linux 内核基础薄弱，调试环境的搭建过程并不顺利，网上很多资料都是用 QEMU 和 Busybox 启动 Linux 内核，并用 GDB 远程调试。我尝试搭建出了这样的环境，但似乎很难解决我遇到的问题。如何利用 Busybox 提供的有限指令编译安装 ply 并运行起来？  

我需要的是调试整个发行版 OS 的运行，安装 ply 工具，并往系统调用打上断点调试。通过请教前辈和搜寻资料，多番尝试终于找到了方法。

# 环境准备
我的环境如下：

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.1 LTS
Release:	22.04
Codename:	jammy
```

安装编译内核需要的依赖：

```
sudo apt update
sudo apt upgrade
sudo apt install libncurses-dev flex bison openssl libssl-dev \
                 dkms libelf-dev libudev-dev libpci-dev       \
                 libiberty-dev autoconf
sudo apt autoremove
```

# 第一部分 编译 Linux 内核

## 下载 Linux 内核源码
根据不同需求，可以选择下载完整的 Linux 内核源码，也可以只下载指定版本号的 Linux 内核源码。

1. 下载完整的 Linux 内核源码  

完整的 Linux 内核源码包含所有版本号的 git commit 记录，可以用 `git checkout` 方便的切换到任意版本号。不过所需下载的代码文件相对较大，官方仓库下载速度可能会很慢，建议用如下方法加速：  
- 先从国内镜像下载部分文件，比如清华的 Tuna Mirror
```
$ git clone https://mirrors.tuna.tsinghua.edu.cn/git/linux.git
正克隆到 'linux'...
remote: Counting objects: 7486886, done.
remote: Total 7486886 (delta 0), reused 0 (delta 0)
接收对象中: 100% (7486886/7486886), 1.49 GiB | 4.15 MiB/s, 完成.
处理 delta 中: 100% (6343453/6343453), 完成.
正在检出文件: 100% (69360/69360), 完成.
$ cd linux/
$ ls
arch  block  certs  COPYING  CREDITS  crypto  Documentation  drivers fs  include  init  ipc  Kbuild  Kconfig  kernel  lib
LICENSES  MAINTAINERS  Makefile  mm  net  README  samples  scripts  security sound  tools  usr  virt
```
- 通过 `git branch` 和 `git branch -ar` 可以看到，目前只下载了 master 分支，还没有其他分支，包括已发布的 stable 版本

```
$ git branch
* master
$ git branch -ar
  origin/HEAD -> origin/master
  origin/master
```
- 通过 `git remote add upstream url` 添加远程仓库, url 为 linux 内核的 git 仓库地址。然后通过git fetch upstream 获取全部的内核源码
```
$ git remote add upstream https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
$ git fetch upstream
remote: Enumerating objects: 1175853, done.
remote: Counting objects: 100% (1175853/1175853), done.
remote: Compressing objects: 100% (163531/163531), done.
接收对象中:   1% (14396/1177036), 9.42 MiB | 5.00 KiB/s
```
- 之后用 `git branch -ar` 就可以看到已下载了所有 stable 版本的内核源码
 
 ```
$  git branch -ar
  origin/HEAD -> origin/master
  origin/master
  upstream/linux-2.6.11.y
  upstream/linux-2.6.12.y
  upstream/linux-2.6.13.y
  upstream/linux-2.6.14.y
  upstream/linux-2.6.15.y
  upstream/linux-2.6.16.y
  upstream/linux-2.6.17.y
  upstream/linux-2.6.18.y
  upstream/linux-2.6.19.y
  ...
```

2. 下载指定版本号的 Linux 内核源码

```
$ wget https://mirrors.aliyun.com/linux-kernel/v5.x/linux-5.15.53.tar.xz
$ tar -xf linux-5.15.53.tar.xz
$ cd linux-5.15.53
```

## 编译 Linux 内核
1. 配置内核选项

在编译内核之前需要配置内核选项，内核选项选不对很可能会导致无法用 Ubuntu 系统启动内核，或者即使成功启动，也无法用 GDB 对内核进行调试。

运行 `make defconfig` 命令会在内核源码根目录下生成针对当前 x86 架构的默认 `.config` 文件。针对 RISCV64 架构的话需要指定 ARCH 参数 `make ARCH=riscv defconfig`

如果对内核选项不熟悉，建议先从 `/boot/` 目录下拷贝可以正常启动的内核配置文件，重命名为 `.config` ，放到内核源码根目录下，再以此为基础进行配置。

进入 Linux 内核源码根目录： 

如果内核源码目录不是干净的，则需要先运行 `make mrproper` ，该命令将清除之前编译的配置文件，中间文件和结果文件，也包括 `.config` 配置文件。

接下来
```
$ cp /boot/config-5.15.0-48-generic .config
$ make menuconfig
```

将会打开图形界面，方便进行选项配置。参考其他教程，进行 x86 Ubuntu 内核调试需要关闭 KPTI、 KASLR，并打开 DEBUG_INFO 选项。具体位置如下，注意内核选项的勾选状态用 < > <*>表示：

> Processor type and features → Randomize the address of the kernel image (KASLR) < >  
> Security options → Remove the kernel mapping in user mode < >
> Kernel hacking → Compile-time checks and compiler options → Compile the kernel with debug info <*>
> Kernel hacking → Compile-time checks and compiler options → Provide GDB scripts for kernel debugging <*>

另外，如果需要用 QEMU 调试，还需要调整以下选项：

> Device drivers → Network device support → Virtio network driver <*>  
> Device drivers → Block devices → Virtio block driver <*>  
> Binary Emulations → x32 ABI for 64-bit mode, turn this OFF < >  
> Enable loadable modules support → Module unloading - Forced module unloading <*>  

以上选项的具体影响还未测试，我参考网上的大量文章，将疑似有关的选项都关掉了，因此目前搭建成功的调试环境不确定与这些内核选项的关系度。（TODO：留待以后验证）

内核选项编辑好之后，退出保存。将在内核源码根目录下生成 `.config` 文件。

最后在 `.config` 文件中找到：  
`CONFIG_SYSTEM_TRUSTED_KEYS="debian/canonical-certs.pem"`  
`CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"`  
并修改为空值：  
`CONFIG_SYSTEM_TRUSTED_KEYS=""`  
`CONFIG_SYSTEM_REVOCATION_KEYS=""`  
否则编译内核会出现认证相关错误。

2. 编译 Linux 内核

这一步我们将把内核编译并打包为 `.deb` 文件，拷贝到虚拟机安装。本文后面也会介绍编译到本地，并用编译好的内核启动 QEMU Ubuntu 操作系统的方法。

在内核根目录下运行以下命令开始内核编译：

```
$ make bindeb-pkg -j4
```
编译过程将持续一段时间，成功编译后将会在内核源码上一层目录生成多个 `.deb` 文件，运行 `cd .. ; ls` 就可以看到。

3. 使用 QEMU 启动 Ubuntu 22.04

QEMU 可以通过源码安装，也可以 `apt` 安装，过程可参考相关文档，暂不详述。

- 首先在 Ubuntu 官网下载好 `ubuntu-22.04.1-desktop-amd64.iso` 镜像文件  
- 创建 QEMU img 文件，命名为 `test.qcow2`  

```
$ qemu-img create -f qcow2 test.qcow2 30G
```
此处的 30G 容量是自定义的，我安装 Ubuntu 之后占用了大概 20G ，剩余 10G 左右的空闲空间。我没有设置太大，是因为我发现随着 `test.qcow2` 虚拟磁盘的使用，它在宿主机上所占据的空间会变的虚高，而删除虚拟机里的文件并不会使 `test.qcow2`虚拟磁盘所占空间变小。我还没有找到压缩虚拟磁盘大小的便捷方法，所以先不设置那么大，等不够用了可以用如下命令增加空间：

```
$ qemu-img resize -f raw test.qcow2 +5G
```
- QEMU 启动 Ubuntu ISO 镜像文件

```
$ qemu-system-x86_64 \
    -m 2048 \
    -enable-kvm \
    -drive if=virtio,file=test.qcow2,cache=none \
    -cdrom ubuntu-22.04.1-desktop-amd64.iso
```

- 在虚拟机中进行 Ubuntu 22.04 的安装过程，完成之后去除 `-cdrom` 参数并重启虚拟机
```
$ qemu-system-x86_64 \
    -m 2048 \
    -enable-kvm \
    -drive if=virtio,file=test.qcow2,cache=none \
```

- 虚拟机重启后，将内核编译打包的 `.deb` 文件拷贝到虚拟机，我用的方法是 `sshfs` 挂载远程目录 。在虚拟机中运行

```
$ mkdir tmp_sshfs
$ sshfs name@192.168.host.ip:/home/name/work/path/ ~/tmp_sshfs
```

输入主机登录密码，成功挂载后就可以在 `tmp_sshfs` 目录下找到 `.deb` 文件，把它们拷贝到虚拟机

- 安装 `*.deb`，替换虚拟机 Ubuntu 中的默认内核

```
$ sudo dpkg -i *.deb
```

- 添加内核启动参数 `nokaslr` ，否则可能会导致无法打断点调试。方法如下：

```
$ sudo vim /etc/default/grub
```

在 `GRUB_CMDLINE_LINUX_DEFAULT` 中添加 ` nokaslr`
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nokaslr"
```

（ 此参数不添加应该也不影响，因为我们在编译内核时已经取消了该选项，我是为保险起见就加上了。 ）

更新 grub ，并关闭虚拟机

```
$ sudo update-grub
$ sudo shutdown now
```

- 在编译的 Linux 内核源码根目录下找到 vmlinux 文件，来制作用于调试的符号文件

```
$ objcopy --only-keep-debug vmlinux kernel.sym
```

- 通过以下命令启动虚拟机

```
$ qemu-system-x86_64 -s -S \
    -m 2048 \
    -chardev stdio,id=gdb0 \
    -drive if=virtio,file=nfstest.qcow2,cache=none \
    -device isa-debugcon,iobase=0x402,chardev=gdb0,id=d1 \
    -vga virtio \
    -enable-kvm \
```

其中 `-s` 用来指定 `localhost:1234` 端口进行 GDB 远程调试， `-S` 会让 QEMU 启动时暂停运行并等待 GDB 连接。

- 运行 GDB ，加载符号文件，并远程连接到 QEMU

```
$ gdb
(gdb) file ./kernel.sym
(gdb) target remote :1234
(gdb) hbreak start_kernel
(gdb) c
```

