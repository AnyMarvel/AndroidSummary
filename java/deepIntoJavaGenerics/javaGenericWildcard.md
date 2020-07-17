# 通配符与嵌套
##上界通配符<? extends T>
我们先来看一个例子：

```java
class Fruit {}
class Apple extends Fruit {}


```
现在我们定义一个盘子类：

```java
class Plate<T>{
    T item;
    public Plate(T t){
        item=t;
    }

    public void set(T t) {
        item=t;
    }

    public T get() {
        return item;
    }
}


```
下面，我们定义一个水果盘子，理论上水果盘子里，当然可以存在苹果

```java
Plate<Fruit> p=new Plate<Apple>(new Apple());


```
你会发现这段代码无法进行编译。装苹果的盘子”无法转换成“装水果的盘子：

```java
cannot convert from Plate<Apple> to Plate<Fruit>


```
从上面代码我们知道，就算容器中的类型之间存在继承关系，但是Plate和Plate两个容器之间是不存在继承关系的。 在这种情况下，Java就设计成Plate<? extend Fruit>来让两个容器之间存在继承关系。我们上面的代码就可以进行赋值了


```java
Plate<? extends Fruit> p=new Plate<Apple>(new Apple());


```
Plate<? extend Fruit>是Plate< Fruit >和Plate< Apple >的基类。
我们通过一个更加详细的例子来看一下上界的界限：
```java
class Food{}

class Fruit extends Food {}
class Meat extends Food {}

class Apple extends Fruit {}
class Banana extends Fruit {}
class Pork extends Meat{}
class Beef extends Meat{}

class RedApple extends Apple {}
class GreenApple extends Apple {}


```
在上面这个类层次中，Plate<? extend Fruit>，覆盖下面的蓝色部分：


![](https://user-gold-cdn.xitu.io/2018/8/2/164fa0eece27880a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
如果我们往盘子里面添加数据，例如：

```java
p.set(new Fruit());
p.set(new Apple());


```
你会发现无法往里面设置数据，按道理说我们将泛型类型设置为? extend Fruit。按理说我们往里面添加Fruit的子类应该是可以的。但是Java编译器不允许这样操作。<font color=red><? extends Fruit>会使往盘子里放东西的set()方法失效。但取东西get()方法还有效</font>

原因是：
Java编译期只知道容器里面存放的是Fruit和它的派生类，具体是什么类型不知道，可能是Fruit？可能是Apple？也可能是Banana，RedApple，GreenApple？编译器在后面看到Plate< Apple >赋值以后，盘子里面没有标记为“苹果”。只是标记了一个占位符“CAP#1”，来表示捕获一个Fruit或者Fruit的派生类，具体是什么类型不知道。所有调用代码无论往容器里面插入Apple或者Meat或者Fruit编译器都不知道能不能和这个“CAP#1”匹配，所以这些操作都不允许。


<font color=red>最新理解：
一个Plate<? extends Fruit>的引用，指向的可能是一个Plate类型的盘子，要往这个盘子里放Banana当然是不被允许的。我的一个理解是：Plate<? extends Fruit>代表某个只能放某种类型水果的盘子，而不是什么水果都能往里放的盘子</font>

但是上界通配符是允许读取操作的。例如代码：

```java
Fruit fruit=p.get();
Object object=p.get();


```
这个我们很好理解，由于上界通配符设定容器中只能存放Fruit及其派生类，那么获取出来的我们都可以隐式的转为其基类（或者Object基类）。所以上界描述符Extends适合频繁读取的场景。


##下界通配符<? super T>
下界通配符的意思是容器中只能存放T及其T的基类类型的数据。我们还是以上面类层次的来看，<? super Fruit>覆盖下面的红色部分：

![](https://user-gold-cdn.xitu.io/2018/8/2/164fb1a62ffd54c3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

下界通配符<? super T>不影响往里面存储，但是读取出来的数据只能是Object类型。
原因是：
下界通配符规定了元素最小的粒度，必须是T及其基类，那么我往里面存储T及其派生类都是可以的，因为它都可以隐式的转化为T类型。但是往外读就不好控制了，里面存储的都是T及其基类，无法转型为任何一种类型，只有Object基类才能装下。

##PECS原则
最后简单介绍下Effective Java这本书里面介绍的PECS原则。

<font color=red>
上界<? extends T>不能往里存，只能往外取，适合频繁往外面读取内容的场景。
下界<? super T>不影响往里存，但往外取只能放在Object对象里，适合经常往里面插入数据的场景。
</font>

##<?>无限通配符

无界通配符 意味着可以使用任何对象，因此使用它类似于使用原生类型。但它是有作用的，原生类型可以持有任何类型，而无界通配符修饰的容器持有的是某种具体的类型。举个例子，在List<\?>类型的引用中，不能向其中添加Object, 而List类型的引用就可以添加Object类型的变量。

<font color=red>
最后提醒一下的就是，List<\Object>与List<?>并不等同，List<\Object>是List<?>的子类。还有不能往List<?> list里添加任意对象，除了null。</font>
