# Introduction #

platfrom\_device  fb\_info   s3c2410fb\_info   s3c2410fb\_mach\_info 之间是怎么关联的.
register\_framebuffer()是如何实现注册的.

# Details #

  * platform\_device   fb\_info   s3c2410fb\_info  s3c2410fb\_mach\_info 之间的关系

```
       platform_device  ---------------- s3c2410fb_mach_info 
    
        mach_info= pdev->dev.platform_data  

       其中 mach_info 为s3c2410fb_info类型的指针   
        其实mach_info中存放的是LCD的初始化时所用的值
        pdev 为结构体platform_device类 型的指针
        还记得我们在驱动移植步骤中的第二步就是填充结构体 mach_info 
       然后 调用  s3c24xx_fb_set_platdata(&qt2410_biglcd_cfg); 就是将 platform_device同mach_info联系起来
```

```
         fb_info---------------------s3c2410fb 
           
            info = fbinfo->par 
            info->fb = fbinfo
       其中 info 为 结构体s3c2410fb_info类型的指针
             fbinfo为结构体fb_info类型的指针
```

```
          platform_device -------------- fb_info
          
         pdev->dev.driver_data ＝ fbinfo  
       其中 pdev为 结构体platform_device类型的指针
             fbinfo为结构体 fb_info 类型的指针
```

```
        platform_device  ------------ platform_driver
        这个在前面介绍着两个结构体的时候都说过了 是通过 name属性相互关联起来的
```
> 这样所有的主要结构体都串起来了
  * 总结一下 如下图 ：
```
     结构体platform_driver

              |               
              | 
              | 通过name 相互关联 
              | 
              | 
              | 
   结构体 platform_device {}                  结构体 s3c2410fb_mach_info     |                                                        |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |       上面两个通过dev. platform_data关联  |
              |--------------- 结构体device---------------|        
 dev.data将上下|                                          |
 两个结构体关联 |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |                                          |
              |       你中有我  我中有你                      |
    结构体 fb_info---------------------------- 结构体 s3c2410fb
```
  * 分析 register\_framebuffer()函数的实现
> > ![http://www.dzjs.net/upimg/userup/0907/01102110Y24.jpg](http://www.dzjs.net/upimg/userup/0907/01102110Y24.jpg)
> > fbmem.c实现了Linux FrameBuffer的中间层，任何一个FrameBuffer驱动，在系统初始化时，必须向fbmem.c注册，即需要调用register\_framebuffer()函数，在这个过程中，设备驱动的信息将会存放入名称为registered\_fb数组中  这个数组定义为struct fb\_info\*registered\_fb[FB\_MAX](FB_MAX.md)、int num\_registered\_fb;它是类型为fb\_info的数组，另外num\_register\_fb则存放了注册过的设备数量
代码如下：
```
register_framebuffer(struct fb_info *fb_info)
{
	int i;
	struct fb_event event;
	struct fb_videomode mode;

	if (num_registered_fb == FB_MAX)
		return -ENXIO;
        //看数组时否已经满了  是否超过最大注册数量
	num_registered_fb++;
	for (i = 0 ; i < FB_MAX; i++)
		if (!registered_fb[i])   //找到空余的空间
			break;
	fb_info->node = i;
        	fb_info->dev = device_create(fb_class, fb_info->device,
				     MKDEV(FB_MAJOR, i), "fb%d", i);
        ////建立的设备节点的次设备号就是该驱动信息在registered_fb存放的位置，即数组下标i 
	if (IS_ERR(fb_info->dev)) {
		/* Not fatal */
		printk(KERN_WARNING "Unable to create device for framebuffer %d; errno = %ld\n", i, PTR_ERR(fb_info->dev));
		fb_info->dev = NULL;
	} else
		fb_init_device(fb_info);
             //初始化设备
	if (fb_info->pixmap.addr == NULL) {
		fb_info->pixmap.addr = kmalloc(FBPIXMAPSIZE, GFP_KERNEL);
		if (fb_info->pixmap.addr) {
			fb_info->pixmap.size = FBPIXMAPSIZE;
			fb_info->pixmap.buf_align = 1;
			fb_info->pixmap.scan_align = 1;
			fb_info->pixmap.access_align = 32;
			fb_info->pixmap.flags = FB_PIXMAP_DEFAULT;
		}
	}	
	fb_info->pixmap.offset = 0;

	if (!fb_info->pixmap.blit_x)
		fb_info->pixmap.blit_x = ~(u32)0;

	if (!fb_info->pixmap.blit_y)
		fb_info->pixmap.blit_y = ~(u32)0;

	if (!fb_info->modelist.prev || !fb_info->modelist.next)
		INIT_LIST_HEAD(&fb_info->modelist);

	fb_var_to_videomode(&mode, &fb_info->var);
	fb_add_videomode(&mode, &fb_info->modelist);
	registered_fb[i] = fb_info;

	event.info = fb_info;
	fb_notifier_call_chain(FB_EVENT_FB_REGISTERED, &event);
	return 0;
}
```
当FrameBuffer驱动进行注册的时候，它将驱动的fb\_info结构体记录到全局数组registered\_fb中，并动态建立设备节点，进行设备的初始化。