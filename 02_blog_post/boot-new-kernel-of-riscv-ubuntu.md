# 前言

在 QEMU 中启动 ubuntu-22.04-preinstalled-server-riscv64+unmatched.img 镜像之后，利用 GDB 调试内核时发现内核代码的执行顺序有时会跳，有时代码执行到某处突然就跳出当前函数了，影响调试。通过查看 `/boot/config-xxx` 文件，发现默认内核开启了编译优化选项，原因可能跟此有关，因此需要编译新的内核并替换。

# 出错过程记录

编译内核先是采用了在宿主机 x86 ubuntu 22.04 下交叉编译出内核安装包，拷贝到 qemu 启动的 riscv ubuntu 下进行安装。

- 宿主机下进入内核源码根目录，我下载的内核版本是 `linux--5.15.72`

- 先从虚拟机中拷贝原内核配置文件 `/boot/config-xxx` -> `.config`

- 在宿主机编译内核
```
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- menuconfig
$ make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j4 bindeb-pkg
```

- 将编译出的 `.deb` 文件拷贝到虚拟机并安装

- 安装之后，在 `/boot/` 目录下可以看到新安装的内核文件 `vmlinuz-5.15.72`

- 重启发现该内核没有启动，而是跳过了，启动信息如下

```
...
...
Scanning virtio 0:1...
Found /boot/extlinux/extlinux.conf
Retrieving file: /boot/extlinux/extlinux.conf
U-Boot menu
1:      Ubuntu 22.04.1 LTS 5.15.72
2:      Ubuntu 22.04.1 LTS 5.15.72 (rescue target)
3:      Ubuntu 22.04.1 LTS 5.15.0-1018-generic
4:      Ubuntu 22.04.1 LTS 5.15.0-1018-generic (rescue target)
Enter choice: 1
1:      Ubuntu 22.04.1 LTS 5.15.72
Retrieving file: /boot/initrd.img-5.15.72
Retrieving file: /boot/vmlinuz-5.15.72
append: root=LABEL=cloudimg-rootfs ro earlycon
kernel_comp_addr_r or kernel_comp_size is not provided!
2:      Ubuntu 22.04.1 LTS 5.15.72 (rescue target)
Retrieving file: /boot/initrd.img-5.15.72
Retrieving file: /boot/vmlinuz-5.15.72
append: root=LABEL=cloudimg-rootfs ro earlycon single
kernel_comp_addr_r or kernel_comp_size is not provided!
3:      Ubuntu 22.04.1 LTS 5.15.0-1018-generic
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
...
...
```

- 在 `/boot/` 目录下，用 `file` 命令查看一下可以正常启动的 `vmlinuz-5.15.0-1018-generic` 文件格式

```
$ file vmlinuz-5.15.0-1018-generic 
vmlinuz-5.15.0-1018-generic: MS-DOS executable PE32+ executable (EFI application) RISC-V 64-bit (stripped to external PDB), for MS Windows
```

- 再看一下新内核 `vmlinuz-5.15.72的文件格式，发现是压缩文件

```
$ file vmlinuz-5.15.72
vmlinuz-5.15.72: gzip compressed data, max compression, from Unix, original size modulo 2^32 19926784
```

- 尝试将 `vmlinuz-5.15.72` 解压缩，备份源文件并替换，命令如下：

```
$ sudo gzip -dc vmlinuz-5.15.72 | sudo dd of=vmlinuz-5.15.72.tmp
$ sudo cp vmlinuz-5.15.72 vmlinuz-5.15.72.bak
$ sudo mv vmlinuz-5.15.72.tmp vmlinuz-5.15.72
```

- 解压后检查一下文件格式，还是不太对

```
$ file vmlinuz-5.15.72 
vmlinuz-5.15.72: data
```

- 果然重启后新内核还是无法启动，启动信息如下

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
Moving Image from 0x84000000 to 0x80000000, end=81396c18
Unhandled exception: Store/AMO access fault
EPC: 00000000fff5b888 RA: 00000000fff61374 TVAL: 0000000080000000
EPC: 0000000080201888 RA: 0000000080207374 reloc adjusted

Code: b383 0385 be03 0405 be83 0485 bf03 0505 (e110)


resetting ...

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
Moving Image from 0x84000000 to 0x80000000, end=81396c18
Unhandled exception: Store/AMO access fault
EPC: 00000000fff5b888 RA: 00000000fff61374 TVAL: 0000000080000000
EPC: 0000000080201888 RA: 0000000080207374 reloc adjusted

Code: b383 0385 be03 0405 be83 0485 bf03 0505 (e110)


resetting ...
...
...
```

原因暂时还未知，未来会知道的。

# “正确” 启动新内核的方法

试了另一种方法可以顺利启动，但不确定这是最好的方式

- 启动虚拟机，在虚拟机中操作如下

- 搜索可以 apt 安装的内核源码

```
$ apt-cache search linux-source
linux-source - Linux kernel source with Ubuntu patches
linux-source-5.15.0 - Linux kernel source for version 5.15.0 with Ubuntu patches
```

- 安装内核源码

```
$ sudo apt install linux-source-5.15.0
$ cd /usr/src
$ tar -jxvf linux-source-5.15.0.tar.bz2
$ sudo cp /usr/src/linux-source-5.15.0 ~/work/path/
$ cd ~/work/path/linux-source-5.15.0
$ sudo chmod 777 * -R
```

- 调整内核选项

```
$ sudo cp /boot/config-5.15.0-1018-generic .config
$ make menuconfig
```

- 需要调整的内核选项，其中 `Enable full Section mismatch analysis` 作用是使能内核的编译选项 `CONFIG_DEBUG_SECTION_MISMATCH` ，防止内联 

1. General setup > Compiler optimization level > Optimize for size (-Os)
2. Kernel hacking > Compile-time checks and compiler options > Enable full Section mismatch analysis <*>

- 确认以下选项是选中状态

1. Kernel hacking > Compile-time checks and compiler options > Compile the kernel with debug info <*>
2. Kernel hacking > Compile-time checks and compiler options > Provide GDB scripts for kernel debugging <*>

- 编译新内核

```
$ make bindeb-pkg -j4
```

- 虚拟机内核编译速度慢，还不知道如何加速，先干点别的，耐心等待

- 编译好后生成版本号为 5.15.53 的内核安装文件

```
$ ls ../
linux-headers-5.15.53_5.15.53-3_riscv64.deb    linux-libc-dev_5.15.53-3_riscv64.deb        linux-upstream_5.15.53-3_riscv64.changes
linux-image-5.15.53-dbg_5.15.53-3_riscv64.deb  linux-source-5.15.0
linux-image-5.15.53_5.15.53-3_riscv64.deb      linux-upstream_5.15.53-3_riscv64.buildinfo
```

- 安装新内核

```
$ cd ..
$ sudo dpkg -i *.deb
```

- 安装后的内核也是压缩格式，解压缩内核文件

```
$ cd /boot
$ sudo gzip -dc vmlinuz-5.15.53 | sudo dd of=vmlinuz-5.15.53.tmp
$ file vmlinuz-5.15.53.tmp
vmlinuz-5.15.53: MS-DOS executable PE32+ executable (EFI application) RISC-V 64-bit (stripped to external PDB), for MS Windows
```

- 目测文件格式正确，备份并替换，重启后新内核可以启动了

```
$ cp vmlinuz-5.15.53 vmlinuz-5.15.53.bak
$ rm vmlinuz-5.15.53
$ cp vmlinuz-5.15.53.tmp vmlinuz-5.15.53
$ sudo shutdown -r now
...
...
Scanning virtio 0:1...
Found /boot/extlinux/extlinux.conf
Retrieving file: /boot/extlinux/extlinux.conf
U-Boot menu
1:	Ubuntu 22.04.1 LTS 5.15.72
2:	Ubuntu 22.04.1 LTS 5.15.72 (rescue target)
3:	Ubuntu 22.04.1 LTS 5.15.53
4:	Ubuntu 22.04.1 LTS 5.15.53 (rescue target)
5:	Ubuntu 22.04.1 LTS 5.15.0-1018-generic
6:	Ubuntu 22.04.1 LTS 5.15.0-1018-generic (rescue target)
Enter choice: 3
3:	Ubuntu 22.04.1 LTS 5.15.53
Retrieving file: /boot/initrd.img-5.15.53
Retrieving file: /boot/vmlinuz-5.15.53
append: root=LABEL=cloudimg-rootfs ro earlycon
Moving Image from 0x84000000 to 0x80200000, end=81f29000
## Flattened Device Tree blob at ff7384a0
   Booting using the fdt blob at 0xff7384a0
   Using Device Tree in place at 00000000ff7384a0, end 00000000ff73ce15

Starting kernel ...

[    0.000000] Linux version 5.15.53 (ubuntu@ubuntu) (gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #3 SMP Wed Oct 19 08:35:10 UTC 2022 (Ubuntu 5.15.0-generic)
[    0.000000] OF: fdt: Ignoring memory range 0x80000000 - 0x80200000
[    0.000000] Machine model: riscv-virtio,qemu
...
...
```

- 新内核启动后，调试内核果然代码的执行顺序基本正常，未见突然跳转的现象

- GDB 调试内核时若提示找不到代码路径，需要用 `dir /src/dir/path` 命令来设置


# 参考链接

- https://www.cnblogs.com/dakewei/p/10756416.html

- https://raspberrypi.stackexchange.com/questions/124759/how-to-extract-aarch64-vmlinuz-gzip-image

