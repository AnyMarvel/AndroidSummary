>smalidea是一个IntelliJ IDEA/Android Studio smali语言插件，可实现动态调试smali代码。
github地址：https://github.com/JesusFreke/smali/wiki/smalidea

![](http://upload-images.jianshu.io/upload_images/2333435-58b44a1312f9fa8f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

#####前言
在开发过程中，debug版本我们可以跟踪调试，查看bug等信息，但是在release版本中只能去打log进行代码进行猜测，还有就是dump堆栈等无法与代码直接交互的方法。无源码调试指的是在没有源代码的情况下可以对app进行代码调试，逆向smali代码，然后查看其运行逻辑。在发现release版本问题的过程中可以让我们更块的定位错误。

#####动态调试Android App
smalidea支持14.1或以上版本的IDEA。android Studio如果是基于14.1或以上版本的IDEA也是支持的，我这里用的是2.3.3版本的Android Studio，IDEA的操作也差不多。
从上面的下载地址中下载以下三个应用

![](http://upload-images.jianshu.io/upload_images/2333435-5adba28e14780c8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

backsmali:可将apk转为smali代码，也可将odex转为smali代码，当然在odex转为smali过程的中需要/system/framework目录下的内容

smail:可将smail文件转为dex文件

下载结束后，开始安装插件：
Android Studio -> Preferences -> Plugins -> Install plugin from disk -> 选择 smalidea插件 ->重启 -> 插件就安装好了。

#####准备工作

1. 第一步当然是要拿到你想debug的apk,这里随机使用一个app
2. 第二步就是要把apk里面的编译后的代码转成smali。
这里可以使用上述过程中的baksmali进行反编译，也可以使用apktool进行反编译
baksmail:
```
java -jar baksmali-2.2.1.jar  d myapp.apk -o ~/projects/myapp/src
```
apktool:
```
java -jar apktool.jar d myapp.apk -o ~/projects/myapp/src
```
如图所示使用baksmali-2.2.1.jar反编译出的目录
![](http://upload-images.jianshu.io/upload_images/2333435-654745336270b556.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 这一步很关键，就是让运行在设备中的程序支持debug。方法有几种：
- 把设备root掉
- 修改测试机的 /default.prop 文件的ro.debuggable=1,目测这一步也可能需要root。
- 使用模拟器
- 修改apk的Manifest application 属性 android:debuggable="true",可以用apktool 解出 Manifest 然后修改，接着重新打包回去。
- 终极办法，自己编译一个debug版 的rom，这个稍微麻烦一点，自己编一个，想怎么玩就怎么玩。

#####开始debug：
假设你已经把apk安装到设备里了。

接下来用Android Studio import一个 Project , 工程的目录定位到刚刚apk反编译后的的文件夹

使用“Create project from existing sources”一路next到底


![](http://upload-images.jianshu.io/upload_images/2333435-672638df3256555e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
设置Project 的 jdk[File->Project Structure]

![](http://upload-images.jianshu.io/upload_images/2333435-3c6643ed84ede90f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完成之后 点击项目邮件    Mark Directory As->Sources Root


工程配好了，配置debug的端口：
EditConfigurations...


![](http://upload-images.jianshu.io/upload_images/2333435-73c7e2c083783130.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加一个remote调试

![](http://upload-images.jianshu.io/upload_images/2333435-a480e7f553771c9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
修改调试的端口
这里我用8700端口，这里我们需要启动ddms去设置端口号映射，然后apply->ok。

![](http://upload-images.jianshu.io/upload_images/2333435-63ad9b3b1089530e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
好了，project方面就准备好了！Nice!

接下来需要准备的就是如何连上设备debug了！
过程也很简单，启动DDMS
Tools->Android->Android Device Monitor选择你要调试的apk的包名

![](http://upload-images.jianshu.io/upload_images/2333435-ea9ea49d9744a54a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后，开始debug
Run->Debug Smali

![](http://upload-images.jianshu.io/upload_images/2333435-1fe99c8b8e8f811f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

针对DDMS端口转发，也可以手动的制定端口信息，操作如下（smalidea的作者推荐的是ddms的方式）：
1. 拿到apk的包名和启动的Activity  把应用启动起来然后等待debug: adb shell am start -D -S -W packageName(packageName的获取可以反编译AndroidManifest.xml里面有包含)

正常的话，你会看到设备里的应用已经跑起来了，并且有个 Waiting For Debugger 的提示，别关掉它。

2. 拿到程序运行的pid: adb shell ps | grep packageName

3. 端口映射： adb forward tcp:8800 jdwp:5413 这里的8800 是上一步在配置工程中自定义的端口。

官方步骤如图所示：

![](http://upload-images.jianshu.io/upload_images/2333435-a949e69a03db0158.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上就是基于smalidea无源码调试的整个过程，有问题的可留言，我们一起交流学习。

PS：Android端 TCP直连，有资料的童鞋请不吝赐教。谢谢

---
