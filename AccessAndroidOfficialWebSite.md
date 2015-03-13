# http://source.android.com和http://developer.android.com可以正常访问！ #

## 方法二(代理不稳定,图片能显示则说明可以使用) ##

不知何时，连source.android.com和developer.android.com都不能访问了。看最新的android文档和下载最新的android sdk/source，可以通过下面appspot代理(图片链接)访问android网站

[![](https://unblock4myspace.appspot.com/developer.android.com/assets/images/bg_logo.png)](https://unblock4myspace.appspot.com/developer.android.com/index.html)
[![](http://androidappdocs.appspot.com/assets/images/home/android_adc.png)](http://androidappdocs.appspot.com/index.html)

[https://unblock4myspace.appspot.com/source.android.com/\_/rsrc/1226370697531/config/app/images/customLogo/customLogo.gif?revision=5](https://unblock4myspace.appspot.com/source.android.com/download)

## 方法一(似乎已经不能用了) ##

Google Android官方开源网站(http://source.android.com/ )现在已经撞墙。


现在介绍一个不用代理访问http://source.android.com/ 的方法，而且还很方便，那就是修改一下hosts文件：


对于Windows操作系统:
> 在 C:\WINDOWS\system32\drivers\etc\hosts 文件中加入
> > 74.125.43.121  source.android.com


对Linux操作系统：

> 在 /etc/hosts文件中加入
> 74.125.43.121  source.android.com


重新打开你的浏览器，输入source.android.com，就会发现可以浏览Google Android官方开源网站了：）


这个方法同样适用于所有域名CNAME指向ghs.google.com的所有服务，例如：Google Apps，Blogger自定义域、Google App Engine等：）


如果你正在使用上述服务，而且将自己的域名CNAME指向ghs.google.com，那你也可以将自己的域名解析到上面那个ip地址上，那样你的Google Apps，Blogger、Google App Engine在国内就可以浏览了：）

come from http://www.cqun.com/2009/01/google-android.html