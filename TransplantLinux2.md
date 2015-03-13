#以下配置供选择，不一定要做

# 详细配置（二） #

> 3)取消LCD 的初始化，不然启动时会有出错信息
```
static struct platform_device *smdk2410_devices[] __initdata = {
&s3c_device_usb,
// &s3c_device_lcd,
&s3c_device_wdt,
&s3c_device_i2c,
&s3c_device_iis,
};
```

> 4）修改网卡驱动的主要文件drivers/net/ne.c。


```
a、添加头文件和定义
添加以下头文件
#include <linux/irq.h>
#include <asm/arch-s3c2410/map.h>
#include <asm/arch-s3c2410/regs-mem.h>
#include <asm/arch-s3c2410/irqs.h>
#include <asm/arch-s3c2410/hardware.h>
#include <asm/arch-s3c2410/regs-gpio.h>
#define AX88796_BASE (vAX88796_BASE+0x200)
#define AX88796_IRQ IRQ_EINT2
#define pAX88796_BASE S3C2410_PA_ISA_NET
#define vAX88796_BASE S3C2410_VA_ISA_NET
#define EXTINT_OFF (IRQ_EINT4 - 4)

......
在static struct { const char *name8, *name16; unsigned char SAprefix[4];}
bad_clone_list[] __initdata 中增加AX88796 的MAC 地址前三位（不一定需要） :
{"AX88796", "NE2000-compatible", {0x08, 0x08, 0x08}},
```

```
b、确保定义总线宽度为16 位
将
#if defined(CONFIG_PLAT_MAPPI)
# define DCR_VAL 0x4b
#elif defined(CONFIG_PLAT_OAKS32R) || \
defined(CONFIG_TOSHIBA_RBTX4927) || defined(CONFIG_TOSHIBA_RBTX4938)
# define DCR_VAL 0x48 /* 8-bit mode */
#else
# define DCR_VAL 0x49
#endif
修改为
#if 0
#if defined(CONFIG_PLAT_MAPPI)
# define DCR_VAL 0x4b
#elif defined(CONFIG_PLAT_OAKS32R) || \
defined(CONFIG_TOSHIBA_RBTX4927) || defined(CONFIG_TOSHIBA_RBTX4938)
# define DCR_VAL 0x48 /* 8-bit mode */
#else
#endif
#endif
# define DCR_VAL 0x49
```

```
c、在do_ne_probe 函数中增加配置总线参数、基地址和中断的语句（其参数参考刘淼的
书）
static int __init do_ne_probe(struct net_device *dev)
{
unsigned long base_addr = dev->base_addr;
#ifdef NEEDS_PORTLIST
int orig_irq = dev->irq;
#endif

static int once=0;
if (once) {
return -ENXIO;
}
unsigned int value;
value = __raw_readl(S3C2410_BWSCON);
value &= ~(S3C2410_BWSCON_WS2|S3C2410_BWSCON_ST2|S3C2410_BWSCON_DW2_32);
value |= (S3C2410_BWSCON_ST2|S3C2410_BWSCON_DW2_16);
__raw_writel(value, S3C2410_BWSCON);
value=0;
value =
(S3C2410_BANKCON_Tacs4|S3C2410_BANKCON_Tcos4|S3C2410_BANKCON_Tacc14|S3C2410_BANKCON
_Tcoh4|S3C2410_BANKCON_Tcah4|S3C2410_BANKCON_Tacp6|S3C2410_BANKCON_PMCnorm);
__raw_writel(value,S3C2410_BANKCON2);
set_irq_type(AX88796_IRQ,IRQ_TYPE_LEVEL_LOW );
s3c2410_gpio_cfgpin(S3C2410_GPF2, S3C2410_GPF2_EINT2);
s3c2410_gpio_pullup(S3C2410_GPF2, 0);
if(base_addr==0){
dev->base_addr = base_addr = AX88796_BASE ;
dev->irq = AX88796_IRQ;
once++;
}

SET_MODULE_OWNER(dev);
/* First check any supplied i/o locations. User knows best. <cough> */
if (base_addr > 0x1ff) /* Check a single specified location. */
return ne_probe1(dev, base_addr);

else if (base_addr != 0) /* Don't probe at all. */
return -ENXIO;
```

```
d、修改ne_probe1 函数
增加自定义的网卡MAC 地址（这个地址可以自行修改，但是MAC 也有一定的规则，最重要
的是千万不要把它配置为广播或组播地址， 请参考网络的相关书籍）：
static int __init ne_probe1(struct net_device *dev, unsigned long ioaddr)
{
int i;
unsigned char ne_defethaddr[]={0x08,0x08,0x08,0x08,0x12,0x27,0};//tekkaman
unsigned char SA_prom[32];
int wordlength = 2;
......
增加网卡MAC 地址的配置语句，屏蔽通过EEPROM 配置网卡的语句
......
struct {unsigned char value, offset; } program_seq[] =
{
{E8390_NODMA+E8390_PAGE0+E8390_STOP, E8390_CMD}, /* Select page 0*/
{0x48, EN0_DCFG}, /* Set byte-wide (0x48) access. */
{0x00, EN0_RCNTLO}, /* Clear the count regs. */
{0x00, EN0_RCNTHI},
{0x00, EN0_IMR}, /* Mask completion irq. */
{0xFF, EN0_ISR},
{E8390_RXOFF, EN0_RXCR}, /* 0x20 Set to monitor */
{E8390_TXOFF, EN0_TXCR}, /* 0x02 and loopback mode. */
{32, EN0_RCNTLO},
{0x00, EN0_RCNTHI},
{0x00, EN0_RSARLO}, /* DMA starting at 0x0000. */
{0x00, EN0_RSARHI},
{E8390_RREAD+E8390_START, E8390_CMD},
};
for (i = 0; i < sizeof(program_seq)/sizeof(program_seq[0]); i++)
outb_p(program_seq[i].value, ioaddr + program_seq[i].offset);
}
{
unsigned char *ep;
ep = (unsigned char * ) &ne_defethaddr[0];
ne_defethaddr[5]++;
for(i=0;i<6;i++) {
SA_prom[i] = ep[i];
}
SA_prom[14] = SA_prom[15]=0x57;
wordlength =2;
}
#if 0 
for(i = 0; i < 32 /*sizeof(SA_prom)*/; i+=2) {
SA_prom[i] = inb(ioaddr + NE_DATAPORT);
SA_prom[i+1] = inb(ioaddr + NE_DATAPORT);
if (SA_prom[i] != SA_prom[i+1])
wordlength = 1;
if (wordlength == 2)
{
#if 0 
for (i = 0; i < 16; i++)
SA_prom[i] = SA_prom[i+i];
/* We must set the 8390 for word mode. */
outb_p(DCR_VAL, ioaddr + EN0_DCFG);
start_page = NESM_START_PG;
/*
* Realtek RTL8019AS datasheet says that the PSTOP register
* shouldn't exceed 0x60 in 8-bit mode.
* This chip can be identified by reading the signature from
* the remote byte count registers (otherwise write-only)...
*/
if ((DCR_VAL & 0x01) == 0 && /* 8-bit mode */
inb(ioaddr + EN0_RCNTLO) == 0x50 &&
inb(ioaddr + EN0_RCNTHI) == 0x70)
stop_page = 0x60;
else
stop_page = NESM_STOP_PG;
#endif 
outb_p(0x49, ioaddr + EN0_DCFG);
start_page = NESM_START_PG;
stop_page = NESM_STOP_PG;
} else {
start_page = NE1SM_START_PG;
stop_page = NE1SM_STOP_PG;
}
......
屏蔽自定检测中断号的语句(参考刘淼的书)：
......
#if 0 
if (dev->irq < 2)
{
unsigned long cookie = probe_irq_on();
outb_p(0x50, ioaddr + EN0_IMR); /* Enable one interrupt. */
outb_p(0x00, ioaddr + EN0_RCNTLO);
outb_p(0x00, ioaddr + EN0_RCNTHI);
outb_p(E8390_RREAD+E8390_START, ioaddr); /* Trigger it... */
mdelay(10); /* wait 10ms for interrupt to propagate */
outb_p(0x00, ioaddr + EN0_IMR); /* Mask it again. */
dev->irq = probe_irq_off(cookie);
if (ei_debug > 2)
printk(" autoirq is %d\n", dev->irq);
} else if (dev->irq == 2)
/* Fixup for users that don't know that IRQ 2 is really IRQ 9,
or don't know which one to set. */
dev->irq = 9;
#endif 
if (! dev->irq) {
printk(" failed to detect IRQ line.\n");
ret = -EAGAIN;
goto err_out;
}
AX88697 的移植到此结束！
```

7、配置内核
在配置内核前，先拷贝s3c2410 开发板的默认配置到内核根目录下，以简化配置过程

```

# cp arch/arm/conf igs/s3c2410_def config .config
# make menuconf ig
```

```
General setup --->
[*] Conf igure standard kernel features (for small systems) --->
选上这项，否则文件系统中的一些选项不会出现
System Type --->
S3C2410 Machines --->
[*] SMDK2410/A9M2410 留下这项就够了，其他全部“N”掉
“N”掉S3C2412 Machines ---> 、S3C2440 Machines ---> 和S3C2443
Machines ---> 里的所有选项，都是和2410 无关的选项。
Boot options --->
将(root=/dev/hda1 ro init=/bin/bash console=ttySAC0) Def ault kernel
command string
改成(noinitrd root=/dev/mtdblock4 rootfstype=cramf s console=ttySAC0,1152
00 init=/linuxrc mem=64M ) Def ault kernel command string
#说明：
#mtdblock4 代表第5 个flash 分区，用来作根文件系统rootfs；
# console=ttySAC0,115200 使kernel 启动期间的信息全部输出到串口0 上，波特率为
115200 ；
# 2.6 内核对于串口的命名改为ttySAC0，但这不影响用户空间的串口编程。
# 用户空间的串口编程针对的仍是/dev/ttyS0 等
# mem=64M 表示内存是64M，如果是32 则设为32M
Userspace binary formats --->
< > Kernel support for a.out and ECOFF binaries （去除该选项， a.out 和ECOFF
是两种可执行文件的格式，在ARM－Linux 下一般都用ELF，所以这两种基本用不上。）
Networking --->
Networking options --->
<*> Packet socket
[*] Packet socket: mmapped IO
－下面可以不选－
Wireless --->
--- Improved wireless configuration API
--- Wireless extensions
<*> Generic IEEE 802.11 Networking Stack (mac80211)
[*] Enable LED triggers
[ ] Enable debugging output (NEW)
<*> Generic IEEE 802.11 Networking Stack
[ ] Enable full debugging output (NEW)
--- IEEE 802.11 WEP encryption (802.1x)
< > IEEE 802.11i CCMP support (NEW)
< > IEEE 802.11i TKIP encryption (NEW)
<*> Software MAC add-on to the IEEE 802.11 networking stack
[ ] Enable full debugging output (NEW)
Device Drivers --->
“N”掉Parallel port support ---> 里的所有选项。
Plug and Play support --->里的所有选项一定要“N”掉，不然编译会出错！ !!!!!!!
Network device support --->
Ethernet (10 or 100Mbit) --->
“N”掉< > DM9000 support 和< > Generic Media Independent Interf ace
device support
[*] Other ISA cards
<*> NE2000/NE1000 support
“N”掉[ ] Ethernet (1000 Mbit) --->和[ ] Ethernet (10000 Mbit) --->
Wireless LAN --->
[*] Wireless LAN (pre-802.11)
[*] Wireless LAN (IEEE 802.11)
USB Network Adapters --->
<*> Multi-purpose USB Networking Framework
<*> MMC/SD card support --->
Real Time Clock --->
“N”掉[ ] Set system time from RTC on startup and resume

接下来做的是针对文件系统的设置，我实验时目标箱上要挂的根文件系统是cramf s,故做如下
配置
File systems -->
< > Second extended fs support #去除对ext2 的支持
< > Ext3 journalling file system support #去除对ext3 的支持
<*> Kernel automounter support
<*> Kernel automounter version 4 support (also supports v3)
<*> Filesystem in Userspace support
Pseudo filesystems -->
[*] Virtual memory file system support (former shm fs)
<*> Userspace-driven configuration filesystem (EXPERIMENTAL)
Miscellaneous filesystems -->
<*> YAFFS2 file system support
“N”掉[ ]Autoselect yaffs2 format 和
[ ]Cache short names in RAM ，因为这是给每页大于1024B 的NAND Flash 设计的
<*> Journalling Flash File System v2 (JFFS2) support
(0) JFFS2 debugging verbosity (0 = quiet, 2 = noisy)
[*] JFFS2 write-buffering support
[ ] JFFS2 summary support (EXPERIMENTAL)
[ ] JFFS2 XATTR support (EXPERIMENTAL)
[*] Advanced compression options for JFFS2
[*] JFFS2 ZLIB compression support
[*] JFFS2 RTIME compression support
[*] JFFS2 RUBIN compression support
JFFS2 default compression mode (priority) --->
Network File Systems -->
<*> NFS file system support
--以下最好选上，因为在挂载NFS 时可能出现protocol 不支持的情况--
[*]Provide NFSv3 client support
[*]Provide client support for the NFSv3 ACL protocol extension
[*] Provide NFSv4 client support (EXPERIMENTAL)
[*] Allow direct I/O on NFS files
<*> NFS server support
[*] Provide NFSv3 server support
[*]Provide server support for the NFSv3 ACL protocol extension
[*] Provide NFSv4 server support (EXPERIMENTAL)
--- Provide NFS server over TCP support
[*] Root file system on NFS
保存退出，产生.config 文件。
```
8、编译内核
```
#make zImage
成功编译成zImage,将它烧写到S3C2410，成功启动，移植成功。
```
# Details #

Add your content here.  Format your content with:
  * Text in **bold** or _italic_
  * Headings, paragraphs, and lists
  * Automatic links to other wiki pages