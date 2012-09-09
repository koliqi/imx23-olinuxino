Building a kernel 3.x for the OLinuXino:
===

Software requirements:  
---
-Cross compiler and git.  

On Debian distributions install as:  
 
$: sudo apt-get install gcc-arm-linux-gnueabi  
$: sudo apt-get install git  

-Hardware requirements:  
---
SD card with two partition, 1st for boot and second for rootfs. See file `Make bootable SD-Card`  
for instructions to make the two partition SD card  

Getting the code:  
===
```  
$: git clone https://github.com/koliqi/imx23-olinuxino  
```
This package contains:  
* patches that have initial usb support with kernel linux-3.X,  
* Freescale imx-bootlets and utility elftosb2,  
* patches for imx-bootlets, and  
* patches for barebox.  

Freescale imx-bootlets and utility elftosb2 are from Freescale package L2.6.31_10.05.02_ER_source.  
Utility elftosb2 is distributed in binary form. Sources can be found at the following link:  
http://repository.timesys.com/buildsources/e/elftosb/elftosb-10.12.01/  

Directory tree is as follows:  

├── boot  
│   ├── 0001-boards-Add-support-for-imx233-olinuxino-board.patch  
│   ├── elftosb-0.3  
│   │   ├── elftosb2  
│   │   └── Makefile  
│   ├── imx23_olinuxino_bootlets.patch  
│   └── imx-bootlets-src-10.05.02.tar.gz  
├── kernel  
│   ├── 0001-usb_led.patch  
│   ├── usb_led.patch  
│   └── usb.patch  
├── Make bootable SD-Card.md  
├── README.md  
└── rootfs  



1) Building a kernel from sources  
===

Switch into directory kernel and download kernel sources:  
```
$: cd imx23-olinuxino/kernel  
$: git clone -b patches-3.6-rc2 git://github.com/Freescale/linux-mainline.git  
```
Inside the new created directory linux-mainline, there are files representing patches-3.6-rc2 branch.  
Switch into directory linux-mainline to apply patch.  
```
$: cd linux-mainline  
$: patch -p1 < ../0001-usb_led.patch  
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
Make sure to enable the smsc95xx driver in the kernel configuration,  otherwise you won't  
get your network up in the olinuxino. Save configuration and exit.  

Compile kernel  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- zImage modules  
```
Kernel is ready at arch/arm/boot/zImage.  

*Create device tree blob .dtb file:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- imx23-olinuxino.dtb  
```
Join zImage and imx23-olinuxino.dtb into a new file zImage_dtb:  
```
$: cat arch/arm/boot/zImage arch/arm/boot/imx23-olinuxino.dtb > arch/arm/boot/zImage_dtb  
```
If you want to repeat this procedure, start with clean-up:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- distclean  
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- clean  
```

2) Bootlets  
===  

"The iMX23 SoC contains a built-in ROM firmware capable of loading and   
executing binary images in special format from different locations including   
MMC/SD card and NAND flash."  *)  
Binary image is called a boot stream (.bs). Boot stream consists of a series of   
smaller bootable images (bootlets) that when all together create the binary image  
of the kernel. Linking bootlets with kernel and converting from elf format to raw  
boot stream is done with utility elftobs. In this package, utility elftobs2 is   
located in directory elftosb-0.3.   

Switch into directory elftosb-0.3 and make symbolic link into compilers default   
PATH:   
```
$: sudo ln -s `pwd`/elftosb2 /usr/sbin/      
```
Check with  locate:  
```
$: locate elftosb2  
```
elftosb2 should be located at /usr/sbin/elftosb2.  
 
Next, switch into directory boot and untar archive imx-bootlets-src-10.05.02.tar.gz  
```
$: tar xvzf imx-bootlets-src-10.05.02.tar.gz  
```
then go into directory imx-bootlets-src-10.05.02 and apply patches:  
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
To install bootstream onto SD/MMC card, type: sudo dd if=sd_mmc_bootstream.raw of=/dev/sdXY     where X is the correct letter for your sd or mmc device (to check, do a ls /dev/sd*) and Y is   the partition number for the bootstream  
```
In my system sd device is /dev/sdb1, so writing bootstream file in card is done by:  
```
$: sudo dd if=sd_mmc_bootstream.raw of=/dev/sdb1  
```
Card is ready.  

It is good practice to work with multiple consoles. Open one into directory linux-mainline,  
second into imx-bootlets-src-10.05.02 and third console for minicom to monitor olinuxino.  

If olinuxino failing to boot because of the error:  
```
Undefined Instruction 
r14_unHTLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLLFC   
```
Probably your toolchain does not support cpu arm926ej-s. Download sources and   
compile --with-arch=armv5te --with-tune=arm926ej-s. If you prefer premade binaries,  
download arm-none-eabi toolchain. It is proven to work with arm926ej-s.  

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
Instead of CROSS_COMPILE=arm-linux-gnueabi- write CROSS_COMPILE=arm-none-eabi-  
and follow instructions in section 1) and 2).  

*) i.MX23 Linux BSP User’s Guide, Freescale Rev. 10.05.03, 05/2010  






 
 
  
