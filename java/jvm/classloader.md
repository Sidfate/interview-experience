### ClassLoader

类加载器就是用来加载类的。它负责将 Class 的字节码形式转换成内存形式的 Class 对象。所以有一个常见的用途就是对.class文件加解密，用工具将字节码加密，然后后特定的Classloader对字节码解密解析出正确的类。

### classloader的类别

JVM 运行并不是一次性加载所需要的全部类的，它是按需加载，也就是延迟加载。程序在运行的过程中会逐渐遇到很多不认识的新类，这时候就会调用 ClassLoader 来加载这些类。加载完成后就会将 Class 对象存在 ClassLoader 里面，下次就不需要重新加载了。

JVM 中内置了三个重要的 ClassLoader，分别是 BootstrapClassLoader、ExtensionClassLoader 和 AppClassLoader。

* BootstrapClassLoader 负责加载 JVM 运行时核心类
* ExtensionClassLoader 负责加载 JVM 扩展类
* AppClassLoader 才是直接面向我们用户的加载器，它会加载 Classpath 环境变量里定义的路径中的 jar 包和目录。
* 自定义类加载器，自定义的类加载器可以使我们跨目录（网络）的加载我们需要的类文件。


### 双亲委派

AppClassLoader 在加载一个未知的类名时，它并不是立即去搜寻 Classpath，它会首先将这个类名称交给 ExtensionClassLoader 来加载，如果 ExtensionClassLoader 可以加载，那么 AppClassLoader 就不用麻烦了。否则它就会搜索 Classpath 。
而 ExtensionClassLoader 在加载一个未知的类名时，它也并不是立即搜寻 ext 路径，它会首先将类名称交给 BootstrapClassLoader 来加载，如果 BootstrapClassLoader 可以加载，那么 ExtensionClassLoader 也就不用麻烦了。否则它就会搜索 ext 路径下的 jar 包。

以上就是双亲委派机制的过程，但是为什么需要这样的机制呢？

> 对于任意一个类，都需要由加载它的类加载器和这个类本身来一同确立其在Java虚拟机中的唯一性。

如果不是同一个类加载器加载，即时是相同的class文件，也会出现判断不相同的情况，从而引发一些意想不到的情况，为了保证相同的class文件，在使用的时候，是相同的对象，jvm设计的时候，采用了双亲委派的方式来加载类。

为什么破坏双亲委派？
上面的机制很好的解决了一个类归属的问题，越基础的类就找越上层的类加载器加载，因为这些类总是被用户代码调用。但是反过来基础类也有可能要调用回用户的代码，例如jdbc的驱动类Driver，它提供了统一的jdbc驱动管理，实际根据用户选用不同的驱动，人称SPI。它实际上就是用到了线程上下文类加载器，在基础类中使用子类加载器去加载用户类的代码，这种行为实际上就是破坏了双亲委派模型的层次结构来逆向使用类加载器。

```java
class Driver {
  static {
    try {
       java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
       throw new RuntimeException("Can't register driver!");
    }
  }
  ...
}
```

### 自定义类加载器

ClassLoader 里面有三个重要的方法 loadClass()、findClass() 和 defineClass()。
loadClass() 方法是加载目标类的入口，它首先会查找当前 ClassLoader 以及它的双亲里面是否已经加载了目标类，如果没有找到就会让双亲尝试加载，如果双亲都加载不了，就会调用 findClass() 让自定义加载器自己来加载目标类。ClassLoader 的 findClass() 方法是需要子类来覆盖的，不同的加载器将使用不同的逻辑来获取目标类的字节码。拿到这个字节码之后再调用 defineClass() 方法将字节码转换成 Class 对象。
