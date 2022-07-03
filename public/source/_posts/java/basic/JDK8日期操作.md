---
title: JDK8 日期操作
categories: java
tags:
  - jdk8
  - LocalDate
  - LocalDateTime
abbrlink: 1cf35d86
---
JDK1.8 之后提供了一套全新的时间日期 API 这套全新的 API 在 java.time 包下。相比于传统工具类，该工具类更加简洁，方便。

# LocalDate,LocalDateTime

获取今天的一些日期和时间

```java
// 2021-05-15
LocalDate now1 = LocalDate.now();
// ISO LocalDate使用ISO日历系统，这是公历；其他日历系统在java.time.chrono包下
IsoChronology chronology = LocalDate.now().getChronology();

// 2021
int year = LocalDate.now().getYear();

// MAY
Month month = LocalDate.now().getMonth();
// 五月
String displayName = LocalDate.now().getMonth().getDisplayName(TextStyle.FULL, new Locale(Locale.CHINESE.getLanguage()));
// 5
int monthValue = LocalDate.now().getMonthValue();

// 15
int dayOfMonth = LocalDate.now().getDayOfMonth();

// SATURDAY
DayOfWeek dayOfWeek = LocalDate.now().getDayOfWeek();
// 星期六
String displayName = LocalDate.now().getDayOfWeek().getDisplayName(TextStyle.FULL, new Locale(Locale.CHINESE.getLanguage()));
// 6
int value = LocalDate.now().getDayOfWeek().getValue();
```

```java
// 2021-05-15T17:28:33.985
LocalDateTime now2 = LocalDateTime.now();

// 1621086485055 / 2021-05-15 21:48:05  当前时间戳
long l = Instant.now().toEpochMilli();
```

# 生成日期

```java
// 2020-02-02
LocalDate of = LocalDate.of(2020, 2, 2);
// 1970-01-02 从1970-01-01 开始+N天
LocalDate localDate = LocalDate.ofEpochDay(1);
// 2020-01-02 哪年哪天
LocalDate localDate = LocalDate.ofYearDay(2020,2);
// 2020-05-15 替换当前年份
LocalDate localDate = LocalDate.now().withYear(2020);
// 2020-04-15 替换当前月份
LocalDate localDate = LocalDate.now().withMonth(4);
// 2021-05-20 替换日期
LocalDate localDate = LocalDate.now().withDayOfMonth(20);
// 2021-01-20 今年的第20天
LocalDate localDate = LocalDate.now().withDayOfYear(20);
// 2021-05-01 五月的第一天，参数为TemporalAdjuster的实现类
LocalDate localDate = LocalDate.now().with(MonthDay.of(5, 1));

// 2021-05-15T05:05:05
LocalDateTime of = LocalDateTime.of(2021, 5, 15, 5, 5, 5);
// 2021-05-15T05:05
LocalDateTime of = LocalDateTime.of(2021, 5, 15, 5, 5);
// 2021-05-15T21:19:28.210 根据时间戳
LocalDateTime of = LocalDateTime.ofInstant(Instant.now(), ZoneId.systemDefault());
// LocalDateTime.now().of XXXX
```

# Instant

表示时刻，不直接对应年月日信息，需要通过时区转换。比如跟 Date 做转换就需要这个类来辅助。

```java
// 2021-05-27T04:12:12.490Z 我们是东八区，时间比我们早8小时
Instant now = Instant.now();
```

# 日期格式化

```java
// 2021-05-15 21:23:34
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String format = dateTimeFormatter.format(LocalDateTime.now());
```

# 日期转换

## 字符串转 LocalDateTime

```java
public static void main(String[] args) {
        String str = "2020-05-01 22:22:22";
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        LocalDateTime parse = LocalDateTime.parse(str, dateTimeFormatter);
        System.out.println(parse);
    }
// 2020-05-01T22:22:22
```

## 时间戳转 LocalDateTime

```java
public static void main(String[] args) {
  		// ms
        Long now = 1621086180000L;
        LocalDateTime localDateTime = LocalDateTime.ofInstant(Instant.ofEpochMilli(now), ZoneId.systemDefault());
        System.out.println(localDateTime);
    }
// 2021-05-15T21:43
```

## LocalDate<—>Date

```java
// LocalDate—>Date Thu May 27 00:00:00 CST 2021
ZonedDateTime zonedDateTime = LocalDate.now().atStartOfDay(ZoneId.systemDefault());
Date date = Date.from(zonedDateTime.toInstant());

// Date—>LocalDate 2021-05-27
Instant instant = new Date().toInstant();
LocalDate from = LocalDate.from(instant.atZone(ZoneId.systemDefault()));
```

## LocalDateTime<—>Date

```java
// LocalDateTime—>Date Thu May 27 12:23:26 CST 2021
LocalDateTime now = LocalDateTime.now();
Instant instant = now.toInstant(ZoneOffset.ofHours(8));
Date date = Date.from(instant);

// Date—>LocalDateTime 2021-05-27T12:25:04.821
Date date = new Date();
ZonedDateTime zonedDateTime = date.toInstant().atZone(ZoneId.systemDefault());
LocalDateTime dateTime = LocalDateTime.from(zonedDateTime);
```

# 日期计算

## 多久以前(之后)

LocalDate

```java
// 2022-05-15 一年以后
LocalDate localDate = LocalDate.now().plusYears(1);
// 2021-06-15 一个月以后
LocalDate localDate = LocalDate.now().plusMonths(1);
// 2021-05-22 一周以后
LocalDate localDate = LocalDate.now().plusWeeks(1);
// 2021-05-20 五天以后
LocalDate localDate = LocalDate.now().plusDays(5);
// 2021-05-16 指定单位
LocalDate localDate = LocalDate.now().plus(1, ChronoUnit.DAYS);

// 2020-05-15 一年以前
LocalDate localDate = LocalDate.now().minusYears(1);
// 2021-04-15 一个月以前
LocalDate localDate = LocalDate.now().minusMonths(1);
// 2021-05-08 一周以前
LocalDate localDate = LocalDate.now().minusWeeks(1);
// 2021-05-14 一天以前
LocalDate localDate = LocalDate.now().minusDays(1);
// 2020-05-15 指定单位
LocalDate localDate = LocalDate.now().minus(1,ChronoUnit.YEARS);
```

LocalDateTime 操作几乎和 LocalDate 一样的，只不过多了小时，分钟，秒的操作。

## 时间间隔

```java
public static void main(String[] args) {
        // 开始时间
        LocalDateTime startDate = LocalDateTime.parse("2021-05-01 12:12:12", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        // 当前时间
        LocalDateTime now = LocalDateTime.now();
        // 计算得到差值
        // 35
        System.out.println(ChronoUnit.DAYS.between(startDate, now));
        // 842
        System.out.println(ChronoUnit.HOURS.between(startDate, now));
        // 50535
        System.out.println(ChronoUnit.MINUTES.between(startDate, now));
        // 3032131
        System.out.println(ChronoUnit.SECONDS.between(startDate, now));
        // 3032131992
        System.out.println(ChronoUnit.MILLIS.between(startDate, now));

        // 或者

        Duration between = Duration.between(startDate, now);
        // 35
        System.out.println(between.toDays());
        // 842
        System.out.println(between.toHours());
        // 50533
        System.out.println(between.toMinutes());
        // 3032011164
        System.out.println(between.toMillis());
    }
```
