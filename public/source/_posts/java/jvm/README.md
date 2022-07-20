---
title: JVM-Java Virtural Machine
categories: jvm
tags:
  - java
  - jvm
keywords: 'java,jvm'
abbrlink: a12f0437
date: 2022-05-15 13:30:00
cover: https://blog.lichenghao.cn/upload/2022/07/FA4B985E-jvm.png
---
[Java SE Specifications 虚拟机规范](https://docs.oracle.com/javase/specs/index.html)

Java Virtural Machine  JAVA程序的运行环境。是 JAVA跨平台的基础，并且可以自动内存管理和垃圾回收。



JVM——>JRE——>JDK——>JAVA-SE——>JAVA-EE，其关系图如下：

{% mermaid %}

graph LR

subgraph 开发JAVA-EE: JDK+应用服务器+IDE 工具
subgraph 开发JAVA-SE: JDK+IDE工具
subgraph JDK: JVM+基础类库+编译工具
subgraph JRE: JVM+基础类库
J(JVM)
end
end
end
end

A(操作系统)
J---A

style J width:100px

{% endmermaid %}



JVM 运行时数据区：

![](https://blog.lichenghao.cn/upload/2022/07/04183808-jvm.png)





