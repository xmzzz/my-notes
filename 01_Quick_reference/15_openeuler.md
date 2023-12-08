# PWM

revyOS 中有关PWM和SYSFS开启的选项：
CONFIG_PWM=y
CONFIG_PWM_SYSFS=y
CONFIG_PWM_LIGHT=y
CONFIG_SYSFS=y
CONFIG_SYSFS_SYSCALL=y
CONFIG_GPIO_SYSFS=y
CONFIG_RTC_INTF_SYSFS=y

## 更换设备树

- 将revyos中的`/boot/dtbs/linux-image-5.10.113-lpi4a/thead/light-lpi4a.dtb`文件重命名为 `light-lpi4a-fan.dtb`
- 拷贝至 oE 中的 `/boot/dtbs/thead/` 目录下
- 重启，按任意键停止uboot启动

```
Hit any key to stop autoboot:  0
Light LPI4A# setenv fdtfile thead/light-lpi4a-fan.dtb
Light LPI4A# saveenv
Light LPI4A# boot
```

# ltp 相关

## fail

runtest/syscalls:1196:sched_rr_get_interval01
runtest/syscalls:1201:sched_setparam02 sched_setparam02
runtest/syscalls:1206:sched_getscheduler01
runtest/syscalls:1732:perf_event_open02
runtest/mm:93:vma05 vma05.sh
runtest/sched:6:time-schedule01
runtest/pty:5:pty04
runtest/controllers:24:memcontrol02
runtest/hugetlb:29:hugemmap24
runtest/cve:64:cve-2020-11494

# mugen 相关

## x86 同样失败用例

- oe_test_bnx2fc

用例找的模块名和实际不一致，实际模块名有.xz后缀

On branch openEuler-22.03-LTS-Next

```
commit 54bbe356d7968d8540a331a3b385ad6dfd1e17a6
Author: Liu Yuntao <liuyuntao10@huawei.com>
Date:   Sat Mar 19 15:53:09 2022 +0800

    Compress modules to xz format in kernel.spec, which reduces disk consumption.

diff --git a/kernel.spec b/kernel.spec
index 5033412..6310b79 100644
--- a/kernel.spec
+++ b/kernel.spec
@@ -12,7 +12,7 @@
 %global upstream_sublevel   0
 %global devel_release       52
 %global maintenance_release .0.0
-%global pkg_release         .25
+%global pkg_release         .26
 
 %define with_debuginfo 1
 # Do not recompute the build-id of vmlinux in find-debuginfo.sh
@@ -501,6 +501,7 @@ popd
         chmod 0755 %{modsign_cmd} \
         %{modsign_cmd} $RPM_BUILD_ROOT/lib/modules/%{KernelVer} || exit 1 \
     fi \
+    find $RPM_BUILD_ROOT/lib/modules/ -type f -name '*.ko' | xargs -n1 -P`nproc --all` xz; \
 %{nil}
 
 # deal with header
@@ -861,6 +862,9 @@ fi
 %endif
 
 %changelog
+* Sat Mar 19 2022 Liu Yuntao <windspectator@gmail.com> - 5.10.0-52.0.0.26
+- Compress modules to xz format in kernel.spec, which reduces disk consumption.
+
```

- oe_test_check_huge_task

CONFIG_BOOTPARAM_HUNG_TASK_PANIC_VALUE=0 在5.10版本内核中存在，6.4内核不存在

被以下commit删除，

```
commit 67fca000e1e173fe2c539a127ccf1bc338d5ff37
Author: Rasmus Villemoes <linux@rasmusvillemoes.dk>
Date:   Fri Apr 29 14:38:00 2022 -0700

    lib/Kconfig.debug: remove more CONFIG_..._VALUE indirections
    
    As in "kernel/panic.c: remove CONFIG_PANIC_ON_OOPS_VALUE indirection",
    use the IS_ENABLED() helper rather than having a hidden config option.
    
    Link: https://lkml.kernel.org/r/20220321121301.1389693-1-linux@rasmusvillemoes.dk
    Signed-off-by: Rasmus Villemoes <linux@rasmusvillemoes.dk>
    Cc: Masahiro Yamada <masahiroy@kernel.org>
    Cc: Kees Cook <keescook@chromium.org>
    Signed-off-by: Andrew Morton <akpm@linux-foundation.org>

diff --git a/kernel/hung_task.c b/kernel/hung_task.c
index 52501e5f7655..cff3ae8c818f 100644
--- a/kernel/hung_task.c                                                          
+++ b/kernel/hung_task.c                                                                                                                                             
@@ -73,7 +73,7 @@ static unsigned int __read_mostly sysctl_hung_task_all_cpu_backtrace;
  * hung task is detected:                                                        
  */                                                                              
 unsigned int __read_mostly sysctl_hung_task_panic =                              
-                               CONFIG_BOOTPARAM_HUNG_TASK_PANIC_VALUE;  
+                               IS_ENABLED(CONFIG_BOOTPARAM_HUNG_TASK_PANIC);     
                                                                                  
 static int                                                                       
 hung_task_panic(struct notifier_block *this, unsigned long event, void *ptr)     
diff --git a/kernel/watchdog.c b/kernel/watchdog.c
index 9166220457bc..ecb0e8346e65 100644                                           
--- a/kernel/watchdog.c
+++ b/kernel/watchdog.c
```

目前以下配置
CONFIG_BOOTPARAM_HUNG_TASK_PANIC_VALUE=0
的等效判断应为
CONFIG_BOOTPARAM_HUNG_TASK_PANIC=n

- oe_test_cifs

测试用例问题，cifs ? vport_geneve? 有cifs_arc4 cifs_md4 干扰？

```
commit 38c8a9a52082579090e34c033d439ed2cd1a462d                                   
Author: Steve French <stfrench@microsoft.com>                                     
Date:   Sun May 21 20:46:30 2023 -0500                                            
                                                                                  
    smb: move client and server files to common directory fs/smb                                                                                                     
                                                                                  
    Move CIFS/SMB3 related client and server files (cifs.ko and ksmbd.ko          
    and helper modules) to new fs/smb subdirectory:                               
                                                                                  
       fs/cifs --> fs/smb/client                                                  
       fs/ksmbd --> fs/smb/server                                                 
       fs/smbfs_common --> fs/smb/common                                          
                                                                                  
    Suggested-by: Linus Torvalds <torvalds@linux-foundation.org>                  
    Acked-by: Namjae Jeon <linkinjeon@kernel.org>                                 
    Signed-off-by: Steve French <stfrench@microsoft.com> 
```

以上 commit 新增了内核选项

```
+obj-$(CONFIG_SMBFS) += cifs_arc4.o
+obj-$(CONFIG_SMBFS) += cifs_md4.o
```

新增了cifs依赖的两个内核模块cifs_md4和cifs_arc4，并且这两个模块已在系统启动时默认加载

```
# modinfo cifs | grep cifs
filename:       /lib/modules/6.4.0-7.0.0.15.oe2309.riscv64/kernel/fs/smb/client/cifs.ko.xz
alias:          fs-cifs
depends:        cifs_md4,cifs_arc4
```

以上原因导致用例执行失败，因此需要修改用例。增加grep的-w参数已排除cifs_md4和cifs_arc4的干扰。


- oe_test_cpu_rand

测试用例需要更新，随机数生成的检测方式有变化

```
commit 9592eef7c16ec5fb9f36c4d9abe8eeffc2e1d2f3
Author: Jason A. Donenfeld <Jason@zx2c4.com>
Date:   Tue Jul 5 20:48:41 2022 +0200

    random: remove CONFIG_ARCH_RANDOM
    
    When RDRAND was introduced, there was much discussion on whether it
    should be trusted and how the kernel should handle that. Initially, two
    mechanisms cropped up, CONFIG_ARCH_RANDOM, a compile time switch, and
    "nordrand", a boot-time switch.
    
    Later the thinking evolved. With a properly designed RNG, using RDRAND
    values alone won't harm anything, even if the outputs are malicious.
    Rather, the issue is whether those values are being *trusted* to be good
    or not. And so a new set of options were introduced as the real
    ones that people use -- CONFIG_RANDOM_TRUST_CPU and "random.trust_cpu".
    With these options, RDRAND is used, but it's not always credited. So in
    the worst case, it does nothing, and in the best case, maybe it helps.
    
    Along the way, CONFIG_ARCH_RANDOM's meaning got sort of pulled into the
    center and became something certain platforms force-select.
    
    The old options don't really help with much, and it's a bit odd to have
    special handling for these instructions when the kernel can deal fine
    with the existence or untrusted existence or broken existence or
    non-existence of that CPU capability.
    
    Simplify the situation by removing CONFIG_ARCH_RANDOM and using the
    ordinary asm-generic fallback pattern instead, keeping the two options
    that are actually used. For now it leaves "nordrand" for now, as the
    removal of that will take a different route.
    
    Acked-by: Michael Ellerman <mpe@ellerman.id.au>
    Acked-by: Catalin Marinas <catalin.marinas@arm.com>
    Acked-by: Borislav Petkov <bp@suse.de>
    Acked-by: Heiko Carstens <hca@linux.ibm.com>
    Acked-by: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
    Signed-off-by: Jason A. Donenfeld <Jason@zx2c4.com>
```

以上commit 删除了 CONFIG_ARCH_RANDOM，因此以内核选项ARCH_RANDOM=y的判断方法已不适用。

在RISC-V架构下，lscpu命令通常不会显示Flags，这是因为RISC-V架构与其他架构的处理器相比，不支持像x86架构那样具有一组标志寄存器（Flags）的概念。Flags是一组位于处理器状态寄存器中的特殊标志位，用于在算术和逻辑运算中跟踪运算结果的特性。

在RISC-V架构中，处理器的设计通常更加简洁，不包含像Flags这样的标志寄存器。相反，RISC-V使用条件分支来处理条件判断和跳转操作，以避免增加额外的标志寄存器。因此，lscpu命令在RISC-V架构下不显示Flags信息，这是正常的行为。

在RISC-V架构下，lscpu命令通常会显示有关处理器型号、架构、主频、缓存等基本信息，但不包括Flags。

考虑到架构兼容性，只保留 `grep -w rng /proc/crypto` 为CPU是否具有随机数生成功能的判别方法

- oe_test_fnic

CONFIG_FCOE_FNIC=n
Cisco FNIC Driver不支持riscv架构

- oe_test_hifc

Huawei hifc Fibre Channel驱动不支持riscv架构，并且6.4内核尚未添加该驱动。

- oe_test_hinic

Huawei hinic 驱动支持x86 arm64架构，不支持riscv架构

- oe_test_hpsa

.xz问题

- oe_test_io_sched

内核没有CONFIG_DEFAULT_IOSCHED选项，因此删除该检测条件

riscv qemu环境中的scheduler文件位置有区别，修改为/sys/block/vda/queue/scheduler后该用例通过。

- oe_test_kernel_cmd_01

cpupower不支持
gpio-event-mon
gpio-hammer 支持

x86 openeuler_defconfig
```
#
# CPU frequency scaling drivers
#
CONFIG_X86_INTEL_PSTATE=y
CONFIG_X86_PCC_CPUFREQ=m
# CONFIG_X86_AMD_PSTATE is not set
# CONFIG_X86_AMD_PSTATE_UT is not set
CONFIG_X86_ACPI_CPUFREQ=m
CONFIG_X86_ACPI_CPUFREQ_CPB=y
CONFIG_X86_POWERNOW_K8=m
CONFIG_X86_AMD_FREQ_SENSITIVITY=m
# CONFIG_X86_SPEEDSTEP_CENTRINO is not set
CONFIG_X86_P4_CLOCKMOD=m
```

arm64 openeuler_defconfig

```
#
# CPU frequency scaling drivers
#
# CONFIG_CPUFREQ_DT is not set
CONFIG_ACPI_CPPC_CPUFREQ=y
CONFIG_ACPI_CPPC_CPUFREQ_FIE=y
CONFIG_ARM_SCPI_CPUFREQ=m
# CONFIG_ARM_QCOM_CPUFREQ_HW is not set
# end of CPU Frequency scaling
# end of CPU Power Management
```

riscv 仅有一个驱动，需要测试验证

```
config CPUFREQ_DT
        tristate "Generic DT based cpufreq driver"
```

- oe_test_libfc

- oe_test_swap_compress

- oe_test_tcm_loop

.xz

- oe_test_wangxun

.xz
依赖libwx模块，需要先加载libwx模块

## 仅在riscv失败用例

- oe_test_lpfc

CONFIG_CPU_FREQ=m
CONFIG_SCSI_LPFC=m

- oe_test_snd_aloop

CONFIG_SND_ALOOP=m

- oe_test_service_cpupower 

arm64:

572:CONFIG_CPU_FREQ=y
573:CONFIG_CPU_FREQ_GOV_ATTR_SET=y
574:CONFIG_CPU_FREQ_GOV_COMMON=y
575:CONFIG_CPU_FREQ_STAT=y
576:CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE=y
577:# CONFIG_CPU_FREQ_DEFAULT_GOV_POWERSAVE is not set
578:# CONFIG_CPU_FREQ_DEFAULT_GOV_USERSPACE is not set
579:# CONFIG_CPU_FREQ_DEFAULT_GOV_ONDEMAND is not set
580:# CONFIG_CPU_FREQ_DEFAULT_GOV_CONSERVATIVE is not set
581:# CONFIG_CPU_FREQ_DEFAULT_GOV_SCHEDUTIL is not set
582:CONFIG_CPU_FREQ_GOV_PERFORMANCE=y
583:CONFIG_CPU_FREQ_GOV_POWERSAVE=y
584:CONFIG_CPU_FREQ_GOV_USERSPACE=y
585:CONFIG_CPU_FREQ_GOV_ONDEMAND=y
586:CONFIG_CPU_FREQ_GOV_CONSERVATIVE=y
587:CONFIG_CPU_FREQ_GOV_SCHEDUTIL=y
4143:CONFIG_CPU_FREQ_THERMAL=y

riscv:

- oe_test_vport-geneve

https://groups.google.com/g/linux.debian.bugs.dist/c/XOnI5PaROnc
https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?id=3de4d22cc9ac7c9f38e10edcf54f9a8891a9c2aa

- oe_test_kernel_cmd_03

[root@openeuler-riscv64 ~]# ll /usr/lib64/perl5/CORE/libperl.so
lrwxrwxrwx 1 root root 23 Jul 31 08:00 /usr/lib64/perl5/CORE/libperl.so -> ../../libperl.so.5.38.0
[root@openeuler-riscv64 ~]# readelf -d /usr/bin/perf | grep perl
 0x0000000000000001 (NEEDED)             Shared library: [libperl.so.5.34]
[root@openeuler-riscv64 ~]# perf
perf: error while loading shared libraries: libperl.so.5.34: cannot open shared object file: No such file or directory

## kernel 构建报错记录

```
[52757s] Bytecompiling .py files below /home/abuild/rpmbuild/BUILDROOT/kernel-6.4.0-7.0.0.15.oe2309.riscv64/usr/lib64/python3.11 using /usr/bin/python3.11
[52759s] + /usr/lib/rpm/brp-python-hardlink                                                                                                                          
[52771s] + '[' 1 -eq 1 ']'                                                                                                                                           
[52771s] + cp certs/signing_key.pem .                                                                                                                                
[52771s] + cp certs/signing_key.x509 .                                                                                                                               
[52771s] + chmod 0755 /home/abuild/rpmbuild/SOURCES/sign-modules                                                                                                     
[52771s] + /home/abuild/rpmbuild/SOURCES/sign-modules /home/abuild/rpmbuild/BUILDROOT/kernel-6.4.0-7.0.0.15.oe2309.riscv64/lib/modules/6.4.0-7.0.0.15.oe2309.riscv64 
[52772s] *** Modules are unsigned! ***                                                                                                                               
[52772s] + exit 1                                                                                                                                                    
[52772s] error: Bad exit status from /var/tmp/rpm-tmp.yWZdru (%install)                                                                                              
...
The buildroot was: /var/tmp/build-root/23.08static-riscv64
```

## 测试环境搭建

- 方法一

1. 准备两台oE虚拟机
2. 第一台虚拟机中，下载mugen代码
3. 安装依赖

```
$ cd mugen/
$ bash dep_install.sh
$ bash mugen.sh --help

Usage:
    -c: configuration environment of test framework
    -a: execute all use cases
    -f: designated test suite
    -r: designated test case
    -x: the shell script is executed in debug mode
    -b: do make for test suite if test suite path have makefile or Makefile file
    -s: runing test case at remote NODE1
```

4. 设置环境变量

```
$ bash mugen.sh -c --ip 192.168.1.8 --password xxxxxx --user root --port 22
$ bash mugen.sh -c --ip 192.168.1.12 --password xxxxxx --user root --port 22
$ cat conf/env.json
$ 测试套中的machine num表示需要的物理机个数
```

5. 执行测试用例

```
$ bash mugen.sh -f nftables -r oe_test_nftables_variable_map -x
```

如果成功, 则存在 results/
如果失败, 可以查看 logs/

> http://devops-dev.com/article/443
> https://z572.online/learn-mugen.html

# dnf

```
$ dnf list --all > LOG_list
$ dnf repolist		# 查看源
$ dnf download vim --destdir ~/rpm/
$ dnf deplist		# 所有包
$ dnf deplist vim	# 依赖关系
$ dnf deplist vim --requires	$ 前置条件和配置	
```

# kernel 编译依赖

```
$ sudo dnf install flex bison ncurses-devel openssl-devel
```
