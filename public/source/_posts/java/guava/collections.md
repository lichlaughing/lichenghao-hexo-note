---
title: Google guava 集合
categories: guava
tags:
  - guava
  - google
keywords: guava
abbrlink: dee584da
date: 2022-02-07 13:53:00
---
## 不可变集合

集合的数据在创建的时候提供，并且在整个生命周期中都不可改变，不可以对集合里面的数据做除了查询的操作。

- 线程安全的：immutable对象在多线程下安全，没有竞态条件。
- 不可变集合不需要考虑变化，因此可以节省时间和空间。
- 可以作为常量来安全使用。

常见的不可变集合：

| **可变集合接口**       | **JDK/Guava** | **不可变版本**              |
| :--------------------- | :------------ | :-------------------------- |
| Collection             | JDK           | ImmutableCollection         |
| List                   | JDK           | ImmutableList               |
| Set                    | JDK           | ImmutableSet                |
| SortedSet/NavigableSet | JDK           | ImmutableSortedSet          |
| Map                    | JDK           | ImmutableMap                |
| SortedMap              | JDK           | ImmutableSortedMap          |
| Multiset               | Guava         | ImmutableMultiset           |
| SortedMultiset         | Guava         | ImmutableSortedMultiset     |
| Multimap               | Guava         | ImmutableMultimap           |
| ListMultimap           | Guava         | ImmutableListMultimap       |
| SetMultimap            | Guava         | ImmutableSetMultimap        |
| BiMap                  | Guava         | ImmutableBiMap              |
| ClassToInstanceMap     | Guava         | ImmutableClassToInstanceMap |
| Table                  | Guava         | ImmutableTable              |

众多的不可变集合。

```properties
AbstractCollection (java.util)
    ImmutableCollection (com.google.common.collect)
        EntryCollection in ImmutableMultimap (com.google.common.collect)
        ImmutableSet (com.google.common.collect)
            IndexedImmutableSet (com.google.common.collect)
                KeySet in RegularImmutableMap (com.google.common.collect)
                JdkBackedImmutableSet (com.google.common.collect)
                ImmutableMapKeySet (com.google.common.collect)
                EntrySet in ImmutableMultiset (com.google.common.collect)
                CellSet in RegularImmutableTable (com.google.common.collect)
            SingletonImmutableSet (com.google.common.collect)
            ImmutableSortedSetFauxverideShim (com.google.common.collect)
                ImmutableSortedSet (com.google.common.collect)
            EntrySet in ImmutableSetMultimap (com.google.common.collect)
            ImmutableEnumSet (com.google.common.collect)
            RegularImmutableSet (com.google.common.collect)
            Indexed in ImmutableSet (com.google.common.collect)
                ElementSet in ImmutableMultiset (com.google.common.collect)
            ImmutableMapEntrySet (com.google.common.collect)
                EntrySetImpl in IteratorBasedImmutableMap in ImmutableMap (com.google.common.collect)
                EntrySet in ImmutableSortedMap (com.google.common.collect)
                RegularEntrySet in ImmutableMapEntrySet (com.google.common.collect)
                InverseEntrySet in Inverse in RegularImmutableBiMap (com.google.common.collect)
        ImmutableList (com.google.common.collect)
            RegularImmutableList (com.google.common.collect)
            Values in RegularImmutableTable (com.google.common.collect)
            StringAsImmutableList in Lists (com.google.common.collect)
            ImmutableAsList (com.google.common.collect)
                RegularImmutableAsList (com.google.common.collect)
                Anonymous in EntrySet in ImmutableSortedMap (com.google.common.collect)
                Anonymous in Indexed in ImmutableSet (com.google.common.collect)
                Anonymous in IndexedImmutableSet (com.google.common.collect)
                Anonymous in RegularContiguousSet (com.google.common.collect)
                Anonymous in InverseEntrySet in Inverse in RegularImmutableBiMap (com.google.common.collect)
                Anonymous in ImmutableMapValues (com.google.common.collect)
            ComplementRanges in ImmutableRangeSet (com.google.common.collect)
            ReverseImmutableList in ImmutableList (com.google.common.collect)
            SubList in ImmutableList (com.google.common.collect)
            Values in RegularImmutableMap (com.google.common.collect)
            SingletonImmutableList (com.google.common.collect)
            InverseEntries in JdkBackedImmutableBiMap (com.google.common.collect)
            Anonymous in ImmutableRangeSet (com.google.common.collect)
            Anonymous in ImmutableRangeMap (com.google.common.collect)
            Anonymous in CartesianList (com.google.common.collect)
            Anonymous in CartesianSet in Sets (com.google.common.collect)
        ImmutableMultisetGwtSerializationDependencies (com.google.common.collect)
            ImmutableMultiset (com.google.common.collect)
                RegularImmutableMultiset (com.google.common.collect)
                Keys in ImmutableMultimap (com.google.common.collect)
                ImmutableSortedMultisetFauxverideShim (com.google.common.collect)
                    ImmutableSortedMultiset (com.google.common.collect)
                        RegularImmutableSortedMultiset (com.google.common.collect)
                        DescendingImmutableSortedMultiset (com.google.common.collect)
                JdkBackedImmutableMultiset (com.google.common.collect)
        ImmutableMapValues (com.google.common.collect)
        Values in ImmutableMultimap (com.google.common.collect)

```



// 生成不可变集合

```java
@Test
public void t1() {
  List<Integer> list = ImmutableList.of(1, 2, 3).asList();
  // list.add(5);  java.lang.UnsupportedOperationException
  log.info("生成不可变集合：{}", list);
}
```



## 新集合类型

- Multiset

- MultiMap

  - 多个实现如下 ↓

  - | 实现                  | 键行为类似     | 值行为类似    |
    | :-------------------- | :------------- | :------------ |
    | ArrayListMultimap     | HashMap        | ArrayList     |
    | HashMultimap          | HashMap        | HashSet       |
    | LinkedListMultimap*   | LinkedHashMap* | LinkedList*   |
    | LinkedHashMultimap**  | LinkedHashMap  | LinkedHashMap |
    | TreeMultimap          | TreeMap        | TreeSet       |
    | ImmutableListMultimap | ImmutableMap   | ImmutableList |
    | ImmutableSetMultimap  | ImmutableMap   | ImmutableSet  |

- BiMap

- Table

- ClassToInstanceMap：Class作为Key, 对应实例作为Value

  - 多个实现 ↓

  - | 实现                        | 描述                                                 |
    | --------------------------- | ---------------------------------------------------- |
    | MutableClassToInstanceMap   | 可变类型的ClassToInstanceMap                         |
    | ImmutableClassToInstanceMap | 不可变更的ClassToInstanceMap，构造完成后就不可再变更 |

- RangeSet
- RangeMap

## 集合工具类



| 集合接口   | 属于JDK仍是Guava | 对应的Guava工具类                               |
| :--------- | :--------------- | :---------------------------------------------- |
| Iterable   | JDK              | Iterables                                       |
| Collection | JDK              | Collections2：不要和 java.util.Collections 混淆 |
| List       | JDK              | Lists                                           |
| Set        | JDK              | Sets                                            |
| SortedSet  | JDK              | Sets                                            |
| Map        | JDK              | Maps                                            |
| SortedMap  | JDK              | Maps                                            |
| Queue      | JDK              | Queues                                          |
| Multiset   | Guava            | Multisets                                       |
| Multimap   | Guava            | Multimaps                                       |
| BiMap      | Guava            | Maps                                            |
| Table      | Guava            | Tables                                          |
