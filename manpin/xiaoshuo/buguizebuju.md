完整的开源上线项目，有兴趣的同学可以start
[漫品官网](http://aimanpin.com/) http://aimanpin.com/
[开源详情](https://github.com/AnyMarvel/ManPinAPP) https://github.com/AnyMarvel/ManPinAPP

今天给大家介绍一种**Android 万能的不规则图形绘制**的方法。

Android绘制不规则图形目前主要有三种方式可以实现：

- 一、 PorterDuffXfermode方式
- 二、 BitmapShader方式
- 三、 ClipPath方式

然而这三个API的坑多的数不胜数，这里不再一一介绍，有兴趣的可以去逐一尝试。今天主要介绍一种万能的不规则图形定义的方法

写这篇文章的来源于我要制作的首页，大家一起来看下效果图：

![图片]()

这看起来是不是挺特别的，但是违背了Android UI排列的常理，所以只能采用自定义view的方式解决了，但这并不仅仅是画几个path切几张图的事情。

---
正文这里开始
