#HelloAndroid  程序自动生成解析

# 系统环境 #

Linux ubuntu 2.6.31-15-generic #50-Ubuntu SMP Tue Nov 10 14:54:29 UTC 2009 i686 GNU/Linux

# 目录各文件含义 #

新建一个工程就能输出helloandroid，并不需要编写程序。

分析android应用程序中hello world!程序的输出。

建立一个android工程，

会发现一个android工程有以下几个文件组成：asset、bin、gen、res、src等5个基本的文件夹、一个AndroidManifest.xml文件和一个default文件。



其中几个文件的作用：
```
1.src目录下的是应用程序的java文件。

2.gen目录里是自动生成的R.java文件，保存的是资源索引。

3.Android1.5(看你选择的sdk版本了)，这个里边是sdk。

4.assets目录，这个是给你放自己的资源的，默认状态下不会更新索引到R.java，可以结合SQLite数据库使用。

5.res资源目录。

-5.1. res/drawable是给你放图片资源的

-5.2. res/layout/main.xml是布局文件

-5.3. res/values是给你放变量的值的

--5.3.1. res/values/string.xml是string对象赋值的一个例子

```
# 程序输入hello world解析 #

每个程序都会有个入口函数，通过观察发现src目录下的helloandriod.java文件下的R.layout.main是入口函数，程序转入res/layout/main.xml文件下，观察发现main.xml中设置了一个TextView，其中android.text中的“android:text”设置了这个TextView要显示的文字内容，可以发现引用的是＠string中的hello（“＠string/hello”）字符串，即在res/values/strings.xml目录下的hello所代表的字符串资源才是真正的输出内容，进一步观察发现在hello字符串的内容是“Hello World,helloandroid”，就是运行helloandroid项目后再屏幕上所看到的字符。


# 后续分析 #
关于字符串“＠string/hello”说明：通过实验发现：在目录res/values/strings.xml中的android:text=" "可以直接显示要输出的内容，之所以要通过＠string引用，是因为手机存储空间有限。当另外有个android:text=" "输入相同的内容后就可以通过引用，不用输入相同的内容，节省手机存储空间。