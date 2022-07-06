---
title: Neo4j-Cypher其他操作语句
categories: neo4j
tags:
  - neo4j
keywords: neo4j
abbrlink: df843612
date: 2022-02-20 12:30:00
---
## 操作标签

以下的 ID 是自定义的 UUID 作为主键 ID。

### 节点添加标签

```properties
# 根据 ID 给节点添加标签
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) set n:groupA return n

# 添加多个标签
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) set n:groupA:groupB:groupC return n
```

### 删除标签

```properties
# 删除一个标签
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) remove n:groupA return n

# 删除多个标签
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) remove n:groupA,n:groupB,n:groupC return n
```



## 操作自定义属性

### 添加属性

```properties
# 添加一个属性
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) set n.group="gourpA" return n

# 添加多个属性
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) set n.group="化工模型库",n.pgroup="A公司" return n
```

### 删除属性

```properties
# 删除一个属性
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) remove n.cat1 return n

# 删除多个属性
match (n {id:"EFE4EBD3FFD143DD8BD16C2A7A636016"}) remove n.cat1,n.cat2 return n
```

