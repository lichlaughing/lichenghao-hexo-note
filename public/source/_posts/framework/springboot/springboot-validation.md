---
title: springboot 参数校验
categories: springboot
tags:
  - springboot
  - validator
abbrlink: c63c8951
date: 2022-02-02 12:00:00
---
使用参数校验插件，避免在代码中不断的 if 判断空，判断空字符串等，并且在同一的异常类中处理相关的校验信息！
Springboot 版本：2.4.4 , Gradle 构建

## spring-boot-starter-validation



引入校验插件的依赖

```properties
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

### 基本校验

在实体上使用注解标注

```java
/**
 * @author chenghao.li
 */
public class UserDTO {
    @NotBlank(message = "用户名不能为空！")
    private String userName;
    @NotBlank(message = "密码不能为空！")
    private String password;
		......省略了get set 等
}

```

使用注解`@Valid`在请求上开启校验

```java
@PostMapping("/login")
    public ResultVO login(@RequestBody @Valid UserDTO user) {
        return loginService.login(user);
}
```

在一个类中处理全局的异常，借助`@RestControllerAdvice` 和 `@ExceptionHandler` 这两个注解实现全局异常的处理

```java
/**
 * @author chenghao.li
 * 全局异常处理
 */
@RestControllerAdvice
public class GlobalExceptionHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    /**
     * 参数绑定时候抛出的异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultVO handleBindException(BindException ex) {
        List<String> defaultMsg = ex.getBindingResult().getAllErrors()
                .stream()
                .map(ObjectError::getDefaultMessage)
                .collect(Collectors.toList());
        String msg = defaultMsg.stream().collect(Collectors.joining(SeparatorConstant.COMMA));
        return ResultVO.OK(false, msg, null, null);
    }

    /**
     * RuntimeException
     */
    @ExceptionHandler(RuntimeException.class)
    public ResultVO handleRuntimeException(RuntimeException ex) {
        String errMsg = ex.getMessage();
        return ResultVO.OK(false, errMsg, null, null);
    }

    /**
     * Exception
     */
    @ExceptionHandler(Exception.class)
    public ResultVO handleException(Exception ex) {
        String msg = ex.getMessage();
        return ResultVO.OK(false, msg, null, null);
    }
}
```

### 分组校验

### 自定义校验

自定义注解，自定义校验类



{% note info disabled %}
附：实体上可以使用的注解
{% endnote %}

| 注解                                  | 说明                                                     |
| ------------------------------------- | -------------------------------------------------------- |
| @Null                                 | 被注释的元素必须为 null                                  |
| @NotNull                              | 被注释的元素必须不为 null                                |
| @AssertTrue                           | 被注释的元素必须为 true                                  |
| @AssertFalse                          | 被注释的元素必须为 false                                 |
| @Min(value)                           | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max(value)                           | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin(value)                    | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @DecimalMax(value)                    | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Size(max=, min=)                     | 被注释的元素的大小必须在指定的范围内                     |
| @Digits (integer, fraction)           | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| @Future                               | 被注释的元素必须是一个将来的日期                         |
| @Pattern(regex=,flag=)                | 被注释的元素必须符合指定的正则表达式                     |
|                                       |                                                          |
| **Hibernate Validator提供的校验注解** |                                                          |
| @NotBlank(message =)                  | 验证字符串非null，调用trim()后，长度必须大于0            |
| @Email                                | 被注释的元素必须是电子邮箱地址                           |
| @Length(min=,max=)                    | 被注释的字符串的大小必须在指定的范围内                   |
| @NotEmpty                             | 不能为null，而且长度必须大于0，用在集合类上面            |
| @Range(min=,max=,message=)            | 被注释的元素必须在合适的范围内                           |

其中，引入了`spring-boot-starter-web` 的话，会自动引入 Hibernate Validator 相关的依赖。

## fluent-validator

GitHub：https://github.com/neoremind/fluent-validator

使用文档：[Java的业务逻辑验证框架fluent-validator](http://neoremind.com/2016/02/java%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91%E9%AA%8C%E8%AF%81%E6%A1%86%E6%9E%B6fluent-validator/)



