# python | pip | venv

```
$ python -m venv envname
$ virtualenv envname
$ source envname/bin/activate
(envname) $
(envname) $ deactivate
$
$ pip list
$ pip show
```

https://wiki.archlinux.org/title/Python/Virtual_environment

# linux

## setterm 关闭终端屏幕

	# setterm -blank 10

## tty 终端字符流

	# cat /dev/vcsN
	# script tmp_file.txt

## tty 屏幕亮度

	$ sudo sh -c 'echo 100 > /sys/class/backlight/intel_backlight/brightness'
	# echo 100 > /sys/class/backlight/intel_backlight/brightness

## 

# shell 出现异常提示

```
$ cd todo/linux/
-bash: openEuler_history: command not found
$ unset PROMPT_COMMAND
```

# 查找哪个包提供某文件

	$ sudo apt-file update
	$ apt-file search 'sys/sdt.h'
	systemtap-sdt-dev: /usr/include/riscv64-linux-gnu/sys/sdt.h

# download opensbi patch from url

```
#! /usr/bin/python3

import sys, re, requests
from bs4 import BeautifulSoup
from html import unescape

url = sys.argv[1]
html = requests.get(url).text

soup = BeautifulSoup(html, 'html.parser')
pre=soup.find('pre')
for a in pre.findAll('a'):
    a.replaceWithChildren()

subject = soup.body.h1.text
patch = 'Subject: ' + subject + '\n' + re.sub(r'(<.*)( at )(.*>)',r'\1@\3', unescape(pre.text))

fname = subject + '.mbox'
print('patch at: ' + fname)
with open(subject + '.mbox', 'w') as f:
    f.write(patch)
```

> https://gist.github.com/wxjstz/ce4a423ad49a2cf5c085676377d344d8 by wxjstz

# patch

- Create a Patch for a Single File in Linux

```
$ diff -u OriginalFile UpdatedFile > PatchFile
$ # -u -- Create a diff file in the unified format
```
- Apply a Patch to a File

```
$ patch OriginalFile < PatchFile
```

- Undo a Patch

```
$ patch -R OriginalFile < PatchFile
```

- Create a Patch for a Directory in Linux

```
$ diff -ruN OriginalDir UpdatedDir > PatchFile
$ # -r -- Recursively compare any subdirectories found
$ # -u -- Create a diff file in the unified format
$ # -N -- Treat absent files as empty
```

- Apply a Patch to a Directory

```
$ patch -p0 < PatchFile
$ # Apply the patch to the same directory structure as when the patch was created
```

- Undo a Patch

```
$ patch -R -p0 OriginalFile < PatchFile
```

> https://www.shellhacks.com/create-patch-diff-command-linux

# vim

## 刷新，重新加载

	:e
	:edit

或者

	:edit!

后者将会放弃当前修改，重新载入

	:set autoread

自动检测是否有外部修改

## 重新加载 vimrc

	:source $MYVIMRC

## 多行剪切、复制、粘贴

剪切：

	dd	剪切光标所在行
	2dd	剪切n行
	p	粘贴

复制： yy
删除： dd

复制1到10行，粘贴至12行：

	:1,10 co 12

删除1到10行：

	:1,10 de

## visual mode

```
V	选区整行

```

## 批量行首添加空格

方法1

1. ESC, ctrl v 进入 VISUAL BLOCK 模式
2. j,k键上下移动选中多行
3. shift i, 进入插入模式
4. 只需在光标当前行中添加缩进
5. ESC，选中的行会全部生效

方法2

在 100 行和 110 行之间行首添加缩进

	:100,110 s/^/    /

## 取消换行自动tab缩进

```
:set paste
:
```

## 设置自动换行

自动换行，是每行超过 n 个字的时候自动加换行符：

	:set textwidth=75

自动折行，是把长的一行用多行显示，不在文件里加换行符。开启和关闭方法：

	:set wrap
	:set nowrap

## 快速搜索

`#` 号可以快速搜索光标所在的单词，并移动到下一个位置。类似索引。

# nmcli WIFI connect

```
$ nmcli dev wifi		# 扫描可用wifi
$ nmcli device wifi list	# 查看可用wifi列表
$ nmcli device wifi connect <SSID> password <password>
$ nmcli connection show
$
$ ip addr			# 列出以太网卡
$ nmcli connection		# 列出当前活动的以太网卡
$ 分配静态IP: nmcli connection modify <interface_name> ipv4.address  <ip/prefix>
$ nmcli con mod enp0s3 ipv4.addresses 192.168.1.4/24
$ nmcli con mod enp0s3 ipv4.gateway 192.168.1.1
$ nmcli con mod enp0s3 ipv4.method manual	# DHCP -> static
$ nmcli con mod enp0s3 ipv4.dns "8.8.8.8"
$ nmcli con up enp0s3
$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
$ ip addr show enp0s3
```

> https://cloud.tencent.com/developer/article/1876071

# static IP

试了检索到的 netplan 方法[1]，无效。
在IRC #ubuntu 上有人回复：
with ubuntu desktop 22.04, shouldn't fiddle with netplan.
the gnome network settings (which will configure NetworkManager are the
right thing). nm-connection-editor should also work.

以下配置有效：
```
$ nm-connection-editor
IPv4 Settings
Address 192.168.0.xx
Netmask 255.255.255.0
Gateway 192.168.0.1
DNS servers 201.207.230.100, 202.106.196.115
Routes Address 192.168.0.1
Routes Netmask 255.255.255.0
```

[1] [Ubuntu22.04静态ip配置](https://zhuanlan.zhihu.com/p/612288061)

# minicom 串口通讯工具

```
$ sudo apt install minicom
$ sudo minicom -D /dev/ttyUSB0 -b 115200
$ # 获取串口名称
$ # dmesg | grep tty*
$ # ls /dev/ | grep tty*
$
$ Ctrl-A C 清屏
$
$ # 设定接受键盘输入
$ Ctrl-A O -> Serial Port Setup -> Hardware flow control
  1. Serial port setup -->Hardware Flow Contorl : NO
  2. Serial port setup -->Software Flow Control : Yes
```

# tmux 相关

- vim 模式

	$ vim ~/.tmux.conf
	:setw -g mode-keys vi
	ctrl b  [	进入 vim 模式
	Space		开始选区	
	Enter		结束选区，复制
	ctrl b  ]	粘贴

	w  |  b		前进 | 光标后退一个 word

- 常用命令

```
$ tmux new -s <session name>
$ tmux attach -t <session name>
$ tmux ls
Ctrl-b '	select the window index
Ctrl-b w	to get an interactive index to choose from (0-9a-z)
Ctrl-b %	左右分屏
Ctrl-b "	上下分屏
Ctrl-b o	切换屏幕
Ctrl-b space	左右、上下分屏模式切换
```

# bash 常用快捷键

## 两种模式

* 默认的 emacs 模式

```
$ set -o emacs

ctrl a	光标回到开头
ctrl e	光标跳到最后
ctrl k	剪切光标往后的内容
ctrl y	粘贴
ctrl u	剪切光标往前的内容
ctrl f/b	向前、后移动一个字符
alt  f/b 	向前、后移动一个单词

ctrl x ctrl e	用 $EDITOR 进行全屏编辑
```

* vi 模式

```
$ set -o vi

0	光标移到开头
$	光标移到最后
b/w	向前、后移动一个单词

```

## 其他

```
alt .	选取之前的参数。也可以 alt 2 .
```

# shell 脚本记录

- 去掉4中与3重合的项，并保存到6
```
while read line; do
  if [[ $line =~ ^[[:blank:]]*$ || $line =~ ^[[:blank:]]*# ]]; then
    continue
  fi
    l=${line/%=*/=}
    if ! grep -q $l 3-config.config; then
      echo $line
    fi
done < 4-config.config >6-config.config
```

# ssh 相关

- 确认宿主机是否安装了 openssh-client, openssh-server
- 安装后确认ssh-server已启动(sshd)
-
```
$ dpkg -l | grep ssh
$ ps -e | grep ssh
$ sudo systemctl status ssh
$ sudo /etc/init.d/ssh start # stop, restart
$ ssh -p 123 user@localhost 
$ # ssh keygen
$ ssh-keygen -t rsa
$ ls ~/.ssh    # id_rsa id_rsa.pub
$ ssh-copy-id user@localhost    # id_rsa.pub -> ~/.ssh/authorized_key
$ # mount sshfs
$ sshfs name@192.168.host.ip:/home/name/work/path/ ~/host_files
$
$ # start_qemu.sh 增加" -net user,hostfwd=::2222-:22 -net nic \ "
$
$ # 公钥添加进目标机 ~/.ssh/authorized_key
$ ssh-copy-id name@192.168.x.x
$ 
$ cat ~/.ssh/config 
Host claudine
	HostName 123.60.61.8
	User mingzheng
	ServerAliveInterval 900
```

> https://zhuanlan.zhihu.com/p/587777394

# 创建新用户及增加sudo权限

```
# adduser newuser
# passwd newuser
# vim /etc/sudoers
```

- 找到如下位置新增

```
## Alow root to run any commands anywhere
root 	ALL=(ALL)	ALL
newuser	ALL=(ALL)	All
```

# python 相关

- 查看编译python时的默认配置，包括module搜索路径

```
$ python -m sysconfig
```

- 查看真实的module搜索路径

```
$ python3 -c “import sys;print(sys.path)”

```
- 通过 `$PATH PYTHONPATH` 可以向 `sys.path` 添加值

# diff

- 比较两个文件的区别并用并排的格式输出

- 参数： -y(并排显示)  -W(指定行宽)

```
$ diff test01.md test02.md -y -W 140
I need to buy apples.                           I need to buy apples.
I need to run the laundry.                    | I need to do the laundry.
I need to wash the dog.                       | I need to wash the car.
I need to get the car detailed.               | I need to get the dog detailed.
```

- 说明:
"|"表示前后2个文件内容有不同
"<"表示后面文件比前面文件少了1行内容
">"表示后面文件比前面文件多了1行内容

- 参考：https://blog.csdn.net/qq646748739/article/details/81182673


# mount 挂载 windows 分区

```
$ sudo fdisk -l
$ sudo mkdir /mnt/nvme0n1p3
$ sudo mount /dev/nvme0n1p3 /mnt/nvme0n1p3
```

# 进程相关

```
$ top -u username  # 查看用户所有进程

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

	$ sudo dpkg -i package_file.deb
	$ sudo apt-get remove package_name

# Convert .rpm files to .deb files

- Install alien
- sudo alien package_file.rpm

# ubuntu 查找已安装的包

	$ dpkg -l | grep -i "name"

# 文件解压 | 压缩

## zstd | zst 解压

```
$ zstd -d XXX.zst
```

## gz 解压

```
$ gzip -dk file.gz	# k 保留源文件
$ gunzip file.gz
```

## .tar.xz | .tar.gz 解压

```
$ apt install xz-utils
$ tar -xf xxx.tar.xz
$ tar -xf xxx.tar.gz
$ xz -dk xxx.xz    # d 解压.xz , k 保留源文件
```

## .tar.bz2 解压

```
$ tar -jxvf ×××.tar.bz2
```

## .tar 解压

```
$ tar -xvf xxx.tar
```

## 压缩为 tar.gz

```
$ tar czvf my.tar.gz file1 file2 ....fileN
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
