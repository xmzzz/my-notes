# Documentation/admin-guide/bug-bisect.rst

简单介绍了如何用 `git bisect` 来定位引出bug的commit

# Documentation/admin-guide/README.rst

1. 包含 Linux 的简介，Linux操作系统的特点和优点，可运行在哪些硬件以及有很好的可移植性。

2. 整体介绍了kernel 文档。README在很多子文件夹都有，里面包含了各个方面内核相关的安装指南。

3. 在升级内核遇到问题时，或者查看依赖最低版本要求，建议阅读：`Documentation/process/changes.rst`

4. 安装内核源码

- 解压内核源码

	$ xz -cd linux-6.x.tar.xz | tar xvf -

或用patch升级，详见：Documentation/process/applying-patches.rst

	$ xz -cd ../patch-6.x.xz | patch -p1

也可用脚本自动完成打patch

	$ linux/scripts/patch-kernel linux

5. 指定build目录

	$ cd /usr/src/linux-6.x
	$ make O=/home/name/build/kernel menuconfig
	$ make O=/home/name/build/kernel
	$ sudo make O=/home/name/build/kernel modules_install install

6. 介绍了各种生成config的方式。

- 三点提示，一是没必要的驱动容易导致问题；二是内嵌数学模拟器编译；

  三是"development","experimental", or "debugging" features非必要时可以关掉。

7. 编译内核

- 一般编译内核的三个步骤，如果有模块需要有最后一步

	$ make && make install && make modules_install

- 增加Verbose输出

	$ make V=1 all  # 增加编译、链接等信息
	$ make V=2 all  # 增加rebuild的原因打印

- 安装新内核前建议备份内核和模块，特别是版本号没变时。可用`CONFIG_LOCALVERSION`指定不同版本号

8. 遇到问题

- 报bug参考：Documentation/admin-guide/reporting-issues.rst

- 读bug参考：Documentation/admin-guide/bug-hunting.rst

- debug kernel 参考：`Documentation/dev-tools/gdb-kernel-debugging.rst`，`Documentation/dev-tools/kgdb.rst`

# Documentation/process/kernel-docs.rst

- 文档

- 邮件列表归档及搜索引擎

- 内核模块编程指南

- 出版读物

- kernel newbies 

- kernel news
