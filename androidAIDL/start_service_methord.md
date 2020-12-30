service的隐式启动和显示启动

有些时候我们使用Service的时需要采用隐私启动的方式，但是Android 5.0一出来后，其中有个特性就是Service Intent  must be explitict，也就是说从Lollipop开始，service服务必须采用显示方式启动。  

而android源码是这样写的（源码位置：sdk/sources/android-21/android/app/ContextImpl.java）：

可产考： http://androidxref.com/5.0.0_r2/xref/frameworks/base/core/java/android/app/ContextImpl.java

```java
private void validateServiceIntent(Intent service) {
        if (service.getComponent() == null && service.getPackage() == null) {
            if (getApplicationInfo().targetSdkVersion >= Build.VERSION_CODES.LOLLIPOP) {
                IllegalArgumentException ex = new IllegalArgumentException(
                        "Service Intent must be explicit: " + service);
                throw ex;
            } else {
                Log.w(TAG, "Implicit intents with startService are not safe: " + service
                        + " " + Debug.getCallers(2, 3));
            }
        }
    }

```

既然，源码里是这样写的，那么这里有两种解决方法：  

1、设置Action和packageName：  

参考代码如下：


```java
Intent mIntent = new Intent();
mIntent.setAction("XXX.XXX.XXX");//你定义的service的action
mIntent.setPackage(getPackageName());//这里你需要设置你应用的包名
context.startService(mIntent);

```

此方式是google官方推荐使用的解决方法。


在此附上地址供大家参考：http://developer.android.com/goo ... tml#billing-service,有兴趣的可以去看看。


2、设置ComponentName：


Intent intent = new Intent();
ComponentName componentName = new ComponentName(pkgName,serviceName);
intent.setComponent(componentName);
context.startService(intent);

补充知识点： 在Android5.0之前的显示和隐式启动service

隐式启动

AndroidManifest.xml 中定义service

```xml
<service
    android:name=".monke.monkeybook.service.DownloadService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.mp.android.apps.monke.monkeybook.service.DownloadService_action" />
    </intent-filter>
</service>

```

java 启动

```java
    Intent serviceIntent = new Intent();
    serviceIntent.setAction("com.mp.android.apps.monke.monkeybook.service.DownloadService_action");
    serviceIntent.setPackage(getPackageName());
    startService(serviceIntent);
```

显示启动

```java

final Intent serviceIntent=new Intent(this,service.class);
startService(serviceIntent);

```

不同进程的显式启动，需要带上applicationId,service的全限定名就可以了
