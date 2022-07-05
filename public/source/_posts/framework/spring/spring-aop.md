---
title: spring-aop
categories: spring
tags:
  - spring
  - aop
abbrlink: 4ecd1f34
date: 2022-02-02 12:00:00
---
## AOP

AOP(Aspect-Oriented Programming) : 面向切面（方面）编程，利用AOP可以对业务的各个部分进行隔离，从而使业务逻辑各部分之间的耦合度降低，提高程序的可用性，同时提高了效率；通俗来讲就是在不修改源代码的情况下，对源代码的功能进行增强。

## 实现原理

底层使用动态代理来实现功能的增强，有两种情况

1. 被代理的类有接口 , 使用JDK的动态代理，创建接口实现类代理对象，增强类的方法

2. 被代理的类没有接口，使用CGLIB代理，创建子类的代理对象，增加类的方法



在JAVA平台中，对于AOP的增强，有三种方式：

1. 编译期：编译器把切面的调用编译进字节码，这种方式需要扩展编译器
2. 类加载器：目标被加载到JVM,通过特殊的加载器，对目标字节码进行增强
3. 运行期：目标对象和切面都是JAVA对象，通过JVM的动态代理或者第三方库运行期动态进行增强

## 抽象的概念(术语)

**连接点：**

类中的哪些方法可以被增强，那么这些方法被称为连接点。例如类中十个方法，那这是十个方法都是连接点。

**切入点（Pointcut）:**

被增强的方法就是切入点。例如类中有十个方法，其中add()方法被增强了，那么这个方法就是切入点。

**增强或者通知（Advice）：**

实际增强的部分，比如记录下日志，判断权限等。增强有多个类型：前置增强、后置增强、环绕增强、异常增强、最终增强。

**切面（Aspect）：**

把增强应用到切入点上的过程就是切面。

## 测试代码

Spring中通常使用AspectJ来实现AOP操作，AspectJ不是Spring的组成部分，是独立的AOP框架。通常在配合Spring进行使用。

测试代码 ——> [基于xml或者注解实现AOP操作](https://github.com/lichlaughing/lichenghao-study/tree/master/springstudytest/spring-aop)

## 切入点表达式

用来表示对哪个类中的哪个方法进行增强

### 标准方式

访问修饰符 返回值 包名.包名.包名...类名.方法名(参数列表)
public void cn.com.lichenghao.service.impl.UserServiceImpl.get()

简化过程↓↓↓

👇

省略访问修饰符

```java
void cn.com.lichenghao.service.impl.UserServiceImpl.get()
```

👇

返回值使用通配符*，表示任意返回值

```java
* cn.com.lichenghao.service.impl.UserServiceImpl.get()
```

👇

包名使用通配符，*代表包名，几级包名就写几个 *

```java
* *.*.*.*.impl.UserServiceImpl.get()
```

👇

包名通配符，*..代表包以及子包

```java
* *..service.impl.UserServiceImpl.get()
```

👇

类名和方法名可以使用通配符*

```java
* *..service.impl.*.get()
* *..service.impl.*.*()
```

👇

参数列表可以写（基本类型直接写例如：int，引用类型写包全路径例如：java.lang.String）

👇

参数类型可以使用通配符*，前提是必须有参数

```java
* *..service.impl.*.*(*)
```

👇

参数类型可以使用..表示参数可以是任意类型

```java
* *..service.impl.*.*(..)
```

### 全通配

全通配方式：

```java
* *..*.*(..) 
```

需要注意，如果这样配置，所有类的都满足这种方式，一般都不会这样配置

### 通常写法

业务类下的所有方法

```java
* cn.com.lichenghao.service.impl.*.*(..)
```





