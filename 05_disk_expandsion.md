```
sudo apt-get install gparted
sudo gparted
// Why is there already 1G of used space in the newly created empty disk partition?
mkdir /home/${USERNAME}/new
sudo mount /dev/partition1 /home/${USERNAME}/new_partition_name
sudo blkid
sudo vim /etc/fstab
// work was on /dev/partition1
UUID=234q250.....asf234198hf /home/${USERNAME}/new_partition_name ext4 defaults 0 3 
```
