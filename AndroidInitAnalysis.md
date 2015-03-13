# Android系统启动分析 #
  * **系统环境 Ubuntu 9.10 Linux-kernel 2.6.31-20 generic
  ***日期：2010/3/16

在Android系统启动时，内核引导参数上一般都会设置“init=/init”,这样的话，如果内核成功挂载了这个文件系统之后，首先运行的就是这个根目录下的init程序。

这个init程序是以一个守护进程的方式进行，源代码的位置在/system/core/init/init.c

下面通过对这个init.c文件的简单分析来了解Android的启动过程

1)安装SIGCHLD信号。（如果父进程不等待子进程结束，子进程将成为僵尸进程（zombie）从而占用系统资源。因此需要对SIGCHLD信号做出处理，回收僵尸进程的资源，避免造成不必要的资源浪费。

2)对umask进行清零。

3)为rootfs建立必要的文件夹，并挂载适当的分区。

4)创建空设备节点，创建ksmg设备节点，初始化log。

5)对启动脚本文件init.rc进行解析。

6)读取保存在/proc/cmdline中的内核启动参数。

7)读取硬件信息。

8)解析init.［硬件信息］.rc，在编译生成的根文件系统下有init.goldfish.rc和init.trout.rc，init程序会根据上一步获得的硬件信息选择一个解析。

9)执行early-init。

10)初始化设备。

11)初始化属性，对属性相关处理和启动logo。

12)启动相关属性对应的服务。

13)初始化工作完成后，进入主进程循环，对设备进行处理。

在初始化的过程中，可以尝试对启动的相关内容进行修改。比如在Android启动后，会出现2次开机画面，第一次是在init.c中调用load\_565rle\_image()打开“initlogo.rle”文件，通过写调用写Framebuffer驱动程序的方式，将内容输出到屏幕上，在模拟器上没有这个文件，那么Android就打开Linux控制台，利用文本方式向其输出字符，代码片段如下：
```
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
        "             W E I W E I ";
        write(fd, msg, strlen(msg));
        close(fd);
    }
    }
```
这里对开机字符进行了更改，重新编译后发现开机画面已经更改。如果需要用到开机画面，就需要构造一个"/initlogo.rle"文件，这是一个RGB565格式的纯数据文件。


