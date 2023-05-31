# OLK-510 内核编译报错 `undefined reference to 'EVP_PKEY_set_alias_type'`

- 检索到“The function EVP_PKEY_set_alias_type() was previously documented on this page. It was removed in OpenSSL 3.0.”，推测原因是 libssl-dev 版本问题

- openssl版本查询

```
$ openssl version
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
```

- 增加包含旧版本libssl的源

```
$ sudo vim /etc/apt/sources.list
$ #新增 `deb http://security.ubuntu.com/ubuntu focal-security main`
$ apt update
$ apt search libssl
libssl-dev/jammy-updates,jammy-security,now 3.0.2-0ubuntu1.9 amd64 [installed]
  Secure Sockets Layer toolkit - development files
...
libssl1.1/focal-security 1.1.1f-1ubuntu2.18 amd64
  Secure Sockets Layer toolkit - shared libraries

libssl3/jammy-updates,jammy-security,now 3.0.2-0ubuntu1.9 amd64 [installed,automatic]
  Secure Sockets Layer toolkit - shared libraries
...
$
$ apt-cache show libssl-dev 
Package: libssl-dev
Architecture: amd64
Version: 3.0.2-0ubuntu1.9
Multi-Arch: same
Priority: optional
Section: libdevel
Source: openssl
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian OpenSSL Team <pkg-openssl-devel@alioth-lists.debian.net>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 12089
Depends: libssl3 (= 3.0.2-0ubuntu1.9)
Suggests: libssl-doc
Conflicts: libssl1.0-dev
Filename: pool/main/o/openssl/libssl-dev_3.0.2-0ubuntu1.9_amd64.deb
Size: 2373472
MD5sum: 5f786940364827633d35250c8e3be024
SHA1: 5966802190d97888872b5ec121e4bdc66eeebcf4
SHA256: 09409630f0d470153334642508efb2b33e77c7bf5c0c95afb931cb251bdc29f9
SHA512: 922be29ed788aa450887fa165bb042fa2ac3b16d2d2a90744371e7e2beed37f24553e3672c01363ceec5a7238b161a63b8aed15186d1ebb9eb79ae7307642050
Homepage: https://www.openssl.org/
Description-en_GB: Secure Sockets Layer toolkit - development files
 This package is part of the OpenSSL project's implementation of the SSL
 and TLS cryptographic protocols for secure communication over the
 Internet.
 .
 It contains development libraries, header files, and manpages for libssl
 and libcrypto.
Description-md5: 27044468897c45b271f879c7c6e135fe

Package: libssl-dev
Architecture: amd64
Version: 3.0.2-0ubuntu1
...
$
$ apt-cache madison libssl-dev
libssl-dev | 3.0.2-0ubuntu1.9 | http://cn.archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages
libssl-dev | 3.0.2-0ubuntu1.9 | http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages
libssl-dev | 3.0.2-0ubuntu1 | http://cn.archive.ubuntu.com/ubuntu jammy/main amd64 Packages
libssl-dev | 1.1.1f-1ubuntu2.18 | http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages
$
$ apt-cache policy libssl-dev 
libssl-dev:
  Installed: (none)
  Candidate: 3.0.2-0ubuntu1.9
  Version table:
     3.0.2-0ubuntu1.9 500
        500 http://cn.archive.ubuntu.com/ubuntu jammy-updates/main amd64 Packages
        500 http://security.ubuntu.com/ubuntu jammy-security/main amd64 Packages
     3.0.2-0ubuntu1 500
        500 http://cn.archive.ubuntu.com/ubuntu jammy/main amd64 Packages
     1.1.1f-1ubuntu2.18 500
        500 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages
$ sudo apt remove libssl-dev
$ sudo apt-get install libssl-dev=1.1.1f-1ubuntu2.18
```
- （TODO:两版本同时存在）

- https://www.openssl.org/docs/man3.0/man3/EVP_PKEY_id.html
- https://packages.ubuntu.com/focal/i386/openssl/download
- https://blog.csdn.net/qq_27011361/article/details/83277209

# openssl

```
fatal error: openssl/opensslv.h: No such file or directory
$ sudo apt install libssl-dev
```

# tained kernel

```
$ cat /proc/sys/kernel/tainted
```

# IO scheduler

```
$ cat /sys/block/vda/queue/scheduler 
[mq-deadline] kyber bfq none
$ echo bfq > /sys/block/vda/queue/scheduler
$ cat /sys/block/vda/queue/scheduler
mq-deadline kyber [bfq] none
```

# hung_task_warning

The maximum number of warnings to report. During a check interval if a hung task is detected, this value is decreased by 1. When this value reaches 0, no more warnings will be reported. This file shows up if CONFIG_DETECT_HUNG_TASK is enabled.

-1: report an infinite number of warnings.

```
# sysctl kernel.hung_task_warnings
kernel.hung_task_warnings = 0
# echo 20 > /proc/sys/kernel/hung_task_warnings
# sysctl kernel.hung_task_warnings
kernel.hung_task_warnings = 20

```

# 当前内核config

```
$ cat /proc/config.gz | gzip -d > config.tmp
```

# 当前内核启动参数

```
$ cat /proc/cmdline
```

# module 版本检查

如果 ko 与内核版本不一致，加载时可能会报 "invalid module format" 错误。

可以尝试关闭模块版本检查内核参数

```
Enable loadable module support  --->[]   Module versioning support
```

# kernel dbgsym path

```
/usr/lib/debug/boot/vmlinux-5.15.53
```

# How to extract vmlinuz gzip image
```
$ sudo gzip -dc /boot/vmlinuz-xxx | dd of=/boot/vmlinuz-tmp
```

# make clean, make mrproper, make distclean

在make的时候，会重新生成objects， 新的object覆盖旧的objects

make clean  删除大多数的编译生成文件， 但是会保留内核的配置文件.config， 还有足够的编译支持来建立扩展模块

make mrproper 删除所有的编译生成文件， 还有内核配置文件， 再加上各种备份文件

make distclean mrproper删除的文件， 加上编辑备份文件和一些补丁文件

# BTF: .tmp_vmlinux.btf: pahole (pahole) is not available.
Getting this error while compiling the kernel version v5.15.46
```
BTF: .tmp_vmlinux.btf: pahole (pahole) is not available
Failed to generate BTF for vmlinux
Try to disable CONFIG_DEBUG_INFO_BTF
make: *** [Makefile:1106: vmlinux] Error 1
```

install dwarves:
```
sudo apt install dwarves
```

This works for Ubuntu and Debian.

	-- from: https://stackoverflow.com/questions/61657707/btf-tmp-vmlinux-btf-pahole-pahole-is-not-available

# No rule to make target 'debian/canonical-certs.pem'
In ubuntu 22.04 LTS, I was compiling the kernel 5.15.46 after adding a new system call, during the execution of make command I got this error:
```
make[1]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.
make: *** [Makefile:1809: certs] Error 2
```
In your kernel configuration file you will find this line:
```
CONFIG_SYSTEM_TRUSTED_KEYS="debian/canonical-certs.pem"
```
Change it to this:
```
CONFIG_SYSTEM_TRUSTED_KEYS=""
```
Another key has been added to the default Canonical kernel configuration since this answer was posted:
```
CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"
```

	-- from: https://askubuntu.com/questions/1329538/compiling-the-kernel-5-11-11

# CC, LD, CC [M]
While compiling Linux from scratch I realize that there are compile codes that appear while compiling.
For example CC filename , LD filename, CC[M] filename.
What do these codes mean?

The different markings specify the following

[CC] - Compiles the C file into an designated object file. The object file contains the archicture assembler code of that .c file. As it might also reference parts outside its scope. For example calling another function in another .c file. The function calls are left open within the object file, which is later included by the linker. Therefore
[LD] is the proces of linking the compiled objects together, and wire up the function calls that has been left open by the compiler. However, many parts are linked together as the core part of the kernel, while some parts are left out. And thus you see
[CC (M)] for those parts which are compiled as points to be loaded into the kernel at runtime. But which are not linked together in the monolithic part of the kernel. But instead can be inserted when the kernel is booted.

	-- from: https://stackoverflow.com/questions/11697800/what-are-the-codes-such-as-cc-ld-and-ccm-output-when-compiling-the-linux-kern

# Manually adding your new kernel to your GRUB boot menu
If you are using GRUB and the GRUB scripts fail to autodetect your new kernel, you can still manually add the relevant information to /etc/grub.d/40_custom. As the root user, open this file in a text editor and add something like:
```
       menuentry 'kernel x.y.z' {
       echo 'Loading Linux x.y.z ...'
       linux /boot/vmlinuz root=PARTUUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx <kernel parameters>
       echo 'Loading initial ramdisk ...'
       initrd /boot/initrd
       }
```
You need to change xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx to the actual PARTUUID of your root partition, and replace <kernel parameters> with valid kernel parameters appropriate to your setup. The kernel cannot read UUIDs or filesystem labels without a working root partition, therefore you cannot use them without an initrd.

The most important lines in the example above are the lines beginning with menuentry, linux and if you are using one, initrd. So let's give a quick explanation of what they do; The line beginning with menuentry signals the start of a menu entry and gives the name that will appear in the menu itself. The line beginning with linux tells GRUB where your kernel executable is located on the filesystem, where your root filesystem is located, along with any kernel parameters you may want to boot the kernel with. The line beginning with initrd tells GRUB where to find your initrd image (this line can be omitted if you are not using an initrd). If you're not using an initrd you need to use either root=PARTUUID= (as shown above) or a device node (such as /dev/sda2) in the line beginning with linux above.

Once you have added the relevant information, you then need to update your GRUB configuration so the changes take effect. We will discuss how you do that in the "Updating GRUB" sub-section below.

Further GRUB boot options may sometimes be desirable, but that is beyond the scope of this article. Please refer to the GRUB documentation if you have any questions about any of GRUB's boot options.

	-- from: https://wiki.linuxquestions.org/wiki/How_to_build_and_install_your_own_Linux_kernel

# How to remove a newly installed kernel?
Delete *6.0.0* from /boot. This is what got installed. Delete /lib/modules/*6.0.0* as well. This is what got modules_installed. Run update-grub afterwards.

Note: before deleting, do an echo with the paths that I gave you to make sure they correspond to a single kernel and that is the kernel you want.

	-- from: https://stackoverflow.com/questions/25993363/how-to-remove-a-newly-installed-kernel


