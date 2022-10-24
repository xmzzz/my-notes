# 前言

记录一下跑 kprobe 例子的过程，源代码位于内核源码目录 `linux/samples/kprobes/kprobe_example.c`

内容涉及

- 编译内核模块
- 安全引导模式下为内核模块签名
- 加载卸载内核模块
- 实时查看系统日志
- 内核模块传递参数

```
$ ls ~/linux-5.15.72/samples/kprobes/
kprobe_example.c  kretprobe_example.c  Makefile
```

# 编译 kenrel module

- 将 `kprobe_example.c` 拷贝到工作目录下

- 创建 Makefile 文件添加如下内容

```
obj-m := kprobe_example.o 

clean:
        rm -f *.mod.c *.ko *.o
```

- 编译内核模块

```
$ make -C /lib/modules/`uname -r`/build M=$PWD
$ ls
kprobe_example.c  kprobe_example.ko  kprobe_example.mod  kprobe_example.mod.c  kprobe_example.mod.o
kprobe_example.o  Makefile  modules.order  Module.symvers
```

- 实时监控 dmesg 信息

```
sudo dmesg -wH
```

- 加载内核模块，等待几秒钟后卸载该模块

```
$ sudo insmod kprobe_example.ko
$
$ sudo rmmod kprobe_example
```

- 这时查看实时监控 dmesg 的终端会有类似以下信息

```
[  +0.018228] kprobe_init: Planted kprobe at 000000002a322dce
[  +0.000348] handler_pre: <kernel_clone> p->addr = 0x000000002a322dce, ip = ffffffffa8cbcde1, flags = 0x206
[  +0.000009] handler_post: <kernel_clone> p->addr = 0x000000002a322dce, flags = 0x206
[  +8.214180] handler_pre: <kernel_clone> p->addr = 0x000000002a322dce, ip = ffffffffa8cbcde1, flags = 0x206
[  +0.000013] handler_post: <kernel_clone> p->addr = 0x000000002a322dce, flags = 0x206
[  +0.006498] handler_pre: <kernel_clone> p->addr = 0x000000002a322dce, ip = ffffffffa8cbcde1, flags = 0x206
[  +0.000003] handler_post: <kernel_clone> p->addr = 0x000000002a322dce, flags = 0x206
[  +0.000233] handler_pre: <kernel_clone> p->addr = 0x000000002a322dce, ip = ffffffffa8cbcde1, flags = 0x206
[  +0.000002] handler_post: <kernel_clone> p->addr = 0x000000002a322dce, flags = 0x206
[  +0.024199] kprobe_exit: kprobe at 000000002a322dce unregistered
```

- 需要注意的是，在系统默认开启的安全引导模式（Secure Boot）下，没有签名的模块是无法安装或加载的

- 一种方式是开机时进入 BIOS , 关闭 `secure boot` 选项。一般不推荐这种方式

- 另一种方式就是给内核模块签名，方法如下

# 为内核模块签名

-  创建 openssl.cnf 配置文件，编辑内容如下，其中 commonName , emailAddress 自己修改下

```
# This definition stops the following lines choking if HOME isn't
# defined.
HOME                    = .
RANDFILE                = $ENV::HOME/.rnd 
[ req ]
distinguished_name      = req_distinguished_name
x509_extensions         = v3
string_mask             = utf8only
prompt                  = no

[ req_distinguished_name ]
countryName             = CA
stateOrProvinceName     = Quebec
localityName            = Montreal
0.organizationName      = cyphermox
commonName              = Secure Boot Signing mz
emailAddress            = mingzheng.xing@gmail.com

[ v3 ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always,issuer
basicConstraints        = critical,CA:FALSE
extendedKeyUsage        = codeSigning,1.3.6.1.4.1.311.10.3.6,1.3.6.1.4.1.2312.16.1.2
nsComment               = "OpenSSL Generated Certificate"
```

- 创建 private key 和 public key

```
$ openssl req -config ./openssl.cnf \
        -new -x509 -newkey rsa:2048 \
        -nodes -days 36500 -outform DER \
        -keyout "MOK.priv" \
        -out "MOK.der"
```

- 注册 keys ，此处会提示输入密码，该密码似乎是临时的，只需记住几分钟

```
sudo mokutil --import MOK.der
```

- 完成注册后，重启计算机。再进入系统引导前会自动进入 MokManager 蓝屏界面

- 根据菜单提示选择 `Enroll MOK`, 然后根据后续菜单提示完成注册。期间会提示输入几分钟前设置的临时密码

- 再次重启后可以检查下是否注册成功，可通过 commonName 信息确认是否存在我们注册的 key

```
$ sudo cat /proc/keys | grep "Secure Boot Signing mz"
```

- 在 secure boot 模式下，没有签名的内核模块是无法安装或加载的，有类似以下错误提示。dmesg 也会有错误信息

```
$ sudo insmod kprobe_example.ko 
insmod: ERROR: could not insert module kprobe_example.ko: Operation not permitted
$ sudo dmesg
[...]
[ +30.876098] Lockdown: insmod: unsigned module loading is restricted; see man kernel_lockdown.7
```

- 现在为该模块签名，命令如下

```
$ kmodsign sha512 MOK.priv MOK.der kprobe_example.ko
```

- 现在就可以加载该模块了

- 可以通过检查模块是否包含字符串 `~Module signature appended~` 来验证模块是否已签名

```
$ hexdump -Cv kprobe_example.ko | tail -n 5
00033b60  97 9c 85 87 32 cb 2b 02  a3 16 06 a3 2d 31 68 37  |....2.+.....-1h7|
00033b70  2d da 23 33 00 00 02 00  00 00 00 00 00 00 02 0c  |-.#3............|
00033b80  7e 4d 6f 64 75 6c 65 20  73 69 67 6e 61 74 75 72  |~Module signatur|
00033b90  65 20 61 70 70 65 6e 64  65 64 7e 0a              |e appended~.|
00033b9c
```

# 给内核模块传递参数
todo 
