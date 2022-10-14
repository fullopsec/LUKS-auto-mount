IF you need to create your partitions on your disk:

sudo fdisk -l  # to see if there is a partition and find the right disk name

MY disk is SDB(find your's name)
create partition:
sudo fdisk /dev/sdb

"n" to create a new partition, "d" to delete the partition, "p" to check the partition table

"w" to write


lsblk -f to verify that it's done corectly with the good file system(ext4 etc...)

____________________
MY PARTITION NAME IS SDB1
AND the mount point name I will choose is: databank

sudo cryptsetup luksFormat /dev/sdb1
(enter a passphrase)

sudo cryptsetup luksOpen /dev/sdb1 storage

sudo mkfs.ext4 /dev/mapper/storage

sudo dd if=/dev/urandom of=/root/keyfile bs=1024 count=4

sudo chmod 0400 /root/keyfile

sudo cryptsetup luksAddKey /dev/sdb1 /root/keyfile

sudo cryptsetup luksUUID /dev/sdb1

sudo nano /etc/crypttab

databank      /dev/sdb1  /root/keyfile  luks
OR
databank     /dev/disk/by-uuid/2965d419-34e6-4a15-a446-5dc4e715e3c8  /root/keyfile  luks

sudo nano /etc/fstab

(find it, its the volume in /dev/mapper that  you see in lsblk -f)
/dev/mapper/databank  /media/databank     ext4    defaults        0       2

sudo mkdir /media/databank

sudo chown daniel /media/databank

sudo reboot now

sudo chown daniel /media/databank

VERIFICATIONS:
lsblk -f 
cd /media/databank/
touch a
ls
