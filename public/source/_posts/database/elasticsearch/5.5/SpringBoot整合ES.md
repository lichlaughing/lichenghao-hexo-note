---
title: Elasticsearch5.5 SpringBoot整合ES
categories: elasticsearch
tags:
  - elasticsearch
  - springboot
keywords: elasticsearch
abbrlink: 448c99cd
date: 2022-02-15 12:41:00
---


# 版本对应关系

官网文档：https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#reference

|                  Spring Data Release Train                   |                  Spring Data Elasticsearch                   | Elasticsearch |                         Spring Boot                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :-----------: | :----------------------------------------------------------: |
|                            Moore                             |                            3.2.x                             |     6.8.4     |                            2.2.x                             |
|                           Lovelace                           |                            3.1.x                             |     6.2.2     |                            2.1.x                             |
| Kay[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#_footnotedef_1)] | 3.0.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#_footnotedef_1)] |     5.5.0     | 2.0.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#_footnotedef_1)] |
| Ingalls[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#_footnotedef_1)] | 2.1.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#_footnotedef_1)] |     2.4.0     | 1.5.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#_footnotedef_1)] |

# 环境

springboot 使用 ES 的基础环境非常简单，添加依赖，添加配置就可以使用

POM 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.5.RELEASE</version>
        <relativePath/>
    </parent>
    <groupId>com.lich</groupId>
    <artifactId>es</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>es</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

添加配置文件 application.yml

```yaml
spring:
  data:
    elasticsearch:
      # 集群名
      cluster-name: elasticsearch
      # 连接节点,注意在集群中通信都是9300端口，否则会报错无法连接上！
      cluster-nodes: 127.0.0.1:9300
```

单元测试：

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class EsApplicationTests {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

    @Test
    public void test1() {
        //分页 match查询
        Pageable pageable = PageRequest.of(0, 10);
        MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("name", "李白");
        SearchQuery searchQuery = new NativeSearchQueryBuilder().withQuery(matchQueryBuilder).withPageable(pageable).build();
        List<Man> mans = elasticsearchTemplate.queryForList(searchQuery, Man.class);
        System.out.println(mans);
    }
}

```

# ElasticsearchTemplate

## 索引相关操作

准备文档实体数据,使用了lombok插件

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Document(indexName = "transport"
        , type = "car"
        , shards = 2
        , replicas = 1
        , createIndex = false
        , refreshInterval = "2s"
        , indexStoreType = "fs")
public class Car implements Serializable {

    @Id
    private String id;

    /**
     * 名称
     */
    @Field(type = FieldType.Keyword)
    private String name;
    /**
     * 价格
     */
    @Field(type = FieldType.Double)
    private Double price;
    /**
     * 颜色
     */
    @Field(type = FieldType.Keyword)
    private String color;
    /**
     * 产地
     */
    @Field(type = FieldType.Keyword)
    private String origin;
    /**
     * 生产日期
     */
    @Field(type = FieldType.Date)
    private String date;
    /**
     * 介绍
     */
    @Field(type = FieldType.Text)
    private String desc;
}
```

@Document 注解参数说明：

 ```properties
indexName: 索引名称

type：索引的类型

shards：分片数，默认5

replicas：每个分片的副本数量，默认1

createIndex：添加文档的时候，如果索引不存在是否自动创建索引,默认是true

refreshInterval: 自动刷新时间，默认1s

indexStoreType: 索引文件的存储类型有以下几种：
	fs:默认文件系统实现，根据当前操作系统选择最佳存储方式。
	simplefs:简单的FS类型，使用随机访问文件实现文件系统存储(映射到Lucene SimpleFsDirectory)。并发性能很差(多线程会出现瓶颈)。当需要索引持久性时，通常最好使用niofs。
	niofs:基于NIOS实现的文件系统，该类型使用NIO在文件系统上存储碎片索引(映射到Lucene NIOFSDirectory)。它允许多个线程同时从同一个文件中读取数据。
	mmapfs:基于文件内存映射机制实现的文件系统实现，该方式将文件映射到内存(MMap)来存储文件系统上的碎片索引(映射到Lucene MMapDirectory)。内存映射使用进程中与被映射文件大小相同的部分虚拟内存地址空间。可以通过node.store.allow_mmapfs属性来禁用基于内存映射机制，如果节点所在的操作系统没有大量的虚拟内存，则可以使用该属性明确禁止使用该文件实现。
 ```



<!-- tabs:start -->

####**索引是否存在**

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class EsIndexTest {
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
     /**
     * 判断索引是否存在
     */
    @Test
    public void testExistsIndex() {
        boolean exists1 = elasticsearchTemplate.indexExists(Car.class);
        Assert.assertTrue(exists1);

        boolean exists2 = elasticsearchTemplate.indexExists(Car.class.getAnnotation(Document.class).indexName());
        Assert.assertTrue(exists2);
    }
}
```

####**添加索引**

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class EsIndexTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    
    /**
     * 创建索引
     */
    @Test
    public void testCreateIndex() {
        boolean index = elasticsearchTemplate.createIndex(Car.class);
        // 如果实体上 createIndex = false，就需要手动添加字段
        boolean mapping = elasticsearchTemplate.putMapping(Car.class);
        Assert.assertTrue(index);
        Assert.assertTrue(mapping);
    }
}
```

####**删除索引**

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class EsIndexTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    
    /**
     * 删除索引
     */
    @Test
    public void testDeleteIndex() {
        boolean deleteIndex = elasticsearchTemplate.deleteIndex(Car.class);
        Assert.assertTrue(deleteIndex);
    }
}
```

  ####**添加/删除索引别名**

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class EsIndexTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    
    /**
     * 索引添加/删除别名
     */
    @Test
    public void testAddAlias() {
        AliasQuery aliasQuery = new AliasQuery();
        aliasQuery.setIndexName("transport");
        aliasQuery.setAliasName("t_all");

        // 添加别名
        //Boolean addAlias = = elasticsearchTemplate.addAlias(aliasQuery);
        //System.out.println(addAlias);

        // 删除别名
        Boolean removeAlias = elasticsearchTemplate.removeAlias(aliasQuery);
        Assert.assertTrue(removeAlias);
    }
}
```

 ####**根据别名获取所有的索引**

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class EsIndexTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    
     /**
     * 根据别名,获取所有的索引
     */
    @Test
    public void testGetAlias() {
        String[] alias = {"trans"};
        ImmutableOpenMap<String, List<AliasMetaData>> aliasMap = elasticsearchTemplate.getClient().admin().indices()
                .prepareGetAliases(alias)
                .get().getAliases();
        Set<String> allIndices = new HashSet<>();
        aliasMap.keysIt().forEachRemaining(allIndices::add);
        System.out.println(allIndices);
    }
}
```

<!-- tabs:end -->



























