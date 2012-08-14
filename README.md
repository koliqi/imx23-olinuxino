Building a kernel 3.x for the OLinuXino:
===

*Software requirements  
You need to have cross compiler and git installed. On Debian distributions install as:
 
$: sudo apt-get install gcc-arm-linux-gnueabi   
$: sudo apt-get install git

*Hardware requirements
SD card with two partition, 1st boot and second rootfs partition. 

Getting the code:  
--- 
```  
$: git clone https://github.com/koliqi/imx23-olinuxino  
```
This package contain initial patches usb support with kernel linux-3.X,  
Freescale imx-bootlets, Freescale utility elftosb2 and patches for imx-bootlets.  
  
Directory tree is as follow:  

imx23-olinuxino    
├── bootlets    
│   ├── imx23_olinuxino_bootlets.patch  
│   ├── elftosb-0.3  
│   └── imx-bootlets-src-10.05.02.tar.gz  
├── kernel  
│   └── usb.patch  
├── rootfs  
│     
└── README.md  

1.) Building a kernel 3.x for the OLinuXino from sources  
===

Change into directory kernel and download kernel sources:  
```
$: cd imx23-olinuxino/kernel  
$: git clone -b patches-3.6-rc1 git://github.com/Freescale/linux-mainline.git   
```
Inside new created directory linux-mainline are files from patches-3.6-rc1 branch.  
Change into directory linux-mainline to  apply usb.patch.  
```
$: cd linux-mainline  
$: patch -p1 < ../usb.patch  
```
response would be:  
```
patching file arch/arm/boot/dts/imx23-olinuxino.dts  
patching file arch/arm/boot/dts/imx23.dtsi  
patching file drivers/usb/otg/mxs-phy.c  
```
Configure kernel  
---
Start from supplied default mxs_defconfig configuration:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- mxs_defconfig  
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig  
```
Select `Boot options --->` and select following options:  
```
 Boot options ---->  
  [*]  Use appended device tree blob to zImage (EXPERIMENTAL)  
  [*]   Supplement the appended DTB with traditional ATAG information  
(console=ttyAMA0,115200 root=/dev/mmcblk0p2 rw rootwait)  Default kernel command string  
```
Save configuration and exit.  

Compile kernel  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- zImage modules
```
Kernel is ready at arch/arm/boot/zImage.  

*Create device tree blob .dtb file:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- imx23-olinuxino.dtb  
```
Join zImage and imx23-olinuxino.dtb file in new file zImage_dtb:  
```
$: cat arch/arm/boot/zImage arch/arm/boot/imx23-olinuxino.dtb > arch/arm/boot/zImage_dtb  
```
If you want to repet this procedure, start with clean-up:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- distclean
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- clean
```

2.) Bootlets  
===  

"The iMX23 SoC contains a built-in ROM firmware capable of loading and   
executing binary images in special format from different locations including   
MMC/SD card and NAND flash."  *)  
Binary image is called a boot stream (.bs). Boot stream consists of a series of   
smaller bootable images (bootlets) and finaly with binary image of kernel. Linking   
bootlets with kernel and converting from elf format to raw boot stream is done   
with utility elftobs. In this package, utility elftobs2 is located in directory  
elftosb-0.3.   

Change into directory elftosb-0.3 and make symbolic link into compilers default   
PATH:   
```
$: sudo ln -s `pwd`/elftosb2 /usr/sbin/      
```
Check with  locate:  
```
$: locate elftosb2  
```
elftosb2 should be located at /usr/sbin/elftosb2.  
 
Next, change into directory bootlets and untar archive imx-bootlets-src-10.05.02.tar.gz  
```
$: tar xvzf imx-bootlets-src-10.05.02.tar.gz  
```
then change into directory imx-bootlets-src-10.05.02 and apply patches:  
```
$: patch -p1 < ../imx23_olinuxino_bootlets.patch  
```
This patched package require zImage in this directory. We have created   
zImage_dtb instead, so make symbolic link as:  
```
$: ln -s ../../kernel/linux-mainline/arch/arm/boot/zImage_dtb ./zImage  

```

Make boot stream file:  
```
$: make CROSS_COMPILE=arm-linux-gnueabi-  clean
$: make CROSS_COMPILE=arm-linux-gnueabi-  
```
Final response would be:  
```
To install bootstream onto SD/MMC card, type: sudo dd if=sd_mmc_bootstream.raw of=/dev/sdXY    where X is the correct letter for your sd or mmc device (to check, do a ls /dev/sd*) and Y is   the partition number for the bootstream  
```
In my system sd device is /dev/sdb1, so writing bootstream file in card is done by:  
```
$: sudo dd if=sd_mmc_bootstream.raw of=/dev/sdb1  
```
Card is ready.  

*Tip 1:  
Is good practice to work with multiple consoles. Open one into directory linux-mainline,  
second into imx-bootlets-src-10.05.02 and third console for minicom to monitor olinuxino.  

*Tip 2:  
If olinuxino failing to boot with the error:  
```
Undefined Instruction 
r14_unHTLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLFC   
```
Probably your toolchain does not support cpu arm926ej-s. Download sources and   
compile --with-arch=armv5te --with-tune=arm926ej-s. Second option is to download  
binaries arm-none-eabi:  

Add PPA in your system:  
```
$: sudo add-apt-repository ppa:germia/archive3  
```
Refresh list of software available, including the PPA you just added:  
```
$: sudo apt-get update  
```

Install packages:
```
sudo apt-get install gcc-arm-none-eabi binutils-arm-none-eabi \
newlib-arm-none-eabi \
gdb-arm-none-eabi
```

3.) Make bootable SD-Card  
===

Create two partition in your SD card. Boot stream file is going to be installed in 1st  
partition (type 53 - OnTrack DM6 Aux3 )and rootfs in second partition (type 83 - Linux).  
Here is an example of SD card 2GB with two partitions :  
  
  
Disk /dev/sdb: 1977 MB, 1977614336 bytes  
61 heads, 62 sectors/track, 1021 cylinders, total 3862528 sectors  
Units = sectors of 1 * 512 = 512 bytes  
Sector size (logical/physical): 512 bytes / 512 bytes  
I/O size (minimum/optimal): 512 bytes / 512 bytes  
Disk identifier: 0x00000000  

   Device Boot      Start         End      Blocks   Id  System  
/dev/sdb1            2048       33297       15625   53  OnTrack DM6 Aux3  
/dev/sdb2           33298     3862527     1914615   83  Linux  

Formatting and partitioning a SD card  
---  
Insert your SD card on card reader of your linux developing machine and list partition  
tables for all disks:  
 
$: sudo fdisk -l  

Identify your SD disk. In this example SD disk is recognized as /dev/sdb.  
Unmount all mounted partitions, i.e. `sudo umount /dev/sdb2 `.  
Run fdisk:  

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


Createting the SD raw partition image   
---

Create and fill with zeros the image file sd_mmc_bootstream.raw with 4  
blocks of 512 byes each:  

$: dd if=/dev/zero of=sd_mmc_bootstream.raw bs=512 count=4  

Append the boot stream file imx23_linux.sb to the file sd_mmc_bootstream.raw  

$: dd if=./imx23_linux.sb of=sd_mmc_bootstream.raw ibs=512 seek=4 conv=sync,notrunc  

Write the file to the boot partition on the 1st partition /dev/sdb1:  

$: sudo dd if=sd_mmc_bootstream.raw of=/dev/sdb1  

Installing rootfs   
---
For now, rootfs is not populated. 

Format the second partition on the SD card:  

$: sudo mkfs.ext3 /dev/sdb2  

Make mount point directory /mnt/mmc   

$: sudo mkdir /mnt/mmc  

Mount the partition /dev/sdb2  on mount point directory /mnt/mmc:  

$: sudo mount /dev/sdb2 /mnt/mmc  

Copy the rootfs to  mount point directory:  

$: sudo cp -a ../../rootfs/* /mnt/mmc  

Unmount SD disk:  

$: sudo umount /dev/sdb2  
$: sync  

At this point the SD card should be ready for use.  






*) i.MX23 Linux BSP User’s Guide, Freescale Rev. 10.05.03, 05/2010  






 
 
  
