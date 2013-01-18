Building a kernel 3.7.1 for the olinuxino
===
Latest Stable Kernel 3.7.1 include USB support for imx23-olinuxino boards. This  
build contain patches for SPI and I2C interface and also patches for easy kernel  
integration of wifi rtl8188cu device. You can download precompiled image from here:  
https://www.dropbox.com/s/rfnmzdbcu21pfgf/sd_mmc_bootstream.raw  

Software requirements  
---
Cross compiler and git.  

On Debian distributions install as:  
 
$: sudo apt-get install gcc-arm-linux-gnueabi  
$: sudo apt-get install git  

Hardware requirements  
---
SD card with two partition:
- boot partition for kernel, and 
-rootfs partition with your prefered linux distribution.  
See file `Make bootable SD-Card` for instructions how to make the two partition SD card  

Getting the code  
===
```  
$: git clone https://github.com/koliqi/imx23-olinuxino  
```  
This package contain also:  
* Freescale imx-bootlets and utility elftosb2,  
* patches for imx-bootlets.  

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
│   ├── 0001-ARM-imx23-olinuxino-Add-spi-support.patch  
│   ├── 0001-MXS-imx23-olinuxino-Add-i2c-support.patch  
│   └── 0001-rtl8192cu.patch  
├── Make bootable SD-Card.md  
├── README.md  
└── rootfs  



1) Building a kernel from sources  
===

Switch into directory kernel and download kernel sources:  
```
$: cd imx23-olinuxino/kernel  
$: wget http://www.kernel.org/pub/linux/kernel/v3.0/linux-3.7.1.tar.bz2
$: tar xvjf linux-3.7.1.tar.bz2
$: mv linux-3.7.1 linux-stable  
```
New directory linux-stable contain kernel sources. Switch into directory linux-stable to apply patches.  
```
$: cd linux-stable  
$: patch -p1 < ../0001-MXS-imx23-olinuxino-Add-i2c-support.patch  
$: patch -p1 < ../0001-ARM-imx23-olinuxino-Add-spi-support.patch  
$: patch -p1 < ../0001-rtl8192cu.patch  
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
Users of imx23-olinuxino-maxi need to enable the smsc95xx driver in the kernel configuration,   
otherwise you won't get your network up in the olinuxino.  
```
       -> Device Drivers  
         [*] Network device support  --->  
                   USB Network Adapters  --->   
                         <*> Multi-purpose USB Networking Framework  
                         <*> SMSC LAN95XX based USB 2.0 10/100 ethernet devices
```

Users of imx23-olinuxino-mini need to enable driver rtl8188cu:  

Enable Multi-purpose USB Networking Framework:  
```
       -> Device Drivers  
         [*] Network device support  --->  
                   USB Network Adapters  --->   
                         <*> Multi-purpose USB Networking Framework   
```
then activate  IEEE 802.11 networking stack:  
```
		[*] Networking support  --->  
                                   [*]   Wireless  ---> 
                                   <*>   cfg80211 - wireless configuration API  
                                   [ ]     nl80211 testmode command  
                                   [ ]     enable developer warnings  
                                   [ ]     cfg80211 regulatory debugging  
                                   [ ]     enable powersave by default  
                                   [ ]     cfg80211 DebugFS entries  
                                   [*]     cfg80211 wireless extensions compatibility  
                                  < >   Common routines for IEEE802.11 drivers  
                                  <*>   Generic IEEE 802.11 Networking Stack (mac80211)  
```
and activate Realtek 8192C USB WiFi and IEEE 802.11 for Host AP:  
```
     -> Device Drivers  
        [*] Network device support  --->  
                     [*]   Wireless LAN  --->  
                     <*>   Realtek 8192C USB WiFi  
                     <*>   IEEE 802.11 for Host AP (Prism2/2.5/3 and WEP/TKIP/CCMP)  
```
Power LED need to have default ON trigger activated on kernel:  
```
       -> Device Drivers  
         -> LED Support  
             <*>   LED Default ON Trigger  
```
Activate other drivers as needed. Save and exit from menuconfig application.  

Compile kernel  
---
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- zImage modules  
```
Kernel is ready at arch/arm/boot/zImage.  

Create device tree blob .dtb file:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- imx23-olinuxino.dtb  
```
Join zImage and imx23-olinuxino.dtb into a new file zImage_dtb:  
```
$: cat arch/arm/boot/zImage arch/arm/boot/dts/imx23-olinuxino.dtb > arch/arm/boot/zImage_dtb  
```
If you want to repeat this procedure, start with clean-up:  
```
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- distclean  
$: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- clean  
```



2) Bootlets  
===  

The iMX23 SoC contains a built-in ROM firmware capable of loading and   
executing binary images in special format from different locations including   
MMC/SD card and NAND flash. Binary image is called a boot stream (.bs).  
Boot stream consists of a series of smaller bootable images (bootlets)  
such clock bootlet, power bootlet etc.  
Linking bootlets with kernel and converting from elf format to raw boot stream is  
done with utility elftobs. 
In this package, utility elftobs2 is located in directory elftosb-0.3.  

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
then switch into directory imx-bootlets-src-10.05.02 and apply patches:  
```
$: patch -p1 < ../imx23_olinuxino_bootlets.patch  
```
This patched package require zImage in this directory. We have created   
zImage_dtb instead, so make symbolic link as:  
```
$: ln -s ../../kernel/linux-stable/arch/arm/boot/zImage_dtb ./zImage  

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






 
 
  

