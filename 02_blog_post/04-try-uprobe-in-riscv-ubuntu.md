文章标题： 尝试 RISC-V Ubuntu 下的 uprobe

- 作 者： 明 政

- 邮 箱： xingmingzheng@iscas.ac.cn

# 正文开始

- 若要使用 uprobe 特性，编译内核需要开启 CONFIG_UPROBE_EVENTS=y 

- 准备一个简单的 C 程序 `$ vim test.c`

```
#include <stdio.h>
#include <stdlib.h>

void test_uprobe()
{
  printf("test_uprobe() called\n");
}

int main()
{
  test_uprobe();
  return 0;
}
```

- 编译为可执行文件

```
$ gcc -o test test.c
```

- 找到 test_uprobe() 函数在 ELF 文件中的偏移量，命令如下

```
$ objdump -D -F test | grep test_uprobe
0000000000001149 <test_uprobe> (File Offset: 0x1149):
    1170:	e8 d4 ff ff ff       	call   1149 <test_uprobe> (File Offset: 0x1149)
```

- 记录一下结果中的 `File Offset: 0x1149` 

- 也可以用同样的方法记录以下 main() 函数的偏移量

```
$ objdump -D -F test | grep main
    1078:	48 8d 3d e4 00 00 00 	lea    0xe4(%rip),%rdi        # 1163 <main> (File Offset: 0x1163)
    107f:	ff 15 53 2f 00 00    	call   *0x2f53(%rip)        # 3fd8 <__libc_start_main@GLIBC_2.34> (File Offset: 0x3fd8)
0000000000001163 <main> (File Offset: 0x1163):
```

- 切换到 root ，通过下面命令确认 ftrace 特性是否开启，以及 tracefs 挂载的位置

```
# mount | grep tracefs
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime)
#
# cd /sys/kernel/tracing
```

- 添加 uprobe event

```
# echo 'p /home/user/path/test:0x1149' > uprobe_events
# echo 'p /home/user/work/path/test:0x1163' >> uprobe_events
# echo 1 > events/uprobes/p_test_0x1149/enable
# echo 1 > events/uprobes/p_test_0x1163/enable
# echo 1 > tracing_on
```

- 可在另一终端运行可执行程序 test

```
$ ./test
test_uprobe() called
```

- 接着回到 root 终端

```
# echo 0 > tracing_on
# cat trace
```

- 会看到类似以下结果

```
# tracer: nop
#
# entries-in-buffer/entries-written: 2/2   #P:8
#
#                                _-----=> irqs-off
#                               / _----=> need-resched
#                              | / _---=> hardirq/softirq
#                              || / _--=> preempt-depth
#                              ||| / _-=> migrate-disable
#                              |||| /     delay
#           TASK-PID     CPU#  |||||  TIMESTAMP  FUNCTION
#              | |         |   |||||     |         |
            test-41995   [006] ..... 30126.051044: p_test_0x1163: (0x55d29791a163)
            test-41995   [006] ..... 30126.051056: p_test_0x1149: (0x55d29791a149)
```

# 参考资料

- https://www.kernel.org/doc/html/latest/trace/uprobetracer.html
