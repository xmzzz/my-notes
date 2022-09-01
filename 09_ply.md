# register reference
$ cat /usr/src/linux-headers-5.15.0-41/arch/riscv/include/uapi/asm/ptrace.h

# Autoconf macro
AC_CANONICAL_HOST
AS_CASE([$host_cpu],
  [i?86], [ARCHDIR=arch/i386],
  [x86_64], [ARCHDIR=arch/x86_64],
  [AC_MSG_ERROR("No assembler code for CPU $host_cpu")]
)
AC_SUBST([ARCHDIR])

(Note that i?86 is used to match i586, i686 etc.) With this, you can use @ARCHDIR@ in your Makefile.am

	from: https://stackoverflow.com/questions/16713205/how-to-build-arch-machine-dependent-code-with-autotools

# ply: error while loading shared libraries: libply.so.0
$ sudo ply 'kprobe:do_sys_open { printf("%v(%v): %s\n", comm, uid, str(arg1)); }'
ply: error while loading shared libraries: libply.so.0: cannot open shared object file: No such file or directory
$ sudo find / -name libply.so.0
/usr/local/lib/libply.so.0
/home/xmz/work/ply/src/libply/.libs/libply.so.0
~/work/ply$ echo $LD_LIBRARY_PATH

~/work/ply$ LD_LIBRARY_PATH=/usr/local/lib
~/work/ply$ echo $LD_LIBRARY_PATH
/usr/local/lib
~/work/ply$ export LD_LIBRARY_PATH
~/work/ply$ ply 'kprobe:do_sys_open { printf("%v(%v): %s\n", comm, uid, str(arg1)); }'
info: creating kallsyms cache
warning: unable to create kallsyms cache: Operation not permitted
warning: could not remove memlock size restriction
warning: total map size is limited to 2022340kB
error: unable to create map 'stdbuf', errno:1
ERR:-1
~/work/ply$ sudo !!
sudo ply 'kprobe:do_sys_open { printf("%v(%v): %s\n", comm, uid, str(arg1)); }'
ply: error while loading shared libraries: libply.so.0: cannot open shared object file: No such file or directory
~/work/ply$ sudo vim /etc/ld.so.conf
~/work/ply$ sudo ldconfig
~/work/ply$ sudo ply 'kprobe:do_sys_open { printf("%v(%v): %s\n", comm, uid, str(arg1)); }'
info: creating kallsyms cache

# 关于模块的打印信息
sudo dmesg -c     // 清理内容，在插入模块，方便查看
sudo dmesg

