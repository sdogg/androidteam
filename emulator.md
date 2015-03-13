# How to use Android Emulator? #
**详细的设置指南可以参照我的google Doc:**http://docs.google.com/View?id=dfn92jnc_09g95fshp

当android源码编译完成后，需要进行相应的环境配置才可以顺利启动模拟器,
在终端里输入：gedit ~/.bashrc
在打开的文档末尾添加如下环境参数：
```
#add other info here just for android
export ANDROID_JAVA_HOME=$JAVA_HOME
export ANDROID_PRODUCT_OUT=/home/sw2/mydroid/out/target/product/generic
export PATH=${PATH}:~/mydroid/out/host/linux-x86/sdk/android-sdk_eng.bluestorm_linux-x86/tools
#指名SDK所在的目录
export PATH=${PATH}:~/mydroid/out/host/linux-x86/bin
```
保存后退出，然后重新打开终端，在终端中输入：
$ emulator -debug-init -skin QVGA-L
```
sw2@sw2-desktop:~/mydroid/out/target/product/generic$ emulator -ramdisk ramdisk.img -system system.img  -data userdata.img 
```

# References #
  * [Get Android source and build it](GetAndroidSource.md) [issue 3](https://code.google.com/p/androidteam/issues/detail?id=3)
  * How to use Android Emulator? [issue 4](https://code.google.com/p/androidteam/issues/detail?id=4)