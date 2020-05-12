>解决Android8.0之后开启service时报错IllegalStateException: Not allowed to start service Intent ...

#背景:

项目测试时发现的，在双击返回键关闭应用后（并未杀死后台）重新打开APP，其他手机都OK，但是8.0的手机会出现较频繁的crash。检查代码，问题锁定在重新开启应用时的startService()上。

查找资料说是Android 8.0 不再允许后台service直接通过startService方式去启动，否则就会引起IllegalStateException

#原因
Android 8.0 有一项复杂功能；系统不允许后台应用创建后台服务。 因此，Android 8.0 引入了一种全新的方法，即 Context.startForegroundService()，以在前台启动新服务。
在系统创建服务后，应用有5秒的时间来调用该服务的 startForeground() 方法以显示新服务的用户可见通知。如果应用在此时间限制内未调用 startForeground()，则系统将停止服务并声明此应用为 ANR。

##遇到的问题
但是目前在调用：context.startForegroundService(intent)时报如下ANR，startForegroundService()文档说明在service启动后要调用startForeground()。

```shell
android.app.RemoteServiceException: Context.startForegroundService() did not then call Service.startForeground()

```

完整解决步骤:

###1. 添加权限

```
<!--android 9.0上使用前台服务，需要添加权限,此权限为级别为nomarl-->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```
###2. 启动server(引用启动5秒内要启动server)
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        context.startForegroundService(new Intent(context, MyService.class));
    } else {
        context.startService(new Intent(context, MyService.class));
    }

```
然后必须在Myservice中调用startForeground():

###3. Server中onCreate方法中调用startForeground()
```java
public static final String CHANNEL_ID_STRING = "service_01";
private Notification notification;
    @Override
    public void onCreate() {
        super.onCreate();
        //适配8.0service
        NotificationManager notificationManager = (NotificationManager) MyApp.getInstance().getSystemService(Context.NOTIFICATION_SERVICE);
        NotificationChannel mChannel = null;
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            mChannel = new NotificationChannel(CHANNEL_ID_STRING, getString(R.string.app_name),
                    NotificationManager.IMPORTANCE_LOW);
            notificationManager.createNotificationChannel(mChannel);
            notification = new Notification.Builder(getApplicationContext(), CHANNEL_ID_STRING).build();
            startForeground(1, notification);
        }
}

```
###4. 在onStart里再次调用startForeground()
```java
@Override
public void onStart(Intent intent, int startId) {
    super.onStart(intent, startId);

    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
        startForeground(1, notification);
    }

}
```

###注解:
- Android 8.0 系统不允许后台应用创建后台服务，故只能使用Context.startForegroundService()启动服务

- 创建服务后，应用必须在5秒内调用该服务的 startForeground() 显示一条可见通知，声明有服务在挂着，不然系统会停止服务 + ANR 套餐送上。

- Notification 要加 Channel，系统的要求

- <font color=red>为什么要在onStart里再次调用startForeground()？</font>答：这一条主要是针对后台保活的服务，如果在服务A运行期间，保活机制又startForegroundService启动了一次服务A，那么这样不会调用服务A的onCreate方法，只会调用onStart方法。如果不在onStart方法里再挂个通知的话，系统会认为你使用了 startForegroundService 却不在 5 秒内给通知，很傻地就停止服务 + ANR 套餐送上了。
