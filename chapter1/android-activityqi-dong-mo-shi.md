一、启动模式介绍
　　启动模式简单地说就是Activity启动时的策略，在[Android](http://www.linuxidc.com/topicnews.aspx?tid=11)Manifest.xml中的标签的android:launchMode属性设置；
　　启动模式有4种，分别为standard、singleTop、singleTask、singleInstance；
讲解启动模式之前，有必要先讲解一下“任务栈”的概念;
　　任务栈
　　每个应用都有一个任务栈，是用来存放Activity的，功能类似于函数调用的栈，先后顺序代表了Activity的出现顺序；比如Activity1-->Activity2-->Activity3,则任务栈为：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2333435-46a0a550419cdd4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

二、启动模式
(1)standard：每次激活Activity时(startActivity)，都创建Activity实例，并放入任务栈；

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2333435-4615b1462a257bfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(2)singleTop：如果某个Activity自己激活自己，即任务栈栈顶就是该Activity，则不需要创建，其余情况都要创建Activity实例；

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2333435-a71a089ab979fcec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(3)singleTask：如果要激活的那个Activity在任务栈中存在该实例，则不需要创建，只需要把此Activity放入栈顶，并把该Activity以上的Activity实例都pop；

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2333435-35cdee065d09ec0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

(4)singleInstance：如果应用1的任务栈中创建了MainActivity实例，如果应用2也要激活MainActivity，则不需要创建，两应用共享该Activity实例；

 
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2333435-594df9ccf11e19de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
SingTask的应用：
       可以用来退出整个应用。
       将主Activity设为SingTask模式，然后在要退出的Activity中转到主Activity，然后重写主Activity的onNewIntent函数，并在函数中加上一句finish。
 
 
附：
退出单个Activity方法：
      调用finish
　　杀死该进程：killprocess(Process.mId)
      终止正在运行的虚拟机：system.exit()
 
退出整个应用：
　　制造抛异常导致整个程序退出
　　将所有的activity放入到一个list中，然后在需要退出的时候，将所有的activity，finish掉
　　通过广播来完成退出功能
     通过广播来完成退出功能，具体实现过程是这样的：在每个Activity创建时（onCreate时）给Activity注册一个广播接收器，当退出时发送该广播即可。
