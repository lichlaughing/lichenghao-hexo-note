---
title: Springboot 参数校验插件 spring-boot-starter-validation
tags:
  - springboot
  - Valid
  - Validated
categories: springboot
keywords: springboot 参数校验
abbrlink: 763e1a35
date: 2022-07-13 10:39:03
---



JSR303 规范（Bean Validation 规范）为 JavaBean 验证定义了相应的元数据模型和API。Hibernate Validator 对其进行了参考实现。

[JSR303](https://jcp.org/en/jsr/detail?id=303)

[Hibernate Validator](https://hibernate.org/validator/)

> springboot 参数校验最终还是利用了 hibernate-validator 

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```



前提准备

{% tabs validator %}

<!-- tab 实体 -->

用户,加上了基本校验注解

```java
/**
 * 用户
 *
 * @author chenghao.li
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String userId;

    @NotBlank(message = "用户名不能为空!")
    private String name;

    @NotNull(message = "性别不能为空!")
    private Integer gender;

    @NotNull(message = "年龄不能为空!")
    private Integer age;

    private LocalDateTime birthday;

    @NotBlank(message = "邮件不能为空!")
    @Email(message = "邮件格式不正确!")
    private String email;

    private String address;
    private List<User> friends;
}
```

统一返回结果,用于统一异常处理等

```java
/**
 * 统一结果
 *
 * @author chenghao.li
 */
@AllArgsConstructor
@NoArgsConstructor
@Data
public class CommonResult<T> implements Serializable {
    private static final long serialVersionUID = -3643217252980590475L;
    /**
     * true / false
     */
    private boolean success;
    /**
     * 提示信息
     */
    private String msg;

    public static <T> CommonResult<T> success() {
        CommonResult<T> result = new CommonResult<>();
        result.setSuccess(true);
        return result;
    }

    public static <T> CommonResult<T> failure(String reason) {
        CommonResult<T> result = new CommonResult<>();
        result.setSuccess(false);
        result.setMsg(reason);
        return result;
    }
}
```

<!-- endtab -->



<!-- tab 全局异常处理 -->

解析校验框架抛出的校验异常，返回

```java
/**
 * 统一异常
 *
 * @author chenghao.li
 */
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * 参数校验的异常
     */
    @ExceptionHandler({MethodArgumentNotValidException.class})
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public CommonResult<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
        BindingResult bindingResult = ex.getBindingResult();
        StringBuilder sb = new StringBuilder("校验失败:");
        for (FieldError fieldError : bindingResult.getFieldErrors()) {
            sb.append(fieldError.getField()).append("：").append(fieldError.getDefaultMessage()).append(", ");
        }
        return CommonResult.failure(sb.toString());
    }
}
```

<!-- endtab -->

<!-- tab 控制器 -->

测试请求

```java
/**
 * 控制器
 *
 * @author chenghao.li
 */
@RestController
@Slf4j
public class IndexController {

    /**
     * 添加用户
     */
    @PostMapping("/addUser")
    public CommonResult<String> addUser(@RequestBody @Validated User user) {
        log.info("添加用户:{}", user);
        return CommonResult.success();
    }
}
```

<!-- endtab -->



<!-- tab 单元测试 -->

在单元测试中进行测试

```java	
/**
 * 测试
 *
 * @author chenghao.li
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {ValidatedApplicatioin.class})
@Slf4j
@AutoConfigureMockMvc
public class AppTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void shouldAnswerWithTrue() {
        assertTrue(true);
    }

    @Test
    public void AddUserTest() {
       // todo 
    }
}
```

<!-- endtab -->



{% endtabs %}

## 基本校验

关于JSR303–Bean Validation 规范，可以参考[官网](http://jcp.org/en/jsr/detail?id=303)

利用校验框架给的内置注解，即可完成基本参数的校验。其中常用校验注解如下所示：

备注：校验注解是 JSR(Java Specification Requests) 规范给出，Hibernate Validator 对这些规范进行了实现，并且扩充了部分，还支持用户自定义。

| 注解                        | 说明                                                     |
| --------------------------- | -------------------------------------------------------- |
| @Null                       | 被注释的元素必须为 null                                  |
| @NotNull                    | 被注释的元素必须不为 null                                |
| @AssertTrue                 | 被注释的元素必须为 true                                  |
| @AssertFalse                | 被注释的元素必须为 false                                 |
| @Min(value)                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max(value)                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin(value)          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @DecimalMax(value)          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Size(max=, min=)           | 被注释的元素的大小必须在指定的范围内                     |
| @Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| @Future                     | 被注释的元素必须是一个将来的日期                         |
| @Pattern(regex=,flag=)      | 被注释的元素必须符合指定的正则表达式                     |
|                             |                                                          |
| @NotBlank(message =)        | 验证字符串非null，调用trim()后，长度必须大于0            |
| @Email                      | 被注释的元素必须是电子邮箱地址                           |
| @Length(min=,max=)          | 被注释的字符串的大小必须在指定的范围内                   |
| @NotEmpty                   | 不能为null，而且长度必须大于0，用在集合类上面            |
| @Range(min=,max=,message=)  | 被注释的元素必须在合适的范围内                           |

所以在实体上添加注解，然后在请求上开启校验即可。

```java
@NotBlank(message = "用户名不能为空!")
private String name;
```

@Validated 开启校验。使用 @Valid 也可以。（他们有一些区别，下面会做个总结）

@Validated：hibernate 提供

@Valid：JSR规范 提供的

```java
/**
	* 添加用户
 */
@PostMapping("/addUser")
public CommonResult<String> addUser(@RequestBody @Validated User user) {
  log.info("添加用户:{}", user);
  return CommonResult.success();
}
```

单元测试下：

```java
    @Test
    public void AddUserTest() {
        User user = new User();
        user.setUserId("2516ddb0-b1a9-4daf-b9fd-9d09073e341a");
        String param = JSONUtil.toJsonStr(user);
        MvcResult mvcResult;
        try {
            mvcResult = mockMvc.perform(MockMvcRequestBuilders.post("/addUser")
                            .contentType(MediaType.APPLICATION_JSON_VALUE)
                            .content(param)
                    )
                    //.andDo(MockMvcResultHandlers.print()) //打印中间的一些参数信息
                    .andReturn();
            log.info(mvcResult.getResponse().getContentAsString(Charset.forName(StandardCharsets.UTF_8.toString())));
        } catch (Exception e) {
            log.error("There are some exceptions", e);
        }
    }
```

结果：

```java
logback [main] INFO  cn.lichenghao.AppTest - {"success":false,"msg":"校验失败:name：用户名不能为空!, gender：性别不能为空!, email：邮件不能为空!, age：年龄不能为空!, "}
```

## 开启 Fail-Fast

通过上面的结果可以看出，框架把所有参数都校验了一遍才抛出异常结果，在实际情况下，我们需要只要有一个参数校验失败就应该立刻抛出校验异常，这需要开启Fali Fast模式。增加如下配置即可。

```java
/**
 * 校验配置
 *
 * @author chenghao.li
 */
@Configuration
public class ValidatedConfig {
    /**
     * 开启 Fali-Fast 模式，一旦校验失败就立即返回
     */
    @Bean
    public Validator validator() {
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
                .configure()
                .failFast(true)
                .buildValidatorFactory();
        return validatorFactory.getValidator();
    }
}
```

再次测试结果,遇到校验失败就返回结果。

```java
logback [main] INFO  cn.lichenghao.AppTest - {"success":false,"msg":"校验失败:gender：性别不能为空!, "}
```



## 分组校验

场景也是很常见的，比如新增用户不需要指定 ID,但是更新用户肯定需要 ID 了。

定义多个接口，作为分组的依据

```java
/**
 * 校验分组：添加
 *
 * @author chenghao.li
 */
public interface ValidGroupAdd {
}

/**
 * 校验分组：更新
 *
 * @author chenghao.li
 */
public interface ValidGroupUpdate {
}
```

然后在实体上增加分组条件,表示在该分组下校验生效

```java
@NotBlank(message = "用户ID不能为空", groups = {ValidGroupAdd.class})
private String userId;

@NotBlank(message = "用户名不能为空!", groups = {ValidGroupAdd.class, ValidGroupUpdate.class})
private String name;
```

然后在请求上，开启分组校验,多个分组用英文逗号隔开。

{% note danger flat %}

​	1. 如果开启了分组校验，那么只会校验分组的参数，其他未分组的参数就不会校验了。

​	2. @Valid 不支持分组校验

{% endnote %}

```java
    /**
     * 添加用户
     */
    @PostMapping("/addUser")
    public CommonResult<String> addUser(@RequestBody @Validated({ValidGroupAdd.class}) User user) {
        log.info("添加用户:{}", user);
        return CommonResult.success();
    }

    /**
     * 更新用户
     */
    @PostMapping("/updateUser")
    public CommonResult<String> updateUser(@RequestBody @Validated({ValidGroupAdd.class, ValidGroupUpdate.class}) User user) {
        log.info("更新用户:{}", user);
        return CommonResult.success();
    }
```

测试下

```java
    @Test
    public void UpdateUserTest() {
        User user = new User();
        user.setName("cheap");
        String param = JSONUtil.toJsonStr(user);
        MvcResult mvcResult;
        try {
            mvcResult = mockMvc.perform(MockMvcRequestBuilders.post("/updateUser")
                            .contentType(MediaType.APPLICATION_JSON_VALUE)
                            .content(param)
                    )
                    //.andDo(MockMvcResultHandlers.print()) //打印中间的一些参数信息
                    .andReturn();
            log.info(mvcResult.getResponse().getContentAsString(Charset.forName(StandardCharsets.UTF_8.toString())));
        } catch (Exception e) {
            log.error("There are some exceptions", e);
        }
    }
```

结果

```java
logback [main] INFO  cn.lichenghao.AppTest - {"success":false,"msg":"校验失败:userId：用户ID不能为空, "}
```



## 自定义校验

在某些特殊的情况下，我们需要的校验内置的注解已经不能满足，需要自定义一些校验规则。

### 直接校验

大家称之为编程式校验，说白了就是代码直接校验。在参数上代替了注解的工作，但是还是需要利用已有的注解。如下

```java
    /**
     * 添加用户
     */
    @PostMapping("/addUser2")
    public CommonResult<String> addUser2(@RequestBody User user) {
        Set<ConstraintViolation<User>> validate = globalValidator.validate(user, ValidGroupAdd.class);
        // 校验失败
        if (!validate.isEmpty()) {
            List<String> msgList = Lists.newArrayList();
            for (ConstraintViolation<User> violation : validate) {
                // 校验失败，做其它逻辑
                msgList.add(violation.getMessage());
            }
            return CommonResult.failure(Joiner.on(",").join(msgList));
        }
        return CommonResult.success();
    }
```

### 自定义注解

自定义的注解的好处就是可以实现校验的复用。

新增前缀校验注解 @Prefix 模仿已有的注解写就可以。

```java
/**
 * 自定义前缀校验注解
 *
 * @author chenghao.li
 */
@Documented
@Constraint(validatedBy = PrefixValidator.class)
@Target({METHOD, FIELD})
@Retention(RUNTIME)
public @interface Prefix {

    String prefix() default "";

    String message() default "";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    @Target({ElementType.METHOD, ElementType.FIELD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @interface List {
        Prefix[] value();
    }
}
```

增加注解的校验实现

```java
/**
 * 自定义前缀校验注解的实现
 *
 * @author chenghao.li
 */
@Component
public class PrefixValidator implements ConstraintValidator<Prefix, String> {
    private String prefix;

    @Override
    public void initialize(Prefix constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
        this.prefix = constraintAnnotation.prefix();
    }

    /**
     * 根据给定的前缀进行校验
     */
    @Override
    public boolean isValid(String s, ConstraintValidatorContext context) {
        if (s == null || "".equals(s)) {
            return false;
        }
        if (prefix == null || "".equals(prefix)) {
            return true;
        }
        return s.startsWith(prefix);
    }
}
```

在实体上增加我们的自定义校验注解

```java
@Prefix(message = "地址必须以'address'开头", prefix = "address", groups = {ValidGroupAdd.class, ValidGroupUpdate.class})
private String address;
```

测试结果

```java
logback [main] INFO  cn.lichenghao.AppTest - {"success":false,"msg":"校验失败:address：地址必须以'address'开头, "}
```



## 集合校验

只需要在集合字段上增加即可 @Valid 表示集合中的实体也要满足注解上的校验。注意使用 @Validated 不可以因为它不支持。

```java
@NotNull
@Valid
private List<User> friends;
```

测试下

```java
    /**
     * 测试集合校验
     */
    @Test
    public void AddUserTest3() {
        User user = new User();
        user.setUserId("2516ddb0-b1a9-4daf-b9fd-9d09073e341a");
        user.setName("vain");
        user.setEmail("vain@admin.com");

        User user2 = new User();
        user2.setUserId("aac6e7af-40a6-4660-a445-808561664a0b");
        List<User> friends = Lists.newArrayList(user2);

        user.setFriends(friends);
        String param = JSONUtil.toJsonStr(user);

        MvcResult mvcResult;
        try {
            mvcResult = mockMvc.perform(MockMvcRequestBuilders.post("/addUser2")
                            .contentType(MediaType.APPLICATION_JSON_VALUE)
                            .content(param)
                    )
                    //.andDo(MockMvcResultHandlers.print()) //打印中间的一些参数信息
                    .andReturn();
            log.info(mvcResult.getResponse().getContentAsString(Charset.forName(StandardCharsets.UTF_8.toString())));
        } catch (Exception e) {
            log.error("There are some exceptions", e);
        }
    }
```

结果

```java
logback [main] INFO  cn.lichenghao.AppTest - {"success":false,"msg":"用户名不能为空!"}
```



## 组合校验

这个场景也很常见，比如开始时间要小于结束时间。

内置的校验注解只能作用在单个字段上，开始时间结束时间往往需要两个字段，甚至复杂的情况需要多个字段组合判断校验。这个时候只能自定义校验实现了。

新增自定义注解

```java
/**
 * 自定义时间校验注解，结束时间要大于等于开始时间
 *
 * @author chenghao.li
 */
@Documented
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = {StartEndTimeValidator.class})
public @interface StartEndTime {

    String startTime() default "";

    String endTime() default "";

    String message() default "开始时间不能大于结束时间!";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    @Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE, ElementType.TYPE_USE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @interface List {
        StartEndTime[] value();
    }
}
```

注解的实现

```java
/**
 * 自定义前缀校验注解的实现
 *
 * @author chenghao.li
 */
@Component
public class StartEndTimeValidator implements ConstraintValidator<StartEndTime, Object> {
    private String startTime;
    private String endTime;

    @Override
    public void initialize(StartEndTime constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
        this.startTime = constraintAnnotation.startTime();
        this.endTime = constraintAnnotation.endTime();
    }

    /**
     * 根据给定的前缀进行校验
     */
    @Override
    public boolean isValid(Object value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        BeanWrapper beanWrapper = new BeanWrapperImpl(value);
        Object startValue = beanWrapper.getPropertyValue(startTime);
        Object endValue = beanWrapper.getPropertyValue(endTime);
        if (Objects.nonNull(startValue) && Objects.nonNull(endValue)) {
            LocalDateTime start = (LocalDateTime) startValue;
            LocalDateTime end = (LocalDateTime) endValue;
            if (start.isAfter(end)) {
                return false;
            }
        }
        return true;
    }
}
```

这个时候需要把注解放在实体类上，因为需要反射获取到字段的值

```java
@StartEndTime(startTime = "startTime", endTime = "endTime", message = "开始时间不能大于结束时间", groups = {ValidGroupAdd.class, ValidGroupUpdate.class})
public class User {
  	....
      
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime startTime;

    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime endTime;
}
```

还需要重新修改下全局异常，因为最开始找的是字段的注解异常,注解放在实体上则会获取不到，所以修改为全部的异常。

```java
/**
 * 统一异常
 *
 * @author chenghao.li
 */
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * 参数校验的异常
     */
    @ExceptionHandler({MethodArgumentNotValidException.class})
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public CommonResult<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
        BindingResult bindingResult = ex.getBindingResult();
        // StringBuilder sb = new StringBuilder("校验失败:");
        // for (FieldError fieldError : bindingResult.getFieldErrors()) {
        //     sb.append(fieldError.getField()).append("：").append(fieldError.getDefaultMessage()).append(", ");
        // }
        List<Object> msgList = Lists.newArrayList();
        List<ObjectError> allErrors = bindingResult.getAllErrors();
        for (ObjectError allError : allErrors) {
            msgList.add(allError.getDefaultMessage());
        }
        return CommonResult.failure(Joiner.on(",").join(msgList));
    }
}
```

测试结果

```java
logback [main] INFO  cn.lichenghao.AppTest - {"success":false,"msg":"开始时间不能大于结束时间"}
```



## @Valid 和 @Validated 区别

| 区别     | @Valid                                           | @Validated                                                   |
| -------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 提供者   | [JSR303](https://jcp.org/en/jsr/detail?id=303)   | spring [Hibernate Validator](https://hibernate.org/validator/) |
| 支持分组 | ×                                                | √                                                            |
| 注解位置 | METHOD, FIELD, CONSTRUCTOR, PAR AMETER, TYPE_USE | TYPE, METHOD, PARAMETER                                      |
| 嵌套校验 | √                                                | ×                                                            |









