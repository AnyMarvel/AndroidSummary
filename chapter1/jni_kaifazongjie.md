Android Ndk开发常用网站收集，真正的高手并不是掌握所有的API而是需要的时候可以快速的找到要使用的API。

---
基础知识请移步：http://blog.csdn.net/xyang81/article/details/41777471

JNI动态加载: http://www.cnblogs.com/skywang12345/archive/2013/05/23/3092491.html
JNI中Ｃ调用java的方法:http://www.cnblogs.com/xitang/p/4174619.html

JNI读取应用签名：https://www.pocketdigi.com/20141129/1398.html

NDK开发之日志打印：http://blog.csdn.net/u012702547/article/details/48222859

---

以上是学习和使用jni常用的几种方式，上述文章内容并不完全正确，稍加修改可正常使用，有需要的可以收藏下。
这篇文章主要介绍JNI开发中遇到的坑以及解决的方法。

![](http://upload-images.jianshu.io/upload_images/2333435-fffb78f81d894be2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
---
####一.静态注册和动态注册
为什么需要注册？其实就是给Java的native函数找到底层C,C++实现的函数指针。

- #####静态注册：
通过包名类名一致来确认，Java有一个命令javah，专门生成某一个JAVA文件所有的native函数的头文件(h文件)，步骤如下，我们只说Android项目下如何实施，其实理解了都一样
静态方法注册JNI有哪些缺点？

1.必须遵循某些规则
2.名字过长
3,多个class需Javah多遍，
4.运行时去找效率不高
- #####动态注册 ：
在JNi层实现的，JAVA层不需要关心，因为在system.load时就会去掉JNI_OnLoad,有就注册，没就不注册。

- #####区别：

静态注册是用到时加载，动态注册一开始就加载好了，这个可以从DVM的源代码看出来。

---
####二.Ｃ反射JAVA 的各种方法

TestClass类包一个构造方法、一个成员方法，一个静态方法，一个内部类，大多数的类都是由这三种方法组成的。下面要做的就是怎么在JNI调用这些方法。
```
package com.lb6905.jnidemo;

import android.util.Log;

public class TestClass {
    private final static String TAG = "TestClass";

    public TestClass(){
        Log.i(TAG, "TestClass");
    }

    public void test(int index) {
        Log.i(TAG, "test : " + index);
    }

    public static void testStatic(String str) {
        Log.i(TAG, "testStatic : " + str);
    }

    public static class InnerClass {
        private int num;
        public InnerClass() {
            Log.i(TAG, "InnerClass");
        }

        public void setInt(int n) {
            num = n;
            Log.i(TAG, "setInt: num = " + num);
        }
    }
}
```
查看方法签名

进入到classpath目录下:
命令: cd app/build/intermediates/classes/debug

查看外部类的签名
javap -s -p com.lb6905.jnidemo.TestClass

查看内部类的签名
javap -s -p com.lb6905.jnidemo.TestClass$InnerClass

结果如下：
```
F:\Apps\jniDemo\JNIDemo\app\build\intermediates\classes\debug>javap -s -p com.lb6905.jnidemo.TestClass
Compiled from "TestClass.java"
public class com.lb6905.jnidemo.TestClass {
  private static final java.lang.String TAG;
    descriptor: Ljava/lang/String;
  public com.lb6905.jnidemo.TestClass();
    descriptor: ()V

  public void test(int);
    descriptor: (I)V

  public static void testStatic(java.lang.String);
    descriptor: (Ljava/lang/String;)V
}

F:\Apps\jniDemo\JNIDemo\app\build\intermediates\classes\debug>javap -s -p com.lb6905.jnidemo.TestClass$InnerClass
Compiled from "TestClass.java"
public class com.lb6905.jnidemo.TestClass$InnerClass {
  private int num;
    descriptor: I
  public com.lb6905.jnidemo.TestClass$InnerClass();
    descriptor: ()V

  public void setInt(int);
    descriptor: (I)V
}
```
在JNI中反射调用上述方法

```
JNIEXPORT void JNICALL Java_com_lb6905_jnidemo_MainActivity_JNIReflect
(JNIEnv *env, jobject thiz)
{
    //实例化Test类
    jclass testclass = (*env)->FindClass(env, "com/lb6905/jnidemo/TestClass");
    //构造函数的方法名为<init>
    jmethodID testcontruct = (*env)->GetMethodID(env, testclass, "<init>", "()V");
    //根据构造函数实例化对象
    jobject testobject = (*env)->NewObject(env, testclass, testcontruct);

    //调用成员方法，需使用jobject对象
    jmethodID test = (*env)->GetMethodID(env, testclass, "test", "(I)V");
    (*env)->CallVoidMethod(env, testobject, test, 1);

    //调用静态方法
    jmethodID testStatic = (*env)->GetStaticMethodID(env, testclass, "testStatic", "(Ljava/lang/String;)V");
    //创建字符串，不能在CallStaticVoidMethod中直接使用"hello world!"，会报错的
    jstring str = (*env)->NewStringUTF(env, "hello world!");
    //调用静态方法使用的是jclass，而不是jobject
    (*env)->CallStaticVoidMethod(env, testclass, testStatic, str);

    //实例化InnerClass子类
    jclass innerclass = (*env)->FindClass(env, "com/lb6905/jnidemo/TestClass$InnerClass");
    jmethodID innercontruct = (*env)->GetMethodID(env, innerclass, "<init>", "()V");
    jobject innerobject = (*env)->NewObject(env, innerclass, innercontruct);

    //调用子类的成员方法
    jmethodID setInt = (*env)->GetMethodID(env, innerclass, "setInt", "(I)V");
    (*env)->CallVoidMethod(env, innerobject, setInt, 2);
}
```
>此处需要注意
在C中：
(*env)->方法名(env,参数列表)
在C++中：
env->方法名(参数列表)

####三.NewStringUTF函数请慎用
经常在使用    jstring     (*NewStringUTF)(JNIEnv*, const char*);函数的过程中遇到如下错误
(1) .JNI DETECTED ERROR IN APPLICATION: input is not valid Modified UTF-8: illegal start byte 0x80

![](http://upload-images.jianshu.io/upload_images/2333435-d632d6cc5237f151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

JNI调用newStringUTF时遇到不认识的字符串就直接出错退出~~,网上原因是dalvik/vm/CheckJni.c里面的checkUtfString函数检查通不过.

遇到问题时请排查以下几种问题
- 1.　检查包名反射引用是否正确
- 2.　检查方法签名，参数签名是否正确
- 3.　将char * 定义更换为const char *

(2). JNI DETECTED ERROR IN APPLICATION: use of deleted weak global reference 0xb305f57b

![](http://upload-images.jianshu.io/upload_images/2333435-1e39094d717aa3f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

string 对象和jstring对象从java传值或者新new 的字符串不能够直接引用，必须经过NewStringUTF进行转换，才可
####四.推荐几种修改过的类型转换
(1)jstring转换为char*
```
//jstring转为char* NewStringUTF所需要的内容位char*格式
const char *jstringTochar(JNIEnv *env, jstring jstr) {
    char *rtn = NULL;
    jclass clsstring = (*env)->FindClass(env, "java/lang/String");
    jstring strencode = (*env)->NewStringUTF(env, "utf-8");
    jmethodID mid = (*env)->GetMethodID(env, clsstring, "getBytes", "(Ljava/lang/String;)[B");
    jbyteArray barr = (jbyteArray) (*env)->CallObjectMethod(env, jstr, mid, strencode);
    jsize alen = (*env)->GetArrayLength(env, barr);
    jbyte *ba = (*env)->GetByteArrayElements(env, barr, JNI_FALSE);
    if (alen > 0) {
        rtn = (char *) malloc(alen + 1);
        memcpy(rtn, ba, alen);
        rtn[alen] = 0;
    }
    (*env)->ReleaseByteArrayElements(env, barr, ba, 0);
    return rtn;
}
```
(2)char * 转为jstring

```
jstring chartoJstring(JNIEnv *env, const char *pat) {
    jclass strClass = (*env)->FindClass(env, "java/lang/String");
    jmethodID ctorID = (*env)->GetMethodID(env, strClass, "<init>", "([BLjava/lang/String;)V");
    jbyteArray bytes = (*env)->NewByteArray(env, (jsize) strlen(pat));
    (*env)->SetByteArrayRegion(env, bytes, 0, (jsize) strlen(pat), (jbyte *) pat);
    jstring encoding = (*env)->NewStringUTF(env, "utf-8");
    return (jstring) (*env)->NewObject(env, strClass, ctorID, bytes, encoding);
}
```
(3)jstring 转为jbyte*

```
// java中的jstring, 转化为c的一个字符数组
jbyte *Jstring2Jbyte(JNIEnv *env, jstring jstr) {
    jclass clsstring = (*env)->FindClass(env, "java/lang/String");
    jstring strencode = (*env)->NewStringUTF(env, "UTF-8");
    jmethodID mid = (*env)->GetMethodID(env, clsstring, "getBytes", "(Ljava/lang/String;)[B");

    jbyte *barr = (jbyteArray) (*env)->CallObjectMethod(env, jstr, mid,
                                                        strencode); // String .getByte("UTF-8");
    return barr;
}
```

以上格式转换大家看到基本上都c反射java进行的格式转换，所以反射这块还是要多多了解的。

---

####五.　内存管理
JNI内存管理请参考:https://www.ibm.com/developerworks/cn/java/j-lo-jnileak/
JNI 编程实现了 native code 和 Java 程序的交互，因此 JNI 代码编程既遵循 native code 编程语言的编程规则，同时也遵守 JNI 编程的文档规范。在内存管理方面，native code 编程语言本身的内存管理机制依然要遵循，同时也要考虑 JNI 编程的内存管理。

1、什么需要释放？　

什么需要什么呢 ？ JNI 基本数据类型是不需要释放的 ， 如 jint , jlong , jchar 等等 。 我们需要释放是引用数据类型，当然也包括数组家族。如：jstring ，jobject ，jobjectArray，jintArray 等等。
当然，大家可能经常忽略掉的是 jclass ，jmethodID ， 这些也是需要释放的哦

2、如何去释放？

1) 释放String
```
jstring jstr = NULL;
char* cstr = NULL;
//调用方法
jstr = (*jniEnv)->CallObjectMethod(jniEnv, mPerson, getName);
cstr = (char*) (*jniEnv)->GetStringUTFChars(jniEnv,jstr, 0);
__android_log_print(ANDROID_LOG_INFO, "JNIMsg", "getName  ---- >  %s",cstr );
//释放资源
(*jniEnv)->ReleaseStringUTFChars(jniEnv, jstr, cstr);
(*jniEnv)->DeleteLocalRef(jniEnv, jstr);
```
2) 释放 类 、对象、方法
```
(*jniEnv)->DeleteLocalRef(jniEnv, XXX);//“XXX” 代表 引用对象
```
3) 释放 数组家族
```
jobjectArray arrays = NULL;
jclass jclsStr = NULL;
jclsStr = (*jniEnv)->FindClass(jniEnv, "java/lang/String");
arrays = (*jniEnv)->NewObjectArray(jniEnv, len, jclsStr, 0);
(*jniEnv)->DeleteLocalRef(jniEnv, jclsStr);  //释放String类
(*jniEnv)->DeleteLocalRef(jniEnv, arrays); //释放jobjectArray数组
```
native method 调用 DeleteLocalRef() 释放某个 JNI Local Reference 时，首先通过指针 p 定位相应的 Local Reference 在 Local Ref 表中的位置，然后从 Local Ref 表中删除该 Local Reference，也就取消了对相应 Java 对象的引用（Ref count 减 1）。

---
end:以上就是开发中对jni的一些总结，有错误的地方请及时支出。本文仅供参考学习，转载请注明出处。谢谢

[https://androidsummary.gitbook.io/androidsummary/](https://androidsummary.gitbook.io/androidsummary/)

关注公众号:喘口仙氣  实时关注更新状态

![](/assets/qrcode_for_gh_db8538619cdd_258.jpg)
