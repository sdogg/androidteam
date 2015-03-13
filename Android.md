# 在android 2.2模拟器里实现上网功能 #

  * 1. 将 android SDK的tool增加到，windows 环境变量 path中。
　　**2. 启动 android 模拟器。**

　　
　　**3. 在window上，打开命令行窗口，输入 adb shell 命令。该命令是进入模拟器的linux系统。**




　　**4. 在 adb shell 模式下执行以下命令
　　sqlite3 /data/data/com.android.providers.settings/databases/settings.db "INSERT INTO system VALUES(99,'http\_proxy','10.10.26.252:1080')"**

　　**5.在 adb shell 模式下执行查询命令
　　sqlite3 /data/data/com.android.providers.settings/databases/settings.db "SELECT** FROM system"
  * 6.重新启动Android模拟器，看看是不是可以上网了？还挺好玩的，在google里随便搜了一下，还真好使，挺有意思的。






# 实现把电脑里的文件下载到SD卡中 #
  * 1.找到android SKD文件夹下的tools 文件夹，先运行里面的emulator.exe，屏幕会有个小黑框闪一下。
  * 2. 首先把你想要下载的文件复制到tools文件夹下。例如我把 a.mp3复制到tools文件夹下了
  * 3. 运行命令行窗口，定位到SDK下的tools文件夹下，运行命令adb push a.mp3 sdcard/a.mp3
  * 4.运行adb -help ，会有很多命令，有兴趣的可以自己看一下。



# 实现安装电脑里的apk安装程序 #
  * 1.找到android SKD文件夹下的tools 文件夹，先运行里面的emulator.exe，屏幕会有个小黑框闪一下。
  * 2. 首先把你想要安装的文件复制到tools文件夹下。例如我把 game.apk复制到tools文件夹下了
  * 3. 运行命令行窗口，定位到SDK下的tools文件夹下，运行命令adb install game.apk，看看模拟器里是不是有这个程序了，