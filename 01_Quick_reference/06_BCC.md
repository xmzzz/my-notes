# USDT

```
$ cat simple-c.c
#include <sys/sdt.h>
#include <sys/time.h>
#include <unistd.h>

int main(int argc, char **argv)
{
    struct timeval tv;

    while(1) {
        gettimeofday(&tv, NULL);
        DTRACE_PROBE1(test-app, test-probe, tv.tv_sec);
        sleep(1);
    }
    return 0;
}
$ sudo tplist.py -vv -l ./simple-c
$ sudo trace.py 'u:/path/to/simple-c:test-probe "%u", arg1' -T -p $(pidof simple-c)
```

# cmake luajit not found

```
$ cmake ..
...
Could NOT find LuaJIT (missing: LUAJIT_LIBRARIES LUAJIT_INCLUDE_DIR)
...

$ sudo apt install libluajit-5.1-dev
```

# BCC install and debug
```
cmake -DCMAKE_BUILD_TYPE=DEBUG ..
make -j4 VERBOSE=1
sudo make install
```

# grep
```
grep -rn keyword /path/
grep -rni keyword /path/
patch -p1 --dry-run < /path/to/patch
patch -p1  < ~/sshfs/01-riscv.patch
grep -wrf keyword .
grep -wrl keyword .
grep -rl keyword .
```

# find
```
find /path/ -name "name"
```

# about file
```
patch -p1 --dry-run < /path/to/patch
patch -p1  < ~/sshfs/01-riscv.patch
dpkg-query -s keyword
lsb_release -a
apt-file search /path/
rm -rf /path/
du -sh /path/
diff -Nur linux-2.6.22.6 linux-2.6.22.6_new > linux-2.6.22.6.patch
tar -xvf vim.tar
tar -xzvf file.tar.gz
tar -xjvf file.tar.bz2
tar -xZvf file.tar.Z
unrar e file.rar
unzip file.zip
```

# python debug
```
python3 -m pdb filename.py
b filename:lineno
tbreak filename:lineno  # tempbreak
cl filename:lineno
p expression
s  # 进入函数体
n  # 不仅函数体
r  # 函数中直接执行返回处
c  # 直到一个断点
j  # 直接跳转到制定行
a  # 在函数中打印函数的参数和参数的值
whatis expression  # 打印表达式的类型
w  # 打印堆栈信息
```

# gdb python
```
make VERBOSE=1
gdb -args python /usr/share/bcc/tools/slabratetop
(gdb) b libbpf.c:480
(gdb) run
(gdb) b bpf_print_hints
(gdb) bt (or where)
(gdb) up
(gdb) info breakpoints
(gdb) del break 1
(gdb) display input[i]
(gdb) watch i
(gdb) set print pretty on
```

# about registers
```
grep pt_regs -rn /usr/src/linux-riscv-headers-5.15.0-1014/
vim /usr/src/linux-riscv-headers-5.15.0-1014/arch/riscv/include/asm/ptrace.h
vim /usr/src/linux-riscv-headers-5.15.0-1014/arch/mips/include/asm/ptrace.h
```
