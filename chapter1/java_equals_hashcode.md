![]()
>java中equals，hashcode和==的区别  相信很多人都很清楚

1. ==运算符是判断两个对象是不是同一个对象，即他们的地址是否相等

2. object类中equals与==是等效的

3. 覆写equals更多的是追求两个对象在逻辑上的相等，你可以说是值相等，也可说是内容相等。（覆盖以后,覆盖equals时总要覆盖hashCode ）

4. hashCode用于返回对象的hash值，主要用于查找的快捷性，因为hashCode也是在Object对象中就有的，所以所有Java对象都有hashCode，在HashTable和HashMap这一类的散列结构中，都是通过hashCode来查找在散列表中的位置的。

知识拓展图(总结拓展如下):

![](/assets/equals_hashcode.jpg)
### 一. Java ==  运算符

java中的数据类型，可分为两类：

1.基本数据类型，也称原始数据类型

byte,short,char,int,long,float,double,boolean   他们之间的比较，应用双等号（==）,比较的是他们的值。

2.引用类型(类、接口、数组)

当他们用（==）进行比较的时候，比较的是他们在内存中的存放地址，所以，除非是同一个new出来的对象，他们的比较后的结果为true，否则比较后结果为false。

对象是放在堆中的，栈中存放的是对象的引用（地址）。由此可见'=='是对栈中的值进行比较的。如果要比较堆中对象的内容是否相同，那么就要重写equals方法了。

### 二. Java里面有==运算符了，为什么还需要equals

equals()的作用是用来判断两个对象是否相等，在Object里面的定义是：

```java
//equals与==是等效的
public boolean equals(Object obj) {
        return (this == obj);
    }
```

这说明在我们实现自己的equals方法之前，equals等价于==,而==运算符是判断两个对象是不是同一个对象，即他们的地址是否相等。而覆写equals更多的是追求两个对象在逻辑上的相等，你可以说是值相等，也可说是内容相等。
String.java中对equals方法的重写

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```
这里对equals重新需要注意五点：
1. 自反性：对任意引用值X，x.equals(x)的返回值一定为true.

2. 对称性：对于任何引用值x,y,当且仅当y.equals(x)返回值为true时，x.equals(y)的返回值一定为true;

3. 传递性：如果x.equals(y)=true, y.equals(z)=true,则x.equals(z)=true

4. 一致性：如果参与比较的对象没任何改变，则对象比较的结果也不应该有任何改变

5. 非空性：任何非空的引用值X，x.equals(null)的返回值一定为false

在以下几种条件中，不覆写equals就能达到目的：

- 类的每个实例本质上是唯一的：强调活动实体的而不关心值得，比如Thread，我们在乎的是哪一个线程，这时候用equals就可以比较了。

- 不关心类是否提供了逻辑相等的测试功能：有的类的使用者不会用到它的比较值得功能，比如Random类，基本没人会去比较两个随机值吧

- 超类已经覆盖了equals，子类也只需要用到超类的行为：比如AbstractMap里已经覆写了equals，那么继承的子类行为上也就需要这个功能，那也不需要再实现了。

- 类是私有的或者包级私有的，那也用不到equals方法：这时候需要覆写equals方法来禁用它：

```java
@Override public boolean equals(Object obj) { throw new AssertionError();}
```

### 三. hashCode，有什么用？

hashCode()方法返回的就是一个数值，从方法的名称上就可以看出，其目的是生成一个hash码。hash码的主要用途就是在对对象进行散列的时候作为key输入，据此很容易推断出，我们需要每个对象的hash码尽可能不同，这样才能保证散列的存取性能。事实上，Object类提供的默认实现确实保证每个对象的hash码不同（在对象的内存地址基础上经过特定算法返回一个hash码）。Java采用了哈希表的原理。哈希（Hash）实际上是个人名，由于他提出一哈希算法的概念，所以就以他的名字命名了。 哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。初学者可以这样理解，hashCode方法实际上返回的就是对象存储的物理地址（实际可能并不是）。


####  hashCode的作用

想要明白，必须要先知道Java中的集合。

总的来说，Java中的集合（Collection）有两类，一类是List，再有一类是Set。前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。

那么这里就有一个比较严重的问题了：要想保证元素不重复，可两个元素是否重复应该依据什么来判断呢？

这就是Object.equals方法了。但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。也就是说，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，它就要调用1000次equals方法。这显然会大大降低效率。
于是，Java采用了哈希表的原理。

这样一来，当集合要添加新的元素时，

先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。

如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；

如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存，不相同就散列其它的地址。所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。

这里要注意：

- 1. equals相等的两个对象，hashCode一定相等
- 2. equals方法不相等的两个对象，hashCode有可能相等

在每个覆盖了equals方法的类中，也必须覆盖hashCode方法，如果不这样做的话，就会违反Object.hashCode的通用约定。从而导致该类无法结合所有基于散列的集合一起正常运作。


参考网站内容如下:
- [equals与hashCode问题记录](https://juejin.im/post/5a4379d4f265da432003874c)
- [java中equals，hashcode和==的区别](https://www.cnblogs.com/kexianting/p/8508207.html)
- [JVM的内存区域划分](https://www.cnblogs.com/dolphin0520/p/3613043.html)
- [集合框架 Collection、Map](https://www.jianshu.com/p/589d58033841)
- [自动装箱与拆箱的实现原理](https://www.jianshu.com/p/0ce2279c5691 )

请关注--------喘口仙氣

![](https://upload-images.jianshu.io/upload_images/2333435-11195db3d5590ecf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
