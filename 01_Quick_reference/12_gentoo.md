# gentoo clang

## tips

```
# mkdir /etc/portage/sets
# vim compiler-clang
# # add a list packages compiled with clang
# emerge @compiler-clang
```

### recompile packages

```
# emerge -pv llvm clang		# 确认后删除 -p
# emerge -pv1 llvm clang	# -1 --oneshot ，不进 @world
```


# SDDM 图形界面相关

	# rc-update add display-manager default
	# rc-service display-manager start
	# rc-service display-manager stop
	# rc-update del xxx default
	# rc-status
	# rc-udpate show

# 修改 /etc/fstab 后重新挂载

	$ mount -o remount /tmp
	$ # or
	$ umount -a & mount -a	# 如果有busy没有umount
	$ mount -o remount /path/to/busy

# network | 无线网络连接

查看网卡名

	# iw dev
	# # (or)
	# ip link

如果网卡未启用

	# ip link set wlp3s0 up

扫描 wifi

	# iw wlp3s0 scan |grep SSID

假设要连 TP-LINK-01 (WPA加密)

	# wpa_passphrase TP-LINK-01 123456pw >> ~/TP-LINK-01.conf
	# # or
	# wpa_passphrase TP-LINK-01 >> ~/TP-LINK-01.conf
	# (输入密码）

连接

	# wpa_supplicant -B -D wext -i wlp3s0 -c ~/TP-LINK-01.conf

分配网址

	# dhcpcd wlp3s0

端开连接

	# iw wlp3s0 disconnect

参考自动脚本：

```
#!/usr/bin/bash

# By archicu
# Version 3.0
# connect wifi by wpa_supplicant

wlst=`ip link |grep 'wlan0' |awk '{print $3}' |awk -F"," '{print $3}'`
# set wlan0 up
if [[ ! $wlst = "UP" ]]; then
	ip link set wlan0 up
	if [ $? -eq 0 ]; then
		echo -e "\e[1;36mwlan0 up...\e[0m"
	else
		exit 0;
	fi
fi

echo -e "\e[1;36mThe wifi you connected: \e[0m"
sum=0
for ifile in `ls /root/*.conf`
do
	ifile=$(basename $ifile |awk -F"." '{print $1}')
	wifi_connect[$sum]=$ifile
	echo -e "\e[1;36m$sum : $ifile\e[0m"
	let sum++
done
echo

echo -e "\e[1;36mDo you want a\e[0m \e[1;32mNEW\e[0m\e[1;36m wifi?\e[0m"
read -p "[yes/no]: " choose

if [ $choose == "no" ]; then
	read -p "Choose wifi: " wilan
	wpa_supplicant -c /root/${wifi_connect[$wilan]}.conf -iwlan0 &
else
	echo -e "\e[1;36mScanning wifi devices...\e[0m"
	wait
	iw dev wlan0 scan |grep SSID
	read -p "Device: " widev
	read -p "Passwd: " wipas
	wpa_passphrase $widev $wipas > /root/$widev.conf
	wpa_supplicant -c /root/$widev.conf -iwlan0 &
fi
sleep 3s

dhcpcd wlan0 &
sleep 9s
```

https://blog.csdn.net/qq_36485711/article/details/106018940

# Portage 包管理

Portage 是 gentoo 的软件包管理器，使用 Python 和 Bash 编写实现。emerge 用户端是与 Portage 交互的接口。

Gentoo 中的包是指用户可以通过 Gentoo repository 获取到的软件。repository 是由多个 ebuild 组成。ebuild 包含了 Portage 维护软件需要的全部信息，如安装、搜索、查询等等。

ebuilds 通常默认存储在 `/var/db/repos/gentoo` 中。用户针对 Portage 的操作都是以 ebuilds 为基准，因此应该定期更新 repository。

## Update Repository

	# emaint sync --auto
	# emaint sync --repo foo
	# emaint sync --allrepos
	# emerge --sync  # 相当于 emaint sync --auto

可以在 `/etc/portage/repos.conf` 文件中设置 `auto-sync = yes/no` 。

有网络限制的情况下，可下载安装最近的 snapshot ：

	# emerge-webrsync

## Search

	$ emerge --search vim

包含描述信息的内容搜索：

	$ emerge --searchdesc vim

搜索可用的 USE 及说明：

	# emerge --ask app-portage/gentoolkit
	# equery --nocolor uses =app-portage/portage-utils-0.93.3

搜索已安装的全部包：

	# qlist -IU
	# emerge world -ep
	# ls /var/db/pkg/*
	# cd /var/db/pkg/ && ls -d */*| sed 's/\/$//'		## remove the end slash
	# cat /var/lib/portage/world		## emerge install

## install

	# emerge --ask xxx/xxx

查看依赖，而不安装：

	# emerge --pretend xxx

## 下载源码，而不安装

	# emerge --fetchonly xxx

默认会先下载源码到 `/var/cache/distfiles/` ，之后解压、编译、安装。

## 查看包的文档

USE 中的 doc 决定是否安装包的文档，可通过下面指令查看是否安装了文档：

	# emerge -vp xxx/xxx

最好在 `/etc/portage/package.use` 中针对每个包控制 USE 中的 doc 是否开启。

文档默认安装在 `/usr/share/doc/` 下。

另外，可以用 app-portage/gentoolkit 中的 equery 安装文档。equery 是用来查询 Portage 数据库的。

	$ equery files --filter=doc alsa-lib

--filter 还可以查看文件安装位置，参看 `man 1 equery`

## 删除包

下面指令告诉 Portage 该包不需要了，可以安全删除了。但并不真正的删除。

	# emerge --deselect xxx

## 更新系统

	# emerge --update --deep --newuse @world
	# emerge --update --deep --newuse --with-bdeps=y @world --exclude=chromium

`--deep` 查询包是否有新的版本，不加则只更新当前系统已有的版本（已安装软件列表在 /var/lib/portage/world 中），并且不查询依赖包的更新。

`newuse` 更新应用 USE 中的变化，通常变化来自 repository 的 profiles，或者系统的 USE settings 调整。

## /etc needs updating

	# emerge --ask app-portage/cfg-update
	# etc-update --automode -3

## 删除孤立的包

	# emerge --update --deep --newuse @world
	# emerge --ask --depclean

## Licenses 及包管理常见问题

- https://wiki.gentoo.org/wiki/Handbook:AMD64/Working/Portage
- https://wiki.gentoo.org/wiki/Upgrading_Gentoo
- https://wiki.gentoo.org/wiki/Ebuild_repository#Repository_synchronization

## 打补丁 | patch

```
$ sudo mv x.patch /etc/portage/patches/dev-util/ply-2.2.0/
$ sudo emerge dev-util/ply
```

## 单独维护包 | injecting

设置为单独维护的包，将不再跟随 Portage 自动更新和处理依赖。

```
# cat /etc/portage/profile/package.provided
sys-kernel/gentoo-sources-6.1.38
```

## linux-firmware

```
# emerge --ask sys-kernel/linux-firmware

All ebuilds that could satisfy "sys-kernel/linux-firmware" have been masked.
``` 

解决办法：

```
# echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" | tee -a /etc/portage/package.license
```

以下方法无效，why?

```
# nano /etc/portage/make.conf
```

添加

```
ACCEPT_LICENSE="-* @FREE"
```

之后执行

```
# emerge --ask --verbose --update --deep --newuse @world
```

无效，仍然报错。

# 安装好后的指南

https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Finalizing

# grub

os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.

```
# echo GRUB_DISABLE_OS_PROBER=false >> /etc/default/grub && sudo update-grub
```

# 安装 OS 时的可选优化

## 镜像 repos 设置文件位置


	nano /etc/portage/repos.conf/gentoo.conf


## USE 减去中文楷体

如果不想用中文楷体，可以通过 USE 设置

```
$ cat etc/portage/package.use/ghostscript-gpl
app-text/ghostscript-gpl -l10n_zh-CN
```

## make.conf

`/etc/portage/make.conf` 中的

```
CHOST="x86_64-pc-linux-gnu"
CPU_FLAGS_x86="aes avx avx2 fma3 mmx mmxext pclmul popcnt sse sse2"
QEMU_SOFTMMU_TARGETS="arm aarch64"
```

## update system

	# emerge -auvDN --with-bdeps=y @world

## 清理系统，查看哪些依赖还没解决

	# emerge @preserved-rebuild

## 查看或修改 gcc 版本

	# eselect gcc list
	# eselect gcc set 2
	# source /etc/profile

## 在内存中编译，设置 tmpfs

在 `/etc/fstab` 中添加：

```
tmpfs  /tmp      tmpfs  size=32G,noatime  0 0
tmpfs  /var/tmp  tmpfs  size=8G,noatime   0 0
```
handbook 相关内容：

OpenRC /etc/fstab

	tmpfs /tmp tmpfs rw,nosuid,noatime,nodev,size=4G,mode=1777 0 0
	
- https://wiki.gentoo.org/wiki/Tmpfs
- https://wiki.gentoo.org/wiki/Portage_TMPDIR_on_tmpfs

	tmpfs /var/tmp         tmpfs rw,nosuid,noatime,nodev,size=4G,mode=1777 0 0
	tmpfs /var/tmp/portage tmpfs rw,nosuid,noatime,nodev,size=4G,mode=775,uid=portage,gid=portage,x-mount.mkdir=775 0 0

每个包独立设置

	/etc/portage/env/notmpfs.conf
	PORTAGE_TMPDIR="/var/tmp/notmpfs"
	
	root #mkdir /var/tmp/notmpfs
	root #chown portage:portage /var/tmp/notmpfs
	root #chmod 775 /var/tmp/notmpfs
	
	/etc/portage/package.env
	app-office/libreoffice		notmpfs.conf
	dev-lang/ghc			notmpfs.conf
	dev-lang/mono			notmpfs.conf
	dev-lang/rust			notmpfs.conf
	dev-lang/spidermonkey		notmpfs.conf
	mail-client/thunderbird		notmpfs.conf
	sys-devel/clang                 notmpfs.conf
	sys-devel/gcc			notmpfs.conf
	sys-devel/llvm                  notmpfs.conf
	www-client/chromium		notmpfs.conf
	www-client/firefox		notmpfs.conf

## tools

Network Manager, sudo, layman, grub, 

## 更新第三方源，编译安装 xanmod 内核

eselect-repository 工具，方便找到 src_prepare-overlay 的源

https://gitlab.com/src_prepare/src_prepare-overlay
https://github.com/xanmod/linux

	# eselect repository list
	# eselect repository enable 364
	# emerge --sync
	# eselect kernel list
	# emerge --search xanmod
	# emerge sys-kernel/xanmod-sources
	# # (config)
	# make nconfig
	# make -j12
	# make modules_install
	# make install
	# genkernel --install initramfs
	# # (grub)

# inittab

修改 inittab 立即生效

	init q


