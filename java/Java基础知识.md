# java基础知识

[基础数据类型](#基础数据类型)

[构造方法](#构造方法)

[Object通用方法](#Object通用方法)

[集合类](#集合类)

[泛型](#泛型)

## 基础数据类型

在 Java 中，数据类型只有`四类八种`

- 整数型：byte、short、int、long

  byte 也就是字节，1 byte = 8 bits，取值10000000（-128）到 01111111（127），byte 的默认值是 0 ；

  

  short 占用两个字节，也就是 16 位，1 short = 16 bits，取值10000000 00000000（-32768）到 01111111 11111111（32767），它的默认值也是 0 ；

  

  int 占用四个字节，也就是 32 位，取值-2^31 （-2,147,483,648）到 2^31-1（2,147,483,647），1 int = 32 bits，默认值是 0 ；

  

  long 占用八个字节，也就是 64 位，1 long = 64 bits，取值-2^63 （-9,223,372,036,854,775,808‬）到 2^63-1（9,223,372,036,854,775,8087），默认值是 0L；

  

  所以整数型的占用字节大小空间为 long > int > short > byte

- 浮点型有两种数据类型：float 和 double

  float 是单精度浮点型，精确到8位，占用 4 位，1 float = 32 bits，默认值是 0.0f；

  

  double 是双精度浮点型，占用 8 位，精确到17位，1 double = 64 bits，默认值是 0.0d；

- 字符型

  字符型就是 char，char 类型是一个单一的 16 位 Unicode 字符，最小值是 `\u0000 (也就是 0 )`，最大值是 `\uffff (即为 65535)`，char 数据类型可以存储任何字符，例如 char a = 'A'。

- 布尔型

  布尔型指的就是 boolean，boolean 只有两种值，true 或者是 false，只表示 1 位，默认值是 false。

## 构造方法

存在继承的情况，初始化顺序：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

## Object通用方法

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

## 集合类

### ArrayList,Vector与Stack

ArrayList：

​		ArrayList是实现List接口的动态数组，所谓动态就是它的大小是可变的。实现了所有可选列表操作，并允许包括 null 在内的所有元素。除了实现 List 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。

​		每个ArrayList实例都有一个容量，该容量是指用来存储列表元素的数组的大小。默认初始容量为10。随着ArrayList中元素的增加，它的容量也会不断的自动增长。如果发生扩容，是扩容1.5倍。	`为什么扩容1.5倍？`

​		ArrayList是线程不安全的。在其迭代器iteator中，如果有多线程操作导致modcount改变，会执行fastfail。抛出异常。

Vector:

​		Vector可以实现可增长的对象数组。与数组一样，它包含可以使用整数索引进行访问的组件。不过，Vector的大小是可以增加或者减小的，以便适应创建Vector后进行添加或者删除操作。

​		扩容方式与ArrayList基本一样，但是扩容时不是1.5倍扩容，而是有一个扩容增量。

```java
public synchronized void ensureCapacity(int minCapacity) { 
    modCount++;//父类中的属性，记录集合变化次数 
    ensureCapacityHelper(minCapacity); 
    } 
 private void ensureCapacityHelper(int minCapacity) { 
    int oldCapacity = elementData.length; 
    if (minCapacity > oldCapacity) {//扩容的条件，数组需要的长度要大于实际长度 
        Object[] oldData = elementData; 
        int newCapacity = (capacityIncrement > 0) ? 
        (oldCapacity + capacityIncrement) : (oldCapacity * 2); 
            if (newCapacity < minCapacity) { 
        newCapacity = minCapacity; 
        } 
        elementData = new Object[newCapacity]; 
        System.arraycopy(oldData, 0, elementData, 0, elementCount); 
    } 
    } 
```

  int newCapacity = (capacityIncrement > 0) ?(oldCapacity + capacityIncrement) : (oldCapacity * 2);

​		Vector可以指定增长因子,当扩容因子大于0时，新数组长度为原数组长度+扩容因子，否子新数组长度为原数组长度的2倍。



Stack:

​		在Java中Stack类表示后进先出（LIFO）的对象堆栈。栈是一种非常常见的数据结构，它采用典型的先进后出的操作方式完成的。

​		Stack通过五个操作对Vector进行扩展，允许将向量视为堆栈。这个五个操作如下：

```
empty()
测试堆栈是否为空。

peek()
查看堆栈顶部的对象，但不从堆栈中移除它。

pop()
移除堆栈顶部的对象，并作为此函数的值返回该对象。

push(E item)
把项压入堆栈顶部。

search(Object o)
返回对象在堆栈中的位置，以 1 为基数。
```

### LinkedList和Queue

​		LinkedList与ArrayList一样实现List接口，只是ArrayList是List接口的大小可变数组的实现，LinkedList是List接口链表的实现。基于链表实现的方式使得LinkedList在插入和删除时更优于ArrayList，而随机访问则比ArrayList逊色些。LinkedList实现所有可选的列表操作，并允许所有的元素包括null。

​		除了实现 List 接口外，LinkedList 类还为在列表的开头及结尾 get、remove 和 insert 元素提供了统一的命名方法。这些操作允许将链接列表用作堆栈、队列或双端队列。

​		此类实现 Deque 接口，为 add、poll 提供先进先出队列操作，以及其他堆栈和双端队列操作。

​		所有操作都是按照双重链接列表的需要执行的。在列表中编索引的操作将从开头或结尾遍历列表（从靠近指定索引的一端）。

同时，与ArrayList一样此实现不是同步的。

Entry节点对象：

```
private static class Entry<E> {
    E element;        //元素节点
    Entry<E> next;    //下一个元素
    Entry<E> previous;  //上一个元素

    Entry(E element, Entry<E> next, Entry<E> previous) {
        this.element = element;
        this.next = next;
        this.previous = previous;
    }
}
  上面为Entry对象的源代码，Entry为LinkedList的内部类，它定义了存储的元素。该元素的前一个元素、后一个元素，这是典型的双向链表定义方式。
```

Queue:

Queue接口定义了队列数据结构，元素是有序的(按插入顺序)，先进先出。

### HashMap和HashTable

HashMap：

​		基于哈希表的 Map 接口的实现，以key-value的形式存在。在HashMap中，key-value总是会当做一个整体来处理，系统会根据hash算法来来计算key-value的存储位置，我们总是可以通过key快速地存、取value。

构造函数：

```
  HashMap提供了三个构造函数：

  HashMap()：构造一个具有默认初始容量 (16) 和默认加载因子 (0.75) 的空 HashMap。

  HashMap(int initialCapacity)：构造一个带指定初始容量和默认加载因子 (0.75) 的空 HashMap。

  HashMap(int initialCapacity, float loadFactor)：构造一个带指定初始容量和加载因子的空 HashMap。
```

```
初始容量，加载因子:
这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中桶的数量，初始容量是创建哈希表时的容量，加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。

对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为0.75，一般情况下我们是无需修改的。
```

HashMap数据结构图

下图的table数组的每个格子都是一个桶。负载因子就是map中的元素占用的容量百分比。比如负载因子是0.75，初始容量（桶数量）为16时，那么允许装填的元素最大个数就是16*0.75 = 12，这个最大个数也被成为阈值，就是map中定义的threshold。超过这个阈值时，map就会自动扩容。

[hasMap 工作原理和扩容机制](https://blog.csdn.net/u010890358/article/details/80496144)

HashTable：

HashMap和HashTable不同点：

​	HashMap线程不安全，HashTable是线程安全的。HashMap内部实现没有任何线程同步相关的代码，所以相对而言性能要好一点。如果在多线程中使用HashMap需要自己管理线程同步。HashTable大部分对外接口都使用synchronized包裹，所以是线程安全的，但是性能会相对差一些。

二者的基类不一样。HashMap派生于AbstractMap，HashTable派生于Dictionary。它们都实现Map, Cloneable, Serializable这些接口。AbstractMap中提供的基础方法更多，并且实现了多个通用的方法，而在Dictionary中只有少量的接口，并且都是abstract类型。

key和value的取值范围不同。HashMap的key和value都可以为null，但是HashTablekey和value都不能为null。对于HashMap如果get返回null，并不能表明HashMap不存在这个key，如果需要判断HashMap中是否包含某个key，就需要使用containsKey这个方法来判断。

算法不一样。HashMap的initialCapacity为16，而HashTable的initialCapacity为11。HashMap中初始容量必须是2的幂,如果初始化传入的initialCapacity不是2的幂，将会自动调整为大于出入的initialCapacity最小的2的幂。HashMap使用自己的计算hash的方法(会依赖key的hashCode方法)，HashTable则使用key的hashCode方法得到。

### LinkedHashMap和LRU缓存

[LinkedHashMap和LRU缓存](https://blog.csdn.net/a724888/article/details/80290276)

### HashSet，TreeSet与LinkedHashSet

HashSet:

​		`Set接口是一种不包括重复元素的Collection，它维持它自己的内部排序`。总所周知HashSet的底层是HashMap，

参考：

[java集合类](https://blog.csdn.net/a724888/category_9274189.html)

## 泛型

### <? extends T>和<? super T>的区别

`<? extends T>`和`<? super T>`是Java泛型中的**“通配符（Wildcards）”**和**“边界（Bounds）”**的概念。

- <? extends T>：是指 **“上界通配符（Upper Bounds Wildcards）”**
- <? super T>：是指 **“下界通配符（Lower Bounds Wildcards）”**

[参考](https://www.cnblogs.com/drizzlewithwind/p/6100164.html)

### LRU缓存实现(Java)

​	[LRU缓存实现(Java)](https://www.cnblogs.com/lzrabbit/p/3734850.html)