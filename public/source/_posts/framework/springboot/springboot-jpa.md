---
title: springboot-jpa
categories: springboot
tags:
  - springboot
  - jpa
abbrlink: f1d78dc2
date: 2022-02-02 12:00:00
---


> 软件环境：
>
> IntelliJ IDEA 2020.1.1 (Ultimate Edition)
>
> mysql 5.7.29
>
> springboot 2.3.1.RELEASE



------



> JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体[对象持久化](https://baike.baidu.com/item/对象持久化/7316192)到数据库中。 --百度



spring 实现了这个标准，就出来了Spring Data Jpa ，Jpa内置了好多现成的接口，简化了增删改查，分页，排序等操作，它封装了sql的入参和执行，你在其他工具中写好了sql语句，直接拿过来在Jpa中，有时候是不能直接使用的。这和mybatis不同，mybatis几乎和写SQL语句一样，只要会写sql应该就会很容易理解mybatis中mapper的写法。

但是往往越是简洁的东西，出错的时候就难发现问题之处~

下面看下如何使用Spring Data Jpa

首先引入主要依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    	<artifactId>mysql-connector-java</artifactId>
   		<version>${mysql.version}</version>
</dependency>
```

引入Spring Data Jpa后，可以使用IDEA的工具自动生成实体的代码，如下图所示，选择对应的包路径即可，确定后便可自动生成实体。

![](https://tva2.sinaimg.cn/large/e444d1c7ly1ggjtp3s1xbj210l0frjto.jpg)



![](https://tva1.sinaimg.cn/large/e444d1c7ly1ggjtpcuvbmj20yv0mv404.jpg)



dao层操作数据库，Jpa有很多接口，如下所示：

<img src="https://tvax4.sinaimg.cn/mw690/e444d1c7ly1ggjutrzxg0j20ra11udj5.jpg" style="zoom:70%;" />

所以我们直接继承JpaRepository就行了

```java
public interface UserRepository extends JpaRepository<TbUserTestEntity, String> {
}
```

那写个service就可以进行简单的使用了

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public TbUserTestEntity addUser(TbUserTestEntity userTestEntity) {
        TbUserTestEntity save = userRepository.save(userTestEntity);
        return save;
    }

    @Override
    public void deleteUser(String userId) {
        userRepository.deleteById(userId);
    }

    @Override
    public TbUserTestEntity getUser(String userId) {
        return userRepository.getOne(userId);
    }
}
```

别忘了配置文件

```yaml
server:
  port: 8888
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/springbootstudytest?characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root1234
  jpa:
    show-sql: true
```



单元测试下：

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class SpringbootSpringDataJpaApplicationTests {

    @Autowired
    private UserService userService;

    @Test
    public void test1() {
        TbUserTestEntity tbUserTestEntity = new TbUserTestEntity();
        tbUserTestEntity.setId("001");
        tbUserTestEntity.setAg(100);
        tbUserTestEntity.setName("Tom");
        TbUserTestEntity tbUserTestEntity1 = userService.addUser(tbUserTestEntity);
        System.out.println(tbUserTestEntity1);
    }

    @Test
    public void test2() {
        TbUserTestEntity user = userService.getUser("001");
        System.out.println(user);
    }
}
```









