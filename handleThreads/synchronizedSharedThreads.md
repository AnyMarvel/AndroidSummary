#Synchronized深入分析

## 1. 基本使用
Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法。

Synchronized的作用主要有三个：

- 1. 原子性: 确保线程互斥的访问同步代码；
- 2. 可见性: 保证共享变量的修改能够及时可见，其实是通过Java内存模型中的 “<font color=red>对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值</font>” 来保证的；
- 3. 有序性: 有效解决重排序问题，即 “一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作”；

从语法上讲，Synchronized可以把任何一个非null对象作为"锁"，在HotSpot JVM实现中，锁有个专门的名字：<font color=red>对象监视器（Object Monitor）</font>。

Synchronized总共有三种用法：
- 1. 当synchronized作用在实例方法时，监视器锁（monitor）便是对象实例（this）；
- 2. 当synchronized作用在静态方法时，监视器锁（monitor）便是对象的Class实例，因为Class数据存在于永久代，因此静态方法锁相当于该类的一个全局锁；
- 3. 当synchronized作用在某一个对象实例时，监视器锁（monitor）便是括号括起来的对象实例；

注意，synchronized 内置锁 是一种 对象锁（锁的是对象而非引用变量），<font color=red>作用粒度是对象 ，可以用来实现对 临界资源的同步互斥访问 ，是 可重入 的。其可重入最大的作用是避免死锁，</font>如：
>> <font color=red>子类同步方法调用了父类同步方法，如没有可重入的特性，则会发生死锁；</font>

## 2. 同步原理

数据同步需要依赖锁，那锁的同步又依赖谁？<font color=red> synchronized给出的答案是在软件层面依赖JVM，而j.u.c.Lock给出的答案是在硬件层面依赖特殊的CPU指令。</font>

当一个线程访问同步代码块时，首先是需要得到锁才能执行同步代码，当退出或者抛出异常时必须要释放锁，那么它是如何来实现这个机制的呢？我们先看一段简单的代码：
