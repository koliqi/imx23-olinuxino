Building barebox bootloader and a kernel 3.x for the OLinuXino:
===

Olinuxino board is defined in kernel 3.x as device tree machine. For systems  
with no bootloader, if kernel is configured with CONFIG_ARM_APPENDED_DTB,  
device tree binary (DTB) file is appended to zImage.  

If system has bootloader with device tree support, bootloader pass  
to kernel location of DTB file.  

This package support both methods.  
To build kernel with no bootloader follow instructions from file:  
* `Building a kernel for the OLinuXino.md`  

Building barebox bootloader is covered with:  
* `barebox on olinuxino.md`  






License:
---
Provided barebox patches are copyrighted and covered by  
the GNU General Public License. See source files for details.  

Documents:   
* `Building a kernel for the OLinuXino.md`   
* `barebox on olinuxino.md`  
* `Make bootable SD-Card.md`  
(C) Copyright 2012 Fadil Berisha and covered by license  
Attribution-ShareAlike 3.0 United States (CC BY-SA 3.0)  

You are free:  
    to Share — to copy, distribute and transmit the work   
    to Remix — to adapt the work  
    to make commercial use of the work  
Under the following conditions:  
    Attribution — You must attribute the work in the manner   
    specified by the author or licensor (but not in any way that  
    suggests that they endorse you or your use of the work).  
    Share Alike — If you alter, transform, or build upon this work,  
    you may distribute the resulting work only under the same or  
    similar license to this one.  




