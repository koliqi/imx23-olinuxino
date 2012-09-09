barebox on OLinuXino:
===

Software requirements:  
---
* Cross compiler armv5 and git.  

On Debian distributions install as:  
```
$: sudo apt-get install gcc-arm-linux-gnueabi  
$: sudo apt-get install git  
```
Hardware requirements:  
---
* SD card with four partitions
 - the first one for the bootlets and barebox (at least 256 kiB)  
 - the second one for the persistant environment and device tree blob file ( at least 256k)  
 - the third one for the kernel (2 MiB ... 4 MiB in size)  
 - the 4th one for the root filesystem which can fill the rest of the available space  

Mark the first partition with the partition ID "53". Second and third partitions no filesystem needed.  
On 4th partition make filesystem of your choise (ext2, ext3..), mount it and copy all required rootfs data and  programs into it.  

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
Utility elftosb2 is distributed in binary form. If you want to compile from sources, they can  
be found at the following link: `http://repository.timesys.com/buildsources/e/elftosb/elftosb-10.12.01/`  

Directory tree is as follow:  
```
.  
├── barebox on olinuxino.md  
├── boot  
│   ├── 0001-boards-Add-support-for-imx233-olinuxino-board.patch  
│   ├── elftosb-0.3  
│   │   ├── elftosb2  
│   │   └── Makefile  
│   ├── imx23_olinuxino_bootlets_barebox.patch  
│   ├── imx23_olinuxino_bootlets.patch  
│   └── imx-bootlets-src-10.05.02.tar.gz  
├── kernel  
│   ├── 0001-usb_led.patch  
│   ├── usb_led.patch  
│   └── usb.patch  
├── Make bootable SD-Card.md  
├── README.md  
└── rootfs  
```



1) Building a kernel 3.x from sources  
===

Switch into directory kernel and download kernel sources:  
```
$: cd imx23-olinuxino/kernel  
$: git clone -b patches-3.6-rc2 git://github.com/Freescale/linux-mainline.git  
```
Inside the new created directory linux-mainline, there are files representing patches-3.6-rc2 branch.  
Switch into directory linux-mainline to apply patch and export env variales:  
```
$: cd linux-mainline  
$: patch -p1 < ../0001-usb_led.patch  
$: export ARCH=arm  
$: export CROSS_COMPILE=arm-linux-gnueabi  
```
* Configure kernel  
---
Start from supplied default mxs_defconfig configuration:  
```
$: make  mxs_defconfig  
$: make  menuconfig  
```
Select `Boot options --->` and select following options:  
```
 Boot options ---->  

 -*- Flattened Device Tree support  
 (0) Compressed ROM boot loader base address  
 (0) Compressed ROM boot loader BSS address  
 [ ] Use appended device tree blob to zImage (EXPERIMENTAL)  
 (console=ttyAMA0,115200 root=/dev/mmcblk0p4 rw rootwait) Default kernel command string  
```

Make sure to enable the smsc95xx driver in the kernel configuration, otherwise you won't  
get your network up in the olinuxino. Save configuration and exit.  

* Compile kernel  
```
$: make uImage modules  
```
Kernel is ready at arch/arm/boot/uImage. Copy kernel on SD Card. In my system SD Card is identified  
as /dev/sdb, so write dd command accordingly with your system:  
```
$: dd if=arch/arm/boot/uImage of=/dev/sdb3  


* Create device tree blob .dtb file:  
```
$: make imx23-olinuxino.dtb  
```
This file would be passed later to barebox bootloader. 

If you want to repeat this procedure, start with clean-up:  
```
$: make  distclean  
$: make  clean  
```

2) Barebox
===
Switch into directory boot and get barebox code:  
```
$: git clone git://git.pengutronix.de/git/barebox.git  
```
then go into directory barebox and apply patches:  
```
$: patch -p1 < ../0001-boards-Add-support-for-imx233-olinuxino-board.patch  
```
Make default configuration:  
```
$: make imx233-olinuxino_defconfig  
```
Patch create new directory arch/arm/boards/imx233-olinuxino with following structure  
```
├── config.h
├── env
│   ├── bin
│   │   ├── boot
│   │   └── init
│   └── config
├── imx23-olinuxino.c
└── Makefile
```
We want to pass blob file imx23-olinuxino.dtb to barebox enviroment, so create  
directory env/oftree and copy file:  
```
$: mkdir  arch/arm/boards/imx233-olinuxino/env/oftree  
$: cp ../../kernel/linux-mainline/arch/arm/boot/imx23-olinuxino.dtb arch/arm/boards/imx233-olinuxino/env/oftree/  
```
Compile barebox bootloader:  
```
$: make  
```
In top directory are following files:  
* `barebox` executable file, and  
* `barebox_default_env` default enviroment file.  

Copy enviroment file in second partition of CD Card. In my system SD Card is identified  
as /dev/sdb, so write dd command accordingly with your system:  
```
$: dd if=barebox_default_env of=/dev/sdb2  
```
Executable file `barebox` will be transfered on SD Card combined with bootlets.  

3) Bootlets  
===  

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
$: patch -p1 < ../imx23_olinuxino_bootlets_barebox.patch  
```
This bootlets package require u-boot in this directory, we have created barebox instead,  
so make symbolic link as:  
```
$: ln -s ../../barebox/barebox u-boot  
```
Make boot stream file:  
```
$: make clean  
$: make  
```
Final response would be:  
```
To install uboot bootstream onto SD/MMC card, type: sudo dd if=sd_mmc_uboot_bootstream.raw of=/dev/sdXY where X is the correct letter for your sd or mmc device (to check, do a ls /dev/sd*) and Y is the partition number for the bootstream
```
In my system sd device is /dev/sdb1, so writing bootstream file in card is done by:  
```
$: sudo dd if=sd_mmc_uboot_bootstream.raw of=/dev/sdb1  
```
CD Card is ready. Insert card on slot and power-up olinuxino. To stop autoboot press any key,  
otherwise following autoboot with messages:  
```
barebox 2012.09.0-00140-g361d5bd-dirty #8 Sat Sep 8 02:33:34 EDT 2012                                                 
                                                                                                                      
                                                                                                                      
Board: Olimex.ltd imx233-olinuxino                                                                                    
registered netconsole as cs1                                                                                          
mxs_mci@mci0: registered as mci0                                                                                      
mci@mci0: registered disk0                                                                                            
ehci@ehci: USB EHCI 1.00                                                                                              
Malloc space: 0x41c00000 -> 0x41ffffff (size  4 MB)                                                                   
Stack space : 0x41bf8000 -> 0x41c00000 (size 32 kB)                                                                   
running /env/bin/init...                                                                                              
                                                                                                                      
Hit any key to stop autoboot:  0                                                                                      
   Image Name:   Linux-3.6.0-rc1-09281-g6fba056                                                                       
   Created:      2012-09-07   1:41:33 UTC                                                                             
   OS:           Linux                                                                                                
   Architecture: ARM                                                                                                  
   Type:         Kernel Image                                                                                         
   Compression:  uncompressed                                                                                         
   Data Size:    2617576 Bytes =  2.5 MB                                                                              
   Load Address: 40008000                                                                                             
   Entry Point:  40008000                                                                                             
booting Linux kernel with devicetree                                                                                  
Uncompressing Linux... done, booting the kernel.                                                                      
[    0.000000] Booting Linux on physical CPU 0                                                                        
[    0.000000] Linux version 3.6.0-rc1-09281-g6fba056 (devel@laptop) (gcc version 4.4.5 (Debian 4.4.5-8) ) #16 Thu Se$
[    0.000000] CPU: ARM926EJ-S [41069265] revision 5 (ARMv5TEJ), cr=00053177                                         
[    0.000000] CPU: VIVT data cache, VIVT instruction cache                                                           
[    0.000000] Machine: Freescale i.MX23 (Device Tree), model: i.MX23 Olinuxino Low Cost Board 
```

