# patsubst 函数

```
$(patsubst 原模式,目标模式,文件列表)
```

在 `$(patsubst %.c,%.o,$(dir))` 中，patsubst 把 `$(dir)` 中的变量符合后缀是 `.c` 的全部替换成 `.o`

在 `$(patsubst pattern,replacement,text)` 中，表示寻找 text 中符合模式 pattern 的字，用 replacement 替换

# What is ?= in Makefile
?= indicates to set the KDIR variable only if it's not set/doesn't have a value.
For example:
```
KDIR ?= "foo"
KDIR ?= "bar"

test:
    echo $(KDIR)
```
Would print "foo"
GNU manual: http://www.gnu.org/software/make/manual/html_node/Setting.html
--from https://stackoverflow.com/questions/24777289/what-is-in-makefile

# gcc: How to ignore standard include paths?
```
gcc -nostdinc -I/custom/include/path/goes/here
```
-nostdinc ignores standard C include directories
-nostdinc++ ignores standard C++ include directories

# -print-file-name=<file>, --print-file-name=<file>, --print-file-name <arg>
Print the full library path of <file>

# -emit-llvm
Use the LLVM representation for assembler and object files

# clang -D 
Preprocessor flags
-D<macro>=<value>, --D<arg>, /D<arg>, -D<arg>, --define-macro <arg>, --define-macro=<arg>
Define <macro> to <value> (or 1 if <value> omitted)

# g++ -Wno-unused-variable
How do you disable the unused variable warnings coming out of gcc in 3rd party code I do not wish to edit?
```
C:\boost_1_52_0/boost/system/error_code.hpp: At global scope:
C:\boost_1_52_0/boost/system/error_code.hpp:214:36: error: 'boost::system::posix_category' defined but not used [-Werror=unused-variable]
```
The -Wno-unused-variable switch usually does the trick. However, that is a very useful warning indeed if you care about these things in your project. It becomes annoying when GCC starts to warn you about things not in your code though.

I would recommend you keeping the warning on, but use -isystem instead of -I for include directories of third-party projects. That flag tells GCC not to warn you about the stuff you have no control over.

For example, instead of -IC:\\boost_1_52_0, say -isystem C:\\boost_1_52_0.
	-- from: https://stackoverflow.com/questions/15053776/how-do-you-disable-the-unused-variable-warnings-coming-out-of-gcc-in-3rd-party-c

# gcc -Wpointer-sign (C and Objective-C only)
Warn for pointer argument passing or assignment with different signedness. This option is only supported for C and Objective-C. It is implied by -Wall and by -Wpedantic, which can be disabled with -Wno-pointer-sign.

	-- from: https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html

# clang -Wno-incompatible-pointer-types
How can I disable -Wcompare-distinct-pointer-types warning in clang?

-Wno-compare-distinct-pointer-types actually does work.
The problem is that if you have any invalid compiler flags elsewhere in the line, then the whole line will be bad, not just the one flag that was bad.

It is also worth noting that the order of the options matters.
For example "-Wno-covered-switch-default -Wcovered-switch-default" will not disable the warning but
"-Wcovered-switch-default -Wno-covered-switch-default" will. 

	-- from: https://stackoverflow.com/questions/34707851/how-can-i-disable-wcompare-distinct-pointer-types-warning-in-clang

# -Wno-gnu-variable-sized-type-not-at-end


# -Wno-address-of-packed-member
Do not warn when the address of packed member of struct or union is taken, which usually results in an unaligned pointer value. This is enabled by default.

# clang -c 
-c, --compile
Only run preprocess, compile, and assemble steps

# clang -emit-llvm
-emit-llvm
Use the LLVM representation for assembler and object files.

# llc -march=<arch>
Specify the architecture for which to generate assembly, overriding the target encoded in the input file.
See the output of llc -help for a list of valid architectures. 
By default this is inferred from the target triple or autodetected to the current architecture.

# llc -filetype=<output file type>
Specify what kind of output llc should generated.
Options are: asm for textual assembly ( '.s'), obj for native object files ('.o') and null for not emitting anything (for performance testing).

Note that not all targets support all options.

	-- from: https://llvm.org/docs/CommandGuide/llc.html
# what the target "all" stands for and what it does.
Make构建包含许多目标, 例如，要构建一个project可能需要：

1. Build file1.o out of file1.c
2. Build file2.o out of file2.c
3. Build file3.o out of file3.c
4. Build executable1 out of file1.o and file3.o
5. Build executable2 out of file2.o

如果使用 makefile 实现了此工作流程，则可以分别制作每个目标。例如:
```
make file1.o
```
在必要情况下，只会构建file1.o文件。

all名字本身不是固定的，这只是一个命名习惯。all 表示如果您调用它，make 将构建完成所需的所有内容。
这通常是一个phony虚拟目标，它不创建任何文件，而仅依赖于其他文件。
对于上面的示例，构建所有必要的东西就是构建可执行文件，其他文件作为依赖项被拉入。 
所以在makefile中：

```
all: executable1 executable2
```

all target 通常是 makefile 中的第一个target，因为如果您只是在命令行中编写 make，而不指定目标，它将构建第一个目标。

all 通常也是一个 .PHONY 目标。

	-- from: https://stackoverflow.com/questions/2514903/what-does-all-stand-for-in-a-makefile

# What does .PHONY mean in a Makefile? 
默认情况下，Makefile 的目标是“file targets”——它们通常由其他文件构建出来。
Make 假设它的目标是一个文件，这使得编写 Makefiles 相对容易：

```
foo: bar
  create_one_from_the_other foo bar
```

但是，有时我们希望 Makefile 运行不代表文件系统中真实文件名的命令，如常见的目标“clean”和“all”。
您的主目录中可能已经有一个名为 clean 的文件。在这种情况下，Make 会困惑。
因为默认情况下，clean的目标将与此文件相关联，并且 Make 只会在文件的依赖项似乎不是最新的情况下运行它。

这些特殊目标被称为假的，我们可以明确告诉 Make 它们与文件无关，例如：

```
.PHONY：clean
clean：
  rm -rf *.o
```

现在 make clean 将按预期运行，即使您确实有一个名为 clean 的文件。

就 Make 而言，.PHONY目标只是一个始终过期的目标，因此无论何时询问 make <phony_target>，它都会运行，与文件系统的状态无关。
一些常见的.PHONY目标是：all、install、clean、distclean、TAGS、info、check。

	-- from: https://stackoverflow.com/questions/2145590/what-is-the-purpose-of-phony-in-a-makefile

# What does these do in a makefile: %.o:%.c , $^ , $@ , $< .
Makefile example:
```
CC = gcc
CFLAGS = -std=c99 -W -Wall
CFLAGSS = -std=c99 -W
LIBS = -lm

prog : main.o double.o coord2D.o coord3D.o
    $(CC) $^ $(LIBS) -o $@

%.o : %.c
    $(CC) $(CFLAGS) $< -c

coord2D.o: coord2D.c coord2D.h double.h
coord3D.o: coord3D.c coord3D.h double.h
double.o: double.c double.h
main.o: main.c double.h coord2D.h coord3D.h
```

简单来说，%.o 是匹配所有以 .o 结尾的文件的target。

"%.o: %.c" 表示任何以 .o 结尾的文件都依赖于以 .c 结尾的相同文件名。

以下以制表符开头的行是生成 %.o 形式的文件时使用的规则。例如：

可执行文件prog，依赖“main.o”（第 6 行）。make 寻找构建 main.o 的规则，将找到存在的两个规则：

特定规则（按名称指定文件名）：
    main.o: main.c double.h coord2D.h coord3D.h
此规则指定 main.o 的所有依赖项。结果是，如果这些文件中的任何一个比 main.o 新，则将重新编译 main.o

一般规则：
    %.o: %.c
             $(CC) $(CFLAGS) $< -c
这将运行命令“gcc -std=c99 -W -Wall main.c -c” “$<”是另一个通配符，它的意思是“在此处包含target行中的第一个先决条件文件名” —— 在本例中为 main.C

例子中，通配符“$@”（target）和 $^（在此处包括完整的先决条件文件名列表）。

prog 的命令将扩展为：

```
gcc main.o double.o coord2D.o coord3D.o -lm -o prog
```

通配符规则使得可用较少的规则构建复杂项目。

	-- from: https://stackoverflow.com/questions/54854128/use-of-o-c-in-makefile
