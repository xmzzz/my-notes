文章标题： 在 macOS ventura 中构建 riscv-gnu-toolchain

- 作 者： 明 政

- 邮 箱： xingmingzheng@iscas.ac.cn

# 系统版本

`macOS Ventura 13.2 (22D49)`

# mac 磁盘确认或改为大小写敏感

我发现目标环境的 mac 磁盘对文件名不区分大小写，这样遇到需要区分大小写的情况就会出错，改为区分大小写

- 打开磁盘工具，添加宗卷，格式为 APFS（区分大小写）

- 新添加的宗卷会自动挂载到 `/Volumes` 下，新宗卷以原磁盘为“容器”，共享原磁盘空间

# 安装 homebrew，用来安装依赖包

我用的是如下命令

``` 
work@192 ~ % /usr/bin/ruby -e "$(curl -fsSL https://cdn.jsdelivr.net/gh/ineo6/homebrew-install/install)"
```

出现如下提示就安装成功了

```
==> Installation successful!
```

查看下是否是国内的镜像源

```
work@192 ~ % brew --repo
/usr/local/Homebrew
work@192 ~ % cd /usr/local/Homebrew
work@192 Homebrew % git remote -v
origin  https://mirrors.ustc.edu.cn/brew.git (fetch)
origin  https://mirrors.ustc.edu.cn/brew.git (push)
work@192 Homebrew % brew --repo homebrew/core
/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
work@192 Homebrew % cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
work@192 homebrew-core % git remote -v
origin  https://mirrors.ustc.edu.cn/homebrew-core.git (fetch)
origin  https://mirrors.ustc.edu.cn/homebrew-core.git (push)
```

# build riscv-gnu-toolchain

过程参考的 [源码仓](https://github.com/riscv-collab/riscv-gnu-toolchain) 的 README

不过用 `git clone https://github.com/riscv/riscv-gnu-toolchain` 下载源码后编译报错，改用以下命令下载可以了

```
git clone --recursive  https://github.com/riscv/riscv-gnu-toolchain
```

安装依赖

```
$ brew install python3 gawk gnu-sed gmp mpfr libmpc isl zlib expat
$ brew tap discoteq/discoteq
$ brew install flock
```

这里发现上面依赖包 flock 已经在 homebrew-core 中了，可以直接安装，不用添加 discoteq/discoteq 仓

开始构建，有两种构建模式，ELF/Newlib toolchain 和 Linux-ELF/glibc toolchain

先构建第一种，其中 prefix 指定的路径需要有可写权限

```
./configure --prefix=/opt/riscv
make
```

第二种模式

```
./configure --prefix=/opt/riscv
make linux
```

编译完成后会生成如下工具：

```
work@192 opt % ls /opt/riscv/bin
riscv64-unknown-linux-gnu-addr2line	riscv64-unknown-linux-gnu-gcc-nm	riscv64-unknown-linux-gnu-lto-dump
riscv64-unknown-linux-gnu-ar		riscv64-unknown-linux-gnu-gcc-ranlib	riscv64-unknown-linux-gnu-nm
riscv64-unknown-linux-gnu-as		riscv64-unknown-linux-gnu-gcov		riscv64-unknown-linux-gnu-objcopy
riscv64-unknown-linux-gnu-c++		riscv64-unknown-linux-gnu-gcov-dump	riscv64-unknown-linux-gnu-objdump
riscv64-unknown-linux-gnu-c++filt	riscv64-unknown-linux-gnu-gcov-tool	riscv64-unknown-linux-gnu-ranlib
riscv64-unknown-linux-gnu-cpp		riscv64-unknown-linux-gnu-gdb		riscv64-unknown-linux-gnu-readelf
riscv64-unknown-linux-gnu-elfedit	riscv64-unknown-linux-gnu-gdb-add-index	riscv64-unknown-linux-gnu-run
riscv64-unknown-linux-gnu-g++		riscv64-unknown-linux-gnu-gfortran	riscv64-unknown-linux-gnu-size
riscv64-unknown-linux-gnu-gcc		riscv64-unknown-linux-gnu-gprof		riscv64-unknown-linux-gnu-strings
riscv64-unknown-linux-gnu-gcc-12.2.0	riscv64-unknown-linux-gnu-ld		riscv64-unknown-linux-gnu-strip
riscv64-unknown-linux-gnu-gcc-ar	riscv64-unknown-linux-gnu-ld.bfd
riscv64-unknown-elf-addr2line		riscv64-unknown-elf-gcc-nm		riscv64-unknown-elf-nm
riscv64-unknown-elf-ar			riscv64-unknown-elf-gcc-ranlib		riscv64-unknown-elf-objcopy
riscv64-unknown-elf-as			riscv64-unknown-elf-gcov		riscv64-unknown-elf-objdump
riscv64-unknown-elf-c++			riscv64-unknown-elf-gcov-dump		riscv64-unknown-elf-ranlib
riscv64-unknown-elf-c++filt		riscv64-unknown-elf-gcov-tool		riscv64-unknown-elf-readelf
riscv64-unknown-elf-cpp			riscv64-unknown-elf-gdb			riscv64-unknown-elf-run
riscv64-unknown-elf-elfedit		riscv64-unknown-elf-gdb-add-index	riscv64-unknown-elf-size
riscv64-unknown-elf-g++			riscv64-unknown-elf-gprof		riscv64-unknown-elf-strings
riscv64-unknown-elf-gcc			riscv64-unknown-elf-ld			riscv64-unknown-elf-strip
riscv64-unknown-elf-gcc-12.2.0		riscv64-unknown-elf-ld.bfd
riscv64-unknown-elf-gcc-ar		riscv64-unknown-elf-lto-dump
```
安装 qemu

```
% brew install qemu
```

# 构建时遇到的错误和解决方法

- xcrun error

```
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

解决方法

```
work@192 homework % xcode-select --install
xcode-select: note: install requested for command line developer tools
```

- HTTP/2 fatal error

```
HTTP/2 stream 1 was not closed cleanly before end of the underlying stream
```

解决方法

```
% git config --global http.version HTTP/1.1
```

- mac 自带 make 版本低，更新

```
work@192 riscv-gnu-toolchain % brew install make
```

更新后还是报错如下

```
checking for gnumake... gnumake
checking version of gnumake... 3.81, bad
checking for gnumsgfmt... no
checking for gmsgfmt... no
checking for msgfmt... msgfmt
checking version of msgfmt... 0.21.1, ok
...
configure: error:
*** These critical programs are missing or too old: make
*** Check the INSTALL file for required versions.
make: *** [Makefile:363: stamps/build-glibc-linux-rv64imafdc-lp64d] Error 1
```

手动建立软连接解决了

```
work@192 riscv-gnu-toolchain % cd /usr/local/opt/make/libexec/gnubin/
work@192 gnubin % ln -s ../../bin/gmake gnumake
work@192 gnubin % ls
gnumake make
work@192 gnubin % ls -al
total 0
drwxr-xr-x  4 work  admin  128  1 30 20:23 .
drwxr-xr-x  5 work  admin  160 10 31 14:23 ..
lrwxr-xr-x  1 work  admin   15  1 30 20:23 gnumake -> ../../bin/gmake
lrwxr-xr-x  1 work  admin   15 10 31 14:23 make -> ../../bin/gmake
work@192 gnubin % source ~/.zshrc
work@192 gnubin % gnumake -v
GNU Make 4.4
```

- mac 自带 bison 版本太低

```
% brew install bison
```

- ulimit 错误：打开了过多文件

`make linux` 过程中遇到了如下错误

```
extra-module.mk:11: *** Too many open files.  Stop.
```

原因是 mac 默认的打开文件数限制值为 256，不够用，修改下可以解决

```
% ulimit -n
256
% ulimit -n 1024
% ulimit -n
1024
```

可以根据需要加入到 `~/.zshrc` 中

- Error: makeinfo command not found

make 过程出现以下错误，原因是缺 texinfo 包，可以直接 brew install

```
/Volumes/case-sensitive/riscv-gnu-toolchain/binutils/missing: line 81: makeinfo: command not found
WARNING: 'makeinfo' is missing on your system.
         You should only need it if you modified a '.texi' file, or
         any other file indirectly affecting the aspect of the manual.
         You might want to install the Texinfo package:
         <http://www.gnu.org/software/texinfo/>
         The spurious makeinfo call might also be the consequence of
         using a buggy 'make' (AIX, DU, IRIX), in which case you might
         want to install GNU make:
         <http://www.gnu.org/software/make/>
make[4]: *** [doc/bfd.info] Error 127
make[3]: *** [info-recursive] Error 1
make[2]: *** [all-bfd] Error 2
make[1]: *** [all] Error 2
make: *** [stamps/build-binutils-newlib] Error 2
```

安装 texinfo

```
work@192 riscv-gnu-toolchain % brew install texinfo
```

- brew 出现 `not in a git directory Error`

```
fatal: not in a git directory
Error: Command failed with exit 128: git
work@192 homework % brew -v
Homebrew 3.6.20-143-g10845a1
fatal: detected dubious ownership in repository at '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core'
To add an exception for this directory, call:

        git config --global --add safe.directory /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
Homebrew/homebrew-core (no Git repository)
fatal: detected dubious ownership in repository at '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask'
To add an exception for this directory, call:

        git config --global --add safe.directory /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask
Homebrew/homebrew-cask (no Git repository)
```

执行提示命令可解决

```
work@192 homework % git config --global --add safe.directory /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
work@192 homework % git config --global --add safe.directory /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask
```

# 其他记录

## 常用命令

- 查看所有用户和组

```
% dscacheutil -q group
```

- 当前用户

```
% groups # 查看当前用户所属组（用户所属组可能有多）
% groups user_name # 查看指定用户所属组
% id -a user_name # 查看指定用户所属组的详细信息
% whoami # 当前用户的用户名
```

- 用户切换

```
% sudo -i // 切换到 root ，exit 退出
% su - name // 切换用户
```

## mac 终端常用快捷键

```
Ctrl + d        删除一个字符，相当于通常的 Delete 键(命令行若无所有字符，则相当于 exit；处理多行标准输入时也表示eof)
Ctrl + h        退格删除一个字符，相当于通常的 Backspace 键
Ctrl + u        删除光标之前到行首的字符
Ctrl + k        删除光标之前到行尾的字符
Ctrl + c        取消当前行输入的命令，相当于 Ctrl + Break
Ctrl + a        光标移动到行首(Ahead of line)，相当于通常的 Home 键
Ctrl + e        光标移动到行尾(End of line)
Ctrl + f        光标向前(Forward)移动一个字符位置
Ctrl + b        光标往回(Backward)移动一个字符位置
Ctrl + l        清屏，相当于执行 clear 命令
Ctrl + p        调出命令历史中的前一条(Previous)命令，相当于通常的上箭头
Ctrl + n        调出命令历史中的下一条(Next)命令，相当于通常的上箭头
Ctrl + r        显示：号提示，根据用户输入查找相关历史命令(reverse-i-search)
Ctrl + w        删除从光标位置前到当前所处单词(Word)的开头
Ctrl + y        粘贴最后一次被删除的单词
Ctrl + C        中止一个错误的或者发疯的命令

open .          使用 Finder 打开当前目录

Command + K     清屏
Command + T     新建标签
Command + W      关闭当前标签页
Command + S     保存终端输出
Command + D     垂直分隔当前标签页
Command + Shift + D       水平分隔当前标签页
Command + shift +  { or }   向左/向右切换标签
```


## 给终端配置网络代理

1. 如果是 macOS Mojave 及更低版本，默认终端命令行是 bash

- 首先修改用户配置文件：

```
% vim ~/.bash_profile
```

- 添加如下内容

```
alias proxy='export http_proxy=127.0.0.1:1088;export https_proxy=$http_proxy'
alias proxyOff='unset http_proxy;unset https_proxy'
```

- 执行以下命令生效

```
% source ~/.bash_profile
```

2. 从 macOS Catalina 开始使用 zsh 为默认终端 shell

- 修改配置文件：

```
% vim ~/.zshrc
```

- 增加以下内容

```
alias proxy='export all_proxy=socks5://127.0.0.1:1080'
alias unproxy='unset all_proxy'
```

- 使生效

```
% source ~/.zshrc
```

- 检验代理是否生效

```
% curl cip.cc
% curl ip.sb
% proxy
% curl cip.cc
% unproxy
```

- 也可单独配置 git 代理

```
% git config --global http.proxy http://127.0.0.1:1088
% git config --global https.proxy https://127.0.0.1:1088
% git config --global --unset http.proxy
% git config --global --unset https.proxy
```

## brew tap

- mac 编译 riscv-gnu-toolchain 时，官方仓库文档有一步是 `brew tap discoteq/discoteq`

- Tap 是什么，A Git repository of Formulae and/or commands

- Tap 存储路径为 `/usr/local/Homebrew/Library/Taps/`

- 比如 `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core`

- 删除 Tap 的方法 `brew untap [-f|--force]  discoteq/discoteq`

- brew 的这个 [文档](https://docs.brew.sh/Formula-Cookbook#homebrew-terminology)，解释了 Tap cask bottle 等名词和存储位置

- 可以 [手动搜索包](https://formulae.brew.sh/)，也可以用 `brew search` 搜索


# 参考链接

- https://cloud.tencent.com/developer/article/1853162
- https://blog.csdn.net/whereismatrix/article/details/50582919
- https://javasgl.github.io/mac-max-limit/
- https://github.com/crosstool-ng/crosstool-ng/issues/591
