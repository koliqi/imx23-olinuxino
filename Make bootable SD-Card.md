Make bootable SD-Card  

Create two partition in your SD card. Boot stream file is going to be installed in 1st  
partition (type 53 - OnTrack DM6 Aux3 )and rootfs in second partition (type 83 - Linux).  
Here is an example of 2G SD card with two partitions :  

 ```
Disk /dev/sdb: 1977 MB, 1977614336 bytes  
61 heads, 62 sectors/track, 1021 cylinders, total 3862528 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
Disk identifier: 0x00000000  

   Device Boot      Start         End      Blocks   Id  System  
/dev/sdb1            2048       33297       15625   53  OnTrack DM6 Aux3  
/dev/sdb2           33298     3862527     1914615   83  Linux  
```
Formatting and partitioning a SD card  
---
Insert your SD card into a card reader of your linux developing machine and list partition  
tables for all disks:  

$: sudo fdisk -l  

Identify your SD disk. In this example SD disk is recognized as /dev/sdb.  
Unmount all mounted partitions, i.e. `sudo umount /dev/sdb2 `.  
Run fdisk:  
```
$: sudo fdisk /dev/sdb  
      o Press 'p' to show the partitions on the card  
      o Press 'd' to delete a partition. Repeat to remove all partitions  
      o Press 'n' to create a new partition  
       + press 'p' to select the primary partitio  
       + press '1' for creating partition 1 on the card  
       + press Enter to start from first block  
       + Type '+16MB' to create the 16MB partitions  
      o Press 't' to change the newly created partition type  
       + Enter '53' for the new partition type  
      o Press 'n' to create a second partition  
       + Press Enter to accept all default setting  
      o Press 'w' to write  the partitions to the card and exit the fdisk  
```

Installing rootfs  
---
In this example SD disk is recognized as /dev/sdb. Format the second partition on the SD card:  
```
$: sudo mkfs.ext3 /dev/sdb2  
```
Make mount point directory /mnt/mmc  
```
$: sudo mkdir /mnt/mmc  
```
Mount the partition /dev/sdb2  on mount point directory /mnt/mmc:  
```
$: sudo mount /dev/sdb2 /mnt/mmc  
```
Copy the rootfs to  mount point directory:  
```
$: sudo cp -a `<path_to_your_rootfs> ` /mnt/mmc  
```
Unmount SD disk:  
```
$: sudo umount /dev/sdb2  
$: sync  
```
At this point the SD card should be ready for use.  
