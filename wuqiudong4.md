
```

注册一个驱动的步骤：
1、定义一个platform_driver结构
2、初始化这个结构，指定其probe、remove等函数，并初始化其中的driver变量
3、实现其probe、remove等函数
看platform_driver结构，定义于include/linux/platform_device.h文件中：
struct platform_driver {
    int (*probe)(struct platform_device *);
    int (*remove)(struct platform_device *);
    void (*shutdown)(struct platform_device *);
    int (*suspend)(struct platform_device *, pm_message_t state);
    int (*suspend_late)(struct platform_device *, pm_message_t state);
    int (*resume_early)(struct platform_device *);
    int (*resume)(struct platform_device *);
    struct device_driver driver;
};

    可见，它包含了设备操作的几个功能函数，同样重要的是，它还包含了一个device_driver结构。刚才提到了驱动程序中需要初始化这个变量。下面看一下这个变量的定义，位于include/linux/device.h中：

struct device_driver {
    const char        * name;
    struct bus_type        * bus;
    struct kobject        kobj;
    struct klist        klist_devices;
    struct klist_node    knode_bus;
    struct module        * owner;
    const char         * mod_name;  /* used for built-in modules */
    struct module_kobject    * mkobj;
    int    (*probe)    (struct device * dev);
    int    (*remove)    (struct device * dev);
    void    (*shutdown)    (struct device * dev);
    int    (*suspend)    (struct device * dev, pm_message_t state);
    int    (*resume)    (struct device * dev);
};
    需要注意这两个变量：name和owner。那么的作用主要是为了和相关的platform_device关联起来，owner的作用是说明模块的所有者，驱动程序中一般初始化为THIS_MODULE。
    下面是一个platform_driver的初始化实例：
static struct platform_driver s3c2410iis_driver = { 
  .probe = s3c2410iis_probe, 
  .remove = s3c2410iis_remove,
  .driver = {
    .name = "s3c2410-iis",
    .owner = THIS_MODULE,
  },
}; 
    上面的初始化是一个音频驱动的实例。注意其中的driver这个结构体，只初始化了其name和owner两个量。接着看一下和driver相关的另一个结构，定义如下：
struct platform_device {
    const char    * name;
    int        id;
    struct device    dev;
    u32        num_resources;
    struct resource    * resource;
};
   该结构中也有一个name变量。platform_driver从字面上来看就知道是设备驱动。设备驱动是为谁服务的呢？当然是设备了。platform_device就描述了设备对象。下面是一个具体的实例： 
struct platform_device s3c_device_iis = {
    .name         = "s3c2410-iis",
    .id         = -1,
    .num_resources     = ARRAY_SIZE(s3c_iis_resource),
    .resource     = s3c_iis_resource,
    .dev = {
        .dma_mask = &s3c_device_iis_dmamask,
        .coherent_dma_mask = 0xffffffffUL
    }
};
    它的name变量和刚才上面的platform_driver的name变量是一致的，内核正是通过这个一致性来为驱动程序找到资源，即platform_device中的resource。这个结构的定义如下，位于include/linux/ioport.h中： 
struct resource {
    resource_size_t start;
    resource_size_t end;
    const char *name;
    unsigned long flags;
    struct resource *parent, *sibling, *child;
};
    下面是一个具体的实例：
static struct resource s3c_iis_resource[] = {
    [0] = {
        .start = S3C24XX_PA_IIS,
        .end = S3C24XX_PA_IIS + S3C24XX_SZ_IIS -1,
        .flags = IORESOURCE_MEM,
    }
};
    这个结构的作用就是告诉驱动程序设备的起始地址和终止地址和设备的端口类型。这里的地址指的是物理地址。
    另外还需要注意platform_device中的device结构，它详细描述了设备的情况，定义如下：
struct device {
    struct klist        klist_children;
    struct klist_node    knode_parent;        /* node in sibling list */
    struct klist_node    knode_driver;
    struct klist_node    knode_bus;
    struct device        *parent;
    struct kobject kobj;
    char    bus_id[BUS_ID_SIZE];    /* position on parent bus */
    struct device_type    *type;
    unsigned        is_registered:1;
    unsigned        uevent_suppress:1;
    struct semaphore    sem;    /* semaphore to synchronize calls to
                     * its driver.
                     */
    struct bus_type    * bus;        /* type of bus device is on */
    struct device_driver *driver;    /* which driver has allocated this
                     device */
    void        *driver_data;    /* data private to the driver */
    void        *platform_data;    /* Platform specific data, device
                     core doesn't touch it */
    struct dev_pm_info    power;
#ifdef CONFIG_NUMA
    int        numa_node;    /* NUMA node this device is close to */
#endif
    u64        *dma_mask;    /* dma mask (if dma'able device) */
    u64        coherent_dma_mask;/* Like dma_mask, but for
                     alloc_coherent mappings as
                     not all hardware supports
                     64 bit addresses for consistent
                     allocations such descriptors. */
    struct list_head    dma_pools;    /* dma pools (if dma'ble) */
    struct dma_coherent_mem    *dma_mem; /* internal for coherent mem
                     override */
    /* arch specific additions */
    struct dev_archdata    archdata;
    spinlock_t        devres_lock;
    struct list_head    devres_head;
    /* class_device migration path */
    struct list_head    node;
    struct class        *class;
    dev_t            devt;        /* dev_t, creates the sysfs "dev" */
    struct attribute_group    **groups;    /* optional groups */
    void    (*release)(struct device * dev);
};
```