# Introduction #
```

s3c2410触摸屏驱动(2.6内核)分析 

#include <linux/config.h>
#include <linux/errno.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/slab.h>
#include <linux/input.h>
#include <linux/init.h>
#include <linux/serio.h>
#include <linux/delay.h>
#include <asm/io.h>
#include <asm/irq.h>

#include <asm/arch/regs-adc.h>
#include <asm/arch/regs-gpio.h>
#include <asm/arch/s3c2410-ts.h>
#include <asm/hardware/clock.h>

/* For ts.dev.id.version */
#define S3C2410TSVERSION    0x0101

#define WAIT4INT(x) (((x)<<8) | S3C2410_ADCTSC_YM_SEN | S3C2410_ADCTSC_YP_SEN | S3C2410_ADCTSC_XP_SEN | S3C2410_ADCTSC_XY_PST(3))

#define AUTOPST (S3C2410_ADCTSC_YM_SEN | S3C2410_ADCTSC_YP_SEN | S3C2410_ADCTSC_XP_SEN | S3C2410_ADCTSC_AUTO_PST | S3C2410_ADCTSC_XY_PST(0))

#define DEBUG_LVL    KERN_DEBUG

MODULE_AUTHOR("Arnaud Patard <arnaud.patard@rtp-net.org>");
MODULE_DESCRIPTION("s3c2410 touchscreen driver");
MODULE_LICENSE("GPL");

/*
 * Definitions & global arrays.
 */


static char *s3c2410ts_name = "s3c2410 TouchScreen";

/*
 * Per-touchscreen data.
 */

struct s3c2410ts {
    struct input_dev dev;
    long xp;
    long yp;
    int count;
    int shift;
    char phys[32];
};

static struct s3c2410ts ts;
static void __iomem *base_addr;

static inline void s3c2410_ts_connect(void)
{
    s3c2410_gpio_cfgpin(S3C2410_GPG12, S3C2410_GPG12_XMON);
    s3c2410_gpio_cfgpin(S3C2410_GPG13, S3C2410_GPG13_nXPON);
    s3c2410_gpio_cfgpin(S3C2410_GPG14, S3C2410_GPG14_YMON);
    s3c2410_gpio_cfgpin(S3C2410_GPG15, S3C2410_GPG15_nYPON);
}

static void touch_timer_fire(unsigned long data)
{
      unsigned long data0;
      unsigned long data1;
      int updown;

      /* 
        读取stylus的状态
        0 = Stylus down state
        1 = Stylus up state 
       */
      data0 = readl(base_addr+S3C2410_ADCDAT0);
      data1 = readl(base_addr+S3C2410_ADCDAT1);

     updown = (!(data0 & S3C2410_ADCDAT0_UPDOWN)) && (!(data1 & S3C2410_ADCDAT1_UPDOWN));
     /*
       更新stylus状态寄存器updown：
       1 = down
       0 = up
      */
    /*
      touch_timer_fire这个函数主要实现以下功能：
      1、stylus down的时候，在中断函数stylus_updown里面被调用，
         此时缓存区没有数据，ts.count为0，所以只是简单的设置ad转换的模式，然后开启ad转换。
      2、但ADC中断函数stylus_action把缓冲区填满的时候，作为中断后半段函数稍后被调用，
         此时ts.count为4，算出其平均值后，交给事件处理层(Event Handler)处理，
         主要是填写缓冲，然后唤醒等待输入数据的进程。
      3、stylus抬起，等到缓冲区填满后(可能会包含一些无用的数据)被调用，
         这时候判断出stylus up，报告stylus up事件，重新等待stylus down。

     if (updown) {                        
         if (ts.count != 0) {    <功能2>
             /* 求平均值 */
             ts.xp >>= ts.shift;
             ts.yp >>= ts.shift;

#ifdef CONFIG_TOUCHSCREEN_S3C2410_DEBUG
             {
                 struct timeval tv;
                 do_gettimeofday(&tv);
                 printk(DEBUG_LVL "T: %06d, X: %03ld, Y: %03ld\n", (int)tv.tv_usec, ts.xp, ts.yp);
             }
#endif
             /* 报告x、y的绝对坐标值 */
             input_report_abs(&ts.dev, ABS_X, ts.xp);
             input_report_abs(&ts.dev, ABS_Y, ts.yp);

             /* 报告按键事件，键值为1(代表触摸屏对应的按键被按下) */
             input_report_key(&ts.dev, BTN_TOUCH, 1);

             /* 报告触摸屏的状态，1表明触摸屏被按下 */
             input_report_abs(&ts.dev, ABS_PRESSURE, 1);
       
             /* 等待接收方受到数据后回复确认，用于同步 */
             input_sync(&ts.dev);
         }
                                     <功能1>
         ts.xp = 0;
         ts.yp = 0;
         ts.count = 0;

         writel(S3C2410_ADCTSC_XP_PULL_UP_DIS | AUTOPST, base_addr+S3C2410_ADCTSC);
         /* 设置触摸屏的模式为AUTOPST */
         writel(readl(base_addr+S3C2410_ADCCON) | S3C2410_ADCCON_ENABLE_START, base_addr+S3C2410_ADCCON);
         /* 启动ADC转换 */
     }
    else {                           <功能3>
     
         ts.count = 0;
         
         /* 报告按键事件，键值为1(代表触摸屏对应的按键被释放) */
         input_report_key(&ts.dev, BTN_TOUCH, 0);
         
         /* 报告触摸屏的状态，0表明触摸屏没被按下 */
         input_report_abs(&ts.dev, ABS_PRESSURE, 0);

         /* 等待接收方受到数据后回复确认，用于同步 */
         input_sync(&ts.dev);

         /* 进入s3c2410触摸屏提供的,Waiting for Interrupt Mode，waits for Stylus down */
         writel(WAIT4INT(0), base_addr+S3C2410_ADCTSC);
     }
}

static struct timer_list touch_timer =
        TIMER_INITIALIZER(touch_timer_fire, 0, 0);

static irqreturn_t stylus_updown(int irq, void *dev_id, struct pt_regs *regs)
{
    unsigned long data0;
    unsigned long data1;
    int updown;
    /* 
       读取ADCDAT0/1寄存器，判断Stylus的状态：
       0 = Stylus down state
       1 = Stylus up state
     */
    data0 = readl(base_addr+S3C2410_ADCDAT0);
    data1 = readl(base_addr+S3C2410_ADCDAT1);
    
    updown = (!(data0 & S3C2410_ADCDAT0_UPDOWN)) && (!(data1 & S3C2410_ADCDAT1_UPDOWN));
    /*
      更新stylus状态寄存器updown：
      1 = down
      0 = up
     */

    /* TODO we should never get an interrupt with updown set while
     * the timer is running, but maybe we ought to verify that the
     * timer isn't running anyways. */

    /* 判断出stylus down，调用touch_timer_fire函数 */
    if (updown)
        touch_timer_fire(0);

    return IRQ_HANDLED;
}


static irqreturn_t stylus_action(int irq, void *dev_id, struct pt_regs *regs)
{
    unsigned long data0;
    unsigned long data1;
    data0 = readl(base_addr+S3C2410_ADCDAT0);
    data1 = readl(base_addr+S3C2410_ADCDAT1);

    
    /**************************modify by lfc********************/
    ts.yp += data0 & S3C2410_ADCDAT0_XPDATA_MASK;
    ts.xp += data1 & S3C2410_ADCDAT1_YPDATA_MASK;
    /***********************************************************/
    ts.count++;
 
    if (ts.count < (1<<ts.shift)) {      
        /* 缓冲区未满，再次激活ADC转换 */
        writel(S3C2410_ADCTSC_XP_PULL_UP_DIS | AUTOPST, base_addr+S3C2410_ADCTSC);
        writel(readl(base_addr+S3C2410_ADCCON) | S3C2410_ADCCON_ENABLE_START, base_addr+S3C2410_ADCCON);
    } else {                              
        /* 缓冲区满，激活下半部处理程序touch_timer_fire，处理接收数据 */
        mod_timer(&touch_timer, jiffies+1);
        writel(WAIT4INT(1), base_addr+S3C2410_ADCTSC);
    }

    return IRQ_HANDLED;
}

static struct clk    *adc_clock;

/*
 * The functions for inserting/removing us as a module.
 */

static int __init s3c2410ts_probe(struct device *dev)
{
    struct s3c2410_ts_mach_info *info;

    info = ( struct s3c2410_ts_mach_info *)dev->platform_data;
    注：
       s3c2410_ts_mach_info这个结构需要我们去填充，里面存放的是触摸屏需要的一些配置参数，见下面的附录部分。

    if (!info)
    {
        printk(KERN_ERR "Hm... too bad : no platform data for ts\n");
        return -EINVAL;
    }

#ifdef CONFIG_TOUCHSCREEN_S3C2410_DEBUG
    printk(DEBUG_LVL "Entering s3c2410ts_init\n");
#endif

    adc_clock = clk_get(NULL, "adc");
    if (!adc_clock) {
        printk(KERN_ERR "failed to get adc clock source\n");
        return -ENOENT;
    }
    clk_use(adc_clock);//这个在高版本下已经不需要了
    clk_enable(adc_clock);

#ifdef CONFIG_TOUCHSCREEN_S3C2410_DEBUG
    printk(DEBUG_LVL "got and enabled clock\n");
#endif

    base_addr=ioremap(S3C2410_PA_ADC,0x20);//映射触摸屏的控制寄存器，应该不需要解析了吧^_^
    if (base_addr == NULL) {
        printk(KERN_ERR "Failed to remap register block\n");
        return -ENOMEM;
    }


    /* Configure GPIOs */
    s3c2410_ts_connect();
 
   /*以下根据我们提供的s3c2410_ts_mach_info结构，配置触摸屏的一些控制寄存器*/
        /* Set the prscvl*/
    if ((info->presc&0xff) > 0)
        writel(S3C2410_ADCCON_PRSCEN | S3C2410_ADCCON_PRSCVL(info->presc&0xFF),\
                 base_addr+S3C2410_ADCCON);
    else
        writel(0,base_addr+S3C2410_ADCCON);


    /* Initialise the adcdly registers */
    if ((info->delay&0xffff) > 0)
        writel(info->delay & 0xffff,  base_addr+S3C2410_ADCDLY);

    /* 进入s3c2410触摸屏提供的Waiting for Interrupt Mode，waits for Stylus down.The controller generates Interrupt (INT_TC) signals when the Stylus is down on Touch Screen Panel.*/

    writel(WAIT4INT(0), base_addr+S3C2410_ADCTSC);

    /* Initialise input stuff */
    memset(&ts, 0, sizeof(struct s3c2410ts));
    /* 以下配置2.6内核划分出来的输入设备 */

    init_input_dev(&ts.dev);
    ts.dev.evbit[0] = BIT(EV_SYN) | BIT(EV_KEY) | BIT(EV_ABS);
    /* 
       evbit字段用来定义该输入设备可以支持的（产生和响应）的事件的类型，
       在此触摸屏设置为支持同步(EN_SYN)、按键(EN_KEY)、绝对坐标(EV_ABS)事件
    */
    ts.dev.keybit[LONG(BTN_TOUCH)] = BIT(BTN_TOUCH);
    /* 设置所支持的按键(键值)，触摸屏可以看成只有一个按键的设备 */

    input_set_abs_params(&ts.dev, ABS_X, 0, 0x3FF, 0, 0);
    /* 设置绝对坐标x的最小最大值，在这是0-0x3FF */

    input_set_abs_params(&ts.dev, ABS_Y, 0, 0x3FF, 0, 0);
    /* 设置绝对坐标y的最小最大值，在这是0-0x3FF */
    input_set_abs_params(&ts.dev, ABS_PRESSURE, 0, 1, 0, 0);

    sprintf(ts.phys, "ts0");

    ts.dev.private = &ts;
    ts.dev.name = s3c2410ts_name;
    ts.dev.phys = ts.phys;
    ts.dev.id.bustype = BUS_RS232;
    ts.dev.id.vendor = 0xDEAD;
    ts.dev.id.product = 0xBEEF;
    ts.dev.id.version = S3C2410TSVERSION;

    ts.shift = info->oversampling_shift;
    /* 
       这个比较重要，配置输入数据的缓存区大小，
       在这里oversampling_shift设为2，也就是缓存区的大小为4(1<<2) 
     */

    /* ADC转换中断，转换结束后触发 */
    if (request_irq(IRQ_ADC, stylus_action, SA_SAMPLE_RANDOM,
        "s3c2410_action", &ts.dev)) {
        printk(KERN_ERR "s3c2410_ts.c: Could not allocate ts IRQ_ADC !\n");
        iounmap(base_addr);
        return -EIO;
    }
    /* 
       检测stylus updown的中断，设置为Waiting for Interrupt Mode时，
       The controller generates Interrupt (INT_TC) signals when the Stylus is down on Touch             Screen Panel，还记得吧^_^
     */
    if (request_irq(IRQ_TC, stylus_updown, SA_SAMPLE_RANDOM,
            "s3c2410_action", &ts.dev)) {
        printk(KERN_ERR "s3c2410_ts.c: Could not allocate ts IRQ_TC !\n");
        iounmap(base_addr);
        return -EIO;
    }

    printk(KERN_INFO "%s successfully loaded\n", s3c2410ts_name);

    /* All went ok, so register to the input system */
    input_register_device(&ts.dev);//注册输入设备

    return 0;
}




```