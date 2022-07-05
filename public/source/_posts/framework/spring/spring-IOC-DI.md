---
title: spring-IOC-DI
categories: spring
tags:
  - spring
abbrlink: 978c7d15
date: 2022-02-02 12:00:00
---
# Spring IOC

控制反转是一种编程的思想，被动的接收对象，把创建对象的的权利交spring框架，这个是spring框架的主要特征。它包括依依赖注入（Dependency Injection 简称DI）和依赖查找（Dependency Lookup）。`注意:依赖注入和依赖查找是Spring控制反转的一种实现，控制反转思想的实现也不仅这一种方式`

所以IOC的主要功能就是削减程序的耦合性（因为程序中不用主动的去创建对象了）

```xml
<!-- 交给容器即可 -->
<bean id="userService" class="cn.com.lichenghao.service.impl.UserServieImpl"></bean>
```

# IOC底层原理

IOC底层就是个对象工厂，主要依赖xml解析、工厂模式、反射去完成解耦操作

第一步：创建XML文件文件

```xml
<bean id="userService" class="cn.com.lichenghao.service.impl.UserServieImpl"></bean>
```

第二步：工厂类解析xml，利用反射生成对象返回

```java
// 伪代码
public Class BeanFactory(){
  public static Object getBean(){
    // 解析xml，得到 cn.com.lichenghao.service.impl.UserServieImpl
    // 利用反射得到对象返回
  }
}
```

# IOC容器的两个主要的接口

IOC原理是使用工厂，那么Spring有两个主要的工厂接口

![](https://cdn.jsdelivr.net/gh/lichlaughing/pic@latest/img/20200712144657.png)

## BeanFactory

Spring内部使用的接口，是IOC容器的基本实现，一般不提供给开发人员使用

## ApplicationContext 

是BeanFactory的子接口，一般开发使用这个接口，有三个主要的实现类

```java
//读取类路径下配置文件
ApplicationContext ac =
	new ClassPathXmlApplicationContext("beans.xml");
UserService userService = (UserService) ac.getBean("userService");
userService.sayHi("Tom");

//读取磁盘上的配置文件
FileSystemXmlApplicationContext fileSystemXmlApplicationContext =
	new FileSystemXmlApplicationContext("/User/local/beans.xml");
UserService userService1 = (UserService) fileSystemXmlApplicationContext.getBean("userService");
userService1.sayHi("Tom1");

//扫描带注解的
AnnotationConfigApplicationContext annotationConfigApplicationContext =
	new AnnotationConfigApplicationContext("cn.com.lichenghao.service");
UserService userService2 = (UserService) annotationConfigApplicationContext.getBean("userService");
```

## 两个接口的区别？

**核心容器中的接口BeanFactory 和 ApplicationContext 的区别 ？**(面试题)

ApplicationContext  创建对象是使用立即加载的方式，也就是说加载完配置文件，对象就已经创建好了，适用于单例模式

BeanFactory 创建对象是懒加载的方式，只有在使用bean的时候才会创建对象

# IOC bean管理







bean管理的两个操作`创建bean对象`和`注入对象的属性` 两个操作又有两种方式，`xml方式`和`注解`的方式

## XML方式

### 创建bean的方式

#### 默认构造函数

没有额外的属性和标签，走默认的构造函数，如果没有默认的构造函数就会报错

```xml
<bean id="userService" class="cn.com.lichenghao.service.impl.UserServieImpl"></bean>
```

如果么有默认的构造函数报错：

```java
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [cn.com.lichenghao.service.impl.UserServieImpl]: No default constructor found; nested exception is java.lang.NoSuchMethodException: cn.com.lichenghao.service.impl.UserServieImpl.<init>()
```

#### 工厂模式

```java
public class UserFactory {
    public UserServieImpl instance() {
        return new UserServieImpl();
    }
}
```

```xml
 <!-- 工厂 -->
 <bean id="userFactory" class="cn.com.lichenghao.factory.UserFactory"></bean>
 <bean id="userService1" factory-bean="userFactory" factory-method="instance"></bean>
```

#### 静态工厂模式

```java
public class StaticUserFactory {
    public static UserServieImpl instance() {
        return new UserServieImpl();
    }
}
```

```xml
<!-- 静态工厂 -->
    <bean id="userService2" class="cn.com.lichenghao.factory.StaticUserFactory" factory-method="instance"></bean>
```



### 属性依赖注入(DI)

在IOC创建完对象之后，才能进行DI，属性的注入

能注入的三类数据：基本数据类型、String，引用类型，复杂类型/集合类型

注入方式：构造函数，set方法

#### set方法注入

```xml
<bean id="user" class="cn.com.lichenghao.model.User">
  <property name="userName" value="张无忌"></property>
</bean>  
```

#### 有参构造注入

```xml
<!-- 根据属性的名称 -->
<bean id="user" class="cn.com.lichenghao.model.User">
  <constructor-arg name="userName" value="张无忌"></constructor-arg>
</bean>  

<!-- 根据属性的位置 -->
<bean id="user" class="cn.com.lichenghao.model.User">
  <constructor-arg index="0" value="张无忌"></constructor-arg>
</bean>  
```

#### 其他属性的注入

**字面量注入, 注入空值和特殊符号**

注入null

```xml
<bean id="user" class="cn.com.lichenghao.model.User">
  <property name="userName">
    <null/>
  </property>
</bean> 
```

注入特殊符号

```xml
<!-- <![CDATA[带有特殊符号的内容]]> -->
<bean id="user" class="cn.com.lichenghao.model.User">
  <property name="userName">
    <value><![CDATA[《张无忌》]]></value>
  </property>
</bean> 
```

**注入外部的bean**

```xml
<bean id="userRepository" class="cn.com.lichenghao.dao.UserRepository"></bean>
<bean id="user" class="cn.com.lichenghao.service.UserService">
  <property name="userDao" ref="userRepository"></property>
</bean>
```

**注入内部bean和级联赋值**

有部门类和员工类，部分和员工的关系，一对多

内部bean注入

```xml
<bean id="emp" class="cn.com.lichenghao.model.Emp">
  <property name="userName" value="张无忌"></property>
  <property name="gender" value="man"></property>
  <property name="dept">
  	<bean id="dept" class="cn.com.lichenghao.model.Dept">
  		<property name="name" value="技术部"></property>
		</bean>
  </property>
</bean>
```

级联注入

```xml
<bean id="emp" class="cn.com.lichenghao.model.Emp">
  <property name="userName" value="张无忌"></property>
  <property name="gender" value="man"></property>
  <property name="dept" ref="dept"></property>
  <property name="dept.name" value="技术部"></property>
</bean>
<bean id="dept" class="cn.com.lichenghao.model.Dept"></bean>
```

**注入复杂类型**

```java
public class User {
    private String[] strings;
    private List<String> list;
    private Map<String, String> map;
    private Properties properties;
    ......
}
```

```xml
<bean id="user" class="cn.com.lichenghao.model.User">
        <property name="strings">
            <array>
                <value>1</value>
                <value>2</value>
            </array>
        </property>
        <property name="list">
            <list>
                <value>1</value>
                <value>2</value>
            </list>
        </property>
        <property name="map">
            <map>
                <entry key="m1" value="m1"></entry>
                <entry key="m2">
                    <value>m2</value>
                </entry>
            </map>
        </property>

        <property name="properties">
            <props>
                <prop key="p1">p1</prop>
                <prop key="p2">p2</prop>
            </props>
        </property>
</bean>
```

## 注解的方式

开启注解扫描：

```xml
<!-- 多个包逗号,隔开 -->
<context:component-scan base-package="cn.com.lichenghao"></context:component-scan>
```

扫描过滤

```xml
<!-- 只扫描@Controller注解 -->
<context:component-scan base-package="cn.com.lichenghao" use-default-filters="false">
  <context:include-filter type="annotation"    expression="org.springframework.stereotype.Controller">
</context:component-scan>
  
<!-- 不扫描@Controller注解 -->
<context:component-scan base-package="cn.com.lichenghao" use-default-filters="false">
  <context:exclude-filter type="annotation"    expression="org.springframework.stereotype.Controller">
</context:component-scan>
```

使用注解创建对象

@Component @Service @Controller  @Repository 

使用注解注入属性

@AutoWired @Qualifer @Resource

## bean的类型

Spring有两种类型的bean: 普通bean , 工厂bean (Factory Bean)

普通bean: 定义什么bean就返回什么bean

工厂bean: 定义的类型和返回的类型可以是不一样的 （实现FactoryBean 接口，自己指定返回什么样类型的bean）

# 生命周期-作用域

[springbean生命周期和作用域](https://www.lichenghao.com.cn/posts/6109.html)













