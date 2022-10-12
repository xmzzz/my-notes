## 1. 安装QEMU依赖包
Required additional packages
git (30 MiB), version manager
glib2.0-dev (9 MiB), this automatically includes zlib1g-dev
libfdt-devel

```
sudo apt-get install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build
```
## 2. 获取QEMU源码并编译安装
```
git clone https://gitlab.com/qemu-project/qemu.git
cd qemu
# 目前尚不明白 init, update 过程的作用
git submodule init
git submodule update --recursive
# Configure QEMU for x86_64 only - faster build
./configure --target-list=x86_64-softmmu --enable-debug
# Build in parallel - my system has 4 CPUs
make -j4
```

## 3. Create a workspace
```
STAGE=~/tl
TOP=$STAGE/teeny-linux
mkdir -p $STAGE
```

## 4. Download and extract the Linux kernel and BusyBox
```
cd $STAGE
curl https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.10.6.tar.xz | tar xJf -
curl https://busybox.net/downloads/busybox-1.26.2.tar.bz2 | tar xjf -
```

## 5. Create a minimal userland with Busybox
```
cd $STAGE/busybox-1.26.2
mkdir -pv $TOP/obj/busybox-x86
make O=$TOP/obj/busybox-x86 defconfig
```

## 6. Enable static linking in Busybox
```
make O=$TOP/obj/busybox-x86 menuconfig
```

### 6.1 Press enter on Busybox Settings  --->

### 6.2 Press the down arrow 26 times until you hit [ ] Build BusyBox as a static binary (no shared libs)

### 6.3 Press Y

### 6.4 Select Exit twice and hit Enter while the cursor is on <Yes> to save

## 7. Build Busybox
```
cd $TOP/obj/busybox-x86
make -j2
make install
```

## 8. Build the directory structure of the initramfs
```
mkdir -pv $TOP/initramfs/x86-busybox
cd $TOP/initramfs/x86-busybox
mkdir -pv {bin,dev,sbin,etc,proc,sys/kernel/debug,usr/{bin,sbin},lib,lib64,mnt/root,root}
cp -av $TOP/obj/busybox-x86/_install/* $TOP/initramfs/x86-busybox
sudo cp -av /dev/{null,console,tty,sda1} $TOP/initramfs/x86-busybox/dev/
```

## 9. Create init and make it executable

### 9.1 Type:
```
vi $TOP/initramfs/x86-busybox/init
```

### 9.2 Paste this in (press i)
```
#!/bin/sh
 
mount -t proc none /proc
mount -t sysfs none /sys
mount -t debugfs none /sys/kernel/debug
 
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
 
exec /bin/sh
```

### 9.3 Type :w to save, then :q to quit

## 10. Make init executable:
```
chmod +x $TOP/initramfs/x86-busybox/init
```

## 11. Create the initramfs:
```
cd $TOP/initramfs/x86-busybox
find . | cpio -H newc -o > ../initramfs.cpio
cd ..
cat initramfs.cpio | gzip > $TOP/obj/initramfs.igz
```

## 12. Config the Linux kernel with the minimal config
```
cd $STAGE/linux-4.10.6
make O=$TOP/obj/linux-x86-allnoconfig allnoconfig
```

## 13. Turn options on for QEMU:
```
cd $STAGE/linux-4.10.6
make O=$TOP/obj/linux-x86-allnoconfig nconfig
```

## 14. Turn these options on:
```
[*] 64-bit kernel

-> General setup
  -> Configure standard kernel features
[*] Enable support for printk

-> General setup
[*] Initial RAM filesystem and RAM disk (initramfs/initrd) support

-> Executable file formats / Emulations
[*] Kernel support for ELF binaries
[*] Kernel support for scripts starting with #!

-> Device Drivers
  -> Character devices
[*] Enable TTY

-> Device Drivers
  -> Character devices
    -> Serial drivers
[*] 8250/16550 and compatible serial support
[*]   Console on 8250/16550 and compatible serial port

-> File systems
  -> Pseudo filesystems
[*] /proc file system support
[*] sysfs file system support

-> Kernel hacking                                                     
    -> Compile-time checks and compiler options 
[*] Debug filesystem

-> Kernel hacking   
[*] Early printk
```
This writes .config to $TOP/obj/linux-x86-allnoconfig

Use ls -a $TOP/obj/linux-x86-allnoconfig to check

## 15. Make the kernel:
```
cd $STAGE/linux-4.10.6
make O=$TOP/obj/linux-x86-allnoconfig -j2
```

## 16. Launch the Linux kernel binary & initrd in QEMU:
```
STAGE=$HOME/tl
TOP=$STAGE/teeny-linux
qemu-system-x86_64 \
    -kernel $TOP/obj/linux-x86-allnoconfig/arch/x86/boot/bzImage \
    -initrd $TOP/obj/initramfs.igz \
    -nographic -append "earlyprintk=serial,ttyS0 console=ttyS0"
```

## 17. Type Control-a x to quit QEMU

## 参考链接
[1] https://wiki.qemu.org/Hosts/Linux
[2] https://www.qemu.org/download/#source
[3] https://www.centennialsoftwaresolutions.com/post/build-the-linux-kernel-and-busybox-and-run-them-on-qemu

