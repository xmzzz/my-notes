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
