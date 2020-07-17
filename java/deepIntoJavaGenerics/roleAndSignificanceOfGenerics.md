深入Java泛型
# 泛型的作用与定义

类型的参数化，就是可以把类型像方法的参数那样传递

泛型使编译器可以在编译期间对类型进行检查以提高类型安全，减少运行时由于对象类型不匹配引发的异常。


##1. 泛型是什么

---

一说到泛型，大伙肯定不会陌生，我们代码里面有很多类似这样的语句：


```java
List<String> list=new ArrayList<>();

```

ArrayList就是个泛型类，我们通过设定不同的类型，可以往集合里面存储不同类型的数据类型（而且只能存储设定的数据类型，这是泛型的优势之一）。“泛型”简单的意思就是泛指的类型（参数化类型）。想象下这样的场景：如果我们现在要写一个容器类（支持数据增删查询的），我们写了支持String类型的，后面还需要写支持Integer类型的。然后呢？Doubel、Float、各种自定义类型？这样重复代码太多了，而且这些容器的算法都是一致的。我们可以通过泛指一种类型T,来代替我们之前需要的所有类型，把我们需要的类型作为参数传递到容器里面，这样我们算法只需要写一套就可以适应所有的类型。最典型的的例子就是ArrayList了，这个集合我们无论传递什么数据类型，它都能很好的工作。

聪明的同学看完上面的描述，灵机一动，写出了下面的代码：

```java
class MyList{
    private Object[] elements=new Object[10];
    private int size;

    public void add(Object item) {
    	elements[size++]=item;
    }

    public Object get(int index) {
    	return elements[index];
    }
}

```

这个代码灵活性很高，所有的类型都可以向上转型为Object类，这样我们就可以往里面存储各种类型的数据了。的确Java在泛型出现之前，也是这么做的。但是这样的有一个问题：如果集合里面数据很多，某一个数据转型出现错误，在编译期是无法发现的。但是在运行期会发生java.lang.ClassCastException。例如：


```java
MyList myList=new MyList();
myList.add("A");
myList.add(1);
System.out.println(myList.get(0));
System.out.println((String)myList.get(1));

```

我们在这个集合里面存储了多个类型（某些情况下容器可能会存储多种类型的数据），如果数据量较多，转型的时候难免会出现异常，而这些都是无法在编译期得知的。而泛型一方面让我们只能往集合中添加一种类型的数据，同时可以让我们在编译期就发现这些错误，避免运行时异常的发生，提升代码的健壮性。

##2. Java泛型介绍
下面我们来介绍Java泛型的相关内容，下面会介绍以下几个方面：

- Java泛型类
- Java泛型方法
- Java泛型接口

###Java泛型类

类结构是面向对象中最基本的元素，如果我们的类需要有很好的扩展性，那么我们可以将其设置成泛型的。假设我们需要一个数据的包装类，通过传入不同类型的数据，可以存储相应类型的数据。我们看看这个简单的泛型类的设计：


```java

class DataHolder<T>{
    T item;

    public void setData(T t) {
    	this.item=t;
    }

    public T getData() {
    	return this.item;
    }
}

```

**泛型类定义时只需要在类名后面加上类型参数即可，当然你也可以添加多个参数，类似于<K,V>,<T,E,K>等。这样我们就可以在类里面使用定义的类型参数。**

泛型类最常用的使用场景就是“元组”的使用。我们知道方法return返回值只能返回单个对象。如果我们定义一个泛型类，定义2个甚至3个类型参数，这样我们return对象的时候，构建这样一个“元组”数据，通过泛型传入多个对象，这样我们就可以一次性方法多个数据了。

###Java泛型方法

前面我们介绍的泛型是作用于整个类的，现在我们来介绍泛型方法。泛型方法既可以存在于泛型类中，也可以存在于普通的类中。<font color=red>如果使用泛型方法可以解决问题，那么应该尽量使用泛型方法。</font>下面我们通过例子来看一下泛型方法的使用：


```java
class DataHolder<T>{
    T item;

    public void setData(T t) {
    	this.item=t;
    }

    public T getData() {
    	return this.item;
    }

    /**
     * 泛型方法
     * @param e
     */
    public <E> void PrinterInfo(E e) {
    	System.out.println(e);
    }
}

```

我们来看运行结果：


```java
1
AAAAA
8.88

```
从上面的例子中，我们看到我们是在一个泛型类里面定义了一个泛型方法printInfo。通过传入不同的数据类型，我们都可以打印出来。在这个方法里面，我们定义了类型参数E。这个E和泛型类里面的T两者之间是没有关系的。哪怕我们将泛型方法设置成这样：


```java
//注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
public <T> void PrinterInfo(T e) {
    System.out.println(e);
}
//调用方法
DataHolder<String> dataHolder=new DataHolder<>();
dataHolder.PrinterInfo(1);
dataHolder.PrinterInfo("AAAAA");
dataHolder.PrinterInfo(8.88f);

```

这个泛型方法依然可以传入Double、Float等类型的数据。泛型方法里面的类型参数T和泛型类里面的类型参数是不一样的类型，从上面的调用方式，我们也可以看出，泛型方法printInfo不受我们DataHolder中泛型类型参数是String的影响。
我们来总结下泛型方法的几个基本特征：


- public与返回值中间非常重要，可以理解为声明此方法为泛型方法。
- 只有声明了的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
- 表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
- 与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。

###Java泛型接口

Java泛型接口的定义和Java泛型类基本相同，下面是一个例子：

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}

```

此处有两点需要注意：

- 泛型接口未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中。

例子如下：


```java

/* 即：class DataHolder implements Generator<T>{
 * 如果不声明泛型，如：class DataHolder implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}

```

- 如果泛型接口传入类型参数时，实现该泛型接口的实现类，则所有使用泛型的地方都要替换成传入的实参类型。例子如下：

```java
class DataHolder implements Generator<String>{
    @Override
    public String next() {
    	return null;
    }
}

```

从这个例子我们看到，实现类里面的所有T的地方都需要实现为String。
