文章标题： 怎么从BCC的ebpf代码转变到libbpf

- 作 者： 明 政  

- 邮 箱： xingmingzheng@iscas.ac.cn

# 背景

最近在做BCC到libbpf的代码转换相关工作，以下内容是阅读 [BCC to libbpf conversion guide - Andrii Nakryiko](https://nakryiko.com/posts/bcc-to-libbpf-howto-guide/) 形成的个人笔记，注意原文写自2020年，有些细节会与现在的实现不一致。

运行BCC程序依赖会Clang/LLVM编译器，相当于在运行时编译BPF程序。采用这种方式是早前在应对不断变化的内核环境中开发可维护的BPF程序唯一好的办法。

现在libbpf的能力在跟踪方面已和BCC不相上下，另外还有BCC没有的新特性，比如全局变量和BPF skeleton。

BCC有自己的用法来方便用户开发，但是这种方便也让一些开发错误难以被发现，比如：
- 必须记住tracepoint的命名约定和自动生成的struct；
- 必须通过rewriter来读kernel数据和获取kprobe参数；
- 在使用map时需要写半面向对象的程序，这跟内核内部实际发生的不一致;
- BCC仍需要在用户态写不少模板类的代码，手工处理一些繁琐的事情

BCC依赖内嵌Clang编译器的运行时编译，依赖庞大的LLVM/Clang库，这会产生一些问题：
1. 编译运行时负载比较高
2. 依赖内核头文件，需要在部署的机器上安装。另外，如果用到内核头文件没有暴露出的数据，需要手动复制定义
3. 任何小错误都需要在编译运行时发现，这会降低开发迭代周期

libbpf会有以下特点：
- libbpf 将自己定位为与用户态程序相似，通过一次性编译为二进制文件，通过一种不可修改的压缩格式去部署到目标机器。
- libbpf 作为加载器，完成枯燥的重定位，加载、验证BPF程序，创建map，附加到BPF hooks等。
- libpf 加载器的负载很低，减轻了复杂的依赖，便于开发。

libbpf的API和代码习惯是尽量没有任何隐式的include header和代码rewrite，而是纯C代码和一些用于简化代码的herlper macro。
另外，我们用libbpf写的代码执行的内容和结构体数据的访问都是与内核内的最终执行是一一对应的。

使用 libbpf 写 bpf 的大致过程：
1. 使用bpftool生成vmlinux.h文件，该文件包含所有kernel里的数据类型信息
2. 用clang（10版本以上）来编译bpf程序，生成object文件(.o)文件
3. 针对object文件生成BPF skeleton头文件
4. 编写用户态程序，在用户态程序里include BPF skeleton文件
5. 编译用户态程序，生成可执行程序。程序包含了BPF程序的二进制。
每一步的实现细节与具体设置和构建系统相关。

BPF 使用 Locked memory 在内核中储存map等数据。libbpf默认设置的非常小，不调整的话几乎不能加载简单的BPF程序。BCC默认设置为无限制。
调整方法具体在不同的生产环境中各不同，一般可以用系统调用setrlimit(2)，并且放在在程序最开始

```
#include <sys/resource.h>

rlimit rlim = {
  .rlim_cur = 512UL << 20, /* 512 MBs */
  .rlim_max = 512UL << 20, /* 512 MBs */
};

err = setrlimit(RLIMIT_MEMLOCK, $rlim);
if (err)
  /* handle error */
```

# libbpf log

log信息对调试非常有帮助，输出的log信息是分等级的，默认是error级别。
建议实现自定义的log回调函数，以用来输出verbose级别的log

```
int print_libbpf_log(enum libbpf_print_level lvl, const char *fmt, va_list args)
{
  if (!FLAGS_bpf_libbpf_debug && lvl >= LIBBPF_DEBUG)
    return 0;
  return vfprintf(stderr, fmt, args);
}

/* ...*/

libbpf_set_print(print_libbpf_log); /* set custom log handler */
```

# BPF skeleton and BPF app lifecycle

使用 libbpf skeleton 和libbpf API 的细节可以参考kernel selftest 和 BCC libbpf-tools。
BPF应用可以包含很多BPF程序，这些BPF程序之间可以是独立的，也可以是相互有联系的。
同时可以有BPF maps和global variable，他们可以在各个BPF程序之间共享，也可以被用户态程序访问。用户态程序可认为是控制程序(control app)。控制程序可以获取和设置这些map、全局变量。

每个BPF程序都需要经过一系列的步骤：
1. Open 阶段
该阶段时会解析BPF object，会发现BPF程序中的map、程序、全局变量等，但没有真正去创建。
open过程之后，在所有这些实体创建之前，我们是可以根据需要再对其进行调整的，比如设置BPF程序类型，或者给全局变量设置初始化值等。
2. Load 阶段
该阶段会创建各种实体（比如map），重定位计算？（various relocation）解决，BPF程序被加载进内核，并进行了验证。该阶段后，BPF程序会在内核里真实存在并且是可用状态，但还没有被运行。该阶段之后，在没有执行BPF程序之前，也可以对程序进行一些全局变量设置、map初始化等操作。
3. Attachment 阶段
该阶段BPF程序被挂载到不同的hook点上，比如tracepoints, kprobes, cgroup hooks, network packet processing pipeline等，这个过程BPF程序真正开始执行并发挥作用，开始更新map和全局变量。
4. Tear down 阶段
该阶段会将内核里的BPF程序解除挂载点并卸载。创建的map会被销毁，所有BPF程序用的资源都会被释放。

生成的 BPF skeleton 有相应阶段的触发函数
- `<name>__open()` - 创建和打开 BPF 应用程序
- `<name>__load()` - 实例化，加载，验证BPF程序
- `<name>__attach()` - 挂载所有 auto-attachable BPF程序，也可以通过libbpf更多的API来更多的控制
- `<name>__destroy()` - 解除挂载所有的BPF程序，释放所有的资源

# BPF code conversion

## Detecting BCC vs libbpf modes

如果想同时有BCC和libbpf模式，在BCC中可用 `BCC_SEC` macro 来决定编译为哪种代码。

```
#ifdef BCC_SEC
#define __BCC__
#endif
```

之后可以在代码中区分编译不同的模式：

```
#ifdef __BCC__
/* BCC-specific code */
#else
/* libbpf-specific code */
#endif
```

这样就可以使BPF程序包含大部分公用代码，同时保留少量BCC和libbpf特有的代码

## Header includes

libbpf/BPF CO-RE 不依赖于内核头文件（例如所有的 `#include <linux/whatever.h>` 文件，而是简单的一个 `vmlinux.h` 和少数的 libbpf helper headers。

```
#ifdef __BCC__
/* linux headers needed for BCC only */
#else /* __BCC__ */
#include "vmlinux.h"		/* all kernel types */
#include <bpf/bpf_helpers.h>	/* most used helpers:SEC, __always_inline, etc */
#include <bpf/bpf_core_read.h>	/* for BPF CO-RE helpers */
#include <bpf/bpf_tracing.h>	/* for getting kprobe arguments */
```

`vmlinux.h` 可能不包含内核里一些有用的macro 常量，这种情况需要在本地重新定义。大部分常用的常量定义会在 `bpf_helpers.h` 中提供。

## Field accesses

BCC 有一个 rewriter，用来重写BPF代码，会将 `tsk->parent->pid` 格式的访问转换为一系列的 `bpf_probe_read()`。
libbpf `bpf_core_read.h` 中提供了一组类C语言的helper，如将 `tsk->parent->pid` 将变为 `BPF_CORE_READ(tsk, parent, pid)`。
自从linux 5.5之后，支持了 tp_btf 和 fentry/fexit BPF程序类型，使用原始C语言的格式访问数据也是有可能的。但是对于老的kernel或者其他BPF program types(如tracepoint，kprobe)，最好还是转换为 `BPF_CORE_READ`。

BCC中也支持 `BPF_CORE_READ` 宏，所以为了避免在源码中保留BCC和libbpf两种代码，可以将所有的field访问都转换为 `BPF_CORE_READ`。
另外注意在BCC中也要确定最终的BPF程序包含 `bpf_core_read.h` 头文件。

## BPF maps

BCC和libbpf两者定义map有区别，见代码：

```
/* Array */
#ifdef __BCC__
BPF_ARRAY(my_array_map, struct my_value, 128);
#else
struct {
  __uint(type, BPF_MAP_TYPE_ARRAY);
  __uint(max_entries, 128);
  __type(key, u32);
  __type(value, struct my_value);
} my_array_map SEC(".maps");
#endif

/* Hashmap */
#ifdef __BCC__
BPF_HASH(my_hash_map, u32, struct my_value);
#else
struct {
  __uint(type, BPF_MAP_TYPE_HASH);
  __uint(max_entries, 10240);
  __type(key, u32);
  __type(value, struct my_value);
} my_hash_map SEC(".maps")
#endif

/* Per-CPU array */
#ifdef __BCC__
BPF_PERCPU_ARRAY(heap, struct my_value, 1);
#else
struct {
  __uint(type, BPF_MAP_TYPE_PERCPU_ARRAY);
  __uint(max_entries, 1);
  __type(key, u32);
  __type(value, struct my_value);
} heap SEC(".maps");
#endif
```

注意BCC中的map默认大小通常是10240,在libbpf中需要手动指定这个值。

`PERF_EVENT_ARRAY`,`STACK_TRACE`，还有一些特别的maps，如DEVMAP，CPUMAP等，不支持key/value的BTF格式，所以需要直接指定key_size/value_size。

```
/* Perf_event_array(for use with perf_buffer API) */
#ifdef __BCC__
BPF_PERF_OUTPUT(events);
#else
struct {
  __uint(type, BPF_MAP_TYPE_PERF_EVENT_ARRAY);
  __uint(key_size, sizeof(u32));
  __uint(value_size, sizeof(u32));
} events SEC(".maps");
#endif
```

## Accessing BPF maps from BPF code

BCC访问map的语法与C语言类似，背后通过rewriter转换为实际的BPF helper调用。

```
some_map.operation(some, args)
```

将重写为以下形式：

```
bpf_map_operation_elem(&some_map, some, args);
```

举例：

```
#ifdef __BCC__
  struct event *data = heap.lookup($zero);
#else
  struct event *data = bpf_map_lookup_elem(&heap, &zero);
#endif

#ifdef __BCC__
  my_hash_map.update(&id, my_val);
#else
  bpf_map_update_elem(&my_hash_map, &id, &my_val, 0/*flag*/);
#endif

#ifdef __BCC__
  events.perf_submit(args, data, data_len);
#else
  bpf_perf_event_output(args, &events, BPF_F_CURRENT_CPU, data, data_len);
#endif
```

## BPF programs

所有BPF程序的函数必须通过 `SEC()` 宏和自定义section name来标记，这些来自 `bpf_helpers.h`，例

```
#if !defined(__BCC__)
SEC("tracepoint/sched/sched_process_exec")
#endif
int tracepoint__sched__sched_process_exec(
#ifdef __BCC__
  struct tracepoint__sched__sched_process_exec *args
#else
  struct trace_event_raw_sched_process_exec *args
#endif
) {
/* ... */
}
```

这只是一个惯例，但如果遵循libbpf的节命名，总体上会获得更好的体验。预期命名的详细列表在 `libbpf/src/libbpf.c` 中。
一些常用的惯例如下

```
tp/<category>/<name> for tracepoints
kprobe/<func_name> for kprobe 和 kretprobe/<func_name> for kretprobe
raw_tp/<name> for raw tracepoint
cgroup_skb/ingress, cgroup_skb/egress, 以及全家族cgroup/<subtype>
```

## Tracepoints

上小节的例子中可以看出BCC和libbpf两者在tracepoint上下文中struct数据类型名命名有微小的区别。
BCC的格式是 `tracepoint__<categoory>__<name>`，并在编译阶段自动生成相对应的类型名。
libbpf没用这个复杂特性，而是用的内核提供的tracepoint 类型名，命名惯例通常为 `trace_event_raw_<name>`。但有一些例外，如果惯例中的命名不工作，那就需要检查内核源码，或者检查 `vmlinux.h` 文件，以找到正确的名字。
有个例子是，我们需要使用 `trace_event_raw_sched_process_template` 而不是 `trace_event_raw_sched_process_exit`

另外需要注意的是，在大部分情况下，访问tracepoint上下文数据情况是相似的，有一些特殊的 variable-length string fields 有所不同。
转换方式为 `data_loc_<some_field>` 变为 `__data_loc_<some_field>`。

## Kprobes

BCC针对kprobe也有一系列的magic方法。比如BPF程序中接受一个 `struct pt_regs` 类型的指针，作为上下文参数。在BCC中，BPF程序可认为可以直接访问内核中函数的参数值。
在libbpf中，可以利用 `BPF_KPROBE` 来完成类似的工作，这个 macro 暂时存在于 kernel selftest 的 bpf_trace_helper.h 头文件中，不过应该很快会成为 libbpf 的一部分。
示例：

```
#ifdef __BCC__
int kprobe__acct_collect(struct pt_regs *ctx, long exit_code, int group_dead)
#else
SEC("kprobe/acct_collect")
int BPF_KPROBE(kprobe__acct_collect, long exit_code, int group_dead)
#endif
{
  /* BPF code accessing exit_code and group_dead here */
}
```

另外有相对应的 `BPF_KRETPROBE` macro 。

系统调用从4.17版本开始进行了重命名，syscall kprobe 被调用时的名字，如 `sys_kill`，在x64架构上会是 `__x64_sys_kill`，其他架构会有不同的前缀。因此在BPF程序挂载到kprobe/kretprobe中时需要处理。
如果可能的话，最好用tracepoint。

如果你在开发一个使用 tracepoint/kprobe/kretprobe 的 BPF 程序，可以看看新的 raw_tp/fentry/fexit probes，这些提供更好的效率和可用性。是从5.5内核开始支持。

## Dealing with compile-time #if's in BCC

BCC 的 `#ifdef #if` 预处理非常有用。通常是处理不同内核版本或者根据应用程序的配置打开和关闭某些选项。
另外，BCC 允许在用户态使用自定义的 `#define`，然后在运行时编译将其替换。这通常用在自定义不同的参数的情况。

这种运行时编译的方式在libbpf+CO-RE中是没有的，因为libbpf是需要一次性编译好，可以处理可能的不同内核版本和应用程序配置。

在libbpf中，针对内核版本不同，提供了两种互补的机制，`Kconfig externs` 和 `struct "flavors"`

通过声明以下 `extern variable`，BPF代码可以指导现在处理的是哪个内核版本：

```
#define KERNEL_VERSION(a, b, c) (((a) << 16) + ((b) << 8) + (c))

extern int LINUX_KERNEL_VERSION __kconfig;

if (LINUX_KERNEL_VERSION < KERNEL_VERSION(5, 2, 0)) {
  /* deal with older kernels */
} else {
  /* 5.2 or newer */
}
```

以上是获取内核版本，类似的，可以提取 Kconfig 中任何 `CONFIG_XXX` 的值。

```
extern int CONFIG_HZ __kconfig;

/* now you can use CONFIG_HZ in  calculations */
```

通常，如果一个field被重命名或者被移到了子结构体里，可以检查判断在目标内核版本里是否存在该field。检查方法可以是 `bpf_core_field_exists(<field>)`，存在返回1,不存在返回0。

再加上 `struct flavors`，基本可以处理大部分内核struct结构变化的情况。
例子：

```
/* struct kernfs_iattrs will come from vmlinux.h */

struct kernfs_iattrs___old {
  struct iattr ia_iattr;
};

if (bpf_core_field_exists(root_kernfs->iattr->ia_mtime)) {
  data->cgroup_root_mtime = BPF_CORE_READ(root_kernfs, iattr, ia_mtime.tv_nsec);
} else {
  struct kernfs_iattrs___old *root_iattr = (void *)BPF_CORE_READ(root_kernfs, iattr);
  data->cgroup_root_mtime = BPF_CORE_READ(root_iattr, ia_iattr.ia_mtime.tv_nsec);
}
```

## Application configuration

BPF CO-RE 可以通过使用 global 变量自定义应用程序的行为。
全局变量可以在BPF程序加载和验证前，让用户态控制程序预设置必要的参数和flags。
全局变量可以是可变的，也可以是不变的常量。
常量一般用于对BPF程序在加载进内核和验证前进行一次性设置。
变量一般用于在BPF程序加载和运行后，内核态和用户态BPF程序双向的交换数据。
在BPF代码一端，可以用 `const volatile` 声明read-only 全局常量。
声明全局变量的话，去掉 `const volatile` 关键字就可以

```
const volatile struct {
  bool feature_enabled;
  int pid_to_filter;
} my_cfg = {};
```

有三点需要注意的：

- `const volatile` 关键字是必须需要加的，作用是避免编译器优化将其错误的变为inline或者0值。
- 如果定义一个全局变量而不是常量，需要确认没有被标记为 `static`。non-static 全局变量最好与compiler交互，在这种情况下 `volatile`通常是不需要的。
- 变量需要初始化，否则libbpf会拒绝加载。可以初始化为0或者其他需要的值。该值将是默认值，除非被用户态控制程序改写。

在 BPF 代码里使用全局变量也很简单：

```
if (my_cfg.feature_enabled) {
  /* ... */
}

if (my_cfg.pid_to_filter && pid == my_cfg.pid_to_filter) {
  /* ... */
}
```

全局变量有更好的用户体验和高效，没有map检索的开销。常量变量在验证器中是已知的，验证器可以根据常量提前将代码裁减，消除死代码。

在控制程序中操作值的方法：

```
struct <name> *skel = <name>__open();
if (!skel)
  /* handle errors */

skel->rodata->my_cfg.feature_enabled = true;
skel->rodata->my_cfg.pid_to_filter = 123;

if (<name>__load(skel))
  /* handle errors */
```

常量全局变量只能在BPF skeleton load之前进行设置和修改值，一旦BPF程序load之后，不管是BPF一端还是用户态一段都不能再修改它。
这也保证了BPF验证器可以再验证时可以将常量全局变量视为不变的值，从而消除死代码。
可变的全局变量在BPF skeleton load之后，在BPF生命周期内都是可以修改的。可以用于交换配置信息，状态值等。

# Common issues

列举几个可能会遇到的问题，这些问题的出现可能是因为误解或者是BCC与libbpf实现的差异。

## Global variables

BPF 全局变量基本上很像用户态的普通变量，可以用于表达式，非常量变量可以更新值，甚至可以获取变量地址传递到helper函数里。
但是这些只适用于BPF code side，对于用于空间代码，读取或者更新只能通过BPF skeleton 。

```
skel->rodata  只读变量
skel->bss  可变的初始化为0的变量
skel->data  可变的初始化不为0的变量
```

还需要注意，从用户端代码里可以读取和更新全局变量并且可以立即反应到BPF程序一端。
但是他们在用户端并不是全局变量，而只是BPF skeleton 中rodata、bss、data的成员。这些成员在skeleton load阶段被初始化。
这也意味着，在BPF和用户态代码里定义通过同名全局变量并不冲突，他们是独立的，没有任何联系。

## Loop unrolling

除非运行在 5.3 内核以上，低版本内核中的所有循环都需要标记 `#pragma unroll`。目的是强制Clang将循环展开，消除任何可能的控制流回路。

```
#pragma unroll
for (i = 0; i < 10; i++) { ... }
```

如果没有将循环展开，或者循环不能确定在固定循环次数之后停止，该BPF程序就不能通过验证，会报错“back-edge from insn X to Y”。

## Helper sub-programs

如果在4.16之前的内核使用静态函数，需要指定 `static __always_inline`。
这样BPF验证器就会认为是一个简单的大函数。

```
static __always_inline unsigned long
probe_read_lim(void *dst, void *src, unsigned long len, unsigned long max)
{
 ...
}
```

在4.16内核时，开始在BPF程序中支持 BPF-to-BPF 函数调用。
libbpf v0.2+ 版本完全支持该特性，并且可以确保执行了所有正确的代码重新定位和调整。
所以我们可以不用 `__always_inline`。甚至可能需要指定 `__noinline`，这样可以提高代码质量，也可以避免一些由于 register-to-stack 溢出的验证错误。

```
static __noinline unsigned long
probe_read_lim(void *dst, void *src, unsigned long len, unsigned long max)
{
 ...
}
```

（?todo） 从 5.5 内核开始支持了 non-inlined global function，与static function相比具有不同的语义和验证限制，因此要确认对他们进行检查。

## bpf_printk debugging

BPF 程序目前没有好的调试方法和工具，不支持断点，检查变量、map等，不支持单步调试。
打印调试信息的log非常有用。使用 `bpf_printk(fmt, args...)`来打印log信息帮助调试。
该函数接受 `printf`类似的格式字符串，可处理最多3个参数。它的负载比较高，因此在最终产品中慎用。

```
char comm[16];
u64 ts = bpf_ktime_get_ns();
u32 pid = bpf_get_current_pid_tgid();

bpf_get_current_comm(&comm, sizeof(comm));
bpf_printk("ts: %lu, comm: %s, pid: %d\n", ts, comm, pid);
```

打印的log信息可以在下面文件中读取：

```
$ sudo cat /sys/kernel/debug/tracing/trace_pipe
...
    [...] ts: 342697952554659, comm: runqslower, pid: 378
    [...] ts: 342697952587289, comm: kworker/3:0, pid: 320
```

参考资料：
- https://nakryiko.com/posts/bcc-to-libbpf-howto-guide/
