#Java泛型擦除及其相关内容

我们下面看一个例子：

```java
Class<?> class1=new ArrayList<String>().getClass();
Class<?> class2=new ArrayList<Integer>().getClass();
System.out.println(class1);		//class java.util.ArrayList
System.out.println(class2);		//class java.util.ArrayList
System.out.println(class1.equals(class2));	//true
```

我们看输出发现，class1和class2居然是同一个类型ArrayList，在运行时我们传入的类型变量String和Integer都被丢掉了。Java语言泛型在设计的时候为了兼容原来的旧代码，Java的泛型机制使用了“擦除”机制。我们来看一个更彻底的例子：

```java
class Table {}
class Room {}
class House<Q> {}
class Particle<POSITION, MOMENTUM> {}
//调用代码及输出
List<Table> tableList = new ArrayList<Table>();
Map<Room, Table> maps = new HashMap<Room, Table>();
House<Room> house = new House<Room>();
Particle<Long, Double> particle = new Particle<Long, Double>();
System.out.println(Arrays.toString(tableList.getClass().getTypeParameters()));
System.out.println(Arrays.toString(maps.getClass().getTypeParameters()));
System.out.println(Arrays.toString(house.getClass().getTypeParameters()));
System.out.println(Arrays.toString(particle.getClass().getTypeParameters()));
/**
[E]
[K, V]
[Q]
[POSITION, MOMENTUM]
 */


```
上面的代码里，我们想在运行时获取类的类型参数，但是我们看到返回的都是“形参”。在运行期我们是获取不到任何已经声明的类型信息的。

<font color=red>注意：
编译器虽然会在编译过程中移除参数的类型信息，但是会保证类或方法内部参数类型的一致性。</font>


泛型参数将会被擦除到它的第一个边界（边界可以有多个，重用 extends 关键字，通过它能给与参数类型添加一个边界）。编译器事实上会把类型参数替换为它的第一个边界的类型。如果没有指明边界，那么类型参数将被擦除到Object。下面的例子中，可以把泛型参数T当作HasF类型来使用。

```java
public interface HasF {
    void f();
}

public class Manipulator<T extends HasF> {
    T obj;
    public T getObj() {
        return obj;
    }
    public void setObj(T obj) {
        this.obj = obj;
    }
}
```
extend关键字后后面的类型信息决定了泛型参数能保留的信息。Java类型擦除只会擦除到HasF类型。

##Java泛型擦除的原理
我们通过例子来看一下，先看一个非泛型的版本：
```java
// SimpleHolder.java
public class SimpleHolder {
    private Object obj;
    public Object getObj() {
        return obj;
    }
    public void setObj(Object obj) {
        this.obj = obj;
    }
    public static void main(String[] args) {
        SimpleHolder holder = new SimpleHolder();
        holder.setObj("Item");
        String s = (String) holder.getObj();
    }
}
// SimpleHolder.class
public class SimpleHolder {
  public SimpleHolder();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        

  public java.lang.Object getObj();
    Code:
       0: aload_0       
       1: getfield      #2                  // Field obj:Ljava/lang/Object;
       4: areturn       

  public void setObj(java.lang.Object);
    Code:
       0: aload_0       
       1: aload_1       
       2: putfield      #2                  // Field obj:Ljava/lang/Object;
       5: return        

  public static void main(java.lang.String[]);
    Code:
       0: new           #3                  // class SimpleHolder
       3: dup           
       4: invokespecial #4                  // Method "<init>":()V
       7: astore_1      
       8: aload_1       
       9: ldc           #5                  // String Item
      11: invokevirtual #6                  // Method setObj:(Ljava/lang/Object;)V
      14: aload_1       
      15: invokevirtual #7                  // Method getObj:()Ljava/lang/Object;
      18: checkcast     #8                  // class java/lang/String
      21: astore_2      
      22: return        
}


```
下面我们给出一个泛型的版本，从字节码的角度来看看:

```java
//GenericHolder.java
public class GenericHolder<T> {
    T obj;
    public T getObj() {
        return obj;
    }
    public void setObj(T obj) {
        this.obj = obj;
    }
    public static void main(String[] args) {
        GenericHolder<String> holder = new GenericHolder<>();
        holder.setObj("Item");
        String s = holder.getObj();
    }
}

//GenericHolder.class
public class GenericHolder<T> {
  T obj;

  public GenericHolder();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        

  public T getObj();
    Code:
       0: aload_0       
       1: getfield      #2                  // Field obj:Ljava/lang/Object;
       4: areturn       

  public void setObj(T);
    Code:
       0: aload_0       
       1: aload_1       
       2: putfield      #2                  // Field obj:Ljava/lang/Object;
       5: return        

  public static void main(java.lang.String[]);
    Code:
       0: new           #3                  // class GenericHolder
       3: dup           
       4: invokespecial #4                  // Method "<init>":()V
       7: astore_1      
       8: aload_1       
       9: ldc           #5                  // String Item
      11: invokevirtual #6                  // Method setObj:(Ljava/lang/Object;)V
      14: aload_1       
      15: invokevirtual #7                  // Method getObj:()Ljava/lang/Object;
      18: checkcast     #8                  // class java/lang/String
      21: astore_2      
      22: return        
}


```
在编译过程中，类型变量的信息是能拿到的。所以，set方法在编译器可以做类型检查，非法类型不能通过编译。但是对于get方法，由于擦除机制，运行时的实际引用类型为Object类型。<font color=red>为了“还原”返回结果的类型，编译器在get之后添加了类型转换。所以，在GenericHolder.class文件main方法主体第18行有一处类型转换的逻辑。它是编译器自动帮我们加进去的。</font>
所以在泛型类对象读取和写入的位置为我们做了处理，为代码添加约束。

##Java泛型擦除的缺陷及补救措施

泛型类型不能显式地运用在运行时类型的操作当中，例如：转型、instanceof 和 new。因为在运行时，所有参数的类型信息都丢失了。类似下面的代码都是无法通过编译的：

```java
public class Erased<T> {
    private final int SIZE = 100;
    public static void f(Object arg) {
        //编译不通过
        if (arg instanceof T) {
        }
        //编译不通过
        T var = new T();
        //编译不通过
        T[] array = new T[SIZE];
        //编译不通过
        T[] array = (T) new Object[SIZE];
    }
}


```

那我们有什么办法来补救呢？下面介绍几种方法来一一解决上面出现的问题。

###类型判断问题
我们可以通过下面的代码来解决泛型的类型信息由于擦除无法进行类型判断的问题：

```java
/**
 * 泛型类型判断封装类
 * @param <T>
 */
class GenericType<T>{
    Class<?> classType;

    public GenericType(Class<?> type) {
        classType=type;
    }

    public boolean isInstance(Object object) {
        return classType.isInstance(object);
    }
}


```
在main方法我们可以这样调用：

```java
GenericType<A> genericType=new GenericType<>(A.class);
System.out.println("------------");
System.out.println(genericType.isInstance(new A()));
System.out.println(genericType.isInstance(new B()));
```
我们通过记录类型参数的Class对象，然后通过这个Class对象进行类型判断。

###创建类型实例
泛型代码中不能new T()的原因有两个，一是因为擦除，不能确定类型；而是无法确定T是否包含无参构造函数。

为了避免这两个问题，我们使用显式的工厂模式：
```java
/**
 * 使用工厂方法来创建实例
 *
 * @param <T>
 */
interface Factory<T>{
    T create();
}

class Creater<T>{
    T instance;
    public <F extends Factory<T>> T newInstance(F f) {
    	instance=f.create();
    	return instance;
    }
}

class IntegerFactory implements Factory<Integer>{
    @Override
    public Integer create() {
    	Integer integer=new Integer(9);
    	return integer;
    }
}
```
我们通过工厂模式+泛型方法来创建实例对象，上面代码中我们创建了一个IntegerFactory工厂，用来创建Integer实例，以后代码有变动的话，我们可以添加新的工厂类型即可。

调用代码如下：
```java
Creater<Integer> creater=new Creater<>();
System.out.println(creater.newInstance(new IntegerFactory()));


```
###创建泛型数组

一般不建议创建泛型数组。尽量使用ArrayList来代替泛型数组。但是在这里还是给出一种创建泛型数组的方法。

```java
public class GenericArrayWithTypeToken<T> {
    private T[] array;

    @SuppressWarnings("unchecked")
    public GenericArrayWithTypeToken(Class<T> type, int sz) {
        array = (T[]) Array.newInstance(type, sz);
    }

    public void put(int index, T item) {
        array[index] = item;
    }

    public T[] rep() {
        return array;
    }

    public static void main(String[] args) {

    }
}
```
这里我们使用的还是传参数类型，利用类型的newInstance方法创建实例的方式。
