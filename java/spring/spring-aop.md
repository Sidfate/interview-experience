# Spring AOP

### 三个问题

1. 为什么要引入AOP的编程范式？ 

    解决非功能性的代码重复问题，例如日志的打印和事务控制等 实现关注点的分离，使得能集中的开发某一个功能点

2. AOP的好处及适用场景是什么？

    好处： 保持编程的内聚性，高内聚也就对应着代码是高可用的，减少代码的耦合性。AOP是低侵入的，易分离的。开发的代码量较少，代码的可读性较好。

    适用场景： 独立于业务功能的服务开发。

3. AOP的两大核心要点是什么？ 

    一是方面，需要定义什么方面，这个方面能够做什么？ 二是切入点。

### AspectJ

Aspect 和 Pointcut

spring-aop的包中集合了很多aspectj的类，其中有几个关键的注解，@Aspect即切面，可以修饰一个类。@Pointcut即切入点，可以修饰方法，通过表达式的参数配置表示需要织入切面的类。

Advice 即通知，有5中：

- @Before，前置通知
- @After，后置通知（finally）
- @AfterReturning，返回通知（有参数）
- @AfterThrowing，异常通知
- Around，环绕通知，整体控制

### spring-aop

jdk代理：

继承 `InvocationHandler`，重写invoke方法。invoke方法中通过反射去执行目标类的实际方法。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1de978dc-1846-44cb-b230-f27ffef9c9d5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1de978dc-1846-44cb-b230-f27ffef9c9d5/Untitled.png)

在使用的时候，我们需要通过 Proxy 去动态的生成一个代理：

```java
Proxy.newProxyInstance(Client.class.getClassLoader, new Class[]{Subject.class}, new JdkProxySubject(new RealSubject()));
subject.hello();
```

需要三个参数，第一个当前类的类加载器，第二个是接口，第三个是代理类。

newProxyInstance→getProxyClass0→ProxyClassFactory→ProxyGenerator

源码中根据以上步骤会动态的生成一个代码类的字节码。字节码实际已经生成好了所有接口的实现方法。所有实现方法中都会最终调用我们定义的handler的invoke方法。

Cglib代理

Cglib与JDK的区别在于：

1. JDK只能对有接口实现类的方法进行动态代理
2. Cglib基于继承来实现代理，因此静态类final类或者类的final静态方法不能被代理

总结，在spring中如果类有接口实现，默认用jdk代理，除非强制设置为cglib代理，如果类没有接口实现，则用cglib。另外存在多个切面时使用了责任链的模式保证切面的顺序执行。

spring-aop的应用：

1. 事务控制：@Transactional
2. 权限验证：@Preauthorize
3. 缓存：@Cachable

注意：

spring-aop没法支持方法的嵌套调用，例如有一个织入的方法A，另一个没有织入的方法B去内部调用A的时候，调用B是不会走切面的，因为用的是this对象的调用，没有走代理，可以获取spring的上下文的bean去调用来解决。