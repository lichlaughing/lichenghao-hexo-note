---
title: spring-拦截器
categories: spring
tags:
  - spring
  - interceptor
abbrlink: 4bfdf057
date: 2022-02-02 12:00:00
---


spring拦截器的实现方式和执行顺序。（用springboot快速搭建个web项目，用于测试）

拦截器有两种方式，继承`HandlerInterceptorAdapter`  或者实现`HandlerInterceptor` 如下三个拦截器

```JAVA
@Component
public class FirstInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("FirstInterceptor: preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("FirstInterceptor: postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("FirstInterceptor: afterCompletion");
    }
}
```

```JAVA
@Component
public class SecondInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("SecondInterceptor: preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("SecondInterceptor: postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("SecondInterceptor: afterCompletion");
    }
}
```

```JAVA
@Component
public class ThirdInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("ThirdInterceptor: preHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("ThirdInterceptor: postHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("ThirdInterceptor: afterCompletion");
    }
}
```

注册拦截器

```JAVA
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        //FirstInterceptor 拦截除了/test2/** 以外的所有请求
        InterceptorRegistration first = registry.addInterceptor(new FirstInterceptor());
        first.addPathPatterns("/**");
        first.excludePathPatterns("/test2/**");

        //SecondInterceptor 拦截所有的请求
        InterceptorRegistration second = registry.addInterceptor(new SecondInterceptor());
        second.addPathPatterns("/**");

        //ThirdInterceptor 拦截所有的请求
        InterceptorRegistration third = registry.addInterceptor(new ThirdInterceptor());
        third.addPathPatterns("/**");
    }
}
```

新建个请求，测试拦截器

```JAVA
@RestController
public class IndexController {

    @RequestMapping("/test1/index")
    public String index1() {
        System.out.println("IndexController test1/index");
        return "test1 index";
    }

    @RequestMapping("/test2/index")
    public String index2() {
        System.out.println("IndexController test2/index");
        return "test2 index";
    }
}
```

正常情况下执行`test1/index`，查看拦截器执行情况

```
FirstInterceptor: preHandle
SecondInterceptor: preHandle
ThirdInterceptor: preHandle
IndexController test1/index
ThirdInterceptor: postHandle
SecondInterceptor: postHandle
FirstInterceptor: postHandle
ThirdInterceptor: afterCompletion
SecondInterceptor: afterCompletion
FirstInterceptor: afterCompletion
```

结论：

执行`顺序`按照注册的顺序来执行，如果我们有N个拦截器，那么执行顺序如下：

首先顺序执行N个拦截器的preHandle()方法

> 1:preHandle()—>2:preHandle()...—>n:preHandle()

然后执行我们的controller

> test1/index

紧接着`倒序`执行postHandle()方法

> n:postHandle()—>n-1:postHandle()...—>1:postHandle()

最后`倒序`执行afterCompletion()方法

> n:afterCompletion()—>n-1:afterCompletion()...—>1:afterCompletion()



如果其中一个拦截器返回false,例如我们让`ThirdInterceptor`返回false，那么执行情况是

```
FirstInterceptor: preHandle
SecondInterceptor: preHandle
ThirdInterceptor: preHandle
SecondInterceptor: afterCompletion
FirstInterceptor: afterCompletion
```

结论：

执行`顺序`按照注册的顺序来执行，如果我们有N个拦截器，第m个拦截器拦截后返回false,那么执行顺序如下：

首先顺序执行到第m个拦截器的preHandle()方法

> 1:preHandle()—>2:preHandle()...—>m:preHandle()

此时第m个拦截器成功拦截了请求，返回false,此时便会倒序执行前面的`afterCompletion()`方法

> m-1:afterCompletion()—>m-2:afterCompletion()...—>1:afterCompletion()

