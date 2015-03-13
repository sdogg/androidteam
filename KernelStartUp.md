# LINUX内核启动过程 #

当PC启动时,Intel系列的CPU首先进入的是实模式,并开始执行位于地址0xFFFF0处
的代码,也就是ROM-BIOS起始位置的代码。BIOS先进行一系列的系统自检,然后初始化位
于地址0的中断向量表。最后BIOS将启动盘的第一个扇区装入到0x7C00,并开始执行此处
的代码。这就是对内核初始化过程的一个最简单的描述。

　　最初,linux核心的最开始部分是用8086汇编语言编写的。当开始运行时,核心将自
己装入到绝对地址0x90000,再将其后的2k字节装入到地址0x90200处,最后将核心的其余
部分装入到0x10000。

　　当系统装入时,会**显示Loading...信息**。装入完成后,控制转向另一个实模式下的汇
编语言代码boot/Setup.S。Setup部分首先设置一些系统的硬件设备,然后将核心从
0x10000处移至0x1000处。这时系统转入保护模式,开始执行位于0x1000处的代码。

　　接下来是内核的解压缩。0x1000处的代码来自于文件Boot/head.S,它用来初始化寄
存器和调用decompress\_kernel( )程序。decompress\_kernel( )程序由Boot/inflate.c,
Boot/unzip.c和Boot../misc.c组成。解压缩后的数据被装入到了0x100000处,这也是
linux不能在内存小于2M的环境下运行的主要原因。

　　解压后的代码在0x1010000处开始执行,紧接着所有的32位的设置都将完成： IDT、
GDT和LDT将被装入,处理器初始化完毕,设置好内存页面,最终调用start\_kernel过程。
这大概是整个内核中最为复杂的部分。

[系统开始运行]
linux kernel 最早的C代码从汇编标记startup\_32开始执行
```
startup_32:
start_kernel
lock_kernel
trap_init
init_IRQ
sched_init
softirq_init
time_init
console_init
#ifdef CONFIG_MODULES
init_modules
#endif
kmem_cache_init
sti
calibrate_delay
mem_init
kmem_cache_sizes_init
pgtable_cache_init
fork_init
proc_caches_init
vfs_caches_init
buffer_init
page_cache_init
signals_init
#ifdef CONFIG_PROC_FS
proc_root_init
#endif
#if defined(CONFIG_SYSVIPC)
ipc_init
#endif
check_bugs
smp_init
rest_init
kernel_thread
unlock_kernel
cpu_idle


?startup_32 [arch/i386/kernel/head.S]
?start_kernel [init/main.c]
?lock_kernel [include/asm/smplock.h]
?trap_init [arch/i386/kernel/traps.c]
?init_IRQ [arch/i386/kernel/i8259.c]
?sched_init [kernel/sched.c]
?softirq_init [kernel/softirq.c]
?time_init [arch/i386/kernel/time.c]
?console_init [drivers/char/tty_io.c]
?init_modules [kernel/module.c]
?kmem_cache_init [mm/slab.c]
?sti [include/asm/system.h]
?calibrate_delay [init/main.c]
?mem_init [arch/i386/mm/init.c]
?kmem_cache_sizes_init [mm/slab.c]
?pgtable_cache_init [arch/i386/mm/init.c]
?fork_init [kernel/fork.c]
?proc_caches_init
?vfs_caches_init [fs/dcache.c]
?buffer_init [fs/buffer.c]
?page_cache_init [mm/filemap.c]
?signals_init [kernel/signal.c]
?proc_root_init [fs/proc/root.c]
?ipc_init [ipc/util.c]
?check_bugs [include/asm/bugs.h]
?smp_init [init/main.c]
?rest_init
?kernel_thread [arch/i386/kernel/process.c]
?unlock_kernel [include/asm/smplock.h]
?cpu_idle [arch/i386/kernel/process.c]
```
start\_kernel( )程序用于初始化系统内核的各个部分,包括：

**设置内存边界,调用paging\_init( )初始化内存页面。**初始化陷阱,中断通道和调度。
**对命令行进行语法分析。**初始化设备驱动程序和磁盘缓冲区。
**校对延迟循环。**

最后的function'rest\_init' 作了以下工作:

?开辟内核线程'init'
?调用unlock\_kernel
?建立内核运行的cpu\_idle环, 如果没有调度,就一直死循环

实际上start\_kernel永远不能终止.它会无穷地循环执行cpu\_idle.

最后,系统核心转向move\_to\_user\_mode( ),以便创建初始化进程（init）。此后,进程0开始进入无限循环。

初始化进程开始执行/etc/init、/bin/init 或/sbin /init中的一个之后,系统内核就不再对程序进行直接控制了。之后系统内核的作用主要是给进程提供系统调用,以及提供异步中断事件的处理。多任务机制已经建立起来,并开始处理多个用户的登录和fork( )创建的进程。

[init](init.md)
init是第一个进程,或者说内核线程
```
init
lock_kernel
do_basic_setup
mtrr_init
sysctl_init
pci_init
sock_init
start_context_thread
do_init_calls
(*call())-> kswapd_init
prepare_namespace
free_initmem
unlock_kernel
execve
```

启动步骤

系统引导：
涉及的文件
./arch/$ARCH/boot/bootsect.s
./arch/$ARCH/boot/setup.s

bootsect.S
　这个程序是linux kernel的第一个程序,包括了linux自己的bootstrap程序,
但是在说明这个程序前,必须先说明一般IBM PC开机时的动作(此处的开机是指
"打开PC的电源"):

　　一般PC在电源一开时,是由内存中地址FFFF:0000开始执行(这个地址一定
在ROM BIOS中,ROM BIOS一般是在FEOOOh到FFFFFh中),而此处的内容则是一个
jump指令,jump到另一个位於ROM BIOS中的位置,开始执行一系列的动作,包
括了检查RAM,keyboard,显示器,软硬磁盘等等,这些动作是由系统测试代码
(system test code)来执行的,随着制作BIOS厂商的不同而会有些许差异,但都
是大同小异,读者可自行观察自家机器开机时,萤幕上所显示的检查讯息。

　　紧接着系统测试码之后,控制权会转移给ROM中的启动程序
(ROM bootstrap routine),这个程序会将磁盘上的第零轨第零扇区读入
内存中(这就是一般所谓的boot sector,如果你曾接触过电脑病
毒,就大概听过它的大名),至於被读到内存的哪里呢? --绝对
位置07C0:0000(即07C00h处),这是IBM系列PC的特性。而位在linux开机
磁盘的boot sector上的正是linux的bootsect程序,也就是说,bootsect是
第一个被读入内存中并执行的程序。现在,我们可以开始来
看看到底bootsect做了什么。

第一步
　首先,bootsect将它"自己"从被ROM BIOS载入的绝对地址0x7C00处搬到
0x90000处,然后利用一个jmpi(jump indirectly)的指令,跳到新位置的
jmpi的下一行去执行,

第二步
　接着,将其他segment registers包括DS,ES,SS都指向0x9000这个位置,
与CS看齐。另外将SP及DX指向一任意位移地址( offset ),这个地址等一下
会用来存放磁盘参数表(disk para- meter table )

第三步
　接着利用BIOS中断服务int 13h的第0号功能,重置磁盘控制器,使得刚才
的设定发挥功能。

第四步
　完成重置磁盘控制器之后,bootsect就从磁盘上读入紧邻着bootsect的setup
程序,也就是setup.S,此读入动作是利用BIOS中断服务int 13h的第2号功能。
setup的image将会读入至程序所指定的内存绝对地址0x90200处,也就是在内存
中紧邻着bootsect 所在的位置。待setup的image读入内存后,利用BIOS中断服
务int 13h的第8号功能读取目前磁盘的参数。

第五步
　再来,就要读入真正linux的kernel了,也就是你可以在linux的根目录下看
到的"vmlinuz" 。在读入前,将会先呼叫BIOS中断服务int 10h 的第3号功能,
读取游标位置,之后再呼叫BIOS 中断服务int 10h的第13h号功能,在萤幕上输
出字串"Loading",这个字串在boot linux时都会首先被看到,相信大家应该觉
得很眼熟吧。

第六步
　接下来做的事是检查root device,之后就仿照一开始的方法,利用indirect
jump 跳至刚刚已读入的setup部份

第七步
setup.S完成在实模式下版本检查,并将硬盘,鼠标,内存参数写入到 INITSEG
中,并负责进入保护模式。

第八步
操作系统的初始化。
来源：http://blog.163.com/tianyin_pang/blog/static/64459248200981494855566/