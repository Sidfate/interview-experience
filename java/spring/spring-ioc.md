# Spring IoC

IoC（Inverse of Control:控制反转）是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由Spring框架来管理。IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。

ApplicationContext 启动过程中，会负责创建实例 Bean，往各个 Bean 中注入依赖等。

### BeanFactory

ApplicationContext 其实就是一个 BeanFactory。

Spring IoC的初始化过程：
![](https://sidfate.oss-cn-hangzhou.aliyuncs.com/upic/20200224121643-uitOuC.jpg)


springboot中如何获取 ApplicationContext：
```java
@Component
public class ApplicationContextHolder implements ApplicationContextAware{
    // 上下文对象实例
    private static ApplicationContext applicationContext;
 
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextHolder.applicationContext = applicationContext;
    }
 
    public static ApplicationContext getApplicationContext(){
        return applicationContext;
    }
 
    public static <T> T getBean(Class<T> clazz){
        return applicationContext.getBean(clazz);
    }
 
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name){
        return (T)applicationContext.getBean(name);
    }
 
    public static <T> T getBean(String name, Class<T> clazz){
        return getApplicationContext().getBean(name, clazz);
    }
}
```
ApplicationContext 继承自 BeanFactory，但是它不应该被理解为 BeanFactory 的实现类，而是说其内部持有一个实例化的 BeanFactory（DefaultListableBeanFactory）。以后所有的 BeanFactory 相关的操作其实是委托给这个实例来处理的。

这里的 BeanDefinition 就是我们所说的 Spring 的 Bean，我们自己定义的各个 Bean 其实会转换成一个个 BeanDefinition 存在于 Spring 的 BeanFactory 中。

所以，如果有人问你 Bean 是什么的时候，你要知道 Bean 在代码层面上可以简单认为是 BeanDefinition 的实例。

BeanDefinition 中保存了我们的 Bean 信息，比如这个 Bean 指向的是哪个类、是否是单例的、是否懒加载、这个 Bean 依赖了哪些 Bean 等等。