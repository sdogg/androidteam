# Android Kernel Porting #

  * **系统环境: Ubuntu 9.10, Kernel 2.6.31-14-generic, i686**
  * **Android Kernel Version: 2.6.25**
  * **Date：3/20/2010**

### 准备工作 ###

  * 在[code.google.com](http://code.google.com/p/android/downloads/list?can=1&q=)下载Android内核，版本2.6.25-1.0-r1；
  * 解压下载的linux-2.6.25-android-1.0\_r1.tar.gz，修改内核源码根目录kernel.git下的Makefile，具体为：
```
ARCH ?= arm
CROSS_COMPILE ?= /home/yang/yangdroid/prebuilt/linux-x86/toolchain/arm-eabi-4.4.0/bin/arm-eabi-
```
  * 在arch/arm/plat-s3c24xx/common-smdk.c中添加NAND Flash分区信息和NAND Flash硬件信息，分区信息与之前vivi分区表/bon分区表一致，具体为：
```
static struct mtd_partition smdk_default_nand_part[] = {
	[0] = {
		.name	= "vivi",
		.size	= 0x00020000,
		.offset	= 0,
	},
	[1] = {
		.name	= "param",
		.offset = 0x00020000,
		.size	= 0x00010000,
	},
	[2] = {
		.name	= "kernel",
		.offset = 0x00030000,
		.size	= 0x00400000,
	},
	[3] = {
		.name	= "root",
		.offset	= 0x00430000,
		.size	= 0x00300000,
	},
	[4] = {
		.name	= "user",
		.offset = 0x00730000,
		.size	= 0x03800000,
	},
	[5] = {
		.name	= "ucos",
		.offset	= 0x03f30000,
		.size	= 0x000cc000,
	}
};

static struct s3c2410_platform_nand smdk_nand_info = {
	.tacls		= 0,
	.twrph0		= 30,
	.twrph1		= 0,
	.nr_sets	= ARRAY_SIZE(smdk_nand_sets),
	.sets		= smdk_nand_sets,
};
```
  * 修改drivers/mtd/nand/s3c2410.c，搜索NAND\_ECC\_SOFT替换为NAND\_ECC\_NONE，以禁止内核对NAND Flash的ECC校验

### 配置编译内核 ###

  * 将s3c2410的默认配置文件拷贝至内核根目录
```
cp arch/arm/configs/s3c2410_defconfig .config
```
  * make menuconfig配置内核
```
N掉Userspace binary formats ---> < > Kernel support for a.out and ECOFF binaries，否则编译出错
N掉Device Drivers ---> Parallel port support ---> 里的所有选项，否则内核运行将出错：kernel panic not syncing Attempted to kill init
```
其它根据[这篇博文](http://blog.chinaunix.net/u1/34474/showart_384863.html)配置
  * 配置内核，添加Android特性，选上如下选项
```
Kernel Features  ---> [*] Use the ARM EABI to compile the kernel
General setup  ---> [*] Use full shmem filesystem
General setup  ---> [*] Enable Android's Shared Memory Subsystem
System Type  ---> [*] Support Thumb user binaries
Device Drivers  ---> Android  ---> [*] Android log driver
Device Drivers  ---> Android  ---> <*> Binder IPC Driver

Device Drivers  ---> Android  ---> [*] RAM buffer console
Device Drivers  ---> Android  ---> [*] Android timed gpio driver
Device Drivers  ---> Android  ---> [*] Only allow certain groups to create sockets
```
  * 保存退出。输入make编译内核

### 内核引导 ###

  * 将编译后得到的arch/arm/boot/zImage利用vivi烧写到kernel分区
  * 将匆忙制作的根文件系统烧写至root分区
  * 使用vivi将linux内核的命令行参数设置为：
```
"noinitrd root=/dev/mtdblock3 rootfstype=cramfs console=ttySAC0,115200 init=/linuxrc mem=64M"
```
  * vivi> boot，出现如下引导信息：
```
vivi> boot          

Copy linux kernel from 0x00030000 to 0x30008000, size = 0x00400000 ... done                                                                           

zImage magic = 0x016f2818                         

Setup linux parameters at 0x30000100                                    

linux command line is: "noinitrd root=/dev/mtdblock3 rootfstype=cramfs console=ttySAC0,115200 init=/linuxrc mem=64M"                                    

MACH_TYPE = 193               

NOW, Booting Linux......                        

Uncompressing Linux.............................................................                                                                                

............................................... done, booting the kernel.                                                                         

Linux version 2.6.25 (yang@TheFallenAngel) (gcc version 4.4.0 (GCC) ) #1 Tue Mar                                                                                

 16 18:22:37 CST 2010                     

CPU: ARM920T [41129200] revision 0 (ARMv4T), cr=c0007177                                                        

Machine: SMDK2410                 

ATAG_INITRD is deprecated; please update your bootloader.                                                         

Memory policy: ECC disabled, Data cache writeback                                                 

CPU S3C2410 (id 0x32410000)                           

S3C2410: core 200.000 MHz, memory 100.000 MHz, peripheral 50.000 MHz                                                                    

S3C24XX Clocks, (c) 2004 Simtec Electronics                                           

CLOCK: Slow mode (1.500 MHz), fast, MPLL on, UPLL on                                                    

CPU0: D VIVT write-back cache                             

CPU0: I cache: 16384 bytes, associativity 64, 32 byte lines, 8 sets                                                                   

CPU0: D cache: 16384 bytes, associativity 64, 32 byte lines, 8 sets                                                                   

Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 16256                                                                          

Kernel command line: noinitrd root=/dev/mtdblock3 rootfstype=cramfs console=ttySAC0,115200 init=/linuxrc mem=64M                                

irq: clearing pending ext status 00000100                                         

irq: clearing subpending status 00000003                                        

irq: clearing subpending s                        

PID hash table entries: 256 (order: 8, 1024 bytes)                                                  

timer tcon=00000000, tcnt a2c1, tcfg 00000200,00000000, usec 00001eb8                                                                     

Console: colour dummy device 80x30                                  

console [ttySAC0] enabled                         

Dentry cache hash table entries: 8192 (order: 3, 32768 bytes)                                                             

Inode-cache hash table entries: 4096 (order: 2, 16384 bytes)                                                            

Memory: 64MB = 64MB total                         

Memory: 61184KB available (3092K code, 523K data, 128K init)                                                            

Mount-cache hash table entries: 512                                   

CPU: Testing write buffer coherency: ok                                       

net_namespace: 152 bytes                        

NET: Registered protocol                        

S3C2410 Power Management, (c) 2004 Simtec Electronics                                                     

S3C2410: Initialising architecture                                  

S3C24XX DMA Driver, (c) 2003-2004,2006 Simtec Electronics                                                         

DMA channel 0 at c4800000, irq 33                                 

DMA channel 1 at c4800040, irq 34                                 

DMA channel 2 at c4800080, irq 35                                 

DMA channel 3 at c48000c0, irq 36                                 

NET: Registered protocol family 2                                 

IP route cache hash table entries: 1024 (order: 0, 4096 bytes)                                                              

TCP established hash table entries: 2048 (order: 2, 16384 bytes)                                                                

TCP bind hash table entries: 2048 (order: 1, 8192 bytes)                                                        

TCP: Hash tables configured (established 2048 bind 2048)                                                        

TCP reno registered                   

ashmem: initialized                   

Installing knfsd (copyright (C) 1996 okir@monad.swb.de).                                                        

JFFS2 version 2.2. (NAND) 漏 2001-2006 Red Hat, Inc.                                                    

fuse init (API version 7.9)                           

yaffs Mar 16 2010 18:17:34 Installing.                                      

io scheduler noop registered                            

io scheduler anticipatory registered (default)                                              

io scheduler deadline registered                                

io scheduler cfq registered                           

s3c2410-lcd s3c2410-lcd: no platform data for lcd, cannot attach                                                                

s3c2410-lcd: probe of s3c2410-lcd failed w                                         

lp: driver loaded but no devices found                                      

ppdev: user-space parallel port driver                                      

Serial: 8250/16550 driver $Revision: 1.90 $ 4 ports, IRQ sharing enabled                                                                        

s3c2410-uart.0: s3c2410_serial0 at MMIO 0x50000000 (irq = 70) is a S3C2410                                                                          

s3c2410-uart.1: s3c2410_serial1 at MMIO 0x50004000 (irq = 73) is a S3C2410                                                                          

s3c2410-uart.2: s3c2410_serial2 at MMIO 0x50008000 (irq = 76) is a S3C2410                                                                          

brd: module loaded                  

loop: module loaded                   

Uniform Multi-Platform E-IDE driver                                   

ide: Assuming 50MHz system bus speed for PIO modes; override with                                                                 

S3C24XX NAND Driver, (c) 2004 Simtec Electronics                                                

s3c2410-nand s3c2410-nand: Tacls=1, 10ns Twrph0=4 40ns, Twrph1=1 10ns                                                                     

NAND device: Manufacturer ID: 0xec, Chip ID: 0x76 (Samsung NAND 64MiB 3,3V 8-bit) 

NAND_ECC_NONE selected by board driver. This is not recommended !!                                                                  

Scanning device for bad blocks                              

Creating 6 MTD partitions on "NAND 64MiB 3,3V 8-bit":                           

0x00000000-0x00020000 : "vivi"

0x00020000-0x00030000 : "param"

0x00030000-0x00430000 : "kernel"

0x00430000-0x00730000 : "root"

0x00730000-0x03f30000 : "user"

0x03f30000-0x03ffc000 : "ucos"

mice: PS/2 mouse device common for all mice

S3C24XX RTC, (c) 2004,2006 Simtec Electronics

s3c2410-i2c s3c2410-i2c: slave address 0x10

s3c2410-i2c s3c2410-i2c: bus frequency set to 390 KHz

s3c2410-i2c s3c2410-i2c: i2c-0: S3C I2C adapter

S3C2410 Watchdog Timer, (c) 2004 Simtec Electronics

s3c2410-wdt s3c2410-wdt: watchdog inactive, reset disabled, irq enabled

logger: created 64K log 'log_main'

logger: created 64K log 'log_events'

logger: created 64K log 'log_radio'

TCP cubic registered

NET: Registered protocol family 1

NET: Registered protocol family 17

RPC: Registered udp transport module.

RPC: Registered tcp transport module.

VFS: Mounted root (cramfs filesystem) readonly.

Freeing init memory: 128K
```

  * 由以上引导信息可见vivi已经成功引导android内核并且挂载上了根文件系统，但是根文件系统无法初始化，无法登录用户和运行shell，可能初始化脚本出了问题。