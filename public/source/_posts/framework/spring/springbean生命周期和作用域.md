---
title: springbean生命周期和作用域
categories: spring
tags:
  - spring
abbrlink: 7e94e32c
date: 2022-02-02 12:00:00
---
# 生命周期

bean从创建——>销毁的过程

spring bean 生命周期分好几个阶段，每个阶段有多个扩展点

参考文章：[请别再问Spring Bean的生命周期了！](https://www.jianshu.com/p/1dec08d290c1)

## 概述阶段

首先主要的阶段分别为：

（1）加载——>（2）实例化——>（3）属性赋值——>（4）初始化,使用——>（5）销毁

容器启动加载对应类到虚拟机中，包装成`BeanDefinition`对象，当调用getBean的时候，就是开始了实例化的阶段

实例化及以后的方法都在`doCreateBean()`中，`createBeanInstance() -> 实例化`、`populateBean() -> 属性赋值`、

`initializeBean() -> 初始化`。

销毁是在容器关闭的时候调用，在单例模式下，当容器创建的时候对象就出生了，只要容器不死，对象就一直活着，容器销毁，对象嗝屁，多例模式下，容器给我们创建对象，对象只要使用就一直活着，不使用的时候垃圾回收器处理

## 阶段扩展点

### 加载

通过调用`BeanFactoryPostProcessor#postProcessBeanFactory()`方法，可以对 `BeanDefinition`对象进行处理

`BeanDefinition `包含了很多属性，例如：类的作用域、是否懒加载、自动注入类型等信息

### 实例化

这里是生命周期的开始

`InstantiationAwareBeanPostProcessor` 扩展作用于实例化阶段的前后阶段，分别为`postProcessBeforeInstantiation()`,`postProcessAfterInstantiation` 。通俗的来讲，A a = new A() 这个阶段就是实例化阶段

### 属性赋值

属性赋值，这个阶段是给类中的属性赋值默认的零值，比如int a = 0;

### 初始化

这个阶段就是我们给属性真正赋值的时候了，这里`BeanPostProcessor` 扩展会作用于这个阶段的前后，分别是 `postProcessBeforeInitialization()` ,`postProcessAfterInitialization()` 。发现和实例化阶段的方法一样，这里可以发现实例化阶段的`InstantiationAwareBeanPostProcessor` 是继承了`BeanPostProcessor` 

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {......}
```

### 其他扩展点

这里的扩展很多，常用于用户自定义扩展

#### Aware类型的接口

Aware类型的接口的作用就是让用户拿到spinrg容器中的一些资源。这里接口很多，下面的排列顺序也是接口的调用顺序。基本生上Aware前面是啥就是去容器中取什么资源，例如：BeanNameAware可以拿到BeanName以此类推。

第一类：

1. BeanNameAware
2. BeanClassLoaderAware
3. BeanFactoryAware

第二类：

1. EnvironmentAware

2. EmbeddedValueResolverAware 这个知道的人可能不多，实现该接口能够获取Spring EL解析器，用户的自定义注解需要支持spel表达式的时候可以使用，非常方便。

3. ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware) 这几个接口可能让人有点懵，实际上这几个接口可以一起记，其返回值实质上都是当前的ApplicationContext对象，因为ApplicationContext是一个复合接口，如下

   ```java
   public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
           MessageSource, ApplicationEventPublisher, ResourcePatternResolver {}
   ```

Aware接口调用在initializeBean()方法中，说明Aware接口都是在初始化阶段之前执行的（如下所示，只保留了源码部分）

```java
    // 见名知意，初始化阶段调用的方法
    protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
        // 这里调用的是三个Bean开头的Aware
        invokeAwareMethods(beanName, bean);
        Object wrappedBean = bean;
        // 这里就是前面所说的BeanPostProcessor的调用点！
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        // 实例化方法
        invokeInitMethods(beanName, wrappedBean, mbd);
        // BeanPostProcessor的另一个调用点
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
        return wrappedBean;
    }
```

上面的代码也体现了，BeanPostProcessor在bean初始化前后执行。



#### 生命周期接口

bean的生命周期，实例化和属性赋值都是容器帮我们完成，用户扩展的地方大多数都在初始化和销毁阶段。

1. InitializingBean 对应生命周期的初始化阶段，在上面源码的`invokeInitMethods(beanName, wrappedBean, mbd);`方法中调用。
    有一点需要注意，因为Aware方法都是执行在初始化方法之前，所以可以在初始化方法中放心大胆的使用Aware接口获取的资源，这也是我们自定义扩展Spring的常用方式。
    除了实现InitializingBean接口之外还能通过注解或者xml配置的方式指定初始化方法，至于这几种定义方式的调用顺序其实没有必要记。因为这几个方法对应的都是同一个生命周期，只是实现方式不同，我们一般只采用其中一种方式。
2. DisposableBean类似于InitializingBean对应生命周期的销毁阶段，以ConfigurableApplicationContext#close()方法作为入口，实现是通过循环取所有实现了DisposableBean接口的Bean然后调用其destroy()方法 



## BeanPostProcessor 注册时机与执行顺序

BeanPostProcessor有很多个，而且每个BeanPostProcessor都影响多个Bean，其执行顺序至关重要，必须能够控制其执行顺序才行。关于执行顺序这里需要引入两个排序相关的接口：PriorityOrdered、Ordered

- PriorityOrdered是一等公民，首先被执行，PriorityOrdered公民之间通过接口返回值排序
- Ordered是二等公民，然后执行，Ordered公民之间通过接口返回值排序
- 都没有实现是三等公民，最后执行

PriorityOrdered、Ordered接口作为Spring整个框架通用的排序接口，在Spring中应用广泛，也是非常重要的接口。



# 作用域

bean的作用域用于确定哪种类型的bean实例应该从容器中返回给调用者。作用域主要有以下五种：

| 作用域         | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| singleton      | 在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，bean作用域范围的默认值。 |
| prototype      | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean()。 |
| request        | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于web的Spring WebApplicationContext环境。 |
| session        | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean。该作用域仅适用于web的Spring WebApplicationContext环境。 |
| global-session | 集群环境会话范围，全局会话范围，不是集群的话它就是session    |

（1）singleton 单实例

```xml
<bean id="test" class="com.lichneghao.Test" scope="singleton"/>
```

单例类型，在容器启动的时候自动创建bean对象，无论你是否使用。对于所有bean请求，只要id与该bean定义的相匹配，那么就会返回同一个bean对象。

**容器会对这个bean负责到底，直到销毁这个bean,这一点和下面prototype作用域是有区别的**

（2）prototype 多实例

```xml
<bean id="test" class="com.lichneghao.Test" scope="prototype"/>
```

容器启动的时候，不会自动创建bean，只有去请求bean才会创建bean，并且每次都返回一个新的bean。

所以对于有状态的bean应该使用prototype作用域。

**需要注意的是：容器把这个bean给了客户端后，客户端就要对这个bean以后的生命负责了，容器就不管了，例如销毁什么的需要客户端自己去处理**

（3）request 

bean定义如下所示：

```xml
<bean id="test" class="com.lichneghao.Test" scope="request"/>
```

对于每个http请求，spring容器都会创建一个test bean实例。在请求内可以随便修改bean的状态，一旦请求结束这个bean也就会会随之销毁。

（4）session

每次会话创建一个实例

```xml
<bean id="test" class="com.lichneghao.Test" scope="session"/>
```

