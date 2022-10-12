# BPF的应用场景
它被用于高性能负载平衡、DDoS 缓解（ DDoS mitigation ）和防火墙、内核和用户空间代码的安全检测等。
网络数据跟踪、内核探测、perf性能事件等
BPF的历史发展

# 一句话解释BPF是什么
Berkeley Packet Filter，一个非常有用且可扩展的内核功能，不仅仅用于数据包过滤。

# BPF特点
安全，最为重要的特点。在大多数情况下，添加内核模块会带来很大风险。而 BPF 程序会在程序加载时进行验证，以确保不会发生越界访问等。
高效，BPF 支持将其字节码实时编译（ JIT ）为机器指令集，因此 BPF 程序也很快。

# BPF安全检测都检测什么？

# BPF可以做什么？或者说我们可以写什么样的程序？
enum bpf_prog_type
1. socket-related program types - SOCKET_FILTER, SK_SKB, SOCK_OPS
过滤、重定向socket数据和监听socket事件。该类型程序与 BPF 的起源（cBPF）有关。

# BPF程序加载并运行后，如何收集信息？
三种方式：BPF maps，perf events, bpf_trace_printk.
- BPF 映射非常适合在不同的 BPF 程序和用户空间之间进行通信;
- 更多自定义数据处理需要使用 perf 事件
- 对于debug logging, bpf_trace_printk() 非常有用。

# BPF kernel internals
eBPF 是内核中的一套指令集，该指令集设计与真实架构的指令集相仿，便于JIT实现一对一映射，从而具有很高的效率。
该指令集与经典BPF不同。
eBPF 相对于 cBPF 的改进：
1. 寄存器数量从 2 个变为 10 个
2. 寄存器宽度从 32 位变为 64 位
3. jt/jf 替换为 jt/fall-through
4. 引入 bpf_call 指令和寄存器使用惯例，已实现零开销调用其他内核函数

        -- from: https://www.kernel.org/doc/Documentation/networking/filter.txt

# iproute/iproute-tc packages
iproute/iproute-tc packages allow us to load BPF programs directly via tc (see tc-bpf(8) for more details).

# 从几个角度来理解 BPF 
- BPF program types
- BPF helper functions for those programs
- BPF userspace communication
- BPF program build environment
- BPF bytecodes and verifier
- BPF Packet Transformation

# BPF sample 怎么跑起来

# Make kernel sample/bpf
## How to fix fatal error: sys/capability.h: No such file or directory
When trying to build a samples/bpf from source and ran into this error. What package or library is needed to fix this?

page[1] contains helpful information when looking for missing dependency errors. Here if we search for sys/capability.h we find in the page:
```
error: sys/capability.h: No such file or directory
```
If you encounter the error sys/capability.h: No such file or directory, it is because a required package is not installed on your system.
On Debian or Ubuntu, something like the following should work:
```
sudo apt-get install libcap-dev
```

[1] https://man7.org/tlpi/code/faq.html

