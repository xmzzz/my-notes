# 磁盘扩容

```
$ qemu-img resize openeuler-qemu.raw +50G
$ # 进虚拟机
$ sudo dnf install cloud-utils-growpart
$ lsblk
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS                                                                                                                  
sda              8:0    0  107G  0 disk                                                                                                                              
├─sda1           8:1    0    1M  0 part                                                                                                                              
├─sda2           8:2    0    1G  0 part /boot                                                                                                                        
└─sda3           8:3    0    6G  0 part                                                                                                                              
  └─sysvg-root 253:0    0    6G  0 lvm  /                                                                                                                            
sr0             11:0    1 1024M  0 rom                                                                                                                               
zram0          252:0    0  3.8G  0 disk [SWAP]
$ sudo growpart /dev/sda 3
$ sudo lvextend -L 50G /dev/mapper/sysvg-root
$ df -hT
Filesystem             Type      Size  Used Avail Use% Mounted on
devtmpfs               devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                  tmpfs     2.0G     0  2.0G   0% /dev/shm
tmpfs                  tmpfs     781M  860K  780M   1% /run
/dev/mapper/sysvg-root xfs       6.0G  4.0G  2.1G  66% /
tmpfs                  tmpfs     2.0G     0  2.0G   0% /tmp
/dev/sda2              ext4      974M  172M  735M  19% /boot
tmpfs                  tmpfs     391M     0  391M   0% /run/user/1000
$ sudo xfs_growfs /dev/mapper/sysvg-root
```
> https://blog.51cto.com/u_12740336/6172988
> https://www.cnblogs.com/5201351/p/17518924.html

# copy-on-write (COW) 写时复制(增量)磁盘

```
$ #创建镜像
$ qemu-img create -o backing_file=openeuler-qemu-xfce.qcow2,backing_fmt=qcow2 -f qcow2 test.qcow2
$
$ #查看映像信息
$ qemu-img info --backing-chain test.qcow2
$
$ #修改基础映像位置
$ qemu-img rebase -b another.qcow2 test.qcow2
$
$ #合并映像
$ qemu-img commit test.qcow2
```

> [通过 QEMU 仿真 RISC-V 环境并启动 OpenEuler RISC-V 系统](https://github.com/ArielHeleneto/Work-PLCT/tree/master/awesomeqemu)

# qemu 8.0

```
$ ../configure --help
```

- If you see an error msg like this:

```
network backend 'user' is not compiled into this binary
$
$ sudo apt install libslirp-dev
$
recompile QEMU with --enable-slirp
```

- 在claudine上编译源码

```
$ cd work/
$ git clone https://gitlab.com/qemu-project/qemu.git
$ cd qemu/
$ git submodule init
$ git submodule update --recursive
$ mkdir build
$ cd build/
$ ../configure --help
$ mkdir ~/install
$ ../configure ----------
$ pip3.7 install --user -U sphinx sphinx-rtd-theme ninja
```

- 9p

```
'virtio-9p-pci' is not a valid device model name
$
$ sudo apt install libattr1 libattr1-dev
$ ../configure --enable-kvm --enable-virtfs
$
$ sudo apt install libglib2.0-dev libpixman-1-dev liburing-dev libnfs-dev \
  libseccomp-dev libpulse-dev libcap-ng-dev
$
$ ../configure --target-list=riscv64-softmmu,riscv64-linux-user --enable-kvm --enable-virtfs --enable-slirp --prefix=/home/xmz/work/kernel_dev/qemu/install
```

- install

```
$ wget https://download.qemu.org/qemu-8.0.0.tar.xz
$ tar xvJf qemu-8.0.0.tar.xz
$ cd qemu-8.0.0
$ mkdir build; cd build
$ ./configure  [...] # see previous section 
$ make
$ make install
$  # update env $PATH
```

# use 9p virtio

- qemu 启动脚本添加

```
  -fsdev local,security_model=passthrough,id=fsdev0,path=/share/path/foo \
  -device virtio-9p-pci,id=fs0,fsdev=fsdev0,mount_tag=hostshare"
```

- guest 启动后运行

```
$ mkdir /tmp/host_files
$ sudo mount -t 9p -o trans=virtio,version=9p2000.L hostshare /tmp/host_files
```

- 若提示 `mount: unknown filesystem type '9p'`，需要打开内核选项

```
# enabled in the kernel configuration
    CONFIG_NET_9P=y
    CONFIG_NET_9P_VIRTIO=y
    CONFIG_NET_9P_DEBUG=y (Optional)
    CONFIG_9P_FS=y
    CONFIG_9P_FS_POSIX_ACL=y
    CONFIG_PCI=y
    CONFIG_VIRTIO_PCI=y
# and these PCI and virtio options:
    CONFIG_PCI=y
    CONFIG_VIRTIO_PCI=y
    CONFIG_PCI_HOST_GENERIC=y (only needed for the QEMU Arm 'virt' board)
```
