[TOC]

# 面试题记录

## 1.谈一谈对java平台的理解

首先，java是一种**面对对象**的语言，具有**易维护**（封装了细节）、**易扩展**（子类可以扩展新功能）等优势。

其次，其显著的特性有两点：

- write once , run anywhere 一次编写，到处运行：提供了跨平台能力

  *java源代码-->编译为字节码-->由JVM解释为机器码。因此可以实现跨平台的能力，因为只需要平台提供虚拟机即可。*

- 垃圾回收机制：不用自己担心内存的分配与回收

最后谈一谈JRE与JDK的区别：

- JRE：java运行环境，包含了JVM和java类库等
- JDK：java开发工具，包含了JRE和编译器及一些诊断工具

![](C:\Users\Jacob\Desktop\新建文件夹\图片\面试1.png)

## 2. Exception与Error的关系，运行时异常与一般异常的区别

首先，**Exception与Error都是继承自Throwable类。**

![](C:\Users\Jacob\Desktop\新建文件夹\图片\面试2.png)

其次：

- Exception是程序正常运行中，可以预料的意外情况，并且可以被捕获。
- 而Error是正常情况下不太可能出现的情况，例如OutOfMemoryError之类的。

而进一步的，Exception又被分为一般异常与运行时异常

- 一般异常：必须在代码中进行显示的捕获处理
- 运行时异常：不属于编译器检查的一部分，例如NPE与数组越界等

还有一个比较经典的问题就是**NoClassDefFoundError**与**ClassNotFoundException**的区别：

- **NoClassDefFoundError**：编译时class存在，但是运行时找不到class文件
- **ClassNotFoundException **：使用Class.forName等出现的运行时异常

## 3.try-with-resource的使用

引入该语句的主要目的就是为了简化我们的代码开发

**任何实现自AutoCloseable的对象都能够被当成是一个resource。**

例如：

```java
public class TryWithResource implements AutoCloseable{
    @Override
    public void close() throws Exception {
        System.out.println("close");
    }
}
```

```java
public class TryWithResourceTest {
    public static void main(String[] args) {
        try(TryWithResource tryWithResource = new TryWithResource()){
            System.out.println("do something");
        }catch (Exception e){

        }
    }
}
```

## 4. 谈谈final、finally、 finalize有什么不同？

首先final可以用来修饰类、方法和变量，其修饰不同的对象有不同的意义。

- 类：该类无法被继承
- 方法：无法被重写（与private区别在于可以被调用）
- 变量：值无法被修改，实际上是不能修改栈中的值（即对应的地址不能被改变）。

finally则是java保证代码一定要被执行的一种机制，但注意使用try-finally实际上会影响性能。

finalize 是object的一个方法，在垃圾回收的第一次扫描会调用该方法完成特定资源的回收。

在并发场景下设计Immutable类注意的点：

- class声明为final
- 成员变量定义为private final ，并且不要实现setter
- 构造方法使用深拷贝
- getter方法使用copy-on-write原则，指的是可以用多个线程请求相同的资源，但是当写入资源的时候需要复用一个专用的副本给写入资源的线程去操作。

## 5.强引用、软引用、弱引用、幻象引用，及其具体使用场景

首先我们要知道，不同的引用类型主要体现的是对象不同的**可达性状态**和对**垃圾收集**的影响

1. 强引用：常见的普通对象引用，只要有一个强引用，就表明对象还“活着”。
2. 软引用：将对象设置为软引用可以让其豁免一些垃圾回收，只有当JVM认为内存不足时（outofmemory），才会试图回收软引用指向的对象。软引用通常用来实现内存敏感的缓存。
3. 弱引用：不能够使对象豁免于垃圾收集
4. 虚引用：不能通过其访问对象，只是利用其finalize方法在其被回收的时候做某些操作。

![](C:\Users\Jacob\Desktop\新建文件夹\图片\面试3.png)



- 软引用和弱引用都可以和引用队列联合使用，如果引用的对象被垃圾回收，JVM会把引用加入与之关联的引用队列中。

- 虚引用的get会得到null。

- 错误的使用强引用会导致内存泄漏，比如赋值为static会导致无法变回弱引用的可达状态。

## 6. String、StringBuffer、StringBuilder 有什么区别？

首先String是一个不可变的类，是因为其被声明为final class并且其关键的char[] 数组也是final定义的，同时String并没有为我们提供修改char[]数组的任何方法，因此我们说String是一个不可变的类。

那为什么String要被设计为一个**不可变**的类呢？这主要是出于一些安全性的考虑：

- 首先是当string作为hashmap的key值时候不可变保证了hashcode的唯一性
- 其次，String经常被用来作为网络URL、文件路径等，如果string不是固定的那么容易引起安全隐患

但任何东西都是一把双刃剑，不可变也会带来一定的性能问题。

因此JDK中引入了**StringBuilder**与**StringBuffer**。

首先**StringBuilder**与**StringBuffer**都是继承自**AbstractStringBuilder**，与String相比较，AbstractStringBuilder的char[] 数组并非final修饰，同时其提供了append方法可以像char[]数组中添加元素。与ArrayList集合一样，作为动态数组，都需要考虑其数组的扩容问题。

在append方法中往char[]数组中添加数据之前首先会判断当前数组可容纳范围，扩容的话是Math.max(双倍扩容+2,添加后最小需要的容量)；初始化默认容量是16.因此在某些能够确定容量的情况下可以通过初始化对应的容量来避免频繁的扩容。

其次，StringBuilder与StringBuffer的区别仅在于StringBuffer在其方法上加锁以应对可能会出现的并发问题，但也会由于加锁而降低性能，因此在线程安全下通常使用StringBuilder。

然后还可以再谈一下String的第二个特性：**常量池的优化**：当多个String对象拥有相同的值时，他们仅引用同一个常量池的拷贝。

-------------------------------------------------------这里可能需要结合一些JVM的知识------------------------------------------------------------------------------

## 7.动态代理是基于什么原理？

**首先，动态代理是基于反射原理的**。

而java的反射机制是指在程序的运行状态时能够动态的获取一个类的属性信息以及调用它的方法。

- 通过反射的方法获取实例的私有属性

```java
public static void privateContent() throws Exception{
    Class<?> scholarClass = Class.forName("Scholar");
    Object instance = scholarClass.getConstructor().newInstance();
    Field publicField = scholarClass.getDeclaredField("privateContent");
    //当调用私有方法时，需要设置权限
    publicField.setAccessible(true);
    publicField.set(instance,"privateContent");
}
```

- 通过反射的方法调用实例的私有方法（带参数）

```java
    Class<?> scholarClass = Class.forName("Scholar");
    Object instance = scholarClass.getConstructor().newInstance();
    Method publicMethod = scholarClass.getDeclaredMethod("privateMethod",String.class);
    publicMethod.setAccessible(true);
    publicMethod.invoke(instance,str);
```

- 通过反射的方法调用实例的公共方法（无参数）

```java
public static void publicMethod() throws Exception{
    Class<?> scholarClass = Class.forName("Scholar");
    Object instance = scholarClass.getConstructor().newInstance();
    Method publicMethod = scholarClass.getMethod("publicMethod");
    publicMethod.invoke(instance);
}
```

**而基于反射的动态代理是一种方便运行时处理方法调用的机制，常被用到RPC调用与面向切面编程，还有例如日志处理、通用的事务框架等。可以简单的看作是对调用目标的一个包装**

最后动态代理的实现方法也有很多：

- JDK动态代理：基于接口，优势在于减少的类的依赖关系
- 基于CGLIB的动态代理：基于子类，优势在于高性能。

**动态代理的实现原理：**

- 在我们调用Proxy.newInstance的时候会为我们生成一个代理类proxy0，这个代理类继承了Proxy并且实现了我们传入的接口。
- 实现接口的方法后，其调用了super.h.invoke（this,method,args）。这个super.h就是我们实现InvocationHandler接口的对象，通过调用其invoke从而完成了动态代理的过程。

**一个简单的例子做一个说明：**

```java
public interface MyInterface {
    void play();
}
```

```java
public class TargetObject implements MyInterface{
    @Override
    public void play() {
        System.out.println("方法调用");
    }
}
```

```java
public class ProxyFactory implements InvocationHandler {
    private Object target = null;
    public Object getInstance(Object o){
        target = o;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        System.out.println("前置增强");
        method.invoke(target,args);
        System.out.println("后置增强");
        return result;
    }
}
```

```java
public class Test {
    public static void main(String[] args) {
        MyInterface targetObject = new TargetObject();
        ProxyFactory proxyFactory = new ProxyFactory();
        MyInterface target = (MyInterface) proxyFactory.getInstance(targetObject);
        target.play();
        System.out.println(target.getClass());
    }
}
```

在我们调用

```java
return Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
```

之后，JVM就会帮我们生成Proxy0对象：

```java
public final class $Proxy0 extends Proxy implements MyInterface {
    private static Method m1;
    private static Method m0;
    private static Method m3;
    private static Method m2;

    static {
        try {
            $Proxy0.m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            $Proxy0.m0 = Class.forName("java.lang.Object").getMethod("hashCode", (Class<?>[])new Class[0]);
            // 实例化 MyInterface 的 play()方法
            $Proxy0.m3 = Class.forName("com.shuitu.test.MyInterface").getMethod("play", (Class<?>[])new Class[0]);
            $Proxy0.m2 = Class.forName("java.lang.Object").getMethod("toString", (Class<?>[])new Class[0]);
        }
        catch (NoSuchMethodException ex) {
            throw new NoSuchMethodError(ex.getMessage());
        }
        catch (ClassNotFoundException ex2) {
            throw new NoClassDefFoundError(ex2.getMessage());
        }
    }

    public $Proxy0(final InvocationHandler invocationHandler) {
        super(invocationHandler);
    }

    public final void play() {
        try {
        	// 这个 h 其实就是我们调用 Proxy.newProxyInstance()方法 时传进去的 ProxyFactory对象(它实现了
            // InvocationHandler接口)，该对象的 invoke()方法 中实现了对目标对象的目标方法的增强。
        	// 看到这里，利用动态代理实现方法增强的实现原理就全部理清咯
            super.h.invoke(this, $Proxy0.m3, null);
        }
        catch (Error | RuntimeException error) {
            throw new RuntimeException();
        }
        catch (Throwable t) {
            throw new UndeclaredThrowableException(t);
        }
    }

    public final boolean equals(final Object o) {
        try {
            return (boolean)super.h.invoke(this, $Proxy0.m1, new Object[] { o });
        }
        catch (Error | RuntimeException error) {
            throw new RuntimeException();
        }
        catch (Throwable t) {
            throw new UndeclaredThrowableException(t);
        }
    }

    public final int hashCode() {
        try {
            return (int)super.h.invoke(this, $Proxy0.m0, null);
        }
        catch (Error | RuntimeException error) {
            throw new RuntimeException();
        }
        catch (Throwable t) {
            throw new UndeclaredThrowableException(t);
        }
    }

    public final String toString() {
        try {
            return (String)super.h.invoke(this, $Proxy0.m2, null);
        }
        catch (Error | RuntimeException error) {
            throw new RuntimeException();
        }
        catch (Throwable t) {
            throw new UndeclaredThrowableException(t);
        }
    }
}
```

## 8. int 和 Integer 有什么区别？谈谈 Integer 的值缓存范围。

首先int属于java的8种基本数据类型：

- boolean、byte : 1个字节
- short、char：2个字节
- int、float：4个字节
- double、long：8个字节

而Integer是int的一个包装类，其提供了一些数学运算啊，int与字符串的转换之类的基本操作。

同时，Integer实现了**自动装箱**与**自动拆箱**。

例如：下面的代码种，JVM会将222通过调用valueOf自动装箱为Integer，也会将i1通过调用intValue自动拆箱为i1。

```java
public class IntegerTest {
    public static void main(String[] args) {
        Integer i1 = 222;
        int i2 = i1;
    }
}
```

通过反编译以上代码，我们能够得出以上的结论：

```
  public static void main(java.lang.String[]);
    Code:
       0: sipush        222
       3: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       6: astore_1
       7: aload_1
       8: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
      11: istore_2
      12: return
```

而谈到valueOf，这里设计一个Integer的缓存机制，与String一样，Integer也做了缓存，默认的缓存是（-128，127）。也就是说，在自动装箱时，调用valueOf会触发缓存机制：

```java
public static void testCatch(){
    Integer i1 = 127;
    Integer i2 = 127;
    System.out.println(i1 == i2);//true
    Integer i3 = 128;
    Integer i4= 128;
    System.out.println(i3 == i4);//false
}
```

为什么会是这样的结果我们可以从Integer的源码中得到答案：

在调用valueOf之后实际上它会做一个判断就是你传入的值是否是处于[IntegerCache.low,IntegerCache.high]之类的，如果是则直接返回缓存的Integer对象。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

**在能够使用int时应当取避免使用Integer，这是因为创建对象的开销大于直接使用int**

对象包含以下三个元素：

- 对象头：Mark Word（锁标记位，标记为偏向锁、轻量级锁还是重量级锁）；指向class对象的指针。
- 实际的对象数据
- 对齐填充

##  9.Vector、ArrayList、LinkedList 有何区别？

首先，他们都实现了List接口，表明他们都像是一个有序集合（可以使用Colletions.sort对其进行排序，实际上底层还是调用的Arrays.sort）。他们的区别在于：

- Vector：**线程安全**的动态数组，扩容时扩大一倍
- ArrayList：线程不安全的动态数组，扩容时增加0.5倍
- LinkedList：线程不安全的双向链表

当应用场景偏向于随机访问时，Vector与ArrayList性能较好，如果偏向于插入删除那么LinkedList性能较好。

**双轴快速排序**，**TimSort**

## 10.对比 Hashtable、HashMap、TreeMap 的不同？谈谈你对 HashMap 的掌握。

首先，他们都实现了Map接口。区别在于：

- hashtable：线程安全的、不支持null键和值，由于同步开销已经不推荐使用
- hashmap：非线程安全的、支持null键和值。
- TreeMap：基于红黑树的顺序访问map，其get、put、remove操作都是logn的。

**HashMap的源码分析：**

如果能够知道 HashMap 要存取的键值对数量，可以考虑预先设置合适的容量大小。具体数值我们可以根据扩容发生的条件来做简单预估，根据前面的代码分析，我们知道它需要符合计算条件：

```
负载因子 * 容量 > 元素数量
```

所以，预先设置的容量需要满足，大于“预估元素数量 / 负载因子”，**同时它是 2 的幂数**，结论已经非常清晰了。

关于为什么HashMap要树化本质上这是一个安全问题，因为在元素放置过程中，如果一个对象哈希冲突，都被放置到同一个桶里，则会形成一个链表，我们知道链表查询是线性的，会严重影响存取的性能。而在现实世界，构造哈希冲突的数据并不是非常复杂的事情，恶意代码就可以利用这些数据大量与服务器端交互，导致服务器端 CPU 大量占用，这就构成了哈希碰撞拒绝服务攻击，国内一线互联网公司就发生过类似攻击事件。

**哈希冲突的解决方法**

## 11.ConcurrentHashMap

首先是为什么要ConcurrentHashMap，这是因为HashTable本身是通过synchronized加锁，实际上比较低效；而利用Collection.SynchronizedMap()实际上也是加锁的方式。

**源码解析**：

首先是参数介绍：

|                   参数名称                    | 默认值 |
| :-------------------------------------------: | :----: |
|           MAXIMUM_CAPACITY 最大容量           | 1<<30  |
|          DEFAULT_CAPACITY  初始容量           |   16   |
|    DEFAULT_CONCURRENCY_LEVEL 默认并发等级     |   16   |
|             LOAD_FACTOR 扩容因子              | 0.75f  |
|          Node<K,V>[] table 底层数组           |  null  |
|        Node<K,V>[] nextTable 底层数组         |        |
| sizeCtl：负数表示正在初始化（-1）或者调整大小 |   0    |

- 首先分析sizeCtl的作用：

如果说是采用默认的构造函数生成的化，sizeCtl会默认为16.

在调用put函数后会最终调用initTable方法，该方法内会初始化大小为sizeCtl的Node数组。并在初始化完成之后将sizeCtl缩小到3/4。

**而initTable为了应对并发问题，首先通过sizeCtl判断是否有某个线程正在对数组进行初始化，这里主要是利用cas操作去完成的。**

源码如下：

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

而其Put函数为了应对并发问题主要利用了两个点：

**一是在数组对应位置没有值得情况下使用cas操作去存放对应得值。**

**二是在出现hash冲突得时候通过synchronized加锁去避免冲突。**

其源码解析如下：

1. 首先key与value都不能为Null,否则抛出NPE异常
2. 计算hash值，最高位通过&0x7fffffff置为0
3. 开始循环
4. 如果node数组为null或者空的，那么调用initable初始化
5. 如果寻址到数组的某个值为空，则利用cas直接放置值
6. 否则对node节点加锁，再进插入操作
   - 遍历链表（每次bincount++），查找是否存在对应的key值：1hash值是否相等 2.地址或者equals是否相等
   - 找到，则保存旧值到一个变量，然后放入新的值（最后返回旧的值）
   - 找不到，插入到最后一个结点
7. 如果bincount大于限定值8，则转化为红黑树

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

## 12.java的io了解吗？NIO是如何实现多路复用的？

首先什么是IO呢，在Unix系统中一切都是基于文件的，而文件是由流来表示的，因此对数据的操作实际上就是对流的输入输出操作。简称IO。

我们所提到的IO通常指的是：

- 网络IO：例如socket之类的，java.net、Java.nio包下的一些api。
- 磁盘IO：文件输入输出流等

具体在Java中的话，IO主要分为

- BIO
- NIO
- AIO

需要注意的地方：什么场景用什么IO，NIO高性能是基于什么原理，存在的问题。

操作字节的、字符的类要熟悉。

## 13.java的文件拷贝方式了解吗？哪一种最好？

首先文件拷贝实际上也是基于IO实现的。

- BIO：基于流实现文件拷贝

- NIO：基于通道实现文件拷贝

```java
![面试4](C:\Users\Jacob\Desktop\新建文件夹\图片\面试4.png)public class FileCopy {
    public static void main(String[] args) {
        String source = "C:/Users/Jacob/Desktop/source.txt";
        String target = "C:/Users/Jacob/Desktop/target.txt";
        copyFileByStream(source, target);
        copyFileByChannel(source,target);
    }
    //基于流实现BIO
    public static void copyFileByStream(String pathSource, String pathTarget) {
        try (InputStream is = new FileInputStream(pathSource);
             OutputStream os = new FileOutputStream(pathTarget)
        ) {
            byte[] buffer = new byte[128];
            int len = 0;
            while ((len = is.read(buffer)) > 0) {
                os.write(buffer, 0, len);
            }
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
    //基于通道实现NIO
    public static void copyFileByChannel(String pathSource, String targetSource) {
        try (FileChannel inputChannel = new FileInputStream(pathSource).getChannel();
             FileChannel outputChannel = new FileOutputStream(targetSource).getChannel()
        ) {
            for (long count = inputChannel.size(); count > 0; ) {
                long transferred = inputChannel.transferTo(inputChannel.position(),count,outputChannel);
                count -= transferred;
            }
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

对比这两种方式的话，由于NIO更能够利用现代操作系统底层机制，因此其性能更好。

这是因为java中基于流的BIO在操作时实际上需要经历以下的操作过程：

1. 将数据从磁盘拷贝到内核中
2. 将内核中的数据拷贝到用户空间

这两个过程都是阻塞的，同时用户态与内核态切换也会带来额外的开销。

但是java 基于NIO的transferTo实现方式在linux下会使用零拷贝的手段，避免了上下文切换的开销。

<img src="C:\Users\Jacob\Desktop\新建文件夹\图片\面试4.png" style="zoom:67%;" />

## 14.接口和抽象类的区别

首先说一下接口。

- 首先从接口本身来说，其是对行为的一种抽象，任何实现某一个接口的类都变现为is like a，表示其具有这样的行为，像是一个什么东西。
- 其次，从接口的属性与方法来说，接口的成员属性全是public static final默认修饰的。其方法要么是抽象的，要么是静态的，如果是静态方法，则需要实现方法体。
- 类通过implements关键字实现接口

java中有许多的接口，例如util包下的List接口。

再谈一谈抽象类。

- 首先从抽象类本身来说，其设计目的就是为了代码重用，把多个类的重复代码抽象在一起然后通过继承以实现代码复用的目的。
- 其次，从抽象的属性上来说与普通的java类并无太大区别。方法上来说区别仅仅是可能有抽象方法的存在。
- 抽象类不能够实例化，但是它可以有构造方法，可以通过子类调用。
- 通过extends关键字继承抽象类

java中有许多的抽象类，例如util包下的AbstractList接口。

## 15.谈一谈java的三大特性，封装、继承和多态

- 封装：封装了代码的内部实现细节，提高了安全性和简化了编程。
- 继承：代码重用的基础
- 多态：提高多态我会想到方法的重写、重载（编译时多态），向上转型等

需要注意的是，实际上java对方法的标注利用的是**类名+方法名+参数**，因此参数不同算是重载，但是返回值不同则会在编译器就抛出一般异常。

## 16.synchronized 和 ReentrantLock 有什么区别？有人说 synchronized 最慢，这话靠谱吗？

首先，这两种方式都可以实现我们的**线程安全**。他们都具有可重入性，但使用ReentrantLock使用起来更加的灵活，例如其可以公平锁的方式去创建，而synchronized则只能够是非公平的。公平的意思是在获取锁时会首先将锁给到等待时间最长的线程，时间上也就是会给到等待队列的首部对应的线程。

**线程安全：**

定义：多线程情况下共享变量可修改的状态的正确性。

线程安全需要保证的特性：

- 原子性：相关操作不会被其他线程打扰
- 可见性：一个线程修改了某个共享变量，其状态能够马上被其他线程知晓
- 有序性：主要避免指令重排序带来的问题

**synchronized：**翻遍以后可以看到内部实际上用monitorenter----monitorexit实现。**而Monitor是实现对象同步的基本单元。**

## 17.synchronized 底层如何实现？什么是锁的升级、降级？

首先synchronized 代码块是由一对monitorenter/monitorexit实现的。**Monitor是实现对象同步的基本单元。**

JVM提供了三种不同Monitor的基本实现：

- 偏向锁
- 轻量级锁
- 重量级锁

而所谓锁的升降级实际上就是JVM优化Synchronized代码块的机制，当检索到不同的竞争状态时会切换到适合的锁实现。

其切换过程大概如下：

- 偏向锁：利用CAS将对象头中Mark word部分设置为该线程的ID。
- 轻量级锁：如果某个线程尝试锁定某个已经被偏向锁锁住的对象，则java会撤销偏向锁并切换到轻量级锁的实现。此时，同样会使用CAS获取锁，如果失败则再升级为重量级锁。
- 重量级锁：普通认知中的锁。

## 18.一个线程两次调用start()方法会出现什么情况？

会报错，当线程不处于新建状态的适合，调用start方法会抛出异常：

```java
throw new IllegalThreadStateException();
```

这是因为在调用start后会对线程的状态进行检查。如下代码所示：

```java
public synchronized void start() {
    if (threadStatus != 0)//0代表的是NEW状态
        throw new IllegalThreadStateException();
    group.add(this);
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
        }
    }
}
```

从start中源码中实际上可以看出，首先先会判断线程的状态是否是NEW（0），如果不是则会抛出违规的线程状态异常，否则将该线程加入线程组然后调用本地方法start0()，实际上start0内部调用了我们的run方法。

在java中，线程的状态被定义在枚举类Thread.State中。

```java
public enum State {
    NEW,//线程刚被new出来，还未调用start
    RUNNABLE,//就绪态，调用了start，可以是正在运行也可以是等待cpu分配时间片
    BLOCKED,//阻塞态，尝试获取锁被阻塞
    WAITING,//等待态，调用wait或者join会进入等待态
    TIMED_WAITING,//调用带超时时间的wait或者join函数
    TERMINATED;//线程中止状态
}
```

转换过程如下：

<img src="C:\Users\Jacob\Desktop\新建文件夹\图片\面试5.png" style="zoom:75%;" />

## 19.死锁了解吗，什么情况话会产生死锁？怎么排查死锁？

死锁产生需要包含几个条件：

- 互斥条件：一个资源要么你用要么我用
- 请求并持有条件：我占了一个还想要另一个
- 资源不可剥夺
- 循环请求

用一张图来表示：

<img src="C:\Users\Jacob\Desktop\新建文件夹\图片\面试6.png" style="zoom:75%;" />

死锁的排查通常是用Jstack/JConsole等工具。

解决死锁的方法：重启、修正程序本身，多采用带超时时间的等待函数。

死锁例子（线程1持有锁1尝试获取锁2，线程2持有锁2尝试获取锁1）：

```java
public class DeadLockThread {
    private static String lock1 = "Lock1";
    private static String lock2 = "Lock2";
    public static void main(String[] args) {
        ExecutorService executors = Executors.newCachedThreadPool();
        executors.execute(new DeadLockOne());
        executors.execute(new DeadLockTwo());
        executors.shutdown();
    }
    static class DeadLockOne implements Runnable {
        @Override
        public void run() {
            synchronized (lock1) {
                System.out.println("线程1获取到锁1");
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    System.out.println(e.getMessage());
                }
                synchronized (lock2) {
                    System.out.println("线程1获取到锁2");
                }
            }
        }
    }
    static class DeadLockTwo implements Runnable {
        @Override
        public void run() {
            synchronized (lock2){
                System.out.println("线程2获取到锁2");
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    System.out.println(e.getMessage());
                }
                synchronized (lock1) {
                    System.out.println("线程2获取到锁1");
                }
            }
        }
    }
}
```

### 19.1 哲学家的进餐问题

请设计一个**并行**的哲学家进餐问题：

基础代码如下：

```java
class DiningPhilosophers {
    public DiningPhilosophers() {

    }
    // 5个哲学家的线程拥有同一个该用餐实例
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
    }
}
```

**该问题的关键在于避免死锁的产生。**

而如何避免死锁的产生呢？实际上只要我们规定了只有4个哲学家可以拿筷子，那么就一定能够避免死锁的产生。

而这显然是一个线程的资源访问控制，使用信号量Semaphore就可以解决。

#### **串行**的例子（错误）：

通过简单的加锁会使得程序串行化。

```
class DiningPhilosophers {
    public DiningPhilosophers() {
        
    }
    // call the run() method of any runnable to execute its code
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
        synchronized(this){
            pickLeftFork.run();
            pickRightFork.run();
            eat.run();
            putLeftFork.run();
            putRightFork.run();
        }
    }
}
```

#### 利用ReentrantLock + Semaphore实现

主要思想：semaphore控制只能有4个线程进行操作，而ReentrantLock保证每个叉子只能被拿起一次

```java
class DiningPhilosophers {
    ReentrantLock forks[] = new ReentrantLock[]{new ReentrantLock(),new ReentrantLock(),new ReentrantLock(),
    new ReentrantLock(),new ReentrantLock()};
    Semaphore semaphore = new Semaphore(4);
    public DiningPhilosophers() {

    }
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
                               semaphore.acquire();//-1
                               forks[philosopher].lock();//尝试拿起左边的筷子
                               pickLeftFork.run();//调用具体逻辑
                               forks[(philosopher + 4)%5].lock();//尝试拿起右边的筷子
                               pickRightFork.run();//调用具体逻辑
                               eat.run();
                               putLeftFork.run();//先调用具体逻辑，再放下筷子，否则如果先释放锁，那么可能还没有调用具体逻辑就被CPU挂起从而导致错误的答案
                               forks[philosopher].unlock();
                               putRightFork.run();
                               forks[(philosopher + 4)%5].unlock();
                               semaphore.release();//+1
    }
}
```

#### 利用CAS+ Semaphore实现



## 20.juc并发包了解吗？（掌握实现以及主要应用场景）

juc包中提供了各种并发基础工具类，主要包括：

- 同步工具类：CountDownLatch、CyclicBarrier、Semaphore等。
- 线程安全的容器：ConcurrentHashMap、ConcurrentSkipListMap、CopyOnWriteArrayList
- 各种并发队列的实现
- 线程池框架：可以创建不同类型的线程池，调度任务的运行。

首先说一下同步工具类：

- CountDownLatch：主要用于一个线程等待某些线程完成。
- CyclicBarrier：一组线程互相等待到某个状态，再一起执行下去。并且与CountDownLatch相比，具有**可重用性**。
- Semaphore：主要用于资源访问的控制。

以一个例子来说这三个工具类：

**一辆出租车能坐4个人，有很多人一起排队等出租车。**

使用信号量Semaphore实现如下：

```java
public class UsualSemaphore {
    public static void main(String[] args) throws InterruptedException {
        Semaphore semaphore = new Semaphore(0);
        int N = new Random().nextInt(50);
        System.out.println(N);
        ExecutorService executorService = Executors.newFixedThreadPool(N);
        for (int i = 0; i < N; i++) {
            executorService.execute(new SemaphoreWorker(semaphore));
        }
        while (N > 0) {
            if (N >= 4) {
                N -= 4;
                semaphore.release(4);
            } else {
                semaphore.release(N);
                N = 0;
            }
            while (semaphore.availablePermits() != 0) {
                TimeUnit.MILLISECONDS.sleep(100);
            }
            System.out.println("wait for permit");
        }
        executorService.shutdown();
    }
    static class SemaphoreWorker implements Runnable {
        private Semaphore semaphore;
        public SemaphoreWorker(Semaphore semaphore) {
            this.semaphore = semaphore;
        }
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName() + "  execute");
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}
```

使用CountDownLatch实现,可以看出由于其不可重用性，实例化了很多CountDownLatch：

```java
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        int N = new Random().nextInt(50);
        System.out.println(N);
        ExecutorService executorService = Executors.newFixedThreadPool(N);
        while (N > 0) {
            CountDownLatch countDownLatch = new CountDownLatch(N>=4?4:N);
            for (int i = 0; i < 4 && i < N; i++) {
                executorService.execute(new CountDownLatchWorker(countDownLatch));
            }
            N -= 4;
            countDownLatch.await();
            System.out.println("-----------");
        }
        executorService.shutdown();
    }
    static class CountDownLatchWorker implements Runnable {
        public CountDownLatch countDownLatch;
        public CountDownLatchWorker(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " 坐上车了");
            countDownLatch.countDown();
        }
    }
}
```

使用CyclicBarrier实现：

```java
public class CyclicBarrierTest {
    public static void main(String[] args) throws BrokenBarrierException, InterruptedException{
        int N = new Random().nextInt(50);
        System.out.println(N);
        ExecutorService executorService = Executors.newFixedThreadPool(N);
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("CyclicBarrier 开始重用");
            }
        });
        while (N > 0) {
            for (int i = 0; i < N && i < 4; i++) {
                executorService.execute(new CyclicBarrierWorker(cyclicBarrier));
            }
            N -= 4;
            try {
                cyclicBarrier.await(1000,TimeUnit.MILLISECONDS);
            } catch (TimeoutException ignore) {
                System.out.println("超时退出");
            }
            System.out.println("--------------");
        }
        executorService.shutdown();
    }
    static class CyclicBarrierWorker implements Runnable {
        public CyclicBarrier cyclicBarrier;
        public CyclicBarrierWorker(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " 坐上车拉");
                cyclicBarrier.await();
            } catch (InterruptedException e) {

            } catch (BrokenBarrierException e) {

            }
        }
    }
}
```

## 21.并发包中的 ConcurrentLinkedQueue 和 LinkedBlockingQueue 有什么区别？

首先一般我们可以认为

- Concurrent的类是lock-free的，也就是基于cas的，多用于高并发的场景。
- LinkedBlockingQueue 内部是基于锁的

从另一个角度来说，LinkedBlockingQueue 实现了**BlockingQueue**。

根据说明**BlockingQueue**文档：

- 阻塞队列不接受Null值，否则抛出NPE异常
- 阻塞队列可能有容量限制，如无指定，则为Integet.MAX_VALUE；ArrayBlockingQueue为有界队列，需要在创建时指定，LinkedBlockingQueue如果未指定则为无界（Integet.MAX_VALUE），其余都是无界
- 阻塞队列主要设计给生产-消费者队列
- 线程安全的，利用锁

简单的生产消费者实现如下：

```java
public class SetUp {
    public static void main(String[] args) {
        BlockingQueue queue = new ArrayBlockingQueue(10);
        Producer producer = new Producer("producer1",queue);
        Producer producer1 = new Producer("producer2",queue);
        Consumer consumer = new Consumer(queue);
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        executorService.execute(producer);
        executorService.execute(producer1);
        executorService.execute(consumer);
        executorService.shutdown();
    }
}
```

```java
public class Producer implements Runnable {
    private BlockingQueue blockingQueue;
    private String id;
    public Producer(String id, BlockingQueue blockingQueue) {
        this.blockingQueue = blockingQueue;
        this.id = id;
    }
    @Override
    public void run() {
        try {
            int index = 10;
            while (index-- > 0) {
                System.out.println("放入" + id);
                blockingQueue.put(id);//put会阻塞到队列出现空位
            }
        } catch (InterruptedException e) {
            //todo
        }
    }
}
```

```java
public class Consumer implements Runnable {
    public BlockingQueue blockingQueue;
    public Consumer(BlockingQueue blockingQueue) {
        this.blockingQueue = blockingQueue;
    }
    @Override
    public void run() {
        try {
            while (true) {
                String id = (String) blockingQueue.take();//take会阻塞到队列中有数据出现
                System.out.println("取出" + id);
            }
        } catch (InterruptedException e) {

        }
    }
}
```

**注意上面的take 和 put函数都是阻塞的，阻塞的实现使用了ReentrantLock锁内的*条件队列*。**

```java
/** Lock held by take, poll, etc */
private final ReentrantLock takeLock = new ReentrantLock();
/** Wait queue for waiting takes */
private final Condition notEmpty = takeLock.newCondition();
/** Lock held by put, offer, etc */
private final ReentrantLock putLock = new ReentrantLock();
/** Wait queue for waiting puts */
private final Condition notFull = putLock.newCondition();
```

**分析LinkedBlockingQueue的take与put函数如下：**

```java
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        int c = -1;
        Node<E> node = new Node<E>(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
                notFull.await();
            }
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();//注意这句话，其保证了不会出现take无限期等待的可能性
    }

public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

假设有三个线程ABC调用take函数，两个线程DE调用put函数。初始化时队列为空的，同时消费者先进行调用，那么会经历以下过程：

1. ABC都阻塞到notEmpty.await();
2. 线程D先拿到锁，入队后调用signalNotEmpty让消费者开始消费

**对比ArrayBlockingQueue 与LinkedBlockingQueue **

- 首先，对ArrayBlockingQueue而言，其put与take都是一个锁来做。也就是说同一时刻只有一个线程能调用put与take。而LinkedBlockingQueue，其put与take是两个锁，也可以可以有一个线程操作take另一个线程操作put。因此官方认为它具有更好的吞吐性能。
- 其次，ArrayBlockingQueue底层基于数组实现，LinkedBlockingQueue则是基于单向链表实现。

## 22.线程池了解吗，线程池有哪几种？分别有什么特点？

Excutors目前提供了5种方法创建线程池：

- newCacheThreadPool：主要用于处理大量的短时间工作的线程池，特点在于其会尝试去**缓存线程并重用**，无缓存线程可用的时候则会创建新的线程，**如果线程闲置60秒，则会被移除缓存**，因此这种线程在**长时间闲置**的时候不会消耗什么资源。内部使用SynchronizedQueue实现。
- newFixedThreadPool(int nThreads)：**固定的工作线程数量n**，如果任务数量大于工作线程n，那么会在**无界**工作队列中等待。
- newSingleThreadExecutor()：**工作线程数为1**，保证了所有的任务都被**顺序的执行**，也使用**无界**的工作队列。
- newSingleThreadScheduledExecutor、newScheduledThreadPool(int corePoolSize)：**定时或周期性的工作调度。**
- newWorkStealingPool(int parallelism)：其内部会构建ForkJoinPool，利用Work-Stealing算法，并行地处理任务，不保证处理顺序。

用一张图理解一下所谓的线程池：

<img src="C:\Users\Jacob\Desktop\新建文件夹\图片\面试7.png" style="zoom:75%;" />

线程池中主要包括四个部分：**工作队列、工作线程（所谓的“线程池”）、线程工厂、拒绝策略**

- 工作队列：存储用户提交的任务，在newCachedThreadPool中可以是**SynchronousQueue**，在newFixedThreadPool与newSingleThreadExecutor中为**LinkedBlockingQueue**
- 工作线程（线程池）：实际上就是工作线程的一个集合。
- 线程工厂（ThreadFactory）：创建工作线程
- 拒绝策略：例如线程处于SHUTDOWN状态时拒绝提交的任务

```java
private final BlockingQueue<Runnable> workQueue;//工作队列
private final HashSet<Worker> workers = new HashSet<Worker>();//线程池，实际上继承自抽象同步队列（实际上每个队列也可以包含多个任务？）
```

**线程池的关键参数：**

- corePoolSize：核心线程数，表示的是**长期驻留**的线程数量（newCachedThreadPool为0，newFixedThreadPool为n）
- maximumPoolSize：线程最多能够创建的数量，newCachedThreadPool为Integer.MAX_VALUE；newFixedThreadPool仍然为n。
- keepAliveTime 和 TimeUnit：这两个参数指定了额外的线程能够闲置多久，显然有些线程池不需要它。
- workQueue：工作队列，必须是 BlockingQueue。

由此，实际上可以从不同的工厂方法中看出不同，在此之前先了解一下类之间的关系图。

<img src="C:\Users\Jacob\Desktop\新建文件夹\图片\面试8.png" style="zoom:75%;" />

**其中，Executors作为工厂提供了许多创建不同类型ThreadPoolExecutor的方法。**

```java
public ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue,
                        ThreadFactory threadFactory,
                        RejectedExecutionHandler handler)

```

线程池的生命周期是如何表征的呢？

实际上是利用一个Integer类型的ctl参数进行配置的。虽然理论上可以设置线程数量为Integer.MAX_VALUE,但实际上最大容量为2^29-1。**因此将ctl的高三位来表征线程的状态。**如下所示：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;
```

那么当前线程状态和工作线程数量便可以由下式决定：

```java
private static int runStateOf(int c)     { return c & ~CAPACITY; }//线程状态
private static int workerCountOf(int c)  { return c & CAPACITY; }//工作线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

线程生命周期如下所示：

<img src="C:\Users\Jacob\Desktop\新建文件夹\图片\面试9.png" style="zoom:75%;" />

**现在分析关键的源码execute：**

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {//如果工作线程少于核心线程，那么尝试创建一个新的工作线程
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {//将任务添加到工作队列等待处理
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))//如果线程已停止，则执行拒绝策略
            reject(command);
        else if (workerCountOf(recheck) == 0)//如果使用的是newCache可能导致线程变为0，此时需要重新创建一个线程
            addWorker(null, false);
    }
    else if (!addWorker(command, false))//尝试扩大线程数量，false代表判断依据从核心线程到了最大线程？？？哪种情况会走到这里呢？
        reject(command);
}
```

**这里请注意，它使用的是blockingQueue的offer方法，这里区分一下offer、put、add的区别：**

- offer：当队列满了直接返回false
- add：当队列满了抛出异常
- put：当队列满了等待他有空闲位置为止

**addWorker(Runnable firstTask, boolean core)**

首先，core代表是否采用核心线程进行判断。

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {//cas自旋避免新增线程数失败
        int c = ctl.get();
        int rs = runStateOf(c);
        //如果线程状态为：①stop、terminated、tidying直接返回false②shutdown但传入的不是null③传入是null但是工作队列是空的
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            //如果线程数大于规定线程数则返回false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            //cas尝试修改新增线程数
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get(); 
            if (runStateOf(c) != rs)
                continue retry;
        }
    }
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);//创建一个新的工作线程（实际上由AQS实现）
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                int rs = runStateOf(ctl.get());
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive())
                        throw new IllegalThreadStateException();
                    workers.add(w);//将新创建的线程加入到线程池
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

## 23.原子类知道吗，AtomicInteger其底层实现原理是什么？CAS在自己的产品代码中如何应用？

首先原子类的设计目的主要是为了使我们常用的更新操作具备原子性从而避免出现并发问题。其底层实现使用了lock-free的Cas操作，相比使用锁实现原子性在性能上有很大的提高。

然后以AtomicInteger为例分析底层原理：

首先我们要知道其使用的Unsafe类实现的CAS操作，那么首先要知道Unsafe在AtomicInteger中是如何起作用的。

要操作unSafe对象需要3个步骤。

1. 调用Unsafe类的静态方法getUnsafe()获取unsafe实例
2. 利用unsafe的objectFieldOffset方法获取value偏移量

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();//这里能够直接获取因为使用相同的类加载器
private static final long valueOffset;//表征的是value的内存地址值，以常量的形式保存
private volatile int value;//实际上就是我们传入的值
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

实例化好了unsafe对象之后就可以利用unsafe做一些原子性的操作。

例如AtomicInteger中的getAndIncrement就是调用了unsafe的getAndAddInt方法。

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

而unsafe底层就是利用cas尝试修改值**(实际上就是compareAndSwapInt)**：

1. 首先从通过内存偏移量从内存获取对应对象的对应属性（保证可见性）
2. 然后尝试用cas操作尝试修改值cas(object,offset,curvalue,setvalue)

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

**那么CAS的底层又是如何实现的呢？**

实际上依赖于CPU提供的特定指令。Linux X86环境下是利用**cmpxchg**指令实现的。

**那么你有没有在你的业务代码中使用过CAS操作呢？**

```java
public class AtomicIntegerTest {
    private AtomicLong atomicLong;
    private void acquireLock() {
        long t = Thread.currentThread().getId();
        while (!atomicLong.compareAndSet(0, t)) {
            //todo
            atomicLong.set(0);
        }
    }
    public static void main(String[] args) {
    }
}
```

**你认为CAS有什么缺点**：在竞争情况激烈的情况下，会导致大量的自旋操作从而消耗大量的CPU资源。

**ABA问题知道吗？如何解决ABA问题？**：CAS操作只是比较当前值，如果对象只是恰好相同，期间发生了A-B-A的转换，往往会出现问题。AtomicStampedReference，其通过为引用建立版本号的方式来解决ABA问题。

```java
public AtomicStampedReference(V initialRef, int initialStamp) {
    pair = Pair.of(initialRef, initialStamp);
}
```

## 24.抽象同步队列了解吗，为什么需要抽象同步队列？

首先，抽象同步队列的设计初衷是通过将同步相关的基础操作抽象在类中从而为我们构建同步的结构提供一个范本。

其主要包含的属性与方法有：

- 表征状态的state
- 一个先进先出的**等待线程**队列
- 各种基于CAS的操作方法与抽象方法acquire/release

## 25.类加载过程了解吗？什么是双亲委派模型？

首先，java的类加载过程主要分为三个步骤：加载、链接、初始化。

- 加载阶段：将字节码数据加载到JVM中映射为class对象，数据源可以是class文件、jar文件等；**用户可以参与加载阶段，通过自定义加载器。**
- 链接：将原始的类信息转化入JVM运行的一个过程。主要分为验证、准备、解析
  - 验证：核验字节信息是否复合JVM规范。否则抛出VerifyError。
  - 准备：创建类、接口中的静态变量，并隐式的初始化（侧重于内存空间的分配）
  - 解析：将常量池中的符号引用替换为直接引用。
- 初始化阶段：执行**类初始化**的代码逻辑。包括静态字段的赋值操作，执行静态代码块。编译器在编译阶段就会**整理好**这部分内容。

考察如下代码

```java
public class CLPreparation {
  public static int a = 100;
  public static final int INT_CONSTANT = 1000;
  public static final Integer INTEGER_CONSTANT = Integer.valueOf(10000);
}
```

```java
     0: bipush      100
   2: putstatic   #2                // Field a:I
   5: sipush      10000
   8: invokestatic  #3                // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
  11: putstatic   #4                  // Field INTEGER_CONSTANT:Ljava/lang/Integer;
```

这能让我们更清楚，普通原始类型静态变量和引用类型（即使是常量），是需要额外调用 putstatic 等 JVM 指令的，这些是在显式初始化阶段执行，而不是准备阶段调用；而原始类型常量，则不需要这样的步骤。

**什么是双亲委派机制**：

简单说就是当类加载器（Class-Loader）试图加载某个类型的时候，除非父加载器找不到相应类型，否则尽量将这个任务代理给当前加载器的父加载器去做。使用委派模型的目的是避免重复加载 Java 类型。试想，如果不同类加载器都自己加载需要的某个类型，那么就会出现多次重复加载，完全是种浪费。

## 26 java深拷贝、浅拷贝了解吗？

对象拷贝的三种方式：

- 直接赋值
- 浅拷贝
- 深拷贝

1. 直接赋值

   直接赋值是最常用的方式，对于引用类型而言，其知识拷贝了对象的引用地址，并没有在内存中生成新的对象。

   如下图：

   ![](C:\Users\Jacob\Desktop\新建文件夹\图片\直接拷贝.jpg)

2. 浅拷贝

   浅拷贝的对象在堆中具有独立的空间，原始对象的属性如果是基本类型那么直接复制就好，如果是引用类型，就直接复制它的引用地址，并没有为它新开辟空间。

3. 深拷贝

   是一种完全拷贝。会为被拷贝对象内的引用属性也在堆中开辟对应的空间。

   ![](C:\Users\Jacob\Desktop\新建文件夹\图片\面试300.jpg)

## 27 java的泛型与类型擦除了解吗？

**Java 编译器是通过先检查代码中泛型的类型，然后在进行类型擦除，再进行编译。**

以一道经典例题开始：

```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();
		
System.out.println(l1.getClass() == l2.getClass());
```

上述结果会输出true，输出的原因就是类型擦除。

**泛型包括：**

- 泛型类

```java
public class Test<T> {
	T field1;
}

```

这里的T称为**类型参数**。

- 泛型方法

```
public class Test1 {
	public <T> void testMethod(T t){
	}
}
```

这里的T称为**参数化类型**，存在于返回值之前。

**实际上需要注意的是，泛型方法和泛型类是可以共存的，同时泛型方法始终以自己定义的为准。**下面是一个例子。

```
public class Test1<T>{
	public  void testMethod(T t){
		System.out.println(t.getClass().getName());
	}
	public  <T> T testMethod1(T t){
		return t;
	}
}
Test1<String> t = new Test1();
t.testMethod("generic");
Integer i = t.testMethod1(new Integer(1));
```

- 泛型接口

**再谈一谈通配符：**

- **<?>**：无限定通配符，其常常与容器类结合使用，需要明确的是，你也只能够调用与类型无关的方法

**最后谈一谈类型擦除 ：**

**泛型信息只存在于代码编译阶段，在进入 JVM 之前，与泛型相关的信息会被擦除掉，专业术语叫做类型擦除**。

实际上，是把泛型信息转成了Object或者说使用<? extend class>的上限class类。

最后是一个利用反射的小例子，其实现了让List装载不同类型的数据。

```
public class ToolTest {
	public static void main(String[] args) {
		List<Integer> ls = new ArrayList<>();
		ls.add(23);
//		ls.add("text");
		try {
			Method method = ls.getClass().getDeclaredMethod("add",Object.class);
			method.invoke(ls,"test");
			method.invoke(ls,42.9f);
		} catch (NoSuchMethodException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SecurityException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		for ( Object o: ls){
			System.out.println(o);
		}
	
	}
}

```

## 28 内存泄漏与内存溢出

内存泄漏指的是无用对象持续占用内存的现象，根本原因是长生命周期的对象持有短生命周期的对象导致其无法被回收。内存泄漏的几大场景：

- 静态集合类引起的内存泄漏，静态成员的生命周期是整个程序运行期间。
- 各种链接对象未关闭（IO、数据库、网络）

内存溢出则是指的是程序运行过程中无法申请到足够的内存而导致的错误。常见的有：

- 堆溢出（OutOfMemoryError）：创建的对象太多了。
- 栈溢出（StackOverflowError ）：递归层数太多导致栈溢出。

## 29 Class.forName和ClassLoad了解吗？

他们都可以对类进行加载。区别在于：

- ClassLoad仅对类进行加载，也就是说只会将.class文件转换为JVM中的class对象。
- 而Class.forName还会对类进行初始化。

## 30 CopyOnWriteArrayList 了解吗?

写时复制容器，其核心思想是当我们往一个容器添加元素的时候，不直接往容器里面添加。而是将容器复制一份，往复制的容器中添加，最后再将原容器指向当前容器。

其核心优势在于，可以对容器进行高并发的读而不需要加锁。常用于读多写少的场景。

其劣势在于：无法保证**数据一致性**，无法立刻读取到写入的数据。

## 31 集合框架的杂记

### 31.1 简述ArrayList的扩容机制

事实上，扩容会发生在调用add函数的时候，如下：

```java
 public boolean add(E e) { 
   //扩容
  ensureCapacityInternal(size + 1); //Increments modCount!!
  elementData[size++] = e; 
  return true;   
  }         
```

在调用ensureCapacityInternal会判断容量是否还够，如果不够了，则会调用grow方法进行扩容，在这个方法内实际上核心就以下几点：

- 将新容量调整至旧容量的1.5倍
- 判断新容量是否满足要求，并且判断是否超过了容量最大值。
- 最后调用Array.copyOf(elementData,newCapacity)。

```java
private void grow(int minCapacity) {
    // 获取到ArrayList中elementData数组的内存空间长度
    int oldCapacity = elementData.length;
    // 扩容至原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 再判断一下新数组的容量够不够，够了就直接使用这个长度创建新数组， 不够就将数组长度设置为需要的长度
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //若预设值大于默认的最大值检查是否溢出
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 调用Arrays.copyOf方法将elementData数组指向新的内存空间时newCapacity的连续空间
    // 并将elementData的数据复制到新的内存空间
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 31.2 HashMap 的底层实现、JDK 1.8 的时候为啥将链表转换成红黑树？HashMap 的负载因子

HashMap底层是用数组+链表/红黑树实现的，当添加一个新的元素到集合的时候，会首先计算元素key的hash值，并根据hash值确定插入数组的位置，如果该位置已经有其他元素存放，那么实用链表来解决hash冲突，进一步，如果链表长度过长，便将链表转换为红黑树来提高搜索的效率。

还有几个需要注意的点是：

- 数组的初始容量为16，并且该容量是以2的幂次方扩充的，一是为了提升性能使用足够大的数组；二是为了能使用位运算代替取模运算。
- 数组是否需要扩充是通过负载因子来判断的，如果当前元素个数为容量*负载因子，就会扩充数组。
- 当链表长度大于8的时候，会将链表转化为红黑树以提升性能；当长度小于6时，又会将红黑树转回链表。

### 31.3 LinkedHashMap，TreeMap 区别在哪？

- LinkedHashMap保存了插入数据的顺序
- TreeMap可以根据键进行排序

