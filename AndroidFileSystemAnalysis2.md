# Android 根文件系统启动分析 #
> init进程是Android启动后系统执行的第一个名称为init的可执行程序。这个程序以一个守护进程的方式运行，它提供了以下功能：
  * 设备管理
  * 解析启动脚本
  * 执行启动脚本中的基本功能
  * 执行启动脚本中的各种功能

## 1、init可执行程序 ##
> init可执行文件是系统运行的第一个用户空间程序，它以守护进程的方式运行。因此这个程序的init.c文件包含main函数的入口，基本分析如下：
```
int main(int argc, char **argv)
{
   (省略若干。。。)
   
    umask(0);  /*对umask进行清零。*/
    
    mkdir("/dev", 0755);/*为rootfs建立必要的文件夹，并挂载适当的分区。 */
    mkdir("/proc", 0755);
    mkdir("/sys", 0755);

    mount("tmpfs", "/dev", "tmpfs", 0, "mode=0755");
    mkdir("/dev/pts", 0755);
    mkdir("/dev/socket", 0755);
    mount("devpts", "/dev/pts", "devpts", 0, NULL);
    mount("proc", "/proc", "proc", 0, NULL);
    mount("sysfs", "/sys", "sysfs", 0, NULL);

    /*创建/dev/null和/dev/kmsg节点*/
    open_devnull_stdio();
    log_init();
   
    /*解析/init.rc，将所有服务和操作信息加入链表。*/
    INFO("reading config file\n");
    parse_config_file("/init.rc");

    /*获取内核命令行参数*/
    qemu_init();
    import_kernel_cmdline(0);
    /*先从上一步获得的全局变量中获取信息硬件信息和版本号，如果没有则从/proc/cpuinfo中提取，
     *并保存到全局变量。根据硬件信息选择一个/init.(硬件).rc，并解析，将服务和操作信息加入链表。
     */
    get_hardware_name();
    snprintf(tmp, sizeof(tmp), "/init.%s.rc", hardware);
    parse_config_file(tmp);
    /*执行链表中带有“early-init”触发的的命令。*/
    action_for_each_trigger("early-init", action_add_queue_tail);
    drain_action_queue();
    /*遍历/sys文件夹，是内核产生设备添加事件（为了自动产生设备节点)。
     *初始化属性系统，并导入初始化属性文件。用于在系统运行过程中动态创建设备节点、删除设备节点等操作
     */
    INFO("device init\n");
    device_fd = device_init();

    property_init();
    
    // 从属性系统中得到ro.debuggable，若为1，则初始化keychord监听。
    debuggable = property_get("ro.debuggable");
    if (debuggable && !strcmp(debuggable, "1")) {
        keychord_fd = open_keychord();
    }
    /*打开console，如果cmdline中没有指定的console则打开默认的/dev/console*/
    
    if (console[0]) {
        snprintf(tmp, sizeof(tmp), "/dev/%s", console);
        console_name = strdup(tmp);
    }

    fd = open(console_name, O_RDWR);
    if (fd >= 0)
        have_console = 1;
    close(fd);
    /*读取/initlogo.rle（一张位图），如果成功则在/dev/graphics/fb0 显示Logo,如果失败则将/dev/tty0
     *设为TEXT模式并打开/dev/tty0,输出文本ANDROID（本人修改为Zhao Rui Jia做为启动项目的修改）。
     */
    if( load_565rle_image(INIT_IMAGE_FILE) ) {
    fd = open("/dev/tty0", O_WRONLY);
    if (fd >= 0) {
        const char *msg;
            msg = "\n"
        "\n"
        "\n"
        "\n"
        "\n"
        "\n"
        "\n"  // console is 40 cols x 30 lines
        "\n"
        "\n"
        "\n"
        "\n"
        "\n"
        "\n"
        "\n"
      /*"             A N D R O I D ";*/
        "          z h a o R u i J i a";
        write(fd, msg, strlen(msg));
        close(fd);
    }
    }
   /* 判断cmdline 中的參數，并设置属性系统中的参数:
    *  1、 如果 bootmode为
    *     - factory,设置ro.factorytest值为1
    *     - factory2,设置ro.factorytest值为2
    *     - 其他的設ro.factorytest值為0
    *  2、如果有serialno参数，则设置ro.serialno，否则为""
    *  3、如果有bootmod参数，则设置ro.bootmod，否则为"unknown"
    *  4、如果有baseband参数，则设置ro.baseband，否则为"unknown"
    *  5、如果有carrier参数，则设置ro.carrier，否则为"unknown"
    *  6、如果有bootloader参数，则设置ro.bootloader，否则为"unknown"
    *  7、通过全局变量（前面从/proc/cpuinfo中提取的）设置ro.hardware和ro.version。
    */
    if (qemu[0])
        import_kernel_cmdline(1); 

    if (!strcmp(bootmode,"factory"))
        property_set("ro.factorytest", "1");
    else if (!strcmp(bootmode,"factory2"))
        property_set("ro.factorytest", "2");
    else
        property_set("ro.factorytest", "0");

    property_set("ro.serialno", serialno[0] ? serialno : "");
    property_set("ro.bootmode", bootmode[0] ? bootmode : "unknown");
    property_set("ro.baseband", baseband[0] ? baseband : "unknown");
    property_set("ro.carrier", carrier[0] ? carrier : "unknown");
    property_set("ro.bootloader", bootloader[0] ? bootloader : "unknown");

    property_set("ro.hardware", hardware);
    snprintf(tmp, PROP_VALUE_MAX, "%d", revision);
    property_set("ro.revision", tmp);

    /*执行所有触发标识为init的action。*/
    
    action_for_each_trigger("init", action_add_queue_tail);
    drain_action_queue();
    property_set_fd = start_property_service();

     /* 为sigchld handler创建信号机制*/
    
   if (socketpair(AF_UNIX, SOCK_STREAM, 0, s) == 0) {
        signal_fd = s[0];
        signal_recv_fd = s[1];
        fcntl(s[0], F_SETFD, FD_CLOEXEC);
        fcntl(s[0], F_SETFL, O_NONBLOCK);
        fcntl(s[1], F_SETFD, FD_CLOEXEC);
        fcntl(s[1], F_SETFL, O_NONBLOCK);
    }

    /* 确认所有初始化工作完成
     * device_fd(device init 完成)
     * property_set_fd(property server start 完成)
     * signal_recv_fd (信号机制建立) 
     */
    if ((device_fd < 0) ||
        (property_set_fd < 0) ||
        (signal_recv_fd < 0)) {
        ERROR("init startup failure\n");
        return 1;
    }

    /* execute all the boot actions to get us started */
    action_for_each_trigger("early-boot", action_add_queue_tail);
    action_for_each_trigger("boot", action_add_queue_tail);
    drain_action_queue();

    /* run all property triggers based on current state of the properties */
    queue_all_property_triggers();
    drain_action_queue();

    /* enable property triggers */   
    property_triggers_enabled = 1;     
  /*
   *    注册轮询事件:
   *   - device_fd
   *   - property_set_fd
   *   -signal_recv_fd
   *   -如果有keychord，则注册keychord_fd
   */
    ufds[0].fd = device_fd;
    ufds[0].events = POLLIN;
    ufds[1].fd = property_set_fd;
    ufds[1].events = POLLIN;
    ufds[2].fd = signal_recv_fd;
    ufds[2].events = POLLIN;
    fd_count = 3;

    if (keychord_fd > 0) {
        ufds[3].fd = keychord_fd;
        ufds[3].events = POLLIN;
        fd_count++;
    } else {
        ufds[3].events = 0;
        ufds[3].revents = 0;
    }

/*如果支持BOOTCHART,则初始化BOOTCHART*/

#if BOOTCHART
    bootchart_count = bootchart_init();
    if (bootchart_count < 0) {
        ERROR("bootcharting init failure\n");
    } else if (bootchart_count > 0) {
        NOTICE("bootcharting started (period=%d ms)\n", bootchart_count*BOOTCHART_POLLING_MS);
    } else {
        NOTICE("bootcharting ignored\n");
    }
#endif
  /*  
   *进入主进程循环:
   *  - 重置轮询事件的接受状态，revents为0
   *  - 查询action队列并执行。
   *  - 重启需要重启的服务
   *  - 轮询注册的事件
   *       - 如果signal_recv_fd的revents为POLLIN，则得到一个信号，获取并处理
   *       - 如果device_fd的revents为POLLIN,调用handle_device_fd
   *       - 如果property_fd的revents为POLLIN,调用handle_property_set_fd
   *       - 如果keychord_fd的revents为POLLIN,调用handle_keychord
   */ 
   for(;;) {
        int nr, i, timeout = -1;

        for (i = 0; i < fd_count; i++)
            ufds[i].revents = 0;

        drain_action_queue();
        restart_processes();

        if (process_needs_restart) {
            timeout = (process_needs_restart - gettime()) * 1000;
            if (timeout < 0)
                timeout = 0;
        }

#if BOOTCHART
        if (bootchart_count > 0) {
            if (timeout < 0 || timeout > BOOTCHART_POLLING_MS)
                timeout = BOOTCHART_POLLING_MS;
            if (bootchart_step() < 0 || --bootchart_count == 0) {
                bootchart_finish();
                bootchart_count = 0;
            }
        }
#endif
        nr = poll(ufds, fd_count, timeout);
        if (nr <= 0)
            continue;

        if (ufds[2].revents == POLLIN) {
            /* we got a SIGCHLD - reap and restart as needed */
            read(signal_recv_fd, tmp, sizeof(tmp));
            while (!wait_for_one_process(0))
                ;
            continue;
        }

        if (ufds[0].revents == POLLIN)
            handle_device_fd(device_fd);

        if (ufds[1].revents == POLLIN)
            handle_property_set_fd(property_set_fd);
        if (ufds[3].revents == POLLIN)
            handle_keychord(keychord_fd);
    }

    return 0;
}


```

## 2、启动脚本init.rc ##

在Android中使用启动脚本init.rc,可以在系统的初始化过程中进行一些简单的初始化操作。这个脚本被直接安装到目标系统的根文件系统中，被init可执行程序解析。
init.rc是在init启动后被执行的启动脚本，其余发主要包含了以下内容：
  * Commands：命令
  * Actions：动作
  * Triggers:触发条件
  * Services：服务
  * Options：选项
  * Propertise：属性
Commands是一些基本的操作，例如：
```
    mkdir /sdcard 0000 system system
    mkdir /system
    mkdir /data 0771 system system
    mkdir /cache 0770 system cache
    mkdir /config 0500 root root
    mkdir /sqlite_stmt_journals 01777 root root
    mount tmpfs tmpfs /sqlite_stmt_journals size=4m
```
这些命令在init可执行程序中被解析，然后调用相关的函数来实现。
Actions(动作)表示一系列的命令，通常在Triggers（触发条件）中调用，动作和触发条件例如：
```
    on init
    export PATH /sbin:/system/sbin:/system/bin:/system/xbin
```

init表示一个触发条件，这个触发事件发生后，进行设置环境变量和建立目录的操作称为一个“动作”
Services（服务）通常表示启动一个可执行程序，Options（选项）是服务的附加内容，用于配合服务使用。

```
service vold /system/bin/vold
    socket vold stream 0660 root mount

service bootsound /system/bin/playmp3
    user media
    group audio
    oneshot
```
vold和bootsound分别是两个服务的名称，/system/bin/vold和/system/bin/playmp3分别是他们所对应的可执行程序。socket、user、group、oneshot就是配合服务使用的选项。
Properties（属性）是系统中使用的一些值，可以进行设置和读取。
```
    setprop ro.FOREGROUND_APP_MEM 1536
    setprop ro.VISIBLE_APP_MEM 2048
    start adbd
```
setprop 用于设置属性，on property可以用于判断属性，这里的属性在整个Android系统运行中都是一致的。

综上如果想要修改启动过程只需要修改init.c或者init.rc里的内容即可，本人只做了修改启动界面显示的实验.

## 参考资料 ##
