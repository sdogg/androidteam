#based on v2.6.26 kernel
# Linux内核Makefile编译生成内核目标文件的过程 #

直接执行make的编译过程
  * 1.先找到入口点([入口点问题](LinuxKernelMakefileStartPoint.md))
```
#编译内核line502,直接执行make默认编译此项
all: vmlinux  
#编译模块line1037,选择编译模块的话会到这里,另外还有其他许多all:target存在,为什么默认执行all: vmlinux ?
all: modules
```
  * 2.继续找vmlinux目标
```
# vmlinux image - including updated kernel symbols
# vmlinux目标在line806
vmlinux: $(vmlinux-lds) $(vmlinux-init) $(vmlinux-main) vmlinux.o $(kallsyms.o) FORCE
# FORCE是伪目标，make假定伪目标的时间戳总是最新的，即总是被修改过，因此以它为“依赖”的“目标”“vmlinux”在每次make的时候都会被编译。
```
  * 3.理解$(vmlinux-lds) $(vmlinux-init) $(vmlinux-main)这个变量的作用
```
# line656
vmlinux-init := $(head-y) $(init-y)
# -y是指配置为yes表示加入内核，-m是指配置为module，-n是指配置为no表示不加入内核
vmlinux-main := $(core-y) $(libs-y) $(drivers-y) $(net-y)
vmlinux-all  := $(vmlinux-init) $(vmlinux-main)
vmlinux-lds  := arch/$(SRCARCH)/kernel/vmlinux.lds # SRCARCH为体系结构名,这里我们使用x86
```
生成的vmlinux.lds目标文件是链接生成vmlinux映像的[链接描述文件ld script](LinkerDescrpitonScript.md),从该文件中我们大致可以知道vmlinux映像的头部是$(head-y) $(init-y),vmlinux映像的主体部分是$(core-y) $(libs-y) $(drivers-y) $(net-y)等.具体我们可以仔细研究[ld script to make i386 Linux kernel](LDScrpitKernelI386.md).
```
vmlinux映像
####################################################
#             #                                    #
#  $(head-y)  #  $(core-y) $(libs-y)               #
#  $(init-y)  #  $(drivers-y) $(net-y),etc.        #
#             #                                    #
####################################################
```
  * 找出$(vmlinux-init)或者说$(head-y) $(init-y)包含那些文件
先找init-y,轻易搞定如下:
```
# line452
init-y		:= init/
# line621
init-y		:= $(patsubst %/, %/built-in.o, $(init-y)) ## 表示将$(init-y)列表中"/"替换为"/built-in.o",也就是最终init-y == init/built-in.o
```
init/built-in.o目标在init目录下生成,其中包含start\_kernel函数,这个函数是从启动代码进入linux kernel的点.

在根目录下的Makefile文件中我们找不到head-y的定义,那么head-y肯定在某个被包含(include)进来的文件中.
通过搜索include我们发现head-y可能在/arch/x86/Makefile中.
```
# line431
include $(srctree)/arch/$(SRCARCH)/Makefile
```
果然,在/arch/x86/Makefile中找到head-y,
```
# line161
head-y := arch/x86/kernel/head_$(BITS).o ## BITS是CPU处理的位数的定义,我们使用的32位CPU,这里直接使用32来代替,文件也就是head_32.o
head-y += arch/x86/kernel/head$(BITS).o # head32.o
head-y += arch/x86/kernel/init_task.o
```
  * vmlinux映像生成的一般规则综述
通过以上分析路径
```
all --> vmlinux --> $(vmlinux-lds) $(vmlinux-init) --> $(head-y) $(init-y) --
--> built-in.o head32.o head_32.o init_task.o --> *.c *.S
```
我们可以有了一个大致的概念,那就是通过内核配置信息,我们有了xxxx-y的目标列表,通过深度遍历依次去生成这些目标,并最终生成了vmlinux.

至于内核配置信息与xxxx-y的目标列表以及依然的目录文件等,是如何映射匹配的,还需要更仔细的分析.

  * bzimage - 对vmlinux映像的后续处理
并且在/arch/x86/Makefile中我们还可以发现对vmlinux映像的后续处理部分,后续处理之后的bzimage将会是
```
bzimage
####################################################
#         #                  #                     #
#  Setup  #  uncompress code #  compressed vmlinux #
#         #                  #                     #
####################################################
```
接下来我们看看对vmlinux映像的后续处理部分的Makefile,首先要找到起点:
```
# line200 of /arch/x86/Makefile
####
# boot loader support. Several targets are kept for legacy purposes

boot := arch/x86/boot

PHONY += zImage bzImage compressed zlilo bzlilo \
         zdisk bzdisk fdimage fdimage144 fdimage288 isoimage install

# Default kernel to build
all: bzImage

# KBUILD_IMAGE specify target image being built
                    KBUILD_IMAGE := $(boot)/bzImage
zImage zlilo zdisk: KBUILD_IMAGE := arch/x86/boot/zImage

zImage bzImage: vmlinux
	$(Q)$(MAKE) $(build)=$(boot) $(KBUILD_IMAGE) ## 进入arch/x86/boot目录执行其Makefile
	$(Q)mkdir -p $(objtree)/arch/$(UTS_MACHINE)/boot
	$(Q)ln -fsn ../../x86/boot/bzImage $(objtree)/arch/$(UTS_MACHINE)/boot/bzImage

compressed: zImage
```
# What's the Next? #

  * vmlinux映像的后续处理并生成zImage bzImage,会大量涉及linux启动的部分,我们在[linux启动分析](KernelStartUp.md)中来具体介绍.

# References #
  * [Linux内核源代码中的Makefile分析](LinuxMakefileAnalysis.md)
  * /Documentation/kbuild/makefiles.txt - This document describes the Linux kernel Makefiles.[中文译稿](http://hi.baidu.com/wjq_qust/blog/item/97ddbdfdfb2e541309244d30.html)
  * [配置内核过程](LinuxKernelConfig.md)
  * [入口点的问题](LinuxKernelMakefileStartPoint.md)
  * [编译生成内核目标文件的过程](LinuxKernelCompiling.md)