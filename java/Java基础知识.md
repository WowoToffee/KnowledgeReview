# java基础知识

[基础数据类型](#基础数据类型)

[构造方法](#构造方法)

[Object 通用方法](#Object 通用方法)

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

## Object 通用方法

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



## 泛型

### <? extends T>和<? super T>的区别

`<? extends T>`和`<? super T>`是Java泛型中的**“通配符（Wildcards）”**和**“边界（Bounds）”**的概念。

- <? extends T>：是指 **“上界通配符（Upper Bounds Wildcards）”**
- <? super T>：是指 **“下界通配符（Lower Bounds Wildcards）”**

[参考](https://www.cnblogs.com/drizzlewithwind/p/6100164.html)

### LRU缓存实现(Java)

​	[LRU缓存实现(Java)](https://www.cnblogs.com/lzrabbit/p/3734850.html)