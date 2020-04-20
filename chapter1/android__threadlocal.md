# Android 之 ThreadLocal简析
### 前言
源码 基于 Android SDK 28    JDK 1.8

---
> 说起 ThreadLocal，大家可能会比较陌生，但是如果想要比较好地理解 Android 的消息机制，ThreadLocal 是必须要掌握的，这是因为 Looper 的工作原理，就跟 ThreadLocal 有很大的关系，理解 ThreadLocal 的实现方式有助于我们理解 Looper 的工作原理，这篇文章就从 ThreadLocal 的用法讲起，一步一步带大家理解 ThreadLocal。

### 一、ThreadLocal 是什么

* * *

ThreadLocal 是一个线程内部的数据存储类，通过它可以在 **指定的线程中** 存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。

一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用 ThreadLocal。

### 二、基本用法

* * *

#### 创建，支持泛型

```java
ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<>();
```

#### set 方法

```java
mBooleanThreadLocal.set(false);
```

#### get 方法

```java
mBooleanThreadLocal.get()
```

接下来用一个完整的例子，帮助大家理解 ThreadLocal

```java


public class MainActivity extends AppCompatActivity {

    String TAG = "ThreadLocal";
    ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<>();

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mBooleanThreadLocal.set(true);
        Log.d(TAG, "Current Thread: mBooleanThreadLocal is : " + mBooleanThreadLocal.get());
        new Thread("Thread#1") {
            @Override
            public void run() {
                super.run();
                mBooleanThreadLocal.set(false);
                Log.d(TAG, "Thread 1: mBooleanThreadLocal is : " + mBooleanThreadLocal.get());
            }
        }.start();
        new Thread("Thread#2") {
            @Override
            public void run() {
                super.run();
                Log.d(TAG, "Thread 2: mBooleanThreadLocal is : " + mBooleanThreadLocal.get());
            }
        }.start();

    }
}

```

在上面的代码中，在主线程中设置 mBooleanThrealLocal 的值为 true，在子线程 1 中设置为 false，在子线程 2 中不设置 mBooleanThrealLocal 的值，然后分别在 3 个线程中通过 get() 方法获取 mBooleanThrealLocal 的值
![](https://upload-images.jianshu.io/upload_images/2333435-065930bf7d33ced4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



从上面的日志中可以看出，虽然在不同的线程中访问的是同一个 ThrealLocal 对象，但是它们通过 ThrealLocal 获取到的值确实不一样的，这就是 ThrealLocal 的奇妙之处了。

ThrealLocal 之所以有这么奇妙的效果，就是因为不同线程访问同一个 ThrealLocal 的 get() 方法，ThrealLocal 内部都会从各自的线程中取出一个数组，然后再从数组中根据当前 ThrealLocal 的索引去查找不同的 value 值。

### 三、ThrealLocal 工作原理

* * *

ThrealLocal 是一个泛型类，它的定义为 public class ThrealLocal<T>，只要弄清楚 ThrealLocal 的 set() 和 get() 方法就可以明白它的工作原理了。

#### 1、ThrealLocal 的 set() 方法

```java
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

可以看到在 set() 方法中，先获取到当前线程，然后通过 getMap(Thread t) 方法获取一个 ThreadLocalMap，如果这个 map 不为空的话，就将 ThrealLocal 和 我们想存放的 value 设置进去，不然的话就创建一个 ThrealLocalMap 然后再进行设置。

```java
    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

ThreadLocalMap 其实是 ThreadLocal 里面的静态内部类，而每一个 Thread 都有一个对应的 ThrealLocalMap，因此获取当前线程的 ThrealLocal 数据就变得异常简单了。

```java
public class Thread implements Runnable {

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

```

下面看一下，ThrealLocal 的值到底是如何在 threadLocals 中进行存储的。
首先我们先看一下set方法中的
```java
map.set(this, value);
```
对应源码为:
```java
/**
 * Set the value associated with key.
 *
 * @param key the thread local object
 * @param value the value to be set
 */
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
可以看到set方法中会获取Entry对象，然后判断当前threadlocal对象是否存储在table数组中，若存在则更新value，若不存在，则新增一个Entry对象'''new Entry(key, value);'''存储threadlocal和对应的值，然后加入到table数组中。
在 threadLocals 内部有一个数组，private Entry[] table，ThrealLocal 的值就存在这个 table 数组中。

```java

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;


```

#### 2、ThrealLocal 的 get() 方法

上面分析了 ThreadLocal 的 set() 方法，这里分析它的 get() 方法，代码如下

```java
    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }

```

可以发现，ThrealLocal 的 get() 方法的逻辑也比较清晰，它同样是取出当前线程的 threadLocals 对象，如果这个对象为 null，就调用 setInitialValue() 方法

```java
    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

在 setInitialValue() 方法中，将 initialValue() 的值赋给我们想要的值，默认情况下，initialValue() 的值为 null，当然也可以重写这个方法。

```java
   /**
     * Returns the current thread's "initial value" for this
     * thread-local variable.  This method will be invoked the first
     * time a thread accesses the variable with the {@link #get}
     * method, unless the thread previously invoked the {@link #set}
     * method, in which case the <tt>initialValue</tt> method will not
     * be invoked for the thread.  Normally, this method is invoked at
     * most once per thread, but it may be invoked again in case of
     * subsequent invocations of {@link #remove} followed by {@link #get}.
     *
     * <p>This implementation simply returns <tt>null</tt>; if the
     * programmer desires thread-local variables to have an initial
     * value other than <tt>null</tt>, <tt>ThreadLocal</tt> must be
     * subclassed, and this method overridden.  Typically, an
     * anonymous inner class will be used.
     *
     * @return the initial value for this thread-local
     */
    protected T initialValue() {
        return null;
    }

```

如果 threadLocals 对象不为 null 的话，那就取出它的 table 数组并找出 ThreadLocal 的 reference 对象在 table 数组中的位置。

从 ThreadLocal 的 set() 和 get() 方法可以看出，他们所操作的对象都是当前线程的 threalLocals 对象的 table 数组，因此在不同的线程中访问同一个 ThreadLocal 的 set() 和 get() 方法，他们对 ThreadLocal 所做的 `读 / 写` 操作权限仅限于各自线程的内部，这就是为什么可以在多个线程中互不干扰地存储和修改数据。

#### 总结

ThreadLocal 是线程内部的数据存储类，每个线程中都会保存一个 `ThreadLocal.ThreadLocalMap threadLocals = null;`，ThreadLocalMap 是 ThreadLocal 的静态内部类，里面保存了一个 `private Entry[] table` 数组，这个数组就是用来保存 ThreadLocal 中的值。通过这种方式，就能让我们在多个线程中互不干扰地存储和修改数据。
