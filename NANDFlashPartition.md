# NANDFlash Partition #
  * **系统环境: Redhat 9.0, Windows XP sp3**
  * **Date： 03/09/2010**
  * **本文仅介绍操作流程，省减了实现细节**

### 为何重分区 ###

博创实验箱bootloader的MTD分区表默认为：
```
mtdpart info. (6 partitions)
name              offset        size        flag
------------------------------------------------
vivi            : 0x00000000    0x00020000 未记录  128k
param           : 0x00020000    0x00010000 未记录   64k
kernel          : 0x00030000    0x00100000 未记录    1M
root            : 0x00130000    0x00300000 未记录    3M
user            : 0x00430000    0x03b00000 未记录   59M
ucos            : 0x03f30000    0x000cc000 未记录  816k
```

NANDFlash的BON分区表默认值同上。
  * 由上表可见，试验箱给kernel的存储容量为1M，而为模拟器编译出的Android内核为1.3M。由于为模拟器编译的Android内核未添加许多特性，从而实际使用时应将NANDFlash重新分区，扩大Kernel分区容量，以适应Android内核。本次分区其它目的有：熟悉bootloader的移植，了解Flash相关知识。

### 准备工作 ###

  * 搭建交叉开发环境。使用实验箱自带的交叉编译工具链，版本2.95.2；运行安装脚本，开发环境即可搭建完成。
  * 下载bootloader源码。为减少工作量，使用专门为s3c2410开发的bootloader-vivi，版本0.1.4。（试验箱自带的bootloader源码不知为啥编译烧写后，启动时超级终端无输出显示。重新下载了官方的源码）

### 修改编译vivi ###

  * 修改vivi根目录下的Makefile。为该文件的LINUX\_INCLUDE\_DIR、CROSS\_COMPILE和ARM\_GCC\_LIBS变量赋上相应的值。这三个变量依次代表：linux头文件目录，交叉编译器目录及其公共名称、arm的C库目录。这三个目录为本机交叉编译工具链自带，分别为：
```
LINUX_INCLUDE_DIR = /opt/host/armv4l/include/
CROSS_COMPILE = /opt/host/armv4l/bin/armv4l-unknown-linux-
ARM_GCC_LIBS = /opt/host/armv4l/lib/gcc-lib/armv4l-unknown-linux/2.95.2
```
  * 修改文件arch/s3c2410/smdk.c。
    1. 修改其中的mtd\_partition\_t default\_mtd\_partitions[.md](.md)结构体数组为期望的分区形式。
    1. 修改char linux\_cmd[.md](.md)字符数组；这是vivi传递给内核的参数，内核根据该参数定位根文件系统。
  * 配置编译vivi。使用make menuconfig进入配置界面，载入默认配置，然后保存并退出；输入make编译vivi，得到用于烧写的文件vivi（大小56.9k）。

### 分区的正确性验证 ###

  * **非专业验证，仅通过使用vivi引导内核并启动根文件系统来验证。**
  * 使用JTAG仿真器将编译好的vivi烧写至NANDFlash0x00000000地址处，用串口链接PC机和试验箱，打开超级终端并配置(115200bps, 8N1, no flow control)。打开试验箱在超级终端上有如下输出：
```
VIVI version 0.1.4 (root@BC) (gcc version 2.95.2 20000516 (release) [Rebel.com])
 #0.1.4 二 3月 9 18:36:09 CST 2010
MMU table base address = 0x33DFC000
Succeed memory mapping.
NAND device: Manufacture ID: 0xec, Chip ID: 0x76 (Samsung K9D1208V0M)
Could not found stored vivi parameters. Use default vivi parameters.
Press Return to start the LINUX now, any other key for vivi
type "help" for help.
vivi>
```
（以上输出在下载内核后获得）
  * 输入part show，可见vivi的MTD分区表如下：
```
vivi> part show
mtdpart info. (5 partitions)
name              offset        size        flag
------------------------------------------------
vivi            : 0x00000000    0x00020000     0  128k
param           : 0x00020000    0x00010000     0   64k
kernel          : 0x00030000    0x00400000     0    4M
root            : 0x00430000    0x00300000     4    3M
user            : 0x00730000    0x03000000     4   48M
```
  * 在刚刚烧写完成vivi后，为与arch/s3c2410/smdk.c中的内核参数配合，将NANDFlash的BON分区修改如下(使用bon part命令；与MTD分区的分配方式相同)
```
vivi> bon part info
BON info. (6 partitions)
No: offset      size            flags     bad
---------------------------------------------
 0: 0x00000000  0x00020000      00000000    0  128k
 1: 0x00020000  0x00010000      00000000    0   64k
 2: 0x00030000  0x00400000      00000000    0    4M
 3: 0x00430000  0x00300000      00000000    0    3M
 4: 0x00730000  0x03000000      00000000    0   48M
 5: 0x03730000  0x008cc000      00000000    0    8M+816k
```
由上表可见，有部分flash空间未分配。(目前只是初步分区，以后会根据各文件实际大小进行调整)
  * 分别烧写试验箱自带的编译好的linux内核和根文件系统至kernel和root分区(用loas flash命令)。输入boot，引导内核和根文件系统。超级终端上显示如下输出：
```
vivi> boot
Copy linux kernel from 0x00030000 to 0x30008000, size = 0x00400000 ... done
zImage magic = 0x016f2818
Setup linux parameters at 0x30000100
linux command line is: "noinitrd root=/dev/bon/3 init=/linuxrc console=ttyS0"
MACH_TYPE = 193
NOW, Booting Linux......
Uncompressing Linux.............................................................
. done, booting the kernel.
Linux version 2.4.18-rmk7-pxa1 (zxt@BC) (gcc version 2.95.2 20000516 (release) [
Rebel.com]) #251 Fri Sep 22 15:11:45 CST 2006
CPU: ARM/CIRRUS Arm920Tsid(wb) revision 0
Machine: Samsung-SMDK2410
On node 0 totalpages: 16384
zone(0): 16384 pages.
zone(1): 0 pages.
zone(2): 0 pages.
Kernel command line: noinitrd root=/dev/bon/3 init=/linuxrc console=ttyS0
DEBUG: timer count 15626
Console: colour dummy device 80x30
Calibrating delay loop... 99.94 BogoMIPS
Memory: 64MB = 64MB total
Memory: 62388KB available (1587K code, 430K data, 64K init)
Dentry-cache hash table entries: 8192 (order: 4, 65536 bytes)
Inode-cache hash table entries: 4096 (order: 3, 32768 bytes)
Mount-cache hash table entries: 1024 (order: 1, 8192 bytes)
Buffer-cache hash table entries: 4096 (order: 2, 16384 bytes)
Page-cache hash table entries: 16384 (order: 4, 65536 bytes)
POSIX conformance testing by UNIFIX
Linux NET4.0 for Linux 2.4
Based upon Swansea University Computer Society NET3.039
Initializing RT netlink socket
BlueZ Core ver 2.4 Copyright (C) 2000,2001 Qualcomm Inc
Written 2000,2001 by Maxim Krasnyansky <maxk@qualcomm.com>
CPU clock = 200.000 Mhz, HCLK = 100.000 Mhz, PCLK = 50.000 Mhz
Initializing S3C2410 buffer pool for DMA workaround
Starting kswapd
devfs: v1.10 (20020120) Richard Gooch (rgooch@atnf.csiro.au)
devfs: boot_options: 0x1
Samsung S3C2410X (i2c) algorithm module version 2.6.1 (20010830)
iic_s3c2410_init: Samsung S3C2410X iic adapter module version 2.6.1 (20010830)
i2c-dev.o: Registered 'Samsung S3C2410X IIC adapter' as minor 0
s3c2410_init: Initialized IIC on S3C2410X, 277kHz clock
iic_s3c2410_init: initialized iic-bus at 0xf4000000.
tts/%d0 at I/O 0x50000000 (irq = 52) is a S3C2410
tts/%d1 at I/O 0x50004000 (irq = 55) is a S3C2410
tts/%d2 at I/O 0x50008000 (irq = 58) is a S3C2410
Console: switching to colour frame buffer device 80x60
Installed S3C2410 frame buffer
pty: 256 Unix98 ptys configured
S3C2410 Real Time Clock Driver v0.1
block: 128 slots per queue, batch=32
ne.c:v1.10 9/23/94 Donald Becker (becker@scyld.com)
Last modified Nov 1, 2000 by Paul Gortmaker
NE*000 ethercard probe at 0xd1000200: 00 d0 cf 00 00 02
eth0: NE2000 found at 0xd1000200, using IRQ 2.
PPP generic driver version 2.4.1
SCSI subsystem driver Revision: 1.00
request_module[scsi_hostadapter]: Root fs not mounted
UDA1341 audio driver initialized
NAND device: Manufacture ID: 0xec, Chip ID: 0x76 (Samsung K9D1208V0M)
Creating 0 MTD partitions on "Samsung K9D1208V0M":
devfs_mk_dir(bon): using old entry in dir: c036e0a0 ""
bon0: 00000000-00020000 (00020000) 00000000
bon1: 00020000-00030000 (00010000) 00000000
bon2: 00030000-00430000 (00400000) 00000000
bon3: 00430000-00730000 (00300000) 00000000
bon4: 00730000-03730000 (03000000) 00000000
bon5: 03730000-03ffc000 (008cc000) 00000000
usb.c: registered new driver usbdevfs
usb.c: registered new driver hub
usb-ohci.c: USB OHCI at membase 0xe9000000, IRQ 26
usb.c: new USB bus registered, assigned bus number 1
hub.c: USB hub found
port #1 suspened!
port #0 alived!
hub.c: 1 port detected
hub.c: USB new device connect on bus1/1, assigned device number 2
usb.c: registered new driver usb_mouse
hub.c: USB hub found
hub.c: 4 ports detected
usbmouse.c: v1.6:USB HID Boot Protocol mouse driver
Initializing USB Mass Storage driver...
usb.c: registered new driver usb-storage
USB Mass Storage support registered.
mice: PS/2 mouse device common for all mice
NET4: Linux TCP/IP 1.0 for NET4.0
IP Protocols: ICMP, UDP, TCP, IGMP
IP: routing cache hash table of 512 buckets, 4Kbytes
TCP: Hash tables configured (established 4096 bind 4096)
NET4: Unix domain sockets 1.0/SMP for Linux NET4.0.
NetWinder Floating Point Emulator V0.95 (c) 1998-1999 Rebel.com
VFS: Mounted root (cramfs filesystem).
Mounted devfs on /dev
Freeing init memory: 64K
mount: Mounting /dev/mtdblock/1 on /mnt/yaffs failed: No such file or directory
mount: Mounting ramfs on /root failed: No such file or directory


BusyBox v1.00 (2005.01.20-11:59+0000) Built-in shell (ash)
Enter 'help' for a list of built-in commands.

runing /etc/profile ok
[/mnt/yaffs]cd /
```
启动引导成功，此时可以运行一些linux命令以测试。

  * 该版本的vivi不支持tftp，只能用串口下载数据，下载速度慢得惊人。
  * 还有其他它法以扩充分区容量，笔者采用了一种比较直观的方法。