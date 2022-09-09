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
