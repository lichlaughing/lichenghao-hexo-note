---
title: Spring 事件监听机制
tags:
  - spring
  - ApplicationEvent
  - ApplicationListener
  - ApplicationEventPublisher
categories: spring
keywords: spring 事件
abbrlink: 3bd56390
cover: https://blog.lichenghao.cn/upload/2022/07/D5E09CE4-spring.png
date: 2022-03-16 11:05:01
---

Spring 事件监听机制也是基于 JDK 规范在框架中进行扩展和完善。	{% post_link java/basic/事件机制 %}

同样也包含三个部分：

- ApplicationEvent：这个抽象类继承了 EventObject
- ApplicationListener：函数式接口继承了 EventListener
- ApplicationEventPublisher：函数式接口，通过 publishEvent 发布事件



## 简单事件监听

模拟用户注册后，发送短信和邮件。

先定义事件

```java
/**
 * 用户注册实体
 *
 * @author chenghao.li
 */
@Getter
@Setter
public class UserRegister extends ApplicationEvent {

    private static final long serialVersionUID = -8392057501920248680L;
    private String userName;
    private String telPhone;
    private String email;

    public UserRegister(Object source) {
        super(source);
    }
}
```

注册监听,有两种方式

```java
/**
 * 注解监听器
 *
 * @author chenghao.li
 */
@Component
public class UserRegisterSendMsgListener {

    @EventListener
    public void handleUserRegisterEvent(UserRegister userRegister) {
        System.out.println("监听到用户注册，准备发送短信:" + userRegister.getTelPhone());
    }
}


/**
 * 监听器（接口实现）
 *
 * @author chenghao.li
 */
@Component
public class UserRegisterSendEmailListener implements ApplicationListener<UserRegister> {

    @Override
    public void onApplicationEvent(UserRegister userRegister) {
        System.out.println("监听到用户注册，准备发送邮件：" + userRegister.getEmail());
    }
}
```

测试事件发布

```java
@RequestMapping("/register")
public String register() {
  UserRegister userRegister = new UserRegister(this);
  userRegister.setUserName("film");
  userRegister.setTelPhone("13333334444");
  userRegister.setEmail("123@admin.com");
  eventPublisher.publishEvent(userRegister);
  return "success";
}
```

上面的代码发生注册事件后，会发现监听会被执行两次。也就是说监听被注册了两次。

原因：

web 项目中如果同时集成了 spring 和 springMVC 的话，上下文中会存在两个容器，即 spring 的applicationContext.xml 父容器和 springMVC 的 applicationContext-mvc.xml 子容器。在通过 ApplicationEventPublisher 发布事件的时候，事件会被两个容器发布，造成上述情况。

解决：

通过父容器发布

```java
@RequestMapping("/register")
public String register() {
  UserRegister userRegister = new UserRegister(this);
  userRegister.setUserName("film");
  userRegister.setTelPhone("13333334444");
  userRegister.setEmail("123@admin.com");
  try {
    WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
    if (context != null) {
      context.publishEvent(userRegister);
    }
  } catch (Exception e) {
    System.out.println("error");
  }
  return "success";
}
```



## 监听器的顺序

默认情况下按照 bean 的加载顺序执行监听器。

### @Order

可以通过 @Order 指定执行顺序，数字越小，优先级越大

```java
@Order(1)
@Component
public class UserRegisterSendMsgListener {

    @EventListener
    public void handleUserRegisterEvent(UserRegister userRegister) {
        System.out.println("监听到用户注册，准备发送短信:" + userRegister.getTelPhone());
    }
}

@Order(2)
@Component
public class UserRegisterSendEmailListener implements ApplicationListener<UserRegister> {

    @Override
    public void onApplicationEvent(UserRegister userRegister) {
        System.out.println("监听到用户注册，准备发送邮件：" + userRegister.getEmail());
    }
}
```

### SmartApplicationListener

通过实现 SmartApplicationListener 接口的 getOrder 方法来指定顺序。

```java
/**
 * 监听器
 *
 * @author chenghao.li
 */
@Component
public class UserRegisterOtherListener implements SmartApplicationListener {

    @Override
    public boolean supportsEventType(Class<? extends ApplicationEvent> aClass) {
        return true;
    }

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        UserRegister event = (UserRegister) applicationEvent;
        System.out.println("监听到用户注册,干点其他的:" + event.getUserName());
    }

    @Override
    public int getOrder() {
        return 3;
    }
}
```



## 异步监听

Spring 事件通知默认是同步的，也就是说需要等待所有的监听都执行完毕后才能继续后序的处理。可以验证下，在任意监听器下睡一会即可。

```java
@Override
public void onApplicationEvent(ApplicationEvent applicationEvent) {
  UserRegister event = (UserRegister) applicationEvent;
  System.out.println("监听到用户注册,干点其他的:" + event.getUserName());
  try {
    TimeUnit.SECONDS.sleep(5);
  } catch (InterruptedException e) {
    System.err.println(e);
  }
}
```

开启异步支持，并配置异步任务的线程池。

```java
/**
 * 线程池配置
 *
 * @author chenghao.li
 */
@EnableAsync
@Configuration
public class ThreadPoolConfig implements AsyncConfigurer {

    @Bean("eventListenerThreadPool")
    @Override
    public Executor getAsyncExecutor() {
        return new ThreadPoolExecutor(5, 10, 60L, TimeUnit.SECONDS
                , new ArrayBlockingQueue<>(50)
                , new MyThreadFactory());
    }

    static class MyThreadFactory implements ThreadFactory {
        final AtomicInteger threadNumber = new AtomicInteger(0);

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(new ThreadGroup("EventListener"), r);
            t.setName("async-thread-" + threadNumber.getAndIncrement());
            t.setDaemon(true);
            return t;
        }
    }
}
```

监听上使用异步 @Async("eventListenerThreadPool") 注解指定配置好的线程池即可。

```java
/**
 * 监听器（接口实现）
 *
 * @author chenghao.li
 */
@Order(2)
@Component
@Async("eventListenerThreadPool")
public class UserRegisterSendEmailListener implements ApplicationListener<UserRegister> {

    @Override
    public void onApplicationEvent(UserRegister userRegister) {
        System.out.println("监听到用户注册，准备发送邮件：" + userRegister.getEmail());
    }
}


/**
 * 注解监听器
 *
 * @author chenghao.li
 */
@Order(1)
@Component
@Async("eventListenerThreadPool")
public class UserRegisterSendMsgListener {
    @EventListener
    public void handleUserRegisterEvent(UserRegister userRegister) {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("监听到用户注册，准备发送短信:" + userRegister.getTelPhone());
    }
}
```

这个时候，SmartApplicationListener 接口的实现是不支持异步注解。使用的话会直接报错。

