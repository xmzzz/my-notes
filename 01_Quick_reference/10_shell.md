# mount 挂载 windows 分区

```
$ sudo fdisk -l
$ sudo mkdir /mnt/nvme0n1p3
$ sudo mount /dev/nvme0n1p3 /mnt/nvme0n1p3
```

# 进程相关

```
$ ctrl + z 进程暂停并放到后台
$ watch -n 5 sh test.sh &	# 每 5s 在后台执行一次 test.sh 脚本
$ jobs -l	# 显示所有任务的 PID
$ fg %jobnumber		# 后台调到前台继续运行
$ bg $jobnumber		# 后台暂停变为后台继续运行

$ kill %jobnumber	# 3 种终止方法
$ kill pid
$ Ctrl + c 
```

# 实时监控 dmesg

```
$ sudo dmesg -wH
$ watch -n 0.1 "dmesg | tail -n $((LINES-6))"
```
from: http://t.zoukankan.com/welhzh-p-5006448.html

# Check which desktop environment you are using

```
$ echo $XDG_CURRENT_DESKTOP
```
	-- from: https://superuser.com/questions/96151/how-do-i-check-whether-i-am-using-kde-or-gnome

# How do I find out which boot loader I have?
If you have the /etc/lilo.conf file then you are using LILO (LInux LOader) This means that if you type lilo for example you should see the command dialog for the lilo booter.

If you have the /boot/grub/ directory then you are using GRUB (Grand Unified Boot Loader) This means that you should be able to use all the grub file like grub-install,grub-reboot...

Ubuntu 9.10 was the first version to use GRUB2

	-- from: https://askubuntu.com/questions/24459/how-do-i-find-out-which-boot-loader-i-have

# whoami
```
$ id -u `whoami`
// 0 root 
```
# deb install
```
sudo dpkg -i package_file.deb
sudo apt-get remove package_name
```
# Convert .rpm files to .deb files
- Install the alien program 
- sudo alien package_file.rpm

# other
```
ls -lthr /usr/lib/riscv64-linux-gnu/libbcc*

export LD_DEBUG=files

ls -ld work/

sudo chmod 765 work/
```

# deb install
sudo dpkg -i package_file.deb
sudo apt-get remove package_name

# Convert .rpm files to .deb files
- Install the alien program 
- sudo alien package_file.rpm

# ubuntu 查找已安装的包
dpkg -l | grep -i "name"

# unzip .tar.xz | .tar.gz

```
$ apt install xz-utils
$ tar -xf xxx.tar.xz
$ tar -xf xxx.tar.gz
```

# unzip .tar.bz2

```
$ tar -jxvf ×××.tar.bz2
```

# 修改终端字体颜色

- ubuntu 22.04 终端默认 ls 目录显示为深蓝色，在黑色背景下看不清

```
$ cd ~
$ dircolors -p > .dircolors
$ vim .dircolors      # 找到 "DIR 01;34" ，修改为 "DIR 01;35" ，保存退出
$ source .bashrc
```

- 默认的颜色表示
蓝色     目录
绿色     可知性文件
红色     压缩文件
浅蓝色   链接文件
灰色     其他文件
红色闪烁 链接的文件有问题
黄色     设备文件，包括 block, char, fifo

- 颜色的数字表示
30  黑色
31  红色
32  绿色
33  黄色
34  蓝色
35  紫红色
36  青蓝色
37  白色
1   透明色

- 第一组代码代表的意义
0  OFF
1  高亮显示
4  underline
5  闪烁
7  反白显示
8  不可见
