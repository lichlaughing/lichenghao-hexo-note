---
title: Google guava 字符串
categories: guava
tags:
  - guava
  - google
keywords: guava
abbrlink: 5a83beeb
date: 2022-02-07 13:51:00
---
## Joiner（连接器）

用指定的分隔符把字符串连接起来。

// 抛弃空值，用；把字符串拼接起来

```java
@Test
public void t1() {
  Joiner joiner = Joiner.on(";").skipNulls();
  String join = joiner.join("Harry", null, "Ron", "Hermione");
  log.info("拼接字符串结果：{}", join);
}
// [main] INFO  cn.lichenghao.GuavaStringTest : 拼接字符串结果：Harry;Ron;Hermione
```

// 空值用&&代替，用；把字符串拼接起来

```java
@Test
public void t2() {
  Joiner joiner = Joiner.on(";").useForNull("&&");
  String join = joiner.join("Harry", null, "Ron", "Hermione");
  log.info("拼接字符串结果：{}", join);
}
// [main] INFO  cn.lichenghao.GuavaStringTest : 拼接字符串结果：Harry;Ron;Hermione
```

// 拼接集合或者 Map

```java
@Test
public void t3() {
  // 空值用&&代替，用；把字符串拼接起来
  Joiner joiner = Joiner.on(";").useForNull("&&");
  String join1 = joiner.join(Ints.asList(1, 2, 3, 4));
  log.info("拼接集合为字符串结果：{}", join1);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 拼接集合为字符串结果：1;2;3;4

  joiner = Joiner.on(",").useForNull("&&");
  String join2 = joiner.withKeyValueSeparator("=").join(ImmutableMap.of("姓名", "李白", "年龄", "20"));
  log.info("拼接MAP为字符串结果：{}", join2);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 拼接MAP为字符串结果：姓名=李白,年龄=20
}
```

// 拼接对象数组

```java
/**
* 拼接对象数组
*/
@Test
public void t4() {
  class Student {
    private final int number;

    public Student(int number) {
      this.number = number;
    }

    @Override
    public String toString() {
      return "Student{" +
        "number=" + number +
        '}';
    }
  }
  Student[] arr = {new Student(1), new Student(2)};
  Joiner joiner = Joiner.on(",").useForNull("&&");
  String join = joiner.join(arr);
  log.info("拼接对象数组为字符串结果：{}", join);
}
// [main] INFO  cn.lichenghao.GuavaStringTest : 拼接对象数组为字符串结果：Student{number=1},Student{number=2}
```



## Splitter（分割器）

// 常用的拆分

```java
@Test
public void t5() {
  // 指定字符拆分
  // Iterable<String> split = Splitter.on(',').split("a,b,c");
  List<String> split = Splitter.on(',').splitToList("a,b,c");
  log.info("用' ，'拆分字符串结果：{}", split);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 用'，'拆分字符串结果：[a, b, c]

  List<String> split2 = Splitter.on("@@").splitToList("a@@b@@c");
  log.info("用' @@ '拆分字符串结果：{}", split2);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 用' @@ '拆分字符串结果：[a, b, c]

  List<String> list = Splitter.on(',').trimResults().omitEmptyStrings().splitToList(" a, , b, ");
  log.info("拆分字符串忽略空白字符结果：{},list size:{}", list, list.size());
  // [main] INFO  cn.lichenghao.GuavaStringTest : 拆分字符串忽略空白字符结果：[a, b],list size:2

  Map<String, String> map = Splitter.on(";").withKeyValueSeparator("=").split("a=1;b=2");
  log.info("拆分字符串结果转 Map：{}", map);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 拆分字符串结果转 Map：{a=1, b=2}
}
```

// 进一步的拆分

```java
@Test
public void t6() {
  // 匹配空白字符
  List<String> list = Splitter.on(CharMatcher.whitespace()).splitToList("a b c");
  log.info("用' CharMatcher.whitespace() '拆分字符串结果：{}", list);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 用' CharMatcher.whitespace() '拆分字符串结果：[a, b, c]

  // 不匹配任何字符，直接把整个字符串放进集合了
  List<String> list2 = Splitter.on(CharMatcher.none()).splitToList("a b c");
  log.info("用' CharMatcher.none() '拆分字符串结果：{}", list2);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 用' CharMatcher.none() '拆分字符串结果：[a b c]

  // 利用正则按照数字拆分
  List<String> list3 = Splitter.onPattern("\\d").splitToList("abc8cba");
  log.info("用'正则'拆分字符串结果：{}", list3);
  // [main] INFO  cn.lichenghao.GuavaStringTest : 用'正则'拆分字符串结果：[abc, cba]
}
```



## CharMatcher（匹配器）

// 匹配字符串

```java
@Test
public void t101() {
  log.info("字符串是否全是 a：{}", CharMatcher.is('a').matchesAllOf("aaa"));
  // [main] INFO  cn.lichenghao.GuavaStringTest : 字符串是否全是 a：true

  log.info("字符串是否有一个是 a：{}", CharMatcher.is('a').matchesAnyOf("aaab"));
  // [main] INFO  cn.lichenghao.GuavaStringTest : 字符串是否有一个是 a：true

  log.info("字符串是不是没有 a：{}", CharMatcher.is('a').matchesNoneOf("b"));
  // [main] INFO  cn.lichenghao.GuavaStringTest : 字符串是不是没有 a：true
}
```

// 计算

```java
/**
     * 计算
     */
@Test
public void t102() {
  log.info("a 在字符串中出现的次数：{}", CharMatcher.is('a').countIn("aaab"));
  log.info("a 在字符串中第一次出现的位置：{}", CharMatcher.is('a').indexIn("aaab"));
}
```

// 操作字符串

```java
/**
     * 操作字符串
     */
@Test
public void t103() {
  log.info("在字符串中把 a 去掉：{}", CharMatcher.is('a').removeFrom("ababac"));
  log.info("在字符串中把 a 以外的去掉：{}", CharMatcher.is('a').retainFrom("ababac"));
  log.info("在字符串中把开头和结尾的 好的 去掉：{}", CharMatcher.anyOf("好的").trimFrom("好的啊a0babacab好的"));
}
```

子类众多

```properties
CharMatcher (com.google.common.base)
    BreakingWhitespace in CharMatcher (com.google.common.base)
    JavaLetterOrDigit in CharMatcher (com.google.common.base)
    JavaDigit in CharMatcher (com.google.common.base)
    And in CharMatcher (com.google.common.base)
    Negated in CharMatcher (com.google.common.base)
        NegatedFastMatcher in CharMatcher (com.google.common.base)
            Anonymous in CharMatcher (com.google.common.base)
    RangesMatcher in CharMatcher (com.google.common.base)
        Invisible in CharMatcher (com.google.common.base)
        SingleWidth in CharMatcher (com.google.common.base)
        Digit in CharMatcher (com.google.common.base)
    ForPredicate in CharMatcher (com.google.common.base)
    AnyOf in CharMatcher (com.google.common.base)
    JavaLowerCase in CharMatcher (com.google.common.base)
    Or in CharMatcher (com.google.common.base)
    JavaUpperCase in CharMatcher (com.google.common.base)
    JavaLetter in CharMatcher (com.google.common.base)
    FastMatcher in CharMatcher (com.google.common.base)
        NamedFastMatcher in CharMatcher (com.google.common.base)
            Ascii in CharMatcher (com.google.common.base)
            Any in CharMatcher (com.google.common.base)
            JavaIsoControl in CharMatcher (com.google.common.base)
            SmallCharMatcher (com.google.common.base)
            None in CharMatcher (com.google.common.base)
            BitSetMatcher in CharMatcher (com.google.common.base)
            Whitespace in CharMatcher (com.google.common.base)
        IsEither in CharMatcher (com.google.common.base)
        IsNot in CharMatcher (com.google.common.base)
        Is in CharMatcher (com.google.common.base)
        InRange in CharMatcher (com.google.common.base)

```

// 子类的匹配 ，取出空白

```java
 /**
     * 子类
     */
@Test
public void t104() {
  log.info("在字符串中把空白替换成*：{}", CharMatcher.whitespace().collapseFrom("a    b c", '*'));
  log.info("在字符串去掉前后空白后把空白替换成*：{}", CharMatcher.whitespace().trimAndCollapseFrom("   a    b c ", '*'));
}
```

