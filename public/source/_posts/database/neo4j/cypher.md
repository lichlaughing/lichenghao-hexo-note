---
title: Neo4j-Cypher查询语句
categories: neo4j
tags:
  - neo4j
keywords: neo4j
abbrlink: 156d3e4e
date: 2022-02-20 12:30:00
---
## 准备测试数据

neo4j自带批量导入的工具，支持通过 neo4j browser 的 load csv 方式以及后台的neo4j-admin import 命令。

这里使用 neo4j-admin import命令，数据可以从开源知识图谱  http://www.openkg.cn/ 中下载，文件类型为 csv文件，比如下载鱼🐟的相关知识。（http://www.openkg.cn/dataset/ocean）

将文件上传到 neo4j 的 import 目录下。

```bash
[root@bogon neo4j]# cd import
[root@bogon import]# ll
总用量 15740
-rwxrwxrwx. 1 root root 14055783 5月  fish.csv
-rwxrwxrwx. 1 root root  1907127 5月  fishplacerelationship.csv
-rwxrwxrwx. 1 root root   149803 5月  place.csv
```

然后执行导入命令。指定数据库名称以及文件的名称即可。

```bash
[root@bogon neo4j]# bin/neo4j-admin import --mode=csv --database=fish.db --nodes /opt/neo4j/import/fish.csv --nodes /opt/neo4j/import/place.csv --relationships /opt/neo4j/import/fishplacerelationship.csv
```

如下结果表示成功

```bash
......
IMPORT DONE in 11s 327ms. 
Imported:
  37449 nodes
  114863 relationships
  572742 properties
Peak memory usage: 740.76 MB
[root@bogon neo4j]# 
```

然后切换配置文件中的数据库信息，修改为上面导入命令中的数据库。

```properties
......
# The name of the database to mount
# dbms.active_database=graph.db
dbms.active_database=fish.db

......
```

重启 neo4j 即可

```bash
[root@bogon neo4j]# bin/neo4j restart
```

进入浏览器查看下

![](https://blog.lichenghao.cn/upload/2022/07/27093056.png)



## 查询所有节点

```properties
match (n) return n
```

## 根据标签查询节点

```properties
match (n:`鱼`) return n
```



## 根据 ID 查询

```properties
# 查询节点的ID是 18 的节点相关
match (n)-[]->(m) where id(n)=18 return *
```

![](https://blog.lichenghao.cn/upload/2022/07/27105144.png)



## 根据节点的属性查询

```properties
# 根据名称查找节点
match (n{name:"眼带石斑鱼"}) return n

```

```properties
# 根据属性查询，返回多个节点
match (n{bio_class_en: "Actinopterygii"}) return n limit 10
```

![](https://blog.lichenghao.cn/upload/2022/07/27110044.png)



## 模糊检索

~ 表示模糊查询    .* 相当于SQL中的 %

```properties
# 根据名称模糊检索
match (n:`鱼`) where n.name=~".*岩石.*" return n

# 前缀匹配检索
match (n:`鱼`) where n.name=~"布氏.*" return n

# 后缀匹配检索
match (n:`鱼`) where n.name=~".*灯鱼" return n

# 一个属性的多个模糊检索
match (n:`鱼`) where n.name=~".*布氏.*|.*大西洋.*" return n
```



## 结果排序

```properties
# 根据属性排序
match (m:`鱼`) return m.fish_name_en order by m.fish_name_en limit 5

╒════════════════════════╕
│"m.fish_name_en"        │
╞════════════════════════╡
│"Aaptosyax grypus"      │
├────────────────────────┤
│"Abactochromis labrosus"│
├────────────────────────┤
│"Abalistes filamentosus"│
├────────────────────────┤
│"Abalistes stellaris"   │
├────────────────────────┤
│"Abalistes stellatus"   │
└────────────────────────┘

# 倒序
match (m:`鱼`) return m.fish_name_en order by m.fish_name_en desc limit 5

╒════════════════════════════╕
│"m.fish_name_en"            │
╞════════════════════════════╡
│"Zungaropsis multimaculatus"│
├────────────────────────────┤
│"Zungaro zungaro"           │
├────────────────────────────┤
│"Zungaro jahu"              │
├────────────────────────────┤
│"Zu elongatus"              │
├────────────────────────────┤
│"Zu cristatus"              │
└────────────────────────────┘

# 多个属性排序
match (m:`鱼`) return m.fish_name_en order by m.fish_name_en,m.bio_class_en limit 5
```

!>如果结果有 null 值的话，在升序情况下 null 始终在末尾，在降序情况下null 始终在开头！



## 分页查询

和其他工具分页查询差不多

```properties
# 第一页，每页显示 10 条
match (m:`鱼`) return m.fish_name_en order by m.fish_name_en limit 10

╒═════════════════════════╕
│"m.fish_name_en"         │
╞═════════════════════════╡
│"Aaptosyax grypus"       │
├─────────────────────────┤
│"Abactochromis labrosus" │
├─────────────────────────┤
│"Abalistes filamentosus" │
├─────────────────────────┤
│"Abalistes stellaris"    │
├─────────────────────────┤
│"Abalistes stellatus"    │
├─────────────────────────┤
│"Abbottina binhi"        │
├─────────────────────────┤
│"Abbottina lalinensis"   │
├─────────────────────────┤
│"Abbottina liaoningensis"│
├─────────────────────────┤
│"Abbottina obtusirostris"│
├─────────────────────────┤
│"Abbottina rivularis"    │
└─────────────────────────┘

# 第二页，每页显示 5 条，可以和上面的对比下结果
match (m:`鱼`) return m.fish_name_en order by m.fish_name_en skip 5 limit 5
╒═════════════════════════╕
│"m.fish_name_en"         │
╞═════════════════════════╡
│"Abbottina binhi"        │
├─────────────────────────┤
│"Abbottina lalinensis"   │
├─────────────────────────┤
│"Abbottina liaoningensis"│
├─────────────────────────┤
│"Abbottina obtusirostris"│
├─────────────────────────┤
│"Abbottina rivularis"    │
└─────────────────────────┘
```

