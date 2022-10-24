# 前言

记录一下跑 kprobe 例子的过程，内容涉及

- 编译内核模块
- 安全引导模式下为内核模块签名
- 加载卸载内核模块
- 实时查看系统日志
- 内核模块传递参数

源代码位于内核源码目录 `linux/samples/kprobes/kprobe_example.c`

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

- 一种方式是开机时进入 BIOS , 关闭 `secure boot` 选项，一般不推荐这种方式

- 另一种方式就是给内核模块签名，方法如下

# 为内核模块签名

- 创建 openssl.cnf 配置文件，编辑内容如下，其中 commonName , emailAddress 可自行修改下

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

阅读 `kprobe_example.c` 代码，有一行负责传递参数的语句，学习了下相关用法，记录以下

```
module_param_string(symbol, symbol, sizeof(symbol), 0644);
```

- 往内核模块里传递参数，与编写应用时通过终端给 main 函数传递参数类似，不过需要按照以下格式

- 传递整型和字符指针类型，需要用 `module_param` 声明要传递的参数名称，类型和权限

```
module_param(变量的名称，类型，权限);
```

如

```
int intval ;
module_param(intval, int , 0664);
MODULE_PARM_DESC(intval, "This is a integer.");  // 用于添加参数描述

char *p = NULL;
module_param(p, charp, 0664);  // 注意参数类型设定为 charp
MODULE_PARM_DESC(p, "This is a char * string.");
```

- 传递数组：module_param_array(数组名, 元素类型, 元素个数（取地址）, 权限);

```
int array[3] = {};
int num = 3;
module_param_array(array, int, &num, 0664);
```

- 传递字符串：module_param_string(传递参数时的字符串名称, 字符串名称, 字符串大小, 权限);

- 传递字符串时注意前两个参数容易混淆，第二个是在程序中的名称，第一个是传递参数时的名称，一般可以取一样的名字。

```
char str[12] = {};
module_param_string(estr, str, sizeof(str), 0664);
```

- 仿照 kprobe_example.c 自己写一个示例程序，用来试验传递参数

- `mod_test.c`

```
// SPDX-License-Identifier: GPL-2.0-only
/*
  This is my first kernel module test with some parameters passed.
 */

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>

int intval = 0;
module_param(intval, int, 0664);
MODULE_PARM_DESC(intval, "This is a integer.");

char *p = NULL;
module_param(p, charp, 0664);
MODULE_PARM_DESC(p, "This is a char * string.");

int array[3] = {};
int num = 3;
module_param_array(array, int, &num, 0644);
MODULE_PARM_DESC(array, "This is a array.");

char str[64] = "kernel_clone";
module_param_string(estr, str, sizeof(str), 0644);
MODULE_PARM_DESC(estr, "This is a char[] array.");

static struct kprobe kp = {
  .symbol_name = str,
};

static int __kprobes handler_pre(struct kprobe *p, struct pt_regs *regs) {
  pr_info("<%s> p->addr = 0x%p, ip = %lx, flags = 0x%lx\n",
           p->symbol_name, p->addr, regs->ip, regs->flags);

  return 0;
}

static void __kprobes handler_post(struct kprobe *p, struct pt_regs *regs, unsigned long flags) {
  pr_info("<%s> handler_post", p->symbol_name);
}

static int __init demo_init(void) {
  int i = 0;
  int ret;

  printk("%s, %d\n", __func__, __LINE__);
  printk("intval: %d\n", intval);
  printk("p: %s\n", p);
  printk("str: %s\n", str);

  for(i = 0; i < num; i++) {
    printk("array[%d] = %d\n", i, array[i]);
  }

  kp.symbol_name = str;
  kp.pre_handler = handler_pre;
  kp.post_handler = handler_post;

  ret = register_kprobe(&kp);
  if (ret < 0) {
    pr_err("register error, return %d\n", ret);
    return ret;
  }
  pr_info("Planted kprobe at %p\n", kp.addr);
  return 0;
}

static void __exit demo_exit(void) {
  printk("%s, %d\n", __func__, __LINE__);

  unregister_kprobe(&kp);
  pr_info("kprobe at %p unregistered\n", kp.addr);
}

module_init(demo_init);
module_exit(demo_exit);

MODULE_LICENSE("GPL");
```

- 编译模块并签名，加载该模块时传递参数如下：

```
sudo insmod mod_test.ko intval=101 p="pppp" array=1,1,1 estr="kernel_thread"
```

- 在 dmesg 中可以看到参数传递进去了

- 另外加载模块后，会在/sys/modules下生成一个模块的文件夹，里面会有parameters文件夹，包含的以参数名命名的文件节点，里面保存了我们传递的参数值

```
$ cat /sys/module/mod_test/parameters/estr 
kernel_thread
```

# 参考链接

- https://www.kernel.org/doc/Documentation/kbuild/modules.txt

- https://ubuntu.com/blog/how-to-sign-things-for-secure-boot

- https://www.freesion.com/article/3130661935/
