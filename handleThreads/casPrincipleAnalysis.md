CAS的英文为Compare and Swap 翻译为比较并交换。

CAS加volatile关键字是实现并发包的基石。没有CAS就不会有并发包，synchronized是一种独占锁、悲观锁，java.util.concurrent中借助了CAS指令实现了一种区别于synchronized的一种乐观锁。

#什么是乐观锁与悲观锁？

悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样当第二个线程想拿这个数据的时候，第二个线程会一直堵塞，直到第一个释放锁，他拿到锁后才可以访问。传统的数据库里面就用到了这种锁机制，例如：行锁，表锁，读锁，写锁，都是在操作前先上锁。java中的synchronized的实现也是一种悲观锁。

乐观锁：乐观锁概念为，每次拿数据的时候都认为别的线程不会修改这个数据，所以不会上锁，但是在更新的时候会判断一下在此期间别的线程有没有修改过数据，乐观锁适用于读操作多的场景，这样可以提高程序的吞吐量。在Java中java.util.concurrent.atomic包下面的原子变量就是使用了乐观锁的一种实现方式CAS实现。

乐观锁的实现方式-CAS（Compare and Swap）：

jdk1.5之前锁存在的问题：

java在1.5之前都是靠synchronized关键字保证同步，synchronized保证了无论哪个线程持有共享变量的锁，都会采用独占的方式来访问这些变量。这种情况下：

- 1.在多线程竞争下，加锁、释放锁会导致较多的上下文切换和调度延时，引起性能问题

- 2.如果一个线程持有锁，其他的线程就都会挂起，等待持有锁的线程释放锁。

- 3.如果一个优先级高的线程等待一个优先级低的线程释放锁，会导致优先级倒置，引起性能风险。

对比于悲观锁的这些问题，另一个更加有效的锁就是乐观锁。 乐观锁就是：每次不加锁而是假设没有并发冲突去操作同一变量，如果有并发冲突导致失败，则重试直至成功。

**乐观锁**：

乐观锁（ Optimistic Locking ）在上文已经说过了，其实就是一种思想。相对悲观锁而言，乐观锁假设认为数据一般情况下不会产生并发冲突，所以在数据进行提交更新的时候，才会正式对数据是否产生并发冲突进行检测，如果发现并发冲突了，则让返回用户错误的信息，让用户决定如何去做。

上面提到的乐观锁的概念中其实已经阐述了它的具体实现细节：主要就是两个步骤：冲突检测和数据更新。其实现方式有一种比较典型的就是 Compare and Swap ( CAS )。

**乐观锁的一种典型实现机制（CAS）**:


乐观锁主要就是两个步骤：冲突检测和数据更新。当多个线程尝试使用CAS同时更新同一个变量时，只有一个线程可以更新变量的值，其他的线程都会失败，失败的线程并不会挂起，而是告知这次竞争中失败了，并可以再次尝试。

CAS操作包括三个操作数：需要读写的内存位置(V)、预期原值(A)、新值(B)。如果内存位置与预期原值的A相匹配，那么将内存位置的值更新为新值B。如果内存位置与预期原值的值不匹配，那么处理器不会做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS其实就是一个：我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。这其实和乐观锁的冲突检测+数据更新的原理是一样的。

乐观锁是一种思想，CAS只是这种思想的一种实现方式。

JAVA对CAS的支持：

在JDK1.5新增的java.util.concurrent(JUC java并发工具包)就是建立在CAS之上的。相比于synchronized这种堵塞算法，CAS是非堵塞算法的一种常见实现。所以JUC在性能上有了很大的提升。

下面通过看下并发包中的原子操作类AtomicInteger来看下，如何在不使用锁的情况下保证线程安全，主要看下getAndIncrement方法，相当于i++的操作：

```java
public class AtomicInteger extends Number implements java.io.Serializable {  
    private volatile int value;

    public final int get() {  
        return value;  
    }  

    public final int getAndIncrement() {  
        for (;;) {  
            int current = get();  
            int next = current + 1;  
            if (compareAndSet(current, next))  
                return current;  
        }  
    }  

    public final boolean compareAndSet(int expect, int update) {  
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
    }  
}

```

首先value使用了volatile修饰，这就保证了他的可见性与有序性，getAndIncrement采用CAS操作，每次从内存中读取数据然后将数据进行+1操作，然后对原数据，+1后的结果进行CAS操作，成功的话返回结果，否则重试直到成功为止。其中调用了compareAndSet利用JNI（java navite Interface navite修饰的方法，都是java调用其他语言的方法来实现的）来完成CPU的操作。其中compareAndSwapInt类似如下逻辑：

```java
if (this == expect) {
     this = update
     return true;
 } else {
     return false;
 }
```
this == expect和this = update，这两个步骤是如何保证原子性的呢？

JAVA实现CAS的原理：

compareAndSwapInt是借助C来调用CPU底层指令实现的。下面从分析比较常用的CPU（intel x86）来解释CAS的实现原理。下面是sun.misc.Unsafe类的compareAndSwapInt()方法的源代码：

```java
public final native boolean compareAndSwapInt(Object o, long offset,
                                               int expected, int x);

```

再看下在JDK中依次调用的C++代码为：

```c
#define LOCK_IF_MP(mp) __asm cmp mp, 0  \
                       __asm je L0      \
                       __asm _emit 0xF0 \
                       __asm L0:

inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  // alternative for InterlockedCompareExchange
  int mp = os::is_MP();
  __asm {
    mov edx, dest
    mov ecx, exchange_value
    mov eax, compare_value
    LOCK_IF_MP(mp)
    cmpxchg dword ptr [edx], ecx
  }
}
```

如上面源代码所示，程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在多处理器上运行，就为cmpxchg指令加上lock前缀（lock cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）。

#CAS的缺陷：

##1.ABA问题

比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但可能存在潜藏的问题。如下所示：

![](https://img-blog.csdn.net/2018081309490438?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

现有一个用单向链表实现的堆栈，栈顶为A，这时线程T1已经知道A.next为B，然后希望用CAS将栈顶替换为B：　head.compareAndSet(A,B); 在T1执行上面这条指令之前，线程T2介入，将A、B出栈，再pushD、C、A，此时堆栈结构如下图，而对象B此时处于游离状态：

![](https://img-blog.csdn.net/20180813094930693?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

此时轮到线程T1执行CAS操作，检测发现栈顶仍为A，所以CAS成功，栈顶变为B，但实际上B.next为null，所以此时的情况变为：

![](https://img-blog.csdn.net/20180813094950294?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

其中堆栈中只有B一个元素，C和D组成的链表不再存在于堆栈中，平白无故就把C、D丢掉了。

解决方法（AtomicStampedReference 带有时间戳的对象引用）：

从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

```java
public boolean compareAndSet(
               V      expectedReference,//预期引用
               V      newReference,//更新后的引用
              int    expectedStamp, //预期标志
              int    newStamp //更新后的标志
)
```

##2.循环时间长开销大

自旋CAS（不成功，就一直循环执行，直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

##3.只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

CAS与Synchronized的使用情景：　　　

1、对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。

2、对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

补充： synchronized在jdk1.6之后，已经改进优化。synchronized的底层实现主要依靠Lock-Free的队列，基本思路是自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。在线程冲突较少的情况下，可以获得和CAS类似的性能；而线程冲突严重的情况下，性能远高于CAS。

concurrent包的实现：

由于java的CAS同时具有 volatile 读和volatile写的内存语义，因此Java线程之间的通信现在有了下面四种方式：

- 1. A线程写volatile变量，随后B线程读这个volatile变量。

- 2. A线程写volatile变量，随后B线程用CAS更新这个volatile变量。

- 3. A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。

- 4. A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。

Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持原子性读-改-写指令的计算机器，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式：

- 1. 首先，声明共享变量为volatile；　　

- 2. 然后，使用CAS的原子条件更新来实现线程之间的同步；

- 3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

AbstractQueuedSynchronizer（AQS抽象的队列同步器），atomic类，非堵塞数据结构这些concurrent包中的基础类都是使用这种模式实现的，而concurrent包中的高层类又是依赖于这些基础类实现的。concurrent包的整体示意图如下：

![](https://img-blog.csdn.net/20180813101953553?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTEzNjA0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

JVM中的CAS（堆中对象的分配）：　

Java调用new object()会创建一个对象，这个对象会被分配到JVM的堆中。那么这个对象到底是怎么在堆中保存的呢？

首先，new object()执行的时候，这个对象需要多大的空间，其实是已经确定的，因为java中的各种数据类型，占用多大的空间都是固定的（对其原理不清楚的请自行Google）。那么接下来的工作就是在堆中找出那么一块空间用于存放这个对象。

在单线程的情况下，一般有两种分配策略：

1. 指针碰撞：这种一般适用于内存是绝对规整的（内存是否规整取决于内存回收策略），分配空间的工作只是将指针像空闲内存一侧移动对象大小的距离即可。

2. 空闲列表：这种适用于内存非规整的情况，这种情况下JVM会维护一个内存列表，记录哪些内存区域是空闲的，大小是多少。给对象分配空间的时候去空闲列表里查询到合适的区域然后进行分配即可。

　　　　但是JVM不可能一直在单线程状态下运行，那样效率太差了。由于再给一个对象分配内存的时候不是原子性的操作，至少需要以下几步：查找空闲列表、分配内存、修改空闲列表等等，这是不安全的。解决并发时的安全问题也有两种策略：

1. CAS：实际上虚拟机采用CAS配合上失败重试的方式保证更新操作的原子性，原理和上面讲的一样。

2. TLAB：如果使用CAS其实对性能还是会有影响的，所以JVM又提出了一种更高级的优化策略：每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲区（TLAB），线程内部需要分配内存时直接在TLAB上分配就行，避免了线程冲突。只有当缓冲区的内存用光需要重新分配内存的时候才会进行CAS操作分配更大的内存空间。
虚拟机是否使用TLAB，可以通过-XX:+/-UseTLAB参数来进行配置（jdk5及以后的版本默认是启用TLAB的）。
