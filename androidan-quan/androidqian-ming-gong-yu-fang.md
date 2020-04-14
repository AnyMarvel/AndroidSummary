---
#一. Android签名背景：
- Android应用使用应用包文件(.apk文件)的形式分发到设备上，由于这个平台的程序主要是用 Java 编写的，所以这种格式与 Java 包的格式 -- jar(Java Archive)有很多共同点，它用于将代码，资源和元数据（来自可选的META-INF目录 ）文件使用 zip 归档算法转换成一个文件。
- 大多数 Android 应用程序都使用开发人员签名的证书（注意 Android 的“证书”和“签名”可以互换使用）。 此证书用于确保原始应用程序的代码及其更新来自同一位置，并在同一开发人员的应用程序之间建立信任关系。 为了执行这个检查，Android 只是比较证书的二进制表示，它用于签署一个应用程序及其更新（第一种情况）和协作应用程序(第二种情况)。
- 但由于Android平台源码的公开性，安全方面也是一个比较严峻的问题。在工作中经常能够遇到恶意破解或严重安全漏洞的情况。Android攻击手段层出不断，目前比较流行的方法就是把签名认证的内容放到动态链接库.so文件中，本文则从JNI签名验证浅谈下Android的攻防问题。
#二. 安全目标
>通常定义的信息安全主要有三大目标：

- 1. 保密性（Confidentiality）：保护信息内容不会被泄露给未授权的实体，防止被动攻击；
- 2. 完整性（Integrity）：保证信息不被未授权地修改，或者如果被修改可以检测出来，防止主动攻击，比如篡改、插入、重放；
- 3. 可用性（Availability）：保证资源的授权用户能够访问到应得资源或服务，防止拒绝服务攻击；
>除了这三点，有时大家也会加上另外两点要求：
- 1. 可控性（Controllability）：限制实体的访问权限，通常是经过认证的合法的实体才可以访问，标识与认证是访问控制的前提；
- 2. 不可抵赖性（Non-repudiation）：防止发送方或者接收方否认传输或者接收过某条消息；
Android提供给我们了一种验证方式：数字签名。但数字签名放到java层代码验证太容易被破解，为了增加破解难度，把验证内容需要转移到native层实现。
#三. JNI注册方式
JNI全称是Java Native Interface（Java本地接口）单词首字母的缩写，本地接口就是指用C和C++开发的接口。由于JNI是JVM规范中的一部份，因此可以将我们写的JNI程序在任何实现了JNI规范的Java虚拟机中运行。同时，这个特性使我们可以复用以前用C/C++写的大量代码。JNI目前提供两种注册方式，静态注册方式实现较为简单，但有一些系列的缺陷，动态注册要复写JNI_OnLoad函数，过程稍微复杂。
##3.1 静态注册方法
>这种方法我们比较常见，但比较麻烦，大致流程如下：
- 1. 实现原理：根据函数名来建立java方法和JNI函数间的一一对应关系。
- 2. 实现过程:
--  先创建Java类，声明Native方法，编译成.class文件。
-- 使用Javah命令生成C/C++的头文件，例如：`javah -jni com.jd.jnidemo.MainActivity`，则会生成一个以.h为后缀的文件`com_jd_jnidemo_MainActivity.h`。
-- 创建.h对应的源文件，然后实现对应的native方法，如下图所示：
![](http://upload-images.jianshu.io/upload_images/2333435-9393c489814fdfff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

- 3. 静态注册的弊端
-- 1. 书写很不方便，因为JNI层函数的名字必须遵循特定的格式，且名字特别长；
-- 2. 会导致程序员的工作量很大，因为必须为所有声明了native函数的java类编写JNI头文件；
-- 3. 程序运行效率低，因为初次调用native函数时需要根据根据函数名在JNI层中搜索对应的本地函数，然后建立对应关系，这个过程比较耗时。

##3.2 动态注册

>动态注册在JNi层实现的，JAVA层不需要关心，因为在system.load时就会去掉JNI_OnLoad,有就注册，没就不注册,因为jni.h里有这么一个结构体，分别如下表示

```
typedef struct {
    const char* name;                                java层函数名
    注:一个签名信息包含JAVA的参数和返回，这个貌似有命令生成javap,应该是
    const char* signature;                         java层函数名的签名信息
    void*       fnPtr;                                      Jni层对应的函数指针。
} JNINativeMethod;
```
- 1.实现原理：直接告诉native函数其在JNI中对应函数的指针；
- 2.实现过程：
-- 1. 利用结构体JNINativeMethod保存Java Native函数和JNI函数的对应关系；
-- 2. 在一个JNINativeMethod数组中保存所有native函数和JNI函数的对应关系；
-- 3. 在Java中通过System.loadLibrary加载完JNI动态库之后，调用JNI_OnLoad函数，开始动态注册；
-- 4. JNI_OnLoad中会调用AndroidRuntime::registerNativeMethods函数进行函数注册；
-- 5. AndroidRuntime::registerNativeMethods中最终调用jniRegisterNativeMethods完成注册。

　　3.优点：克服了静态注册的弊端。


#四. 签名验证
一般情况下为了防止被反编译，会把关键代码写到so文件中(比如加解密)，一般使用到的是在so里加上判断APk包签名是否一致的代码，避免so被二次打包。其实用JNI读签名就是用了Java的反射机制。
##4.1 本地校验
反射代码如下所示：
```Java
    PackageManager pm = context.getPackageManager();
    String packageName = context.getPackageName();
    try {
        PackageInfo packageInfo = pm.getPackageInfo(packageName,
            PackageManager.GET_SIGNATURES);
            Signature sign = info.signatures[0];
            Log.i("test", "hashCode : " + sign.hashCode());
        } catch (NameNotFoundException e) {
            e.printStackTrace();
        }
```
以上我们做了一件事情，获取 PackageInfo 中的 Signature。当然也可以继续获取公钥SHA1如下
```
private byte[] getCertificateSHA1(Context context) {
    PackageManager pm = context.getPackageManager();
    String packageName = context.getPackageName();

    try {
        PackageInfo packageInfo = pm.getPackageInfo(packageName,
            PackageManager.GET_SIGNATURES);
        Signature[] signatures = packageInfo.signatures;
        byte[] cert = signatures[0].toByteArray();
        X509Certificate x509 = X509Certificate.getInstance(cert);
        MessageDigest md = MessageDigest.getInstance("SHA1");
        return md.digest(x509.getEncoded());
    } catch (PackageManager.NameNotFoundException | CertificateException |
            NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
    return null;
}
```
计算出 Signature或计算出SHA1 之后，我们就可以进行对比了。下面我们看看对应的 native 代码。(由于篇幅原因这里列举只计算到Signature的过程)
```
int getSignHashCode(JNIEnv *env, jobject context) {

    jclass context_clazz = (*env)->GetObjectClass(env, context);//Context的类

    jmethodID methodID_getPackageManager = (*env)->GetMethodID(env, context_clazz,
        "getPackageManager", "()Landroid/content/pm/PackageManager;");// 得到 getPackageManager 方法的 ID


    jobject packageManager = (*env)->CallObjectMethod(env, context,
        methodID_getPackageManager);// 获得PackageManager对象

    jclass pm_clazz = (*env)->GetObjectClass(env, packageManager);// 获得 PackageManager 类

    jmethodID methodID_pm = (*env)->GetMethodID(env, pm_clazz, "getPackageInfo",
        "(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;");// 得到 getPackageInfo 方法的 ID

    jmethodID methodID_pack = (*env)->GetMethodID(env, context_clazz,
        "getPackageName", "()Ljava/lang/String;");// 得到 getPackageName 方法的 ID


    jstring application_package = (*env)->CallObjectMethod(env, context,
        methodID_pack);// 获得当前应用的包名

    const char *str = (*env)->GetStringUTFChars(env, application_package, 0);
    __android_log_print(ANDROID_LOG_DEBUG, "JNI", "packageName: %s\n", str);

    jobject packageInfo = (*env)->CallObjectMethod(env, packageManager,
        methodID_pm, application_package, 64);// 获得PackageInfo

    jclass packageinfo_clazz = (*env)->GetObjectClass(env, packageInfo);
    jfieldID fieldID_signatures = (*env)->GetFieldID(env, packageinfo_clazz,
        "signatures", "[Landroid/content/pm/Signature;");
    jobjectArray signature_arr = (jobjectArray)(*env)->GetObjectField(env,
        packageInfo, fieldID_signatures);

    jobject signature = (*env)->GetObjectArrayElement(env, signature_arr, 0);//Signature数组中取出第一个元素

    jclass signature_clazz = (*env)->GetObjectClass(env, signature);//读signature的hashcode
    jmethodID methodID_hashcode = (*env)->GetMethodID(env, signature_clazz,
        "hashCode", "()I");
    jint hashCode = (*env)->CallIntMethod(env, signature, methodID_hashcode);

    __android_log_print(ANDROID_LOG_DEBUG, "JNI", "hashcode: %d\n", hashCode);
    return hashCode;
}
```
本示例中这种认证方式在 Android Studio 中会有一个 Lint 警告，“android-fake-id-vulnerability”，受影响系统版本：部分Android 4.4及所有4.4以下版本,这个问题属于系统bug，在获取cert的方法findCert中判断有缺陷，但在4.4以后大google已经对此修复。
示例中需要传入context,其实context也也可以在native层通过反射的方式拿到，本人感受：native的代码不难就是写起来比较复杂,只要有耐心就可以了。
## 4.2 证书完整性校验
4.1内容是通过context获取的signature获取的签名验证，我们知道在签名后apk文件会多出以下文件
![](http://upload-images.jianshu.io/upload_images/2333435-ead7b8e0c29fd5a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中我们上述过程其实就获取CERT.RSA中的签名，但上述过程依赖context进行依赖认证，攻击者可获取context进行内容替换修改，截取签名替换等方式显示二次打包。
所以，我们可以解析RSA文件，通过本地验证的方式来完成 '证书完整性校验' 。
###4.2.1 了解证书
查看证书指纹.keystores命令

```
keytool   –list –v –keystore debug.keystore
```
查看证书指纹.RSA文件命令
```
keytool –printcert –file CERT.RSA
```
使用openssl查看.RSA文件
```
openssl pkcs7 -inform DER -in CERT.RSA -noout -print_certs –text  
```
查看证书指纹后会发现，RSA文件和.keystores，证书指纹相同，MD5,SHA1,SHA256三种指纹均相同。
###4.2.2 证书格式
1．X.509证书格式如下图所示：

![](http://upload-images.jianshu.io/upload_images/2333435-9be474cfd2a31865.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

• 这里看到证书中并不包含apk签名流程中生成CERT.RSA时对用私钥计算出的签名。所以证书的信息是不会改变的，这也验证了上面所说的RSA中证书指纹和.keystone中的指纹相同的问题

2．对CERT.RSA进行详细解析

明确了上面的问题之后，对CERT.RSA 文件进行详细解析，得到下图：

![](http://upload-images.jianshu.io/upload_images/2333435-c3e05ae4e4b88b05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)



>说明：
- （1）首先，我们的通常所说的证书的签名，是生成证书的时候CA对整个证书的所有域签名的得到的，而不是对某一部分签名得到的。整个签名就是上图中部分一的最下面的一段十六进制的内容；
- （2）编程中的获取到的内容实质上是就是上图中的部分二，这是一个证书的所有内容；
```
    openssl pkcs7  -inform DER -in CERT.RSA  -print_certs
```
- （3）部分一中的公钥等信息就是从部分二中得来的，可以直接在部分二中找到。
- （4）可以猜测，部分一中的其他信息也是从部分二中得来，只不过编码方式不一样，所以显示不同而已。
#4.2.3 RSA加解密实现
由于Android生成的apk文件是以zip文件格式生成的，我们可以查看源码查看Android签名校验机制
可参考：Apk在安装的过程中核心类：
frameworks\base\services\core\java\com\android\server\pm\PackageManagerService.java

```
private void installPackageLI(InstallArgs args, PackageInstalledInfo res) {  
……
}
```
Apk 包中的META-INF目录下，CERT.RSA，它是一个PKCS7 格式的文件。
获取证书的方法如下（上面几张中已经使用openssl获取相关信息）：

```
import sun.security.pkcs.PKCS7;  //注意：需要引入jar包android-support-v4
import java.io.FileInputStream;  
import java.io.IOException;  
import java.security.cert.CertificateException;  
import java.security.cert.X509Certificate;  

public class Test {  
    public static void main(String[] args) throws CertificateException, IOException {  
        FileInputStream fis = new FileInputStream("/home/AnyMarvel/CERT.RSA");  
        PKCS7 pkcs7 = new PKCS7(fis);  
        X509Certificate publicKey = pkcs7.getCertificates()[0];  

        System.out.println("issuer1:" + publicKey.getIssuerDN());  
        System.out.println("subject2:" + publicKey.getSubjectDN());  
        System.out.println(publicKey.getPublicKey());  
    }  
}  
```
也可以转化为native代码进行校验，加固安全性。以上就是目前主流的两种通过签名校验的方式。
#五. 常见的破解方式及加固方案总结
>**破解条件**

- 1.Java层通过
```
   getPackageManager().getPackageInfo.signatures 获取签名信息；
```

- 2.Native方法
/DLL/Lua脚本等通过获取Java的context/Activity对象，反射调用getPackageInfo等来获取签名；
- 3.首先获取apk的路径，定位到META-INF*.RSA文件，获取其中的签名信息；
- 4.能自动识别apk的加固类型；

>**破解方式**
luckypathsign作者提供

- 方式一：substrate框架libhooksig By空道

  - 1.so文件用于hook sign

  - 2.应用于在程序运行时获取当前进程的签名信息而进行的验证；

- 方式二：重写继承类packageInfo和PackageManager By小白

  - 1.适用于Java层packageInfo获取签名信息的方式；

  - 2.亦适用于Native/DLL/LUA层反射packageInfo获取签名信息的方式；

  - 3.该种方式可能会使PackageInfo中的versionCode和versionName为NULL，对程序运行有影响的话，需自主填充修复；

- 方式三：重写继承类，重置Sign信息；

  - 1.适用于Java层packageInfo获取签名信息的方式；

  - 2.亦适用于Native/DLL/LUA层反射packageInfo获取签名信息的方式；

  - 3.该种方式可能会使PackageInfo中的versionCode和versionName为NULL，对程序运行有影响的话，需自主填充修复；

- 方式四：针对定位到具体RSA文件路径获取签名的验证方式；

  - 1.针对定位到具体RSA文件路径获取签名的验证方式；

  - 2.曾经破解过消消乐_Ver1.27,但是如果程序本身对META-INF签名文件中的MANIFEST.MF进行了校验，此方式无效，那就非签名校验，而是文件校验了；

-  方式五：hook android 解析的packageparse类中的两个验证方法
```
  pp.collectCertificates(pkg, parseFlags);
  pp.collectManifestDigest(pkg);
```
修改实现方式。

>**应对方案**

通过以上信息，我们可以得到的是，证书作为不变的内容放在PKCS7格式的.RSA文件中，我们在RSA文件上验证的也只有证书。

- 方案一：通过PackageManag对象可以获取APK自身的签名 这里得到的sign为证书的所有数据，对其做摘要算法，例如:
MD5可以得到MD5指纹，对比指纹可以进行安全验证。
Java程序都可以使用jni反射在native实现，Java代码太容易破解，不建议防止到Java端。方法有很多，最后是都通过

```
  context.getPackageManager().getPackageInfo(
                    this.getPackageName(), PackageManager.GET_SIGNATURES).signatures）
```


- 方案二：调用隐藏类PackageParser.java中的collectCertificates方法，从源头获取cert证书

- 方案三：使用openssl使用JNI做RSA解析破解难度是相当的大，同样的解析出x.509证书，java解析转换为native解析so文件，但得到的文件比较大1.3M。。。。显然不可取（太大）

- 方案四：通过源码解析我们可以知道，apk文件验证是按照zip文件目录形式查找到.RSA文件结尾，我们可以直接去取文件的绝对路径，拿到证书的公钥信息进行验证（但需要引入PKCS7的库）

- 方案五：由棒棒加固和爱加密做的思路可以知道，自己重新定制摘要算法，在asset
里面重新搞一套验证流程。思路就是生成一个定制的CERT，另外开辟一套验证流程，不使用Android固有的签名认证流程。

文章有涉及侵权行为请及时提醒，谢谢
