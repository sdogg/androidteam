# 移植Android系统之LCD驱动移植 #

博创s3c2410的LCD使用的是SHARP LQ080V3DG01，我们使用的Android内核版本是linux-2.6.25-android-1.0\_r1.tar.gz，要移植LCD的驱动，需要在内核源码arch/arm/mach-s3c2410/mach-smdk2410.c里添加初始化s3c2410的LCD控制器时所需要的参数。而这些参数可以参考arch/arm/mach-s3c2410/mach-qt2410.c文件中对SHARP LQ080V3DG01型LCD控制器初始化的相关代码。

## 移植步骤 ##
1．修改arch/arm/mach-s3c2410/mach-smdk2410.c文件

1）添加头文件
```
#include <asm/arch/fb.h>
```
2）添加初始化s3c2410的LCD控制器时所需的参数
```
参考linux2.6.25内核arch/arm/mach-s3c2410/mach-qt2410.c文件中对SHARP LQ080V3DG01型LCD控制器初始化的相关代码，添加如下代码：
/*Configuration for 640x480 SHARP LQ080V3DG01 */

static struct s3c2410fb_display qt2410_lcd_cfg __initdata = {

/* lcd configuration registers */

.lcdcon5 = S3C2410_LCDCON5_FRM565 |

S3C2410_LCDCON5_INVVLINE |

S3C2410_LCDCON5_INVVFRAME |

S3C2410_LCDCON5_PWREN |

S3C2410_LCDCON5_HWSWP,

/* LCD type */

.type = S3C2410_LCDCON1_TFT,

/* Screen size */

.width = 640,

.height = 480,

/* Screen info */                       

.xres = 640,

.yres = 480,

.bpp = 16,

.pixclock = 40000, 

.left_margin = 44,                            

.right_margin = 116,                            

.hsync_len = 96,                               

.upper_margin = 19,                            

.lower_margin = 11,                            

.vsync_len = 15,                              

};

//LCD driver info

static struct s3c2410fb_mach_info qt2410_lcd_info __initdata = {

.displays = &qt2410_lcd_cfg,     /* attached diplays info */

.num_displays = 1,            /* number of defined displays */                

.default_display = 0,                         

/* GPIOs */

.gpccon = 0xaa8002a8,

.gpccon_mask = 0xffc003fc,

.gpcup = 0xf81e,

.gpcup_mask = 0xf81e,

.gpdcon = 0xaa80aaa0,

.gpdcon_mask = 0xffc0fff0,

.gpdup = 0xf8fc,

.gpdup_mask = 0xf8fc,

/* lpc3600 control register */

.lpcsel = ((0xCE6) & ~7) | 1<<4,

};
```
3）添加LCD控制器的寄存器参数设置函数
```
查找smdk2410_init，添加如下代码：
static void __init smdk2410_init(void)
{
	platform_add_devices(smdk2410_devices, ARRAY_SIZE(smdk2410_devices));
	s3c24xx_fb_set_platdata(&qt2410_lcd_info);//设置LCD控制器的寄存器参数 
	smdk_machine_init();
}
```

2.配置内核
```
输入make menuconfig
添加如下选项：
Device Drivers  --->  

   Graphics support  --->    

      Display device support  ---> 

         <*> Display panel/monitor support 

      <*> Support for frame buffer devices 

         <*> S3C2410 LCD framebuffer support  

      Console display driver support  --->               

         <*> Framebuffer Console support             

         [*]   Framebuffer Console Rotation        

         [*] Select compiled-in fonts               

         [*]   VGA 8x8 font                    

         [*]   VGA 8x16 font                   

         [*]   Mini 4x6 font 

         [*] Sparc console 8x16 font           

      [*] Bootup logo  --->                       

         --- Bootup logo

         [*]   Standard 224-color Linux logo   
```
以上都是必须选择的，其他相关选项也可以选择

重新编译，生成zImage映像

## LCD测试 ##
重新烧写内核映像后，用NFS服务挂载Android的根文件系统，启动后首先发现LCD屏幕上显示了Linux的企鹅logo，之后当挂载根文件系统时，在屏幕上会显示字符提示：
```

                                  A N D R O I D_

```
说明开始了Android系统的引导程序，之后会出现Android的滚动条，大概等待三四分钟就可以进入到Android的系统，可以显示Android的系统界面。

## 遗留问题 ##
目前进入系统会显示No Service，而且没有键盘和触摸屏驱动，无法对Android的各种功能进行操作，minicom上会提示一些log信息：
```
#binder:release 1571:1576 transaction 1212 in, still active
binder:send failed reply for transaction 1212 to 1858:1858
init :untracked pid 1849 exited
init :untracked pid 1839 exited
init :untracked pid 1812 exited
init :untracked pid 1828 exited
init :untracked pid 1858 exited
init :untracked pid 1785 exited
init :untracked pid 1792 exited
binder:1505:1918 transaction failed 29189,size 168-0
过一段时间没有操作的话，系统会进入挂起状态，然后再自动加载
#android_power_suspend:503627634000
android sleep state 0->2 at 503627634000
active wake lock PowerManagerService
之后的提示信息：
binder: 1505:1519 transaction failed 29189, size 168-0
rild invoked oom-killer: gfp_mask=0x1201d2, order=0, oomkilladj=-16
[<c002d348>] (dump_stack+0x0/0x14) from [<c006b088>]
(oom_kill_process+0x5c/0x198)
[<c006b02c>] (oom_kill_process+0x0/0x198) from [<c006b5a0>]
(out_of_memory+0x194/0x1e0)
[<c006b40c>] (out_of_memory+0x0/0x1e0) from [<c006d7c8>]
(__alloc_pages+0x26c/0x2f0)
[<c006d55c>] (__alloc_pages+0x0/0x2f0) from [<c006a7b8>]
(filemap_fault+0x284/0x3bc)
[<c006a534>] (filemap_fault+0x0/0x3bc) from [<c00752b8>]
(__do_fault+0x74/0x400)[<c0075244>] (__do_fault+0x0/0x400) from [<c0076230>]
(handle_mm_fault+0x2d8/0x608)
[<c0075f58>] (handle_mm_fault+0x0/0x608) from [<c002f3b4>]
(do_page_fault+0xe4/0x230)
[<c002f2d0>] (do_page_fault+0x0/0x230) from [<c002f5a0>]
(do_translation_fault+0x18/0x80)
[<c002f588>] (do_translation_fault+0x0/0x80) from [<c00281e8>]
(do_PrefetchAbort+0x18/0x1c)
 r5:b0015b04 r4:ffffffff
[<c00281d0>] (do_PrefetchAbort+0x0/0x1c) from [<c0028960>]
(ret_from_exception+0x0/0x10)
Exception stack(0xc09b7fb0 to 0xc09b7ff8)
7fa0:                                     afe075cb a9c01f17 afe075cb 0000005f
7fc0: 00000208 b0015b04 afe041dc afe0215c afe0636c 0ec0047a a9c01f17 b00162ac
7fe0: b0009999 beb80bec b0000dc8 b00099a6 80000030 ffffffff
Mem-info:
DMA per-cpu:
CPU    0: hi:   18, btch:   3 usd:  12
Active:9992 inactive:2577 dirty:0 writeback:0 unstable:0
 free:339 slab:873 mapped:11 pagetables:727 bounce:0
DMA free:1356kB min:1016kB low:1268kB high:1524kB active:39968kB
inactive:10308kB present:65024kB pages_scanned:81557 all_unreclaimable? yes
lowmem_reserve[]: 0 0 0
DMA: 117*4kB 7*8kB 0*16kB 0*32kB 1*64kB 0*128kB 1*256kB 1*512kB 0*1024kB
0*2048kB 0*4096kB = 1356kB
5334 total pagecache pages
Swap cache: add 0, delete 0, find 0/0
Free swap  = 0kB
Total swap = 0kB
Free swap:            0kB
16384 pages of RAM
557 free pages
1265 reserved pages
873 slab pages
13340 pages shared
0 pages swap cached
Out of memory: kill process 2123 (app_process) score 793444352 or a child
Killed process 2123 (app_process)
```
看来Android的相关服务启动还有一定的问题，需要进一步进行分析修改。