#Android根文件系统启动过程.

# Android根文件系统启动过程介绍 #

```
在Android系统启动时，内核引导参数上一般都会设置 “init=/init”,这样的话，如果内核成功挂载了这个文件系统之后，首先运行的就是这个根目录下的init程序 。这个程序所了什么呢？ 我们只有RFSC（Readthe Fucking Source code）!!
init程序源码在Android官方 源码的system/core/init中，main在init.c里。我们的分析就从main开始。
init： 
（1）安装SIGCHLD信号 。（如果父进程不等待子进程结束，子进程将成为僵尸进程（zombie）从而占用系统资源。因此需要对SIGCHLD信号做出处理，回收僵尸进程的资源，避免造成不必要的资源浪费。
（2）对umask进行清零。

（3）为root fs建立必要的文件夹，并挂载适当的分区。
    /dev （tmpfs）
        /dev/pts (devpts)
        /dev/socket
    /proc (proc)
    /sys  (sysfs)
  (4)创建/dev/null和/dev/kmsg节点。
（5）解析/init.rc，将所有服务和操作信息加入链表。
  (6)从/proc/cmdline中提取信息内核启动参数,并保存到全局变量。
（7）先从上一步获得的全局变量中获取信息硬件信息和版本号，如果没有则从/proc/cpuinfo中提取,并保存到全局变量。
（8）根据硬件信息选择一个/init.(硬件).rc，并解析，将服务和操作信息加入链表。
         在G1的ramdisk根目录下有两个/init.(硬件).rc：init.goldfish.rc和init.trout.rc，init程序会根据上一步获得的硬件信息选择一个解析。
（9）执行链表中带有“early-init”触发的的命令。
（10）遍历/sys文件夹，是内核产生设备添加事件（为了自动 产生设备节点)。
（11）初始化属性系统，并导入初始化属性文件。
（12）从属性系统中得到ro.debuggable，若为1，則初始化keychord监听。
（13）打开console,如果cmdline中沒有指定console則打开默认的/dev/console。
（14）读取/initlogo.rle（一章565 rle 压缩的位图），如果成功则在/dev/graphics/fb0显示Logo,如果失败则將/dev/tty0设置TEXT模式并打开/dev/tty0,输出文本“ANDROID”字样。
（15）判断cmdline 中的参数，并设置属性系统中的参数:
         1、 如果 bootmode为
         - factory,设置ro.factorytest值为1
         - factory2,设置ro.factorytest值为2
         - 其他的设ro.factorytest值为0
       2、如果有serialno参数，则设置ro.serialno，否则为""
       3、如果有bootmod参数，则设置ro.bootmod，否则为"unknown"
       4、如果有baseband参数，则设置ro.baseband，否则为"unknown"
       5、如果有carrier参数，则设置ro.carrier，否则为"unknown"
       6、如果有bootloader参数，则设置ro.bootloader，否则为"unknown"
       7、通过全局变量（前面从/proc/cpuinfo中提取的）否则ro.hardware和ro.version。
（16执行所有触发标识为init的action。
（17）开始property服务，读取一些property文件，这一动作必须在前面那些ro.foo设置后做，以便/data/local.prop不能干欲到他们。
      - /system/build.prop
      - /system/default.prop
      - /data/local.prop
      - 在读取默认的property后读取presistent propertie，在/data/property中
（18）为sigchld handler创建信号机制。
（19）确认所有初始化工作完成：
          device_fd(device init 完成)
          property_set_fd(property server start 完成)
          signal_recv_fd (信信号机制建立)
（20） 执行所有触发标识为early-boot的action
（21）执行所有触发标识为boot的action
（22）基于当前property状态，执行所有触发标识为property的action
（23）注册轮循事件:
           - device_fd
           - property_set_fd
           -signal_recv_fd
           -如果有keychord，則注册keychord_fd
（24）如果支持BOOTCHART,則初始化BOOTCHART
（25）进入主进程循环:
      - 重置循环事件的接受状态，revents为0
      - 查询action队列，并执行。
      - 重启需要重启的服务
      - 轮循注册的事件
          - 如果signal_recv_fd的revents为POLLIN，則得到一个信号，获取并处理
          - 如果device_fd的revents为POLLIN,调用handle_device_fd
          - 如果property_fd的revents为POLLIN,调用handle_property_set_fd
          - 如果keychord_fd的revents为POLLIN,调用handle_keychord


```


# Details #

Add your content here.  Format your content with:
  * Text in **bold** or _italic_
  * Headings, paragraphs, and lists
  * Automatic links to other wiki pages