---
title: springboot-mybatis
categories: springboot
tags:
  - springboot
abbrlink: '18775504'
date: 2022-02-02 12:00:00
---


**springboot 记录~**



> 软件环境：
>
> IntelliJ IDEA 2020.1.1 (Ultimate Edition)
>
> mysql 5.7.29
>
> springboot 2.3.1.RELEASE



------



新建springboot 项目，引入需要的依赖

```xml
...省略
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

    <groupId>cn.com.lichenghao</groupId>
    <artifactId>springboot-mybatis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-mybatis</name>
    <description>springboot-mybatis</description>

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
...省略
```



## Free Mybatis plugin 生成通用代码

新建一张测试表，然后利用mybatis的插件（[Free MyBatis plugin](https://github.com/wuzhizhan)）生成对应的model,dao,mapper 等

### 新建数据库连接

下载安装插件完毕后，在右侧Database里面新建个本地数据库连接

![](https://tvax4.sinaimg.cn/large/e444d1c7ly1ggeqr4xgcqj215s0o6781.jpg)



> 说明：
>
> 在图中③的位置测试链接，初次使用会在下面提示安装对应数据库驱动，安装即可。在图中④的位置可以切换数据库驱动的版本等。



### 插件生成代码选项

![](https://tvax1.sinaimg.cn/large/e444d1c7ly1ggeqvk9fdkj21fk0obacu.jpg)



> 说明：
>
> 1. 在图中①的位置右键选择 mybatis-genetator  就是为这个表生成对应的代码
> 2. 图中②的位置理论上可以放在任何位置，这里建议直接放在自己对应的项目中，就不用在手动移动了
> 3. 图中⑤的位置就是model代码的位置，④是包名称，③是实体的名称
> 4. dao层的代码和model的设置一样，就是包名不同
> 5. ⑥是mapper所在的文件夹名称，⑦是mapper所在的路径
> 6. 下面的options为其他设置，例如不用lombox可以取消掉



最终生成的代码如下所示：

![](https://tva4.sinaimg.cn/large/e444d1c7ly1gger73hjkyj20g40e3jrw.jpg)



## 配置文件



application.yml

```yaml
server:
  port: 8888
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/springbootstudytest?characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root1234
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
  type-aliases-package: cn.com.lichenghao.springbootmybatis.model

```

application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/springbootstudytest?characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=root1234
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
# mybatis
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
mybatis.type-aliases-package=cn.com.lichenghao.springbootmybatis.model
```

## 启用包扫描

然后在启动类中声明扫描Dao层的路径 **@MapperScan("cn.com.lichenghao.springbootmybatis")**

```java
@SpringBootApplication
@MapperScan("cn.com.lichenghao.springbootmybatis")
public class SpringbootMybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisApplication.class, args);
    }
}
```



## 单元测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class TbUserTestDaoTest {

    @Autowired
    private TbUserTestDao tbUserTestDao;

    /**
     * 增
     */
    @Test
    //@Ignore
    public void testA() {
        TbUserTest tbUserTest = new TbUserTest("1", "Tom", 100);
        int insert = tbUserTestDao.insert(tbUserTest);
        System.out.println("insert:" + insert);
    }

    /**
     * 查
     */
    @Test
    //@Ignore
    public void testB() {
        TbUserTest tbUserTest = tbUserTestDao.selectByPrimaryKey("1");
        System.out.println("selectByPrimaryKey:" + tbUserTest);
    }

    /**
     * 改
     */
    @Test
    //@Ignore
    public void testC() {
        TbUserTest tbUserTest = tbUserTestDao.selectByPrimaryKey("1");
        tbUserTest.setName("Tom01");
        int i = tbUserTestDao.updateByPrimaryKey(tbUserTest);
        System.out.println("updateByPrimaryKey:" + i);
    }

    /**
     * 删
     */
    @Test
    //@Ignore
    public void testD() {
        TbUserTest tbUserTest = tbUserTestDao.selectByPrimaryKey("1");
        int i = tbUserTestDao.deleteByPrimaryKey(tbUserTest.getId());
        System.out.println("deleteByPrimaryKey:" + i);
    }
}
```







## Questions

- Caused by: org.xml.sax.SAXParseException; lineNumber: 1; columnNumber: 1; 前言中不允许有内容

这个错误很有可能是你的配置文件路径不对，系统启动的时候没有找到配置文件

- CONDITIONS EVALUATION REPORT  

这个错误可能是在启动类中没有配置扫描DAO层的路径，启动类上加上**@MapperScan("dao层的路径")**

```java
============================
CONDITIONS EVALUATION REPORT
============================


Positive matches:
-----------------

   AopAutoConfiguration matched:
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

   AopAutoConfiguration.ClassProxyingConfiguration matched:
      - @ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice' (OnClassCondition)
      - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)

   DataSourceAutoConfiguration matched:
      - @ConditionalOnClass found required classes 'javax.sql.DataSource', 'org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType' (OnClassCondition)
      - @ConditionalOnMissingBean (types: io.r2dbc.spi.ConnectionFactory; SearchStrategy: all) did not find any beans (OnBeanCondition)

   DataSourceAutoConfiguration.PooledDataSourceConfiguration matched:
      - AnyNestedCondition 1 matched 1 did not; NestedCondition on DataSourceAutoConfiguration.PooledDataSourceCondition.PooledDataSourceAvailable PooledDataSource found supported DataSource; NestedCondition on DataSourceAutoConfiguration.PooledDataSourceCondition.ExplicitType @ConditionalOnProperty (spring.datasource.type) did not find property 'type' (DataSourceAutoConfiguration.PooledDataSourceCondition)
      - @ConditionalOnMissingBean (types: javax.sql.DataSource,javax.sql.XADataSource; SearchStrategy: all) did not find any beans (OnBeanCondition)
```





