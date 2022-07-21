---
title: Neo4j 安装及入门的使用
categories: neo4j
tags:
  - neo4j
  - cypher
  - springboot
keywords: neo4j
cover: https://blog.lichenghao.cn/upload/2022/07/2c72a246-neo4j.png
abbrlink: 2c72a246
date: 2022-02-20 12:30:00
---
Neo4j

官网地址：https://neo4j.com/

下载地址：https://neo4j.com/download-center/#community

官方入门文档：https://neo4j.com/docs/getting-started

## 安装

### Window 安装

window 下安装下载对应的安装包，然后配置环境变量即可。

增加系统环境变量 NEO4J_HOME；

修改 path 环境变量增加%NEO4J_HOME%\bin

cmd 窗口执行 ：neo4j console 便可以启动数据库。

```bash
C:\Users\lich0>neo4j console
2021-02-24 07:24:33.778+0000 INFO  ======== Neo4j 3.5.26 ========
2021-02-24 07:24:33.795+0000 INFO  Starting...
2021-02-24 07:24:37.133+0000 INFO  Bolt enabled on 127.0.0.1:7687.
2021-02-24 07:24:38.110+0000 INFO  Started.
2021-02-24 07:24:38.840+0000 INFO  Remote interface available at http://localhost:7474/
```

### Linux 安装

下载安装包，解压后，简单修改配置文件就可以使用。云服务器的话需要放开对应的端口。

配置文件修改：

```properties
# 可以读写
dbms.read_only=false
# 可以远程访问
dbms.connectors.default_listen_address=0.0.0.0
```

启动服务

```bash
[root@VM-0-2-centos bin]# ./neo4j start
Active database: graph.db
Directories in use:
  home:         /opt/neo4j-community-3.5.26
  config:       /opt/neo4j-community-3.5.26/conf
  logs:         /opt/neo4j-community-3.5.26/logs
  plugins:      /opt/neo4j-community-3.5.26/plugins
  import:       /opt/neo4j-community-3.5.26/import
  data:         /opt/neo4j-community-3.5.26/data
  certificates: /opt/neo4j-community-3.5.26/certificates
  run:          /opt/neo4j-community-3.5.26/run
Starting Neo4j.
Started neo4j (pid 20669). It is available at http://localhost:7474/
There may be a short delay until the server is ready.
See /opt/neo4j-community-3.5.26/logs/neo4j.log for current status.
```

查看状态

```sh
[root@VM-0-2-centos bin]# ./neo4j status
Neo4j is running at pid 20669
```

关闭服务

```sh
[root@VM-0-2-centos bin]# ./neo4j stop
Stopping Neo4j.. stopped
```

访问服务即可：http://xxxxx:7474/browser/

### Docker 部署

拉取镜像，启动就行了呀

拉取镜像

```shell
[root@VM-0-2-centos ~]# docker pull neo4j:3.5.26
[root@VM-0-2-centos ~]# docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
docker.io/neo4j           3.5.26              27b84ff7b769        6 weeks ago         333 MB
```

启动容器，建议写个启动的脚本：neo4j_start.sh

这里需要注意脚本权限，映射的目录的访问权限

```sh
sudo docker run -it --name neo4j -e NEO4J_AUTH=neo4j/123456 -p 7474:7474 -p 7687:7687 \
-v /data/neo4j/data:/data \
-v /data/neo4j/neo4j/logs:/logs \
-v /data/neo4j/conf:/conf \
-v /data/neo4j/metrics:/metrics \
-v /data/neo4j/plugins:/plugins \
-v /data/neo4j/import:/import \
-d neo4j:3.5.26
```

查看启动的容器

```sh
[root@VM-0-2-centos neo4j]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
b92f8beb7d82        neo4j:3.5.26        "/sbin/tini -g -- ..."   3 minutes ago       Up 3 minutes        0.0.0.0:7474->7474/tcp, 7473/tcp, 0.0.0.0:7687->7687/tcp   neo4j

```

访问服务即可：http://xxxxx:7474/browser/

## 基本概念

`Node 节点`

图的基本单位主要是节点和关系，节点和关系都可以包含多个 k-v 结构的属性。

`Relationships 关系`

关系的主要功能就是连接节点，一个关系连接两个节点，一个开始和一个结束节点，这种关系是有进出方向的。

`Properties 属性`

属性就可以理解为关系型数据库中表中的所有字段了，节点和关系都可以有多个属性，并且属性的 key 为字符串，值可以是其他类型例如数值，布尔

`Lables 标签`

标签和 ES 的别名是一样的作用，可以用来划分一类的节点，查询的时候可以通过标签来分类查询。

`Travelsal 遍历`

查询的时候往往都是遍历图谱，遍历的过程中会有一个开始节点，然后根据 Cypher 语句，遍历相关的节点和关系得到最终的结果

`Paths 路径`

路径是一个或者多个节点通过关系连接起来，例如通过 Cypher 查询得到的结果

`Schema 模式`

neo4j 是类似 ES 一样的无模式图谱数据库，使用它无需要定义 schema

`Indexes 索引`

和关系型数据库中的索引一样为了加快检索的速度，这里构建索引是一个异步的过程，只有创建成功后才会生效。如果创建失败可以删除后从新创建。

`Constraints 约束`

约束可以定义在某个字段上，限制字段值唯一，创建约束会自动创建索引

## Cypher

语句 cypher 是 neo4j 官网提供的声明式查询语言，非常强大，用它可以完成任意的图谱里面的查询过滤。就好像关系型数据库中的 SQL 语句一样

### 创建节点

创建电影相关的节点

```sql
-- 创建一个节点：黑客帝国； 标签：电影；属性：标题-The Matrix，发布年份-1997
CREATE (matrix:Movie { title:"The Matrix",released:1997 })
-- 下面的同理
CREATE (cloudAtlas:Movie { title:"Cloud Atlas",released:2012 })
CREATE (forrestGump:Movie { title:"Forrest Gump",released:1994 })
```

创建演员相关的节点

```sql
CREATE (keanu:Person { name:"Keanu Reeves", born:1964 })
CREATE (robert:Person { name:"Robert Zemeckis", born:1951 })
CREATE (tom:Person { name:"Tom Hanks", born:1956 })
CREATE (jerry:Person { name:"Jerry", born:1986 })
```

创建关系

```sql
-- Tom Hanks在电影Forrest Gump中出演Forrest角色
CREATE (tom)-[:ACTED_IN { roles: ["Forrest"]}]->(forrestGump)
-- Tom Hanks在电影Forrest Gump中出演Forrest角色
CREATE (tom)-[:ACTED_IN { roles: ['Zachry']}]->(cloudAtlas)
-- jerry 在电影 The Matrix 出演 Jerry角色
CREATE (jerry)-[:ACTED_IN { roles: ["J"]}]->(matrix)
-- Robert Zemeckis指导电影Forrest Gump
CREATE (robert)-[:DIRECTED]->(forrestGump)
```

!>注意：上面创建节点，关系的语句需要同时执行后，节点才会有关联关系。

!>如果你的创建节点和创建关系的语句是分开执行的话，会是如下的结果：因为创建的关系的时候没有指定节点是属性会被认为是新的节点

如果是分开创建的话，创建关系的语句应该是如下所示：

```sql
-- Tom Hanks在电影Forrest Gump中出演Forrest角色
MATCH (tom:Person{name:"Tom Hanks"}),(forrestGump:Movie{title:"Forrest Gump"}) CREATE (tom)-[:ACTED_IN { roles: ["Forrest"]}]->(forrestGump)

-- Tom Hanks在电影Forrest Gump中出演Forrest角色
MATCH (tom:Person{name:"Tom Hanks"}),(cloudAtlas:Movie{title:"Cloud Atlas"}) CREATE (tom)-[:ACTED_IN { roles: ['Zachry']}]->(cloudAtlas)

-- jerry 在电影 The Matrix 出演 Jerry角色
MATCH (jerry:Person{name:"Jerry"}),(matrix:Movie{title:"The Matrix"}) CREATE (jerry)-[:ACTED_IN { roles: ["J"]}]->(matrix)

-- Robert Zemeckis指导电影Forrest Gump
MATCH (robert:Person{name:"Robert Zemeckis"}),(forrestGump:Movie{title:"Forrest Gump"}) CREATE (robert)-[:DIRECTED]->(forrestGump)
```

### 过滤查询

#### 查询所有节点

```sql
MATCH (nodes) return nodes
```

结果:

```json
╒════════════════════════════════════════╕
│"nodes"                                 │
╞════════════════════════════════════════╡
│{"title":"The Matrix","released":1997}  │
├────────────────────────────────────────┤
│{"title":"Cloud Atlas","released":2012} │
├────────────────────────────────────────┤
│{"title":"Forrest Gump","released":1994}│
├────────────────────────────────────────┤
│{"name":"Keanu Reeves","born":1964}     │
├────────────────────────────────────────┤
│{"name":"Robert Zemeckis","born":1951}  │
├────────────────────────────────────────┤
│{"name":"Tom Hanks","born":1956}        │
└────────────────────────────────────────┘
```

#### 查询标签是 Movie 的节点

```sql
MATCH (m:Movie) return m
```

结果：

```json
╒════════════════════════════════════════╕
│"m"                                     │
╞════════════════════════════════════════╡
│{"title":"The Matrix","released":1997}  │
├────────────────────────────────────────┤
│{"title":"Cloud Atlas","released":2012} │
├────────────────────────────────────────┤
│{"title":"Forrest Gump","released":1994}│
└────────────────────────────────────────┘
```

#### 根据标题查询

```sql
-- 查询标题是The Matrix的节点
MATCH (m:Movie) WHERE m.title = "The Matrix" RETURN m
```

结果：

```json
╒══════════════════════════════════════╕
│"m"                                   │
╞══════════════════════════════════════╡
│{"title":"The Matrix","released":1997}│
└──────────────────────────────────────┘
```

#### 多个条件查询

```sql
-- 条件查询 和SQL语句一样
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie) WHERE p.name =~ "K.+" OR m.released > 2000 OR "Neo" IN r.roles RETURN p,r,m
```

结果：

```json
╒═════════════════════════════════════════════════════════════════════════════════════════════╕
│"p"                             │"r"                 │"m"                                    │
╞═════════════════════════════════════════════════════════════════════════════════════════════╡
│{"name":"Tom Hanks","born":1956}│{"roles":["Zachry"]}│{"title":"Cloud Atlas","released":2012}│
└─────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 删除

#### 删除所有节点和所有关系

```sql
MATCH (r)
DETACH DELETE r
```

#### 根据 ID 删除节点及所有关系

```sql
MATCH (r)
WHERE id(r) = 493
DETACH DELETE r
```

### 结果增加列

和 sql 语句一样，可以给返回结果增加列

```sql
MATCH (p:Person)
RETURN p, p.name AS name, toUpper(p.name), coalesce(p.nickname,"n/a") AS nickname,
  { name: p.name, label:head(labels(p))} AS person
```

JSON 结果：

```json
[
  {
    "p": {
      "identity": 65,
      "labels": ["Person"],
      "properties": {
        "name": "Keanu Reeves",
        "born": 1964
      }
    },
    "name": "Keanu Reeves",
    "toUpper(p.name)": "KEANU REEVES",
    "nickname": "n/a",
    "person": {
      "name": "Keanu Reeves",
      "label": "Person"
    }
  },
  {
    "p": {
      "identity": 66,
      "labels": ["Person"],
      "properties": {
        "name": "Robert Zemeckis",
        "born": 1951
      }
    },
    "name": "Robert Zemeckis",
    "toUpper(p.name)": "ROBERT ZEMECKIS",
    "nickname": "n/a",
    "person": {
      "name": "Robert Zemeckis",
      "label": "Person"
    }
  },
  {
    "p": {
      "identity": 67,
      "labels": ["Person"],
      "properties": {
        "name": "Tom Hanks",
        "born": 1956
      }
    },
    "name": "Tom Hanks",
    "toUpper(p.name)": "TOM HANKS",
    "nickname": "n/a",
    "person": {
      "name": "Tom Hanks",
      "label": "Person"
    }
  }
]
```

### 排序分页

演员按照出演电影的数量排序取前 10 个

```sql
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
RETURN a, count(*) AS appearances
ORDER BY appearances DESC SKIP 0 LIMIT 10;
```

结果：

```json
╒════════════════════════════════╤═════════════╕
│"a"                             │"appearances"│
╞════════════════════════════════╪═════════════╡
│{"name":"Tom Hanks","born":1956}│2            │
├────────────────────────────────┼─────────────┤
│{"name":"Jerry","born":1986}    │1            │
└────────────────────────────────┴─────────────┘
```

### 聚合统计查询

查看演员的总数量

```sql
MATCH (:Person)
RETURN count(*) AS people
```

结果：

```json
╒════════╕
│"people"│
╞════════╡
│4       │
└────────┘
```

统计出演电影的演员

```sql
MATCH (m:Movie)<-[:ACTED_IN]-(a:Person)
RETURN m.title AS movie, collect(a.name) AS cast, count(*) AS actors
```

结果：

```json
╒══════════════╤═════════════╤════════╕
│"movie"       │"cast"       │"actors"│
╞══════════════╪═════════════╪════════╡
│"The Matrix"  │["Jerry"]    │1       │
├──────────────┼─────────────┼────────┤
│"Cloud Atlas" │["Tom Hanks"]│1       │
├──────────────┼─────────────┼────────┤
│"Forrest Gump"│["Tom Hanks"]│1       │
└──────────────┴─────────────┴────────┘
```

### UNION / WITH

union 用来组合查询的结果，查询电影的演职人员表

```sql
MATCH (actor:Person)-[r:ACTED_IN]->(movie:Movie)
RETURN actor.name AS name, type(r) AS type, movie.title AS title
UNION
MATCH (director:Person)-[r:DIRECTED]->(movie:Movie)
RETURN director.name AS name, type(r) AS type, movie.title AS title
```

结果：

```json
╒═════════════════╤══════════╤══════════════╕
│"name"           │"type"    │"title"       │
╞═════════════════╪══════════╪══════════════╡
│"Jerry"          │"ACTED_IN"│"The Matrix"  │
├─────────────────┼──────────┼──────────────┤
│"Tom Hanks"      │"ACTED_IN"│"Cloud Atlas" │
├─────────────────┼──────────┼──────────────┤
│"Tom Hanks"      │"ACTED_IN"│"Forrest Gump"│
├─────────────────┼──────────┼──────────────┤
│"Robert Zemeckis"│"DIRECTED"│"Forrest Gump"│
└─────────────────┴──────────┴──────────────┘
```

或者和下面的查询是等价的

```sql
MATCH (actor:Person)-[r:ACTED_IN|DIRECTED]->(movie:Movie)
RETURN actor.name AS name, type(r) AS type, movie.title AS title
```

WITH 关键字可以看成是查询的语句的一个子条件，主要是用来过滤数据用的，比如

查询出演次数大于一次的演员和相关的电影

```sql
MATCH (person:Person)-[:ACTED_IN]->(m:Movie)
WITH person, count(*) AS appearances, collect(m.title) AS movies
WHERE appearances > 1
RETURN person.name, appearances, movies
```

结果：

```json
╒═════════════╤═════════════╤══════════════════════════════╕
│"person.name"│"appearances"│"movies"                      │
╞═════════════╪═════════════╪══════════════════════════════╡
│"Tom Hanks"  │2            │["Cloud Atlas","Forrest Gump"]│
└─────────────┴─────────────┴──────────────────────────────┘
```

## Springboot 集成

### 测试版本：2.0.3.RELEASE

#### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
```

#### 配置文件 application.yml

注意格式的正确

```yaml
spring:
  data:
    neo4j:
      uri: bolt://localhost:7687
      username: neo4j
      password: 123456
```

#### 定义节点、关系对象

（省略了 Get/Set 等方法）

```java
/**
 * @author chenghao.li
 * @Description 电影节点
 */
@NodeEntity(label = "Movie")
public class MovieGraph implements Serializable {
    /**
     * 主键
     */
    @Id
    @GeneratedValue
    private Long id;

    /**
     * 电影标题
     */
    @Property(name = "title")
    private String title;

    /**
     * 发布年份
     */
    @Property(name = "released")
    private Integer released;
}
```

```java
/**
 * @author chenghao.li
 * @Description 演员节点
 */
@NodeEntity(label = "People")
public class PeopleGraph implements Serializable {
    @Id
    @GeneratedValue
    private Long id;

    @Property(name = "name")
    private String name;

    @Property(name = "born")
    private Integer born;
}
```

```java
/**
 * @author chenghao.li
 * @Description 出演关系
 */
@RelationshipEntity(type = "ACTED_IN")
public class ActedInRelationShip implements Serializable {
    @Id
    @GeneratedValue
    private Long id;

    @StartNode
    private PeopleGraph startNode;

    @EndNode
    private MovieGraph endNode;

    @Property(name = "roles")
    private String roles;
}
```

```java
/**
 * @author chenghao.li
 * @Description 指导关系
 */
@RelationshipEntity(type = "DIRECTED")
public class DirectedRelationShip implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @StartNode
    private PeopleGraph startNode;

    @EndNode
    private MovieGraph endNode;
}
```

涉及到的一些注解说明：

`@NodeEntity(label = "People")` 表示图中的一个节点

`@Id` 主键 ID,使用 Long 类型

`@GeneratedValue` 自动生成主键 ID 的值

`@Property(name = "name")` 表示图中节点的一个属性

`@RelationshipEntity(type = "ACTED_IN")` 表示图中连线的关系

#### 定义 Repository

以演员为例，继承`Neo4jRepository` 即可实现增删改查分页排序等操作；其他的都是如此

```java
/**
 * @author chenghao.li
 */
@Repository
public interface PeopleRepository extends Neo4jRepository<PeopleGraph, Long> {

    /**
     * 根据姓名查找
     *
     * @param name 姓名
     * @return PeopleGraph
     */
    @Query("MATCH (p:People) where p.name={0} return p")
    PeopleGraph findByName(String name);

}
```

#### 单元测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class Neo4jApplicationTests {

    @Autowired
    private PeopleRepository peopleRepository;
    @Autowired
    private MovieRepository movieRepository;
    @Autowired
    private ActedInRelationShipRepository actedInRelationShipRepository;
    @Autowired
    private DirectedRelationShipRepository directedRelationShipRepository;

    /**
     * 添加演员节点
     */
    @Test
    public void t1() {
        PeopleGraph peopleGraph1 = new PeopleGraph();
        peopleGraph1.setName("Keanu Reeves");
        peopleGraph1.setBorn(1964);

        PeopleGraph peopleGraph2 = new PeopleGraph();
        peopleGraph2.setName("Robert Zemeckis");
        peopleGraph2.setBorn(1951);

        PeopleGraph peopleGraph3 = new PeopleGraph();
        peopleGraph3.setName("Tom Hanks");
        peopleGraph3.setBorn(1956);

        PeopleGraph peopleGraph4 = new PeopleGraph();
        peopleGraph4.setName("Jerry");
        peopleGraph4.setBorn(1986);

        List<PeopleGraph> peopleGraphList = new ArrayList<>();
        peopleGraphList.add(peopleGraph1);
        peopleGraphList.add(peopleGraph2);
        peopleGraphList.add(peopleGraph3);
        peopleGraphList.add(peopleGraph4);

        Iterable<PeopleGraph> peopleGraphs = peopleRepository.saveAll(peopleGraphList);
        Assert.assertNotNull(peopleGraphs);
    }

    /**
     * 添加电影节点
     */
    @Test
    public void t2() {
        MovieGraph movieGraph1 = new MovieGraph();
        movieGraph1.setTitle("The Matrix");
        movieGraph1.setReleased(1997);

        MovieGraph movieGraph2 = new MovieGraph();
        movieGraph2.setTitle("Cloud Atlas");
        movieGraph2.setReleased(2012);

        MovieGraph movieGraph3 = new MovieGraph();
        movieGraph3.setTitle("Forrest Gump");
        movieGraph3.setReleased(1994);

        List<MovieGraph> movieGraphs = new ArrayList<>();
        movieGraphs.add(movieGraph1);
        movieGraphs.add(movieGraph2);
        movieGraphs.add(movieGraph3);

        Iterable<MovieGraph> movieGraphIterable = movieRepository.saveAll(movieGraphs);
        Assert.assertNotNull(movieGraphIterable);
    }

    /**
     * 添加关系
     */
    @Test
    public void t3() {

        // Tom Hanks在电影Forrest Gump中出演Forrest角色
        PeopleGraph tom_hanks = peopleRepository.findByName("Tom Hanks");
        MovieGraph forrest_gump = movieRepository.findByTitle("Forrest Gump");

        ActedInRelationShip tom_ActedIn_relationShip_forrest = new ActedInRelationShip();
        tom_ActedIn_relationShip_forrest.setStartNode(tom_hanks);
        tom_ActedIn_relationShip_forrest.setEndNode(forrest_gump);
        tom_ActedIn_relationShip_forrest.setRoles("Forrest");

        ActedInRelationShip save = actedInRelationShipRepository.save(tom_ActedIn_relationShip_forrest);
        Assert.assertNotNull(save);

        // Tom Hanks在电影Forrest Gump中出演Forrest角色
        MovieGraph cloud = movieRepository.findByTitle("Cloud Atlas");

        ActedInRelationShip tom_ActedIn_relationShip_Cloud = new ActedInRelationShip();
        tom_ActedIn_relationShip_Cloud.setStartNode(tom_hanks);
        tom_ActedIn_relationShip_Cloud.setEndNode(cloud);
        tom_ActedIn_relationShip_Cloud.setRoles("Zachry");
        ActedInRelationShip save1 = actedInRelationShipRepository.save(tom_ActedIn_relationShip_Cloud);
        Assert.assertNotNull(save1);

        // Robert Zemeckis指导电影Forrest Gump
        PeopleGraph robert_zemeckis = peopleRepository.findByName("Robert Zemeckis");
        DirectedRelationShip directedRelationShip = new DirectedRelationShip();
        directedRelationShip.setStartNode(robert_zemeckis);
        directedRelationShip.setEndNode(forrest_gump);
        DirectedRelationShip save2 = directedRelationShipRepository.save(directedRelationShip);
        Assert.assertNotNull(save2);

        // jerry 在电影 The Matrix 出演 J 角色
        PeopleGraph jerry = peopleRepository.findByName("Jerry");
        MovieGraph the_matrix = movieRepository.findByTitle("The Matrix");
        ActedInRelationShip jerry_ActedIn_relationShip_matrix = new ActedInRelationShip();
        jerry_ActedIn_relationShip_matrix.setStartNode(jerry);
        jerry_ActedIn_relationShip_matrix.setEndNode(the_matrix);
        jerry_ActedIn_relationShip_matrix.setRoles("J");
        actedInRelationShipRepository.save(jerry_ActedIn_relationShip_matrix);
    }
}
```

#### 附报错信息

报错：检查下配置文件 application.yml 是不是有多余的空格等格式问题

```properties
org.springframework.transaction.CannotCreateTransactionException: Could not open Neo4j Session for transaction; nested exception is org.neo4j.driver.v1.exceptions.AuthenticationException: Unsupported authentication token, scheme 'none' is only allowed when auth is disabled.
```



### 测试版本：2.3.12.RELEASE

如果不使用 `spring-boot-starter-data-neo4j` 依赖的话，需要手动配置 session 等信息。

配置文件设置为非内嵌模式

```yaml
server:
  port: 8091
spring:
  data:
    neo4j:
      uri: bolt://172.16.127.50:7687
      username: neo4j
      password: 123456
      embedded:
        enabled: false
```

增加 sessionFactory等配置，增加配置类Neo4jConfig。

```java
@Configuration
public class Neo4jConfig {
    @Value("${spring.data.neo4j.uri}")
    private String uri;
    @Value("${spring.data.neo4j.username}")
    private String userName;
    @Value("${spring.data.neo4j.password}")
    private String password;

    @Bean
    public org.neo4j.ogm.config.Configuration getConfiguration() {
        org.neo4j.ogm.config.Configuration configuration =
                new org.neo4j.ogm.config.Configuration.Builder()
                        .uri(uri)
                        .credentials(userName, password)
                        .build();
        return configuration;
    }
		
  	//ModelConstant.NODE_PACKAGE_PATH 节点实体所在的包路径
    @Bean
    public SessionFactory sessionFactory() {
        return new SessionFactory(getConfiguration(), ModelConstant.NODE_PACKAGE_PATH);
    }

    @Bean("transactionManager")
    public Neo4jTransactionManager neo4jTransactionManager(SessionFactory sessionFactory) {
        return new Neo4jTransactionManager(sessionFactory);
    }
}
```

其他使用的话，就一样了。

!> 存放节点实体的包尽量不要放其他实体，比如枚举或者其他常量类等，就只存放节点的实体。不然加载的时候会报出一些奇怪的错误。



## 压测

从网上摘抄

3台节点， Intel(R) Xeon(R) Silver 4114 CPU @ 2.20GHz 10 核 20 线程

| 数据量\读取速度 | 无索引无预热      | 无索引有预热     | 有索引无预热    | 有索引有预热   |
| :-------------- | :---------------- | :--------------- | :-------------- | :------------- |
| 1百万           | 531.839375‬ms/条   | 524.1652 ms/条   | 1.3464705ms/条  | 1.24ms/条      |
| 1千万           | 4,187.741666ms/条 | 5,580.71333ms/条 | 0.9362ms/条     | 0.788793ms/条  |
| 1亿             | 77926 ms/条       | 70924 ms/条      | 0.75012ms/条    | 0.6263ms/条    |
| 10亿            | 超过7分钟/条      | 超过7分钟/条     | 333.854736ms/条 | 1.4726315ms/条 |
| 数据量\写入速度 |                   |                  |                 |                |
| 1百万           | 0.88625ms/条      | 0.5988ms/条      | 0.928529ms/条   | 0.926ms/条     |
| 1千万           | 1.3516ms/条       | 0.64666ms/条     | 0.92431ms/条    | 0.95741ms/条   |
| 1亿             | -                 | -                | 1.9227710ms/条  | 1.86063ms/条   |
| 10亿            | -                 | -                | 1.1052631ms/条  | 0.64631ms/条   |
