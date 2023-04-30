# Luks Automount

These commands are used to create a partition on a disk, encrypt it, and mount it on a specific mount point. All of this automatically without entering a password at boot.

## Video tutorial

https://www.youtube.com/watch?v=UXJrSji-nNo

## If you need to create your partitions on your disk:

### List partitions

    sudo fdisk -l

This command lists all the available disks and their partitions on the system. It's used to find the name of the disk that you want to partition.

My disk is "sdb" (find your's name)

### Create partition:

#### Format

    sudo fdisk <DISK>

#### Example

    sudo fdisk /dev/sdb
  
 This command starts the fdisk partitioning utility on the /dev/sdb disk. From here, you can create a new partition by selecting "n" and following the prompts.
 
"n" to create a new partition, "d" to delete the partition, "p" to check the partition table

"w" to write

### Verify partition was created

    lsblk -f 
  
It's used to verify that the partition was created with the correct filesystem.


##  Encrypting with LUKS 

#### Format

    sudo cryptsetup luksFormat <DISK_PARTITION>

#### Example

    sudo cryptsetup luksFormat /dev/sdb1

This command encrypts the newly created partition using the LUKS encryption format. You'll be prompted to enter a passphrase to secure the encryption.

## Unlock the LUKS partition

#### Format

    sudo cryptsetup luksOpen <DISK_PARTITION> <DISK_NAME>

#### Example

    sudo cryptsetup luksOpen /dev/sdb1 storage
  
This command unlocks the encrypted partition and maps it to the device named "storage".
  
## Create the LUKS filesystem

#### Format

    sudo mkfs.ext4 <MAPPED_PARTITION>

#### Example

    sudo mkfs.ext4 /dev/mapper/storage
  
This command creates a new ext4 filesystem on the encrypted partition.

## Using a keyfile

### Create the keyfile

    sudo dd if=/dev/urandom of=/root/keyfile bs=1024 count=4

This command generates a random keyfile to be used to unlock the encrypted partition.

### Set keyfile permissions

#### Format
  
    sudo chmod 0400 <KEYFILE>
  
#### Example

    sudo chmod 0400 /root/keyfile

This command sets the permissions of the keyfile so that only the root user can access it.

### Adding keyfile to LUKS

#### Format

    sudo cryptsetup luksAddKey <DISK_PARTITION> <KEYFILE>

#### Example

    sudo cryptsetup luksAddKey /dev/sdb1 /root/keyfile

This command adds the generated keyfile as an additional key to the LUKS encryption.

## Retrieve the UUID of LUKS partition

#### Format

    sudo cryptsetup luksUUID <DISK_PARTITION>

#### Example

    sudo cryptsetup luksUUID /dev/sdb1

This command retrieves the UUID of the encrypted partition, which is needed to configure the crypttab file.

## Edit crypttab

### Open crypttab

    sudo nano /etc/crypttab

This command opens the crypttab file for editing. This file is used to configure the encrypted partition to be automatically unlocked at boot time.

### Add the unlock partiton in crypttab

#### Format

    <MOUNT_NAME>      <DISK_PARTITION>  <KEYFILE>  luks
    
#### Example

    databank      /dev/sdb1  /root/keyfile  luks
  
OR

    databank     /dev/disk/by-uuid/<YOUR_DISK_UUID>  /root/keyfile  luks

This line is added to the crypttab file. It tells the system to unlock the partition located at /dev/sdb1 OR the UUID (better) using the keyfile located at /root/keyfile, and to map it to the device named "databank".

## Edit FSTAB

### Open FSTAB

    sudo nano /etc/fstab

This command opens the fstab file for editing. This file is used to configure the system to automatically mount the encrypted partition at boot time.

### Add the mountpoint to FSTAB

It is the volume in /dev/mapper that you see using lsblk -f

#### Format

    <MAPPER>  <MOUNT_POINT>     ext4    defaults        0       2

#### Example

    /dev/mapper/databank  /media/databank     ext4    defaults        0       2

 This line is added to the fstab file. It tells the system to mount the device named "databank" (which was created in the previous step) at the mount point /media/databank using the ext4 filesystem.

## Mount point

### Create the mount point

#### Format

    sudo mkdir <MOUNT_POINT>
    
#### Example

    sudo mkdir /media/databank

This command creates the mount point directory for the encrypted partition.

### Setting the ownership

#### Format

    sudo chown <USER> <MOUNT_POINT>

#### Example

    sudo chown daniel /media/databank

This command sets the ownership of the mount point directory to the user "daniel".

### Reboot

    sudo reboot now

### Reset the ownsership again

#### Format

    sudo chown <USER> <MOUNT_POINT>

#### Example

    sudo chown daniel /media/databank

This command sets the ownership of the mounted partition to the user "daniel".

## Verifications

### Verify partitions

    lsblk -f 
  
### Verify that mountpoint is readable and writable
```
   cd /media/databank/
   touch a
   ls
```
