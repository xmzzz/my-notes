文章标题： 用 GDB 远程调试 QEMU Ubuntu 下的 Linux 内核

- 作 者： 明 政

- 邮 箱： xingmingzheng@iscas.ac.cn

希望本文可以帮你：

- 在 x86 Ubuntu 22.04 环境下，使用 QEMU 启动 x86 Ubuntu 并用 GDB 调试该操作系统
- 在 x86 Ubuntu 22.04 环境下，使用 QEMU 启动 RISCV64 Ubuntu 并用 GDB 调试该操作系统
- 为 Ubuntu 22.04 更换 Linux 内核

本文在一些细节上描述并不够详细，相关依赖包在过程中也没有仔细记录（TODO：后续补充）。因此如果过程中遇到奇怪的错误，有可能是缺少相关依赖。

文章末尾标注了参考链接，感谢来自网络的资料为我提供了帮助。如果有涉及版权问题，可以联系我修改或删除。

# 前言

最近在做 ply 工具往 RISCV64 上的移植，目前可以在 RISCV64 上编译安装并可以运行部分示例，而使用 BEGIN 和 END 功能时会出错。从而也使`sudo ply -T`会报错。  
具体可参阅：  

- <https://github.com/xmzzz/ply/tree/Initial-support-for-riscv64>   
- <https://github.com/iovisor/ply/issues/81>  

阅读 ply 代码后发现错误发生在 Linux 内核中的 sys_perf_event_open 系统调用中，因此考虑搭建可以调试 Linux 内核的环境，单步跟踪这部分的内核代码运行情况，便于分析错误原因。  

本人 Linux 内核基础薄弱，调试环境的搭建过程并不顺利，网上很多资料都是用 QEMU 和 Busybox 启动 Linux 内核，并用 GDB 远程调试。我尝试搭建出了这样的环境，但似乎很难解决我遇到的问题。  

我需要的是调试整个发行版 OS 的运行，安装 ply 工具，并往系统调用打上断点调试。

# 环境准备

我的环境如下：

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.1 LTS
Release:	22.04
Codename:	jammy

$ cat /proc/version_signature 
Ubuntu 5.15.0-48.54-generic 5.15.53

$ uname -rp
5.15.0-48-generic x86_64

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

# 第一部分 更换 QEMU 中 x86 Ubuntu 默认内核并用 GDB 远程调试

## 下载 Linux 内核源码

根据不同需求，可以选择下载完整的 Linux 内核源码，也可以只下载指定版本号的 Linux 内核源码。

1. 下载完整的 Linux 内核源码  

完整的 Linux 内核源码包含所有已发布的 stable 内核代码，可以用 `git checkout v5.15.0` 方便的切换到需要的版本号。不过所需下载的代码文件相对较大，官方仓库下载速度可能会慢，建议用如下方法加速：  

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
$ git branch -ar
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

- 进入 Linux 内核源码根目录 

- 如果内核源码目录不是干净的，则需要先运行 `make mrproper` ，该命令将清除之前编译的配置文件，中间文件和结果文件，也包括 `.config` 配置文件。

接下来
```
$ cp /boot/config-5.15.0-48-generic .config
$ make menuconfig
```

- 将会打开图形界面，方便进行选项配置。进行 x86 Ubuntu 内核调试需要关闭 KPTI、 KASLR 选项，并打开 DEBUG_INFO 选项。具体位置如下，注意内核选项的勾选状态用 < > <*>表示：

> Processor type and features → Randomize the address of the kernel image (KASLR) < >  
> Security options → Remove the kernel mapping in user mode < >  
> Kernel hacking → Compile-time checks and compiler options → Compile the kernel with debug info <*>  
> Kernel hacking → Compile-time checks and compiler options → Provide GDB scripts for kernel debugging <*>  

- 另外，因为需要用 QEMU 调试，还需要调整以下选项：

> Device drivers → Network device support → Virtio network driver <*>  
> Device drivers → Block devices → Virtio block driver <*>  
> Binary Emulations → x32 ABI for 64-bit mode, turn this OFF < >  
> Enable loadable modules support → Module unloading - Forced module unloading <*>  

- 以上选项的具体影响还未测试，我在尝试的过程中将疑似有关的选项都关掉了，因此目前搭建成功的调试环境不确定与这些内核选项的关系度。（TODO：留待以后验证）

- 内核选项编辑好之后，退出并保存。将在内核源码根目录下生成 `.config` 文件。

- 最后在 `.config` 文件中找到：
 
`CONFIG_SYSTEM_TRUSTED_KEYS="debian/canonical-certs.pem"`  
`CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"`  

- 并修改为空值：  

`CONFIG_SYSTEM_TRUSTED_KEYS=""`  
`CONFIG_SYSTEM_REVOCATION_KEYS=""`  

- 否则编译内核会出现认证相关错误。

2. 编译 Linux 内核

- 这一步我们将把内核编译并打包为 `.deb` 文件，之后拷贝到虚拟机里进行安装。本文后面也会介绍编译到宿主机本地，并用编译好的内核启动 QEMU Ubuntu 操作系统的方法。

- 在内核根目录下运行以下命令开始内核编译：

```
$ make bindeb-pkg -j4
```

- 编译过程将持续一段时间，成功编译后将会在内核源码上一层目录生成多个 `.deb` 文件，运行 `cd .. ; ls` 就可以看到。

## 使用 QEMU 启动 Ubuntu 22.04

QEMU 可以通过源码安装，也可以 `apt` 安装，过程可参考相关文档，暂不详述。

- 首先在 Ubuntu 官网下载好 `ubuntu-22.04.1-desktop-amd64.iso` 镜像文件

```
wget https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-desktop-amd64.iso
```

- 创建 QEMU img 文件，命名为 `test.qcow2`  

```
$ qemu-img create -f qcow2 test.qcow2 30G
```

- 此处的 30G 容量是自定义的，安装 Ubuntu 之后占用大概 20G ，剩余 10G 左右的空闲空间。我发现随着 `test.qcow2` 虚拟磁盘的使用，它在宿主机上所占据的空间会变的虚高，而删除虚拟机里的文件并不会使 `test.qcow2`虚拟磁盘所占空间变小。我还没有找到压缩虚拟磁盘大小的便捷方法，所以没有设置太大，等不够用了可以用如下命令增加空间：

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

- 虚拟机重启后，将内核编译打包的 `.deb` 文件拷贝到虚拟机，我用的方法是 `sshfs` 挂载远程目录。在虚拟机中运行

```
$ mkdir tmp_sshfs
$ sshfs name@192.168.host.ip:/home/name/work/path/ ~/tmp_sshfs
```

- 输入主机登录密码，成功挂载后就可以在 `tmp_sshfs` 目录下找到 `.deb` 文件，把它们拷贝到虚拟机

- 安装 `*.deb`，就会更换虚拟机 Ubuntu 中的默认启动内核

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

（ 此参数不添加应该也不影响，因为我们在编译内核时已经取消了该选项，为了保险起见我加上了。 ）

- 更新 grub ，并关闭虚拟机

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
    -drive if=virtio,file=test.qcow2,cache=none \
    -device isa-debugcon,iobase=0x402,chardev=gdb0,id=d1 \
    -vga virtio \
    -enable-kvm \
```

- 其中 `-s` 用来指定 `localhost:1234` 端口进行 GDB 远程调试， `-S` 会让 QEMU 启动时暂停运行并等待 GDB 连接。

## 用 GDB 调试 Ubuntu 内核

- 新开一个终端，在 Linux 内核源码根目录下运行 GDB ，加载符号文件，并远程连接到 QEMU

```
$ gdb
(gdb) file ./kernel.sym
(gdb) target remote :1234
(gdb) hb start_kernel
(gdb) c
```

- 此时，虚拟机中的 Ubuntu 22.04 操作系统将会被启动，并先在 `start_kernel` 断点处停下，尝试 GDB 中的 `s`、`l`、`p`、`n`等命令，都可以有正确结果。

- 现在便可以使用 `b __do_sys_perf_event_open` 命令在系统调用上打断点，运行 ply 工具的示例并调试。

# 第二部分 用 GDB 远程调试 QEMU 中的 RISCV64 Ubuntu 22.04 内核

## 一点失败过程的记录

调试 RISCV64 版本的 Ubuntu 内核，我尝试了类似上面 第一部分 的方法，在宿主机上编译内核 `.deb` 文件，拷贝到虚拟机中安装替换内核。

需要先安装 riscv-gnu-toolchain 交叉编译工具，可以参考 汪辰老师 的文章 https://zhuanlan.zhihu.com/p/544827596 

也可以 `apt` 安装，我试了也可以正常编译内核 

```
$ sudo apt install binutils-riscv64-linux-gnu
$ sudo apt install gcc-riscv64-linux-gnu
$ riscv64-linux-gnu-gcc --version
riscv64-linux-gnu-gcc (Ubuntu 11.2.0-16ubuntu1) 11.2.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

先拷贝 RISCV64 虚拟机中 `/boot/` 下的 config 文件，注意交叉编译内核用如下指令

```
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- menuconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j4 bindeb-pkg
```

编译的新内核在虚拟机中安装后启动失败了，原因还未知，U-boot 信息如下：

```
...
...
Scanning virtio 0:1...
Found /boot/extlinux/extlinux.conf
Retrieving file: /boot/extlinux/extlinux.conf
U-Boot menu
1:	Ubuntu 22.04.1 LTS 5.15.72
2:	Ubuntu 22.04.1 LTS 5.15.72 (rescue target)
3:	Ubuntu 22.04.1 LTS 5.15.0-1018-generic
4:	Ubuntu 22.04.1 LTS 5.15.0-1018-generic (rescue target)
Enter choice: 1
1:	Ubuntu 22.04.1 LTS 5.15.72
Retrieving file: /boot/initrd.img-5.15.72
Retrieving file: /boot/vmlinuz-5.15.72
append: root=LABEL=cloudimg-rootfs ro earlycon
kernel_comp_addr_r or kernel_comp_size is not provided!
2:	Ubuntu 22.04.1 LTS 5.15.72 (rescue target)
Retrieving file: /boot/initrd.img-5.15.72
Retrieving file: /boot/vmlinuz-5.15.72
append: root=LABEL=cloudimg-rootfs ro earlycon single
kernel_comp_addr_r or kernel_comp_size is not provided!
3:	Ubuntu 22.04.1 LTS 5.15.0-1018-generic
Retrieving file: /boot/initrd.img-5.15.0-1018-generic
Retrieving file: /boot/vmlinuz-5.15.0-1018-generic
append: root=LABEL=cloudimg-rootfs ro earlycon
Retrieving file: /lib/firmware/5.15.0-1018-generic/device-tree/qemu-riscv.dtb
** File not found /lib/firmware/5.15.0-1018-generic/device-tree/qemu-riscv.dtb **
Moving Image from 0x84000000 to 0x80200000, end=81f2a000
## Flattened Device Tree blob at ff7384a0
   Booting using the fdt blob at 0xff7384a0
   Using Device Tree in place at 00000000ff7384a0, end 00000000ff73ce15

Starting kernel ...

[    0.000000] Linux version 5.15.0-1018-generic (buildd@riscv64-qemu-lgw01-030) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #21-Ub
untu SMP Mon Aug 22 13:48:15 UTC 2022 (Ubuntu 5.15.0-1018.21-generic 5.15.46)

```

5.15.72 版本内核是新编译安装的，可见并没有成功启动，而是跳过后启动了原有的内核。

我换了种方式搭建内核调试环境，方法如下：

## 开始搭建调试环境

- 首先在工作目录下载 Ubuntu 官方提供的 Ubuntu 22.04.1 preinstalled SD-card images for SiFive HiFive Unmatched，并解压

```
$ wget https://cdimage.ubuntu.com/releases/22.04.1/release/ubuntu-22.04.1-preinstalled-server-riscv64+unmatched.img.xz
$ xz -dk ubuntu-22.04.1-preinstalled-server-riscv64+unmatched.img.xz
```

- 安装以下依赖包

```
$ sudo apt install qemu-system-misc opensbi u-boot-qemu qemu-utils
```

- 下载的镜像默认的根目录磁盘总容量约 4.2G，剩余可用空间约 1.4G 。之后安装更换新内核以及安装调试符号文件需要 10G 左右的空间，所以这里建议先增加 15G

```
$ qemu-img resize -f raw ubuntu-22.04.1-preinstalled-server-riscv64+unmatched.img +15G
```

- ubuntu Wiki 提供了 QEMU 启动该镜像的脚本，为了方便用宿主机 ssh 访问，修改如下

- 创建 `boot.sh` 文件，并添加以下内容

```
#!/bin/bash
qemu-system-riscv64 \
        -machine virt \
        -nographic \
        -m 2048 \
        -smp 4 \
        -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.elf \
        -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
        -device e1000,netdev=net0 \
        -netdev user,id=net0,hostfwd=tcp::5555-:22 \
        -drive file=ubuntu-22.04.1-preinstalled-server-riscv64+unmatched.img,format=raw,if=virtio \
```

- 保存退出，修改文件权限，执行脚本启动虚拟机

```
$ chmod +x boot.sh
$ ./boot.sh
```

- 虚拟机启动后可以在宿主机 ssh 访问该虚拟机

```
$ ssh ubuntu@localhost -p 5555
The authenticity of host '[localhost]:5555 ([127.0.0.1]:5555)' can't be established.
ED25519 key fingerprint is SHA256:5yI9WUQ8Q7BgfU4rUNBcN0nUBKcaN8717DBJlI3IlEM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[localhost]:5555' (ED25519) to the list of known hosts.
ubuntu@localhost's password: 
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-1016-generic riscv64)
...
...
```

- 可以编辑 `vim ~/.ssh/config` 文件，添加如下内容，方便以后的 ssh 连接

```
Host riscv
 Hostname localhost
 User ubuntu
 Port 5555
```

- 下次访问就会方便一些

```
$ ssh riscv
```

## 安装 dbgsym 符号文件

- 首先添加符号文件对应的源

```
$ codename=$(lsb_release -c | awk '{print $2}')
$ sudo tee /etc/apt/sources.list.d/ddebs.list << EOF
deb http://ddebs.ubuntu.com/ ${codename} main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-security main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-updates main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-proposed main restricted universe multiverse
EOF
```

- 添加访问符号文件服务器的 GPG 密钥

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C8CAB6595FDFF622
```

- 更新源文件

```
$ sudo apt-get update
```

- 这时运行以下命令就可以看到包含 dbgsym 符号文件的内核版本

```
$ apt-cache search linux-image | grep 5.15.0
linux-image-5.15.0-1007-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1008-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1011-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1012-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1014-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1015-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1016-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1017-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1018-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1019-generic - Linux kernel image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1020-generic-dbgsym - Linux kernel debug image for version 5.15.0 on RISC-V SMP
linux-image-5.15.0-1018-generic-dbgsym - Linux kernel debug image for version 5.15.0 on RISC-V SMP

```

- 搜索可以 apt 安装的内核源码

```
$ apt-cache search linux-source
linux-source - Linux kernel source with Ubuntu patches
linux-source-5.15.0 - Linux kernel source for version 5.15.0 with Ubuntu patches
```

- 从以上结果看到，linux-image-5.15.0-1018-generic 版本的内核比较全，接下来安装内核以及调试符号文件，并重启更换内核

```
$ sudo apt install linux-image-5.15.0-1018-generic
$ sudo apt install linux-image-5.15.0-1018-generic-dbgsym
$ sudo shutdown -r now
```

- 将调试符号文件拷贝到宿主机

- 调试符号文件被安装在了 `/usr/lib/debug/boot/` 目录下，可以用 sshfs 挂载远程目录的方式拷贝到宿主机

```
$ mkdir tmp_sshfs
$ sshfs name@192.168.host.ip:/home/name/work/path/ ~/tmp_sshfs
$ cp /usr/lib/debug/boot/vmlinux-5.15.0-1018-generic ~/tmp_sshfs/work/path/
```

- 宿主机安装内核源码，也可以直接下载源码包

```
$ sudo apt install linux-source-5.15.0
```

- 安装后可以在 `/usr/src/` 目录下找到 `linux-source-5.15.0.tar.bz2`

- 解压内核源码

```
$ tar -jxvf linux-source-5.15.0.tar.bz2
```

- 这时内核源码就存放在 `/usr/src/linux-source-5.15.0/` 目录下

- 我想在宿主机上同时启动 RISCV64 和 x86 两个架构的 Ubuntu 虚拟机并对比调试，所以在启动脚本中没有使用 `-s`，改用了 `gdb-socket` 的方式

- 修改启动脚本 `vim boot.sh` ，添加调试参数，并启动

```
#!/bin/bash
qemu-system-riscv64 \
        -machine virt \
        -nographic \
        -m 8192 \
        -smp 6 \
        -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.elf \
        -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
        -device e1000,netdev=net0 \
        -netdev user,id=net0,hostfwd=tcp::5555-:22 \
        -drive file=ubuntu-22.04-preinstalled-server-riscv64+unmatched.img,format=raw,if=virtio \
        -chardev socket,path=/tmp/gdb-socket,server=on,wait=off,id=gdb0 -gdb chardev:gdb0 -S  
```

- 此时终端会暂停并等待远程 GDB 连接

- 新开一个终端，使用 gdb-multiarch 远程调试

```
$ sudo apt install gdb-multiarch
GNU gdb (Ubuntu 12.0.90-0ubuntu1) 12.0.90
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
(gdb) set architecture riscv
The target architecture is set to "riscv".
(gdb) file ./vmlinux-5.15.0-1018-generic 
Reading symbols from ./vmlinux-5.15.0-1018-generic...
(gdb) target remote /tmp/gdb-socket
Remote debugging using /tmp/gdb-socket
0x0000000000001000 in ?? ()
(gdb) hb start_kernel 
Hardware assisted breakpoint 1 at 0xffffffff80a00e58 (2 locations)
(gdb) c
Continuing.

Thread 1 hit Breakpoint 1, start_kernel () at /build/linux-riscv-6fOnN1/linux-riscv-5.15.0/init/main.c:935
935	/build/linux-riscv-6fOnN1/linux-riscv-5.15.0/init/main.c: 没有那个文件或目录.

(gdb) 

```

- 从找不到源码文件的报错信息可以看出 vmlinux 符号文件应该是用绝对路径编译的，在相应的文件夹下建立快捷方式可以解决

```
$ sudo mkdir /build/linux-riscv-6fOnN1
$ sudo ln -s /usr/src/linux-source-5.15.0/ /build/linux-riscv-6fOnN1/linux-riscv-5.15.0
```

- 重新启动 gdb-multiarch 就可以看到源代码了

```
...
...
Thread 2 hit Breakpoint 1, start_kernel () at /build/linux-riscv-6fOnN1/linux-riscv-5.15.0/init/main.c:935
935	{
(gdb) l
930			&unknown_options[1]);
931		memblock_free_ptr(unknown_options, len);
932	}
933	
934	asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
935	{
936		char *command_line;
937		char *after_dashes;
938	
939		set_task_stack_end_magic(&init_task);
(gdb) 
...
...
```

- 调试环境搭建完成

调试过程中发现代码执行顺序有时会异常，打印变量有时会显示值为 `<optimized out>` 。估计与内核中的编译优化有关，留待后续解决（TODO）  

如果打印变量值显示 `<error reading variable: dwarf2_find_location_expression: Corrupted DWARF expression.>` ，升级 GDB 版本到 11 以上可以解决， 

可参考 https://sourceware.org/bugzilla/show_bug.cgi?id=27999

# 参考链接

- https://www.josehu.com/memo/2021/01/02/linux-kernel-build-debug.html
- https://yulistic.gitlab.io/2018/12/debugging-linux-kernel-with-gdb-and-qemu/
- https://www.nuanyun.cloud/?p=1481
- https://www.hiroom2.com/2016/07/20/ubuntu-16-04-debug-ubuntu-16-04-kernel-with-qemu-gdb-stub/
- https://www.ebpf.top/post/ubuntu-21-10-dbgsym/
- https://bbs.pediy.com/thread-249192.htm
- https://wiki.ubuntu.com/RISC-V
