# Building Android Application Development Environment #
  * **系统环境：Windows 7 Ultimate**
  * **Date：03/11/2010**

### 安装JDK ###
  * 在[java.sun.com](http://java.sun.com/javase/downloads/index.jsp)下载JDK并安装
  * 在“系统属性”的“高级”选项卡中点击“环境变量”，然后添加如下系统环境变量：
    1. 在PATH环境变量后追加 JDK安装路径中的bin路径，本机为
```
C:\Program Files\Java\jdk1.6.0_18\bin
```
    1. 新建CLASSPATH环境变量或在CLASSPATH环境变量后追加JDK安装路径中的lib路径和demo路径，本机为
```
C:\Program Files\Java\jdk1.6.0_18\demo;C:\Program Files\Java\jdk1.6.0_18\lib
```

### 安装Eclipse ###
  * 在[eclipse.org](http://www.eclipse.org/downloads/)下载Eclipse IDE for Java Developers的Windows 32bit版本
  * 下载完成后解压即可使用

### 安装Android SDK ###
  * 在[Android Developers](https://unblock4myspace.appspot.com/developer.android.com/sdk/index.html)下载android-sdk\_r05-windows.zip，下载完成后解压到任意路径
  * 运行SDK Setup.exe，点击Available Packages，如果没有出现可安装的包请点击Settings，选中Misc中的"Force https://..."这项，再点击Available Packages
  * 选择希望安装的SDK及其文档或者其它包，点击Installation Selected、Accept All、Install Accepted，开始下载安装所选包
  * 添加SDK安装目录中的tools文件夹路径至系统PATH环境变量，本机为
```
C:\Android\android-sdk-windows\tools
```

### 安装Eclipse插件 ADT ###
  * Start Eclipse, then select Help > Install New Software.
  * In the Available Software dialog, click Add...
  * In the Add Site dialog that appears, enter a name for the remote site (for example, "Android Plugin") in the "Name" field.

In the "Location" field, enter this URL:
```
https://dl-ssl.google.com/android/eclipse/
```
如果无法通过上面的地址获得插件，可将https替换为http。(https is preferred for security reasons)
  * Back in the Available Software view, you should now see "Developer Tools" added to the list.
  * Select the checkbox next to Developer Tools, which will automatically select the nested tools Android DDMS and Android Development Tools. Click Next.
  * In the resulting Install Details dialog, the Android DDMS and Android Development Tools features are listed.
  * Click Next to read and accept the license agreement and install any dependencies, then click Finish.
  * Restart Eclipse.

### 配置ADT ###
在Eclipse中：
  * 选择Window > Preferences...
  * 在左边的面板选择Android，然后在右侧点击Browse...并选中SDK路径，本机为：
```
C:\Android\android-sdk-windows
```
  * 点击Apply、OK。配置完成。

### 创建AVD ###
为使Android应用程序可以在模拟器上运行，必须创建AVD
  * 在Eclipse中。选择Windows > Android SDK and AVD Manager
  * 点击左侧面板的Virtual Devices，在右侧点击New
  * 填入Name，选择Target的API，SD Card大小任意，Skin随便选，Hardware目前保持默认值
  * 点击Create AVD即可完成创建AVD

  * **现在即可创建一个Android工程，然后运行。Android模拟器会启动Android并运行工程默认的HelloWorld程序**