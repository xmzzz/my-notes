文章标题： kexec 和 kdump

- 作 者： 明 政

- 邮 箱： xingmingzheng@iscas.ac.cn

在使用 Linux 系统时，难免会出现内核崩溃的情况，这在 Windows 系统中通常会“蓝屏”，显示一些文本信息，想恢复正常使用就得断电重启了。

Linux 系统在内核崩溃时可以做些什么呢？最近看了下 Linux kernel 里的 kexec 和 kdump 相关的源码，初步了解了下这两个技术之前的关系和大概的用法，记录一下。

# kexec

据说这项技术是为了快速启动新的内核。通过 `man kexec` 可以看到它的用法，如 `$ kexec -l [相关参数]` 加载一个新内核， `$ kexec -e` 就可以从当前内核快速启动并切换到新内核。

通过 kexec 启动新内核跳过了 BIOS 或者 firmware 初始化硬件的过程，所以缩短了启动时间。

# kdump

kdump 离不开 kexec ，kdump 可以让 Linux 系统当前内核（生产内核）崩溃时，根据系统管理员的配置，调用所有注册的崩溃处理函数、在屏幕上打印信息等等。之后快速启动到第二内核（捕获内核）。启动捕获内核的动作就是由 kexec 完成的。

捕获内核以很小的内存启动，由于跳过了 BIOS ，崩溃的生产内核内存会被保留。这样就可以获取内核崩溃时的快照信息，用来查找原因，发现问题。

这大概就是 kdump 内核转储的作用。

# 两种启动新内核的方式

在当前第一内核的上下文中，用 `$ kexec -l [相关参数]` 加载的第二内核，可以被分段存放在当前内存中的不连续位置。在运行命令 `$ kexec -e` 启动第二内核时，之前存储在不同位置的内核内容将会重组并覆盖原第一内核。也就是说，启动后第二内核会替换掉第一内核。所以这样也会丢失第一内核的数据。

而 kdump 加载捕获内核使用的是 `$ kexec -p [相关参数]`，这样加载的内核将被存放在当前内存的一块固定位置，当系统崩溃时会跳转到捕获内核。因此保留了原内核的数据信息。这块固定位置是在第一内核中提前配置好的，具体可参考 kdump 相关文档。

# kdump 触发点

kdump 配置好并且加载捕获内核后，一旦内核崩溃时就会触发 kdump 。具体触发点有 `panic()`, `die()`, `die_nmi()`, `sysrq handler`，它们都会调用 `crash_kexec()` 来启动捕获内核，进而进行故障处理。`crash_kexec()` 会调用 `__crash_kexec()` 。

```
/*
 * No panic_cpu check version of crash_kexec().  This function is called
 * only when panic_cpu holds the current CPU number; this is the only CPU
 * which processes crash_kexec routines.
 */
void __noclone __crash_kexec(struct pt_regs *regs)
{
        /* Take the kexec_lock here to prevent sys_kexec_load
         * running on one cpu from replacing the crash kernel
         * we are using after a panic on a different cpu.
         *
         * If the crash kernel was not located in a fixed area
         * of memory the xchg(&kexec_crash_image) would be
         * sufficient.  But since I reuse the memory...
         */
        if (kexec_trylock()) {
                if (kexec_crash_image) {
                        struct pt_regs fixed_regs;

                        crash_setup_regs(&fixed_regs, regs);
                        crash_save_vmcoreinfo();		// 用于保存 vmcore 信息，比如 crash time 等
                        machine_crash_shutdown(&fixed_regs);	// 保存寄存器数据到 vmcore
                        machine_kexec(kexec_crash_image);	// 启动捕获内核
                }    
                kexec_unlock();
        }
}
STACK_FRAME_NON_STANDARD(__crash_kexec);
```

# 捕获内核获取 dump file

在内核崩溃前，系统内核所有必要的快照数据都会编码为 ELF 格式文件，并存放在提前预留好的内存空间里。这个 ELF 文件的 ELF header 内存地址通过 `elfcorehdr=` 启动参数传递给捕获内核。ELF header 的大小也可以一起传递 `elfcorehdr=[size[KMG]@]offset[KMG]` 。

内核崩溃后，在启动的捕获内核中，将通过 `/proc/vmcore` 输出 ELF 格式的 dump 文件，该文件可以通过 `cp` 或 `scp` 命令保存到特定位置。也可以用 `makedumpfile` 工具来分析或者通过选项来过滤内容，比如可以只保存内核数据。

kdump 会根据配置文件的默认设置，生成 dump 文件到 `/var/crash` 目录下。除此之外还有 dmesg 信息。

有了 dump 文件，就可以用 GDB 和 Crash 工具来 debug 。具体方法可参考相关文档。

# panic(), crash_kexec(), 和 kdump

`crash_kexec()` 主要的功能就是启动捕获内核，启动之前进行了 vmcore 信息保存和保存寄存器数据等动作。

`panic()` 是内核崩溃的触发点之一，该函数里会尽可能的打印错误信息，调用各个模块注册的异常回调函数等。期间会尝试调用 `__crash_kexec()`启动捕获内核。

kdump 机制可以预先加载好捕获内核，在内核崩溃触发之后，启动捕获内核，并决定是否保存 dmesg 等信息、如何生成 vmcore、确定 dumpfile 的默认保存位置等。这些是通过配置文件和脚本控制的。具体可参考 `man kdump-tools` 、 `man kdump-config` 和相关文档。
