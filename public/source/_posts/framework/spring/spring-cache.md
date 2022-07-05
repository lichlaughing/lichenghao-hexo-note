---
title: spring-cache
categories: spring
tags:
  - spring
  - cache
abbrlink: 6378ef32
date: 2022-02-02 12:00:00
---
从3.1开始，Spring引入了对Cache的支持,作用在方法上，将方法的参数和返回结果缓存起来。当我们第一次调用方法的时候，方法会执行，当我们第二次拿着同样的参数去调用方法的时候，就会从缓存中拿到结果，不需要再执行方法了。
springCache是抽象的org.springframework.cache.Cache,
org.springframework.cache.CacheManage
实现具体还需要具体的缓存实现来完成。cacheManager有很多的实现，根据不同的场景。

| CacheManger               | 描述                                                   |
| :------------------------ | :----------------------------------------------------- |
| SimpleCacheManager        | 使用简单的Collection来存储缓存，主要用于测试           |
| ConcurrentMapCacheManager | 使用ConcurrentMap作为缓存技术（默认）                  |
| NoOpCacheManager          | 测试用                                                 |
| EhCacheCacheManager       | 使用EhCache作为缓存技术，以前在hibernate的时候经常用   |
| GuavaCacheManager         | 使用google guava的GuavaCache作为缓存技术               |
| HazelcastCacheManager     | 使用Hazelcast作为缓存技术                              |
| JCacheCacheManager        | 使用JCache标准的实现作为缓存技术，如Apache Commons JCS |
| RedisCacheManager         | 使用Redis作为缓存技术                                  |

这里我们使用Ehcache来实现缓存功能。



springboot 版本 2.x



## 添加依赖

```java
<!-- Spring Cache -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
</dependency>
    
<dependency>
   <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
</dependency>
```

## 编写ehcache配置文件

ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <!--timeToIdleSeconds 当缓存闲置n秒后销毁 -->
    <!--timeToLiveSeconds 当缓存存活n秒后销毁 -->
    <!-- 缓存配置
        name:缓存名称。
        maxElementsInMemory：缓存最大个数。
        maxElementsOnDisk：硬盘最大缓存个数。
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。
        timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
        timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
        overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
        diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
        memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。
                默认策略是 LRU（Least Recently Used:最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（Less Frequently Used:较少使用）。
        clearOnFlush：内存数量最大时是否清除。
    -->
    <!-- 磁盘缓存位置
      diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
      user.home – 用户主目录
      user.dir  – 用户当前工作目录
      java.io.tmpdir – 默认临时文件路径
    -->
    <diskStore path="java.io.tmpdir"/>
    <!-- 默认缓存 -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
    </defaultCache>
    <!-- OrgCache -->
    <cache name="OrgCache"
           eternal="false"
           timeToIdleSeconds="1200"
           timeToLiveSeconds="1200"
           maxEntriesLocalHeap="10000"
           maxEntriesLocalDisk="10000000"
           diskExpiryThreadIntervalSeconds="120"
           overflowToDisk="false"
           memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>
```

ehcache.xsd 这个不是编写，官网上应该有的。

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" elementFormDefault="qualified" version="1.7">
    <xs:element name="ehcache">
        <xs:complexType>
            <xs:sequence>
                <xs:element maxOccurs="1" minOccurs="0" ref="diskStore"/>
                <xs:element maxOccurs="1" minOccurs="0" ref="sizeOfPolicy"/>
                <xs:element maxOccurs="1" minOccurs="0" ref="transactionManagerLookup"/>
                <xs:element maxOccurs="1" minOccurs="0" ref="cacheManagerEventListenerFactory"/>
                <xs:element maxOccurs="1" minOccurs="0" ref="managementRESTService"/>
                <xs:element maxOccurs="unbounded" minOccurs="0" ref="cacheManagerPeerProviderFactory"/>
                <xs:element maxOccurs="unbounded" minOccurs="0" ref="cacheManagerPeerListenerFactory"/>
                <xs:element maxOccurs="1" minOccurs="0" ref="terracottaConfig"/>
                <xs:element maxOccurs="1" minOccurs="0" ref="defaultCache"/>
                <xs:element maxOccurs="unbounded" minOccurs="0" ref="cache"/>
            </xs:sequence>
            <xs:attribute name="name" use="optional"/>
            <xs:attribute default="true" name="updateCheck" type="xs:boolean" use="optional"/>
            <xs:attribute default="autodetect" name="monitoring" type="monitoringType" use="optional"/>
            <xs:attribute default="true" name="dynamicConfig" type="xs:boolean" use="optional"/>
            <xs:attribute default="15" name="defaultTransactionTimeoutInSeconds" type="xs:integer" use="optional"/>
            <xs:attribute default="0" name="maxBytesLocalHeap" type="memoryUnitOrPercentage" use="optional"/>
            <xs:attribute default="0" name="maxBytesLocalOffHeap" type="memoryUnit" use="optional"/>
            <xs:attribute default="0" name="maxBytesLocalDisk" type="memoryUnit" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="managementRESTService">
        <xs:complexType>
            <xs:attribute name="enabled" type="xs:boolean" use="optional"/>
            <xs:attribute name="bind" use="optional"/>
            <xs:attribute name="securityServiceLocation" use="optional"/>
            <xs:attribute name="securityServiceTimeout" use="optional" type="xs:integer"/>
            <xs:attribute name="sslEnabled" use="optional" type="xs:boolean"/>
            <xs:attribute name="needClientAuth" use="optional" type="xs:boolean"/>
            <xs:attribute name="sampleHistorySize" use="optional" type="xs:integer"/>
            <xs:attribute name="sampleIntervalSeconds" use="optional" type="xs:integer"/>
            <xs:attribute name="sampleSearchIntervalSeconds" use="optional" type="xs:integer"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="diskStore">
        <xs:complexType>
            <xs:attribute name="path" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="transactionManagerLookup">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheManagerEventListenerFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheManagerPeerProviderFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheManagerPeerListenerFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="terracottaConfig">
        <xs:complexType>
            <xs:sequence>
                <xs:element maxOccurs="1" minOccurs="0" name="tc-config">
                    <xs:complexType>
                        <xs:sequence>
                            <xs:any maxOccurs="unbounded" minOccurs="0" processContents="skip"/>
                        </xs:sequence>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
            <xs:attribute default="localhost:9510" name="url" use="optional"/>
            <xs:attribute name="rejoin" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="wanEnabledTSA" type="xs:boolean" use="optional" default="false"/>
        </xs:complexType>
    </xs:element>
    <!--
    add clone support for addition of cacheExceptionHandler. Important!
   -->
    <xs:element name="defaultCache">
        <xs:complexType>
            <xs:sequence>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheEventListenerFactory"/>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheExtensionFactory"/>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheLoaderFactory"/>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheDecoratorFactory"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="bootstrapCacheLoaderFactory"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="cacheExceptionHandlerFactory"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="pinning"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="terracotta"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="cacheWriter"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="copyStrategy"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="elementValueComparator"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="sizeOfPolicy"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="persistence"/>
            </xs:sequence>
            <xs:attribute name="diskExpiryThreadIntervalSeconds" type="xs:integer" use="optional"/>
            <xs:attribute name="diskSpoolBufferSizeMB" type="xs:integer" use="optional"/>
            <xs:attribute name="diskPersistent" type="xs:boolean" use="optional"/>
            <xs:attribute name="diskAccessStripes" type="xs:integer" use="optional" default="1"/>
            <xs:attribute name="eternal" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="maxElementsInMemory" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxEntriesLocalHeap" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="clearOnFlush" type="xs:boolean" use="optional"/>
            <xs:attribute name="memoryStoreEvictionPolicy" type="xs:string" use="optional"/>
            <xs:attribute name="overflowToDisk" type="xs:boolean" use="optional"/>
            <xs:attribute name="timeToIdleSeconds" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="timeToLiveSeconds" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxElementsOnDisk" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxEntriesLocalDisk" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="transactionalMode" type="transactionalMode" use="optional" default="off"/>
            <xs:attribute name="statistics" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="copyOnRead" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="copyOnWrite" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="cacheLoaderTimeoutMillis" type="xs:integer" use="optional" default="0"/>
            <xs:attribute name="overflowToOffHeap" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="maxMemoryOffHeap" type="xs:string" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cache">
        <xs:complexType>
            <xs:sequence>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheEventListenerFactory"/>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheExtensionFactory"/>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheLoaderFactory"/>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="cacheDecoratorFactory"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="bootstrapCacheLoaderFactory"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="cacheExceptionHandlerFactory"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="pinning"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="terracotta"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="cacheWriter"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="copyStrategy"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="searchable"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="elementValueComparator"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="sizeOfPolicy"/>
                <xs:element minOccurs="0" maxOccurs="1" ref="persistence"/>
            </xs:sequence>
            <xs:attribute name="diskExpiryThreadIntervalSeconds" type="xs:integer" use="optional"/>
            <xs:attribute name="diskSpoolBufferSizeMB" type="xs:integer" use="optional"/>
            <xs:attribute name="diskPersistent" type="xs:boolean" use="optional"/>
            <xs:attribute name="diskAccessStripes" type="xs:integer" use="optional" default="1"/>
            <xs:attribute name="eternal" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="maxElementsInMemory" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxEntriesLocalHeap" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="memoryStoreEvictionPolicy" type="xs:string" use="optional"/>
            <xs:attribute name="clearOnFlush" type="xs:boolean" use="optional"/>
            <xs:attribute name="name" type="xs:string" use="required"/>
            <xs:attribute name="overflowToDisk" type="xs:boolean" use="optional"/>
            <xs:attribute name="timeToIdleSeconds" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="timeToLiveSeconds" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxElementsOnDisk" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxEntriesLocalDisk" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="maxEntriesInCache" type="xs:nonNegativeInteger" use="optional"/>
            <xs:attribute name="transactionalMode" type="transactionalMode" use="optional" default="off"/>
            <xs:attribute name="statistics" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="copyOnRead" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="copyOnWrite" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="logging" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="cacheLoaderTimeoutMillis" type="xs:integer" use="optional" default="0"/>
            <xs:attribute name="overflowToOffHeap" type="xs:boolean" use="optional" default="false"/>
            <xs:attribute name="maxMemoryOffHeap" type="xs:string" use="optional"/>
            <xs:attribute default="0" name="maxBytesLocalHeap" type="memoryUnitOrPercentage" use="optional"/>
            <xs:attribute default="0" name="maxBytesLocalOffHeap" type="memoryUnitOrPercentage" use="optional"/>
            <xs:attribute default="0" name="maxBytesLocalDisk" type="memoryUnitOrPercentage" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheEventListenerFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
            <xs:attribute name="listenFor" use="optional" type="notificationScope" default="all"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="bootstrapCacheLoaderFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheExtensionFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheExceptionHandlerFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheLoaderFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="cacheDecoratorFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="searchAttribute">
        <xs:complexType>
            <xs:attribute name="name" use="required" type="xs:string"/>
            <xs:attribute name="expression" type="xs:string"/>
            <xs:attribute name="class" type="xs:string"/>
            <xs:attribute name="type" type="xs:string" use="optional"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="searchable">
        <xs:complexType>
            <xs:sequence>
                <xs:element minOccurs="0" maxOccurs="unbounded" ref="searchAttribute"/>
            </xs:sequence>
            <xs:attribute name="keys" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="values" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="allowDynamicIndexing" use="optional" type="xs:boolean" default="false"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="pinning">
        <xs:complexType>
            <xs:attribute name="store" use="required" type="pinningStoreType"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="terracotta">
        <xs:complexType>
            <xs:sequence>
                <xs:element minOccurs="0" maxOccurs="1" ref="nonstop"/>
            </xs:sequence>
            <xs:attribute name="clustered" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="coherentReads" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="localKeyCache" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="localKeyCacheSize" use="optional" type="xs:positiveInteger" default="300000"/>
            <xs:attribute name="orphanEviction" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="orphanEvictionPeriod" use="optional" type="xs:positiveInteger" default="4"/>
            <xs:attribute name="copyOnRead" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="coherent" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="consistency" use="optional" type="consistencyType" default="eventual"/>
            <xs:attribute name="synchronousWrites" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="concurrency" use="optional" type="xs:nonNegativeInteger" default="0"/>
            <xs:attribute name="localCacheEnabled" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="compressionEnabled" use="optional" type="xs:boolean" default="false"/>
        </xs:complexType>
    </xs:element>
    <xs:simpleType name="consistencyType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="strong"/>
            <xs:enumeration value="eventual"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:element name="nonstop">
        <xs:complexType>
            <xs:sequence>
                <xs:element minOccurs="0" maxOccurs="1" ref="timeoutBehavior"/>
            </xs:sequence>
            <xs:attribute name="enabled" use="optional" type="xs:boolean" default="true"/>
            <xs:attribute name="immediateTimeout" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="timeoutMillis" use="optional" type="xs:positiveInteger" default="30000"/>
            <xs:attribute name="searchTimeoutMillis" use="optional" type="xs:positiveInteger" default="30000"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="timeoutBehavior">
        <xs:complexType>
            <xs:attribute name="type" use="optional" type="timeoutBehaviorType" default="exception"/>
            <xs:attribute name="properties" use="optional" default=""/>
            <xs:attribute name="propertySeparator" use="optional" default=","/>
        </xs:complexType>
    </xs:element>
    <xs:simpleType name="timeoutBehaviorType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="noop"/>
            <xs:enumeration value="exception"/>
            <xs:enumeration value="localReads"/>
            <xs:enumeration value="localReadsAndExceptionOnWrite"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="monitoringType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="autodetect"/>
            <xs:enumeration value="on"/>
            <xs:enumeration value="off"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="pinningStoreType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="localMemory"/>
            <xs:enumeration value="inCache"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="terracottaCacheValueType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="serialization"/>
            <xs:enumeration value="identity"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="transactionalMode">
        <xs:restriction base="xs:string">
            <xs:enumeration value="off"/>
            <xs:enumeration value="xa_strict"/>
            <xs:enumeration value="xa"/>
            <xs:enumeration value="local"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:element name="cacheWriter">
        <xs:complexType>
            <xs:sequence>
                <xs:element minOccurs="0" maxOccurs="1" ref="cacheWriterFactory"/>
            </xs:sequence>
            <xs:attribute name="writeMode" use="optional" type="writeModeType" default="write-through"/>
            <xs:attribute name="notifyListenersOnException" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="minWriteDelay" use="optional" type="xs:nonNegativeInteger" default="1"/>
            <xs:attribute name="maxWriteDelay" use="optional" type="xs:nonNegativeInteger" default="1"/>
            <xs:attribute name="rateLimitPerSecond" use="optional" type="xs:nonNegativeInteger" default="0"/>
            <xs:attribute name="writeCoalescing" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="writeBatching" use="optional" type="xs:boolean" default="false"/>
            <xs:attribute name="writeBatchSize" use="optional" type="xs:positiveInteger" default="1"/>
            <xs:attribute name="retryAttempts" use="optional" type="xs:nonNegativeInteger" default="0"/>
            <xs:attribute name="retryAttemptDelaySeconds" use="optional" type="xs:nonNegativeInteger" default="1"/>
            <xs:attribute name="writeBehindConcurrency" use="optional" type="xs:nonNegativeInteger" default="1"/>
            <xs:attribute name="writeBehindMaxQueueSize" use="optional" type="xs:nonNegativeInteger" default="0"/>
        </xs:complexType>
    </xs:element>
    <xs:simpleType name="writeModeType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="write-through"/>
            <xs:enumeration value="write-behind"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:element name="cacheWriterFactory">
        <xs:complexType>
            <xs:attribute name="class" use="required"/>
            <xs:attribute name="properties" use="optional"/>
            <xs:attribute name="propertySeparator" use="optional"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="copyStrategy">
        <xs:complexType>
            <xs:attribute name="class" use="required" type="xs:string"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="elementValueComparator">
        <xs:complexType>
            <xs:attribute name="class" use="required" type="xs:string"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="sizeOfPolicy">
        <xs:complexType>
            <xs:attribute name="maxDepth" use="required" type="xs:integer"/>
            <xs:attribute name="maxDepthExceededBehavior" use="optional" default="continue"
                          type="maxDepthExceededBehavior"/>
        </xs:complexType>
    </xs:element>
    <xs:element name="persistence">
        <xs:complexType>
            <xs:attribute name="strategy" use="required" type="persistenceStrategy"/>
            <xs:attribute name="synchronousWrites" use="optional" default="false" type="xs:boolean"/>
        </xs:complexType>
    </xs:element>
    <xs:simpleType name="persistenceStrategy">
        <xs:restriction base="xs:string">
            <xs:enumeration value="localTempSwap"/>
            <xs:enumeration value="localRestartable"/>
            <xs:enumeration value="none"/>
            <xs:enumeration value="distributed"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="maxDepthExceededBehavior">
        <xs:restriction base="xs:string">
            <xs:enumeration value="continue"/>
            <xs:enumeration value="abort"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="notificationScope">
        <xs:restriction base="xs:string">
            <xs:enumeration value="local"/>
            <xs:enumeration value="remote"/>
            <xs:enumeration value="all"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="memoryUnit">
        <xs:restriction base="xs:token">
            <xs:pattern value="[0-9]+[bBkKmMgG]?"/>
        </xs:restriction>
    </xs:simpleType>
    <xs:simpleType name="memoryUnitOrPercentage">
        <xs:restriction base="xs:token">
            <xs:pattern value="([0-9]+[bBkKmMgG]?|100%|[0-9]{1,2}%)"/>
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```

## 启用缓存功能

这里写个缓存的配置类，统一来管理缓存

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactoryBean() {
        EhCacheManagerFactoryBean cacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        cacheManagerFactoryBean.setConfigLocation(new ClassPathResource("ehcache.xml"));// 缓存信息配置文件，位于resource下
        cacheManagerFactoryBean.setShared(true);
        return cacheManagerFactoryBean;
    }

    @Bean
    public EhCacheCacheManager ehCacheCacheManager(EhCacheManagerFactoryBean cacheManagerFactoryBean) {
        return new EhCacheCacheManager(cacheManagerFactoryBean.getObject());
    }
}


```

## 缓存注解

有几个常用的注解：

### 1.@EnableCaching

开启缓存功能，一般放在启动类上，或者上面一样，我放在配置类上。

### 2.@CacheConfig

配置在类上，统一指定缓存的value值，这样在方法的缓存注解上就不用再指定value啦。

### 3.@Cacheable

根据方法对其返回结果进行缓存，下次请求时，如果缓存存在，则直接读取缓存数据返回；如果缓存不存在，则执行方法，并把返回的结果存入缓存中。通常用在查询方法中。

### 4.@CachePut

使用该注解标志的方法，每次都会执行，并将结果存入指定的缓存中。其他方法可以直接从响应的缓存中读取缓存数据，而不需要再去查询数据库。通常在新增方法上。

### 5.@CacheEvict

使用该注解标志的方法，会清空指定的缓存。通常用在更新或者删除方法中。

### 6.@Caching

该注解可以实现同一个方法上同时使用多种注解,其源码如下：

```java
public @interface Caching {

	Cacheable[] cacheable() default {};

	CachePut[] put() default {};

	CacheEvict[] evict() default {};

}
```



## 代码上配置缓存

首先在一个接口实现类中，声明缓存

```java
@Service
@CacheConfig(cacheNames = "OrgCache")//上面配置文件配置的缓存
public class OrgServiceImpl implements OrgService {
}
```

查询方法上配置注解缓存

```java
@Cacheable(key = "#root.targetClass+'_'+#root.methodName+'_'+#id")
@Override
public OrgEntity getById(String id) {
   return dao.selectById(id);
}
```

插入

```java
@CacheEvict(allEntries = true, value = "OrgCache")
@Override
public Serializable insert(OrgEntity obj) {
    return dao.save(obj);
}
```

更新

```java
@CacheEvict(allEntries = true, value = "OrgCache")
@Override
public void update(OrgEntity obj) {
    dao.update(obj);
}
```

删除

```java
@CacheEvict(allEntries = true, value = "OrgCache")
@Override
public boolean deleteById(String id) {
    return dao.deleteById(id);
}
```

上面增删改，我都用了@CacheEvict，一操作增删改就清除缓存，图个方便。其实应该在新增，修改的时候使用@CachePut 去处理缓存，这样查询的时候会直接去查询缓存中。

毕竟是小缓存，怎么方便怎么使用被。。。

