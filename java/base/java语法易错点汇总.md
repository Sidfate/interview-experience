# JAVA 语法易错点

### JDK，JRE 和 JVM

从包含关系上来说 JVM < JRE < JDK。

JVM是java虚拟机，主要作用就是将字节码转化为机器语言。

JRE是java运行时环境，包含了java虚拟机，java基础类库。

JDK是java开发工具包，里面包含了JRE和一些开发工具。

### String，StringBuffer 和 StringBuilder

String 是不可变的，因为String类中用 `private　final　char　value[]` 表示字符串，带了final标识。而 StringBuffer 和 StringBuilder 继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串 `char[] value` 但是没有用 final 关键字修饰，所以这两种对象都是可变的。

String 和 StringBuffer 是线程安全的，String相当于常量，StringBuffer在字符串操作方法中加了同步锁，所以也是线程安全的。StringBuilder 不是线程的安全的，相对效率更高。

总结：

1. 操作少量的数据: 适用String
2. 单线程操作字符串缓冲区下操作大量数据: 适用StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据: 适用StringBuffer

### **== 和 equals**

== : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

equals() ****: 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

1. 类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
2. 类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

说明：

- String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
- 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。

### hashCode 和 equals

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

hashCode 主要用于 hashtable，例如hashmap。equals可以用于比较对象相等，但是消耗比较大，所以提前使用 hashCode 计算出hash值，判断当前hash值有没有元素，没有说明肯定没有相等，有就继续比较 equals。

### 值传递

java 中对象引用是按值传递的。因为对象的传递不是传递同一个引用，而是对象的拷贝。

![Untitled/Untitled.png](https://sidfate.oss-cn-hangzhou.aliyuncs.com/upic/20200221120115-4lZ6lA.jpg)

### 异常处理

![Untitled/Untitled%201.png](https://sidfate.oss-cn-hangzhou.aliyuncs.com/upic/20200221120259-r8k5vY.jpg)

Error（错误）:是程序无法处理的错误，表示运行应用程序中较严重问题。

Exception（异常）:是程序本身可以处理的异常。

当try语句和finally语句中都有return语句时，在方法返回之前，finally语句的内容将被执行，并且finally语句的返回值将会覆盖原始的返回值。如下：

    public static int f(int value) { 
    	try { 
    		return value * value; 
    	} finally { 
    		if (value == 2) { 
    			return 0; 
    		} 
    	} 
    }

如果调用 `f(2)`，返回值将是0，因为finally语句的返回值覆盖了try语句块的返回值。