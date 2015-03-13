杨绍文的页面 [返回Team96](http://code.google.com/p/androidteam/wiki/Team96)
## 学习资料 ##

  * [引导加载程序vivi 的分析和移植研究](http://hi.baidu.com/zkheartboy/blog/item/7699404ef831520bb2de055d.html)
  * [linux下编译和烧写vivi以及kernel的操作步骤](http://hi.baidu.com/zkheartboy/blog/item/309d59c29e720137e5dd3b84.html)
  * [嵌入式技术教程](http://idc81.net/qrspx/jiaoshi/class/70.html)
  * [移植根文件系统](http://hi.baidu.com/zjsxycli/blog/item/ee0e682181a152589922ed6a.html)
  * [使用SkyEye仿真](http://hi.baidu.com/zkheartboy/blog/item/26dd17e989a33439b90e2d12.html)
  * [Android移植 开发](http://www.android123.com.cn/androidyizhi/)
# linux-2.6.25-android 内核驱动 #
## USB驱动过程 ##

  * 在arch/arm/mach-s3c2410/mach-smdk2410.c中添加:
```
//usb

#include <asm/arch/usb-control.h>

#include <asm/arch/regs-clock.h>

#include <linux/device.h>

#include <linux/delay.h>
//-------------------------------------------------usb

struct s3c2410_hcd_info usb_s3c2410_info = {

              .port[0] = {

                     .flags = S3C_HCDFLG_USED

              }

};

int smdk2410_usb_init(void) /* USB */

{
              unsigned long upllvalue = (0x78<<12)|(0x02<<4)|(0x03);

              printk("USB Control, (c) 2006 s3c2410\n");

              s3c_device_usb.dev.platform_data = &usb_s3c2410_info;

              while(upllvalue!=__raw_readl(S3C2410_UPLLCON))

              { __raw_writel(upllvalue,S3C2410_UPLLCON);

                     mdelay(1);

             }

              return 0;
}
```
  * 在smdk2410\_map\_io函数最后添加:
```
smdk2410_usb_init();
```
  * 编译内核，选中所装驱动,配置USB鼠标键盘:
```
#make menuconfig
Device Drivers >

       USB support  --->  
            <*> Support for Host-side USB

            <*> OHCI HCD support

--- USB Input Devices

    <*> USB Human Interface Devices (full HID) support

    [*] HID input layer support

```



## 触摸屏驱动过程 ##
  * 首先打一个补丁：s3c2410\_touchscreen.patch（或手动修改补丁中显示相关文件的代码）
> > [s3c2410\_touchscreen.patch代码](http://code.google.com/p/androidteam/wiki/s3c2410_ts_patch_code?ts=1270512750&updated=s3c2410_ts_patch_code)
  * 修改`arch/arm/mach-s3c2410/mach-smdk2410.c`添加如下代码：
> > 添加头文件：
```
   #include <asm/arch/ts.h> 
```
> > 添加结构体：
```
   static struct s3c2410_ts_mach_info smdk2410_ts_cfg __initdata = { 
        .delay = 20000, 
        .presc = 49, 
        .oversampling_shift = 2, 
   }; 
```
> > 修改driver/input/touchscreen/Makefile,添加如下内容：
```
obj-$(CONFIG_TOUCHSCREEN_S3C2410) += s3c2410_ts.o
```
> > 添加支持触摸屏平台代码的信息，这个找到一个名为＊smdk2410\_devices[.md](.md)的结构体指针数组里添加：&s3c\_device\_ts,然后在smdk2410\_map\_io函数里添加：
```
set_s3c2410ts_info(&smdk2410_ts_cfg); 
```

  * 在arch/arm/mach-s3c2410/include/mach/下新建ts.h文件
> > ts.h文件内容如下：
```
#ifndef __ASM_ARM_S3C2410_TS_H 
#define __ASM_ARM_S3C2410_TS_H 
struct s3c2410_ts_mach_info {      
     int delay; 
     int presc;        
     int oversampling_shift; 
}; 
void set_s3c2410ts_info(struct s3c2410_ts_mach_info *hard_s3c2410ts_info); 


```

  * 在driver/input/touchscreen/下面创建驱动文件[s3c2410\_ts.c](http://code.google.com/p/androidteam/wiki/s3c2410_ts_code?ts=1270512980&updated=s3c2410_ts_code)
  * 配置内核支持触摸屏驱动make menuconfig 配置支持触摸平驱动
```
Device Drivers  ---> 
    Input device support  --->     
          [*]   Touchscreens  --->                                     
                <*>   Samsung S3C2410 touchscreen input driver                                           
                [*]     Samsung S3C2410 touchscreen debug messages 
```
  * 然后make zImage。系统启动以后会在dev目录下产生event0 和 mouse0两个设备节点，它们就是触摸屏的设备节点。






## 参考链接 ##
  * [Embedded技术相关](http://hi.baidu.com/zkheartboy/blog/category/Embedded)
  * [嵌入式linux](http://hi.baidu.com/zjsxycli/blog/category/%C7%B6%C8%EB%CA%BDlinux)