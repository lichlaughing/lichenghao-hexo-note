---
title: JDK8 Stream
categories: java
tags:
  - java
  - stream
abbrlink: 4be4a15a
---
Stream 是 Java8 API 推出的一个新的抽象，它可以像写 SQL 语句一样去处理 Java 集合或者更高阶的抽象处理，比如排序，过滤，分组聚合等。

了解 Stream 之前，需要了解下函数式接口，或者函数式编程。

函数式接口：有且只有一个未实现的方法的接口，一般通过`@FunctionalInterface`注解来标识，最常见的这样的接口比如`Runnable`

还有比如说 jdk8 新增的`Consumer`接口

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```

如果要实现这个接口，那么通常来讲我们会用匿名内部类的方式

```java
Consumer consumer = new Consumer() {
            @Override
            public void accept(Object o) {
                System.out.println(o);
            }
        };
```

如果使用函数式编程的话，就会变得十分的简洁

```java
//变的简单
Consumer c = (obj) -> {System.out.println(obj);};
//可以继续简写
Consumer c = (obj)-> System.out.println(obj);
//甚至更简单
Consumer c = System.out::println;
```

---

Stream 流是 Java 8 新提供给开发者的一组操作集合的 API，将要处理的元素集合看作一种流，流在管道中传输，并且可以在管道的节点上进行处理，比如筛选、排序、聚合等。元素流在管道中经过中间操作（intermediate operation）的处理，最后由终值操作 (terminal operation) 得到前面处理的结果。

{% note red 'fas fa-fan' simple%}
**Stream 有三点非常重要的特性**
{% endnote %}

1. Stream 是不会存储元素的。
2. Stream 不会改变原对象，相反，他们会返回一个持有结果的新 Stream。
3. Stream 操作是延迟执行的。意味着它们会等到需要结果的时候才执行，也就是说的有终值操作执行。

{% note red 'fas fa-fan' simple%}
**Stream 分为中间操作和终值操作**
{% endnote %}

**中间操作(Intermediate**)：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。

常见的操作：map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

**终值操作(Terminal)**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果。

常见的终值操作：forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator。

{% note red 'fas fa-fan' simple%}
**创建流的几种方式**
{% endnote %}

```java
List< String> createStream = new ArrayList< String>();
// 顺序流
Stream< String> stream = createStream.stream();
// 并行流
Stream< String> parallelStream = createStream.parallelStream();
// of()方法创建
Stream< String> stringStream = Stream.of(
       createStream.toArray(new String[createStream.size()]));
```

# Stream API 的语法格式

Stream API 使用 `->` 箭头操作符，将操作分成左右两侧。左侧主要是参数列表，右侧主要是需要执行的方法体。

## 格式 1：无参，无返回值

```java
格式：() -> xxx
```

举栗子：

```java
Runnable runnable = () -> System.out.println("Hello World");
```

## 格式 2：一个参数，无返回值

```java
格式：(x) -> xxx
```

举栗子：

```java
Consumer consumer = (x)-> System.out.println(x);
consumer.accept("Hello World");
```

如果只有一个参数，那么参数的括号可以省略

举栗子：

```java
Consumer consumer = x -> System.out.println(x);
consumer.accept("Hello World");
```

## 格式 3：多个参数，有返回值，方法多条语句

```java
格式：  （x,y）-> {
 				......
       }
```

举栗子：

```java
Comparator<Integer> comparator = (x, y) -> {
    System.out.println("比较大小");
    return Integer.compare(x, y);
};
```

如果有返回值，并且方法只有一条语句，那么`return` 和`{}` 都可以省略

举栗子：

```java
Comparator<Integer> comparator1 = (x, y) -> Integer.compare(x, y);
```

**可以把省略的情形总结下：**

左侧一个参数，括号可以省略；右侧方法一条语句，返回值和大括号省略

除此之外，参数列表的数据类型可以省略不写，JVM 会根据上下文推断出数据类型，称之为`类型推断`

# JDK8 内置函数式接口

## Consumer< T >

消费型接口，提供方法：`void accept(T t)`

```java
@Test
public void test1() {
  goConsumer(100.0, (x) -> System.out.println("消费了：" + x + "RMB"));
}
public void goConsumer(Double m, Consumer consumer) {
  consumer.accept(m);
}
```

## Supplier< T >

Supplier< T > 生产型接口，提供方法：`T get()`

```java
@Test
public void test2() {
  List<Student> stu = getStus(5, () -> new Student());
  System.out.println(stu);
}

// 获取个学生集合
public List<Student> getStus(Integer num, Supplier<Student> supplier) {
  List<Student> stus = new ArrayList<>();
  for (int i = 0; i < num; i++) {
    stus.add(supplier.get());
  }
  return stus;
}
```

## Function<T, R>

Function<T, R> 函数接口，将 Function 对象应用到输入的参数上，然后返回计算结果。提供方法：`R apply(T t)`

```java
@Test
public void test3() {
  Student jerry = new Student("Jerry", "20");
  String stuNameAndAge = getStuNameAndAge(jerry, (x) -> "姓名：" + x.getName() + "|年龄：" + x.getAge());
  System.out.println(stuNameAndAge);
}

// 拼接姓名和年龄格式为：姓名：Jerry|年龄：20
public String getStuNameAndAge(Student stu, Function<Student, String> function) {
  return function.apply(stu);
}
```

除此之外，还有两个方法`andThen` `compose` 这两个方法可以改变函数的执行顺序，如下所示：

```java
public static void main(String[] args) {
        // 定义两个函数
        Function<Integer, Integer> f1 = f -> f + 1;
        Function<Integer, Integer> f2 = f -> f * 2;
        // 先执行f1,在执行f2 结果4
        Integer r1 = f1.andThen(f2).apply(1);
        System.out.println(r1);
        // 先执行f2,在执行f1 结果3
        Integer r2 = f1.compose(f2).apply(1);
        System.out.println(r2);
 }
```

## Predicate< T >

Predicate< T > 断言型接口，接口方法`boolean test(T t)` 根据指定的参数判断真假

```java
@Test
public void test4() {
  List<Student> students = Arrays.asList(
    new Student("李白", "10"),
    new Student("杜甫", "13"),
    new Student("李世民", "34"));
  // 查找姓李的学生
  List<Student> res = filterStu(students, (student -> student.getName().contains("李")));
  System.out.println(res);
}

// 过滤学生
public List<Student> filterStu(List<Student> students, Predicate<Student> predicate) {
  List<Student> res = new ArrayList<>();
  for (Student student : students) {
    if (predicate.test(student)) {
      res.add(student);
    }
  }
  return res;
}
```

除了上面，还有三个默认方法和一个静态方法

1. `default Predicate<T> and(Predicate<? super T> other)`

和逻辑运算符&&一样，一个为假就不会去计算下一个；多个都为真才是真

```java
public class Test1 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(6);
        list.add(9);
        list.add(20);

        // 使用&&过滤数据 [6, 9]
        List<Integer> c1 = list.stream()
                .filter(integer -> integer > 5 && integer < 10)
                .collect(Collectors.toList());

        // 使用 Predicate<T> and  [20]
        List<Integer> c2 = list.stream()
                .filter(integer -> fil(integer, integer1 -> integer1 > 10, integer1 -> integer1 < 30))
                .collect(Collectors.toList());
    }

    /**
     * 过滤方法
     */
    public static boolean fil(Integer num, Predicate<Integer> p1, Predicate<Integer> p2) {
        return p1.and(p2).test(num);
    }
}
```

2. `default Predicate<T> or(Predicate<? super T> other)`

这个和第一个几乎就是一回事了，就是把 and 变成了 or

```java
There's no code here...
```

3. `default Predicate<T> negate()`

negate 表示结果取反

```java
public class Test1 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(6);
        list.add(9);
        list.add(20);

        // 使用negate 取反 得到 [1, 6, 9]
        List<Integer> c1 = list.stream()
                .filter(integer -> fil(integer, integer1 -> integer1 > 10, integer1 -> integer1 < 30))
                .collect(Collectors.toList());
        System.out.println(c1);
    }

    /**
     * 过滤方法，negate表示结果取反
     */
    public static boolean fil(Integer num, Predicate<Integer> p1, Predicate<Integer> p2) {
        return p1.and(p2).negate().test(num);
    }
}
```

4. `static <T> Predicate<T> isEqual(Object targetRef)`

这个就是直接调用 Object 的 equals 方法

```java
public class Test1 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(6);
        list.add(9);
        list.add(20);

        // 使用Object#equals方法 取反 得到 [6]
        List<Integer> c1 = list.stream()
                .filter(Predicate.isEqual(6))
                .collect(Collectors.toList());
        System.out.println(c1);
    }
}
```

## 其他的子接口

```java
BiFunction<T, U, R> # R apply(T t, U u); //两个参数返回结果
```

```java

```

......

# Lambda 方法引用和构造器引用

要求 lambda 方法体中的方法参数和返回值类型与函数式接口中的抽象方法的参数和返回值类型一致，就可以使用方法引用

## 方法引用：对象::实例方法名

```java
// 打印
Consumer<String> consumer = System.out::println;
consumer.accept("Hello World");
// 获取学生姓名
Student student = new Student("Jerry", "100");
Supplier<String> studentSupplier = student::getName;
System.out.println(studentSupplier.get());
```

## 方法引用：类::静态方法名

```java
Comparator<Integer> comparator = Integer::compare;
int compare = comparator.compare(2, 1);
System.out.println(compare);
```

## 方法引用：类::实例方法名

这种方式比较特殊：如果 lambda 方法体中的第一个参数是方法的调用者，第二个参数是方法的参数，可以使用`类::实例方法名`

```java
BiPredicate<String, String> b1 = (x, y) -> x.equals(y);
System.out.println(b1.test("abc", "abc"));
// 类::实例方法名
BiPredicate<String, String> b2 = String::equals;
System.out.println(b2.test("abc", "abc"));
```

## 构造器引用：ClassName::new

这里需要注意的是：构造器引用需要对应的类有对应的构造方法，例如下面第一个需要有无参构造方法；第二个例子需要有一个参数的构造方法。也就说构造方法中的参数列表需要和抽象方法中的参数列表一致。

```java
Supplier<Student> s1 = () -> new Student();
System.out.println(s1.get());
//  转换ClassName::New
Supplier<Student> s2 = Student::new;
System.out.println(s2.get());
// 或者
Function<String, Student> function = Student::new;
System.out.println(function.apply("Jerry"));
```

## 数组引用：Type[]::new

```java
// 测试创建数组
Function<Integer, String[]> function = (x) -> new String[x];
System.out.println(function.apply(2));
// 使用数组引用
Function<Integer, String[]> function1 = String[]::new;
System.out.println(function1.apply(10));
```

# Stream 基础的三步操作

Stream 的三步走：创建 Stream——>中间操作（无数个）——>终值操作（终值操作）

## 创建 Stream

```java
// 创建Stream
// 1.Collection集合提供的Stream()方法
List<String> list = new ArrayList<>();
Stream<String> stream1 = list.stream();

// 2.Arrays中的静态方法获取数组流
Student[] students = new Student[10];
Stream<Student> stream2 = Arrays.stream(students);

// 3.通过Stream中的静态方法
Stream<String> stream3 = Stream.of("a", "b", "c");

// 4.创建无限流
// (1)迭代
Stream<Integer> iterate = Stream.iterate(0, (x) -> x + 1);
iterate.limit(5).forEach(System.out::println);
// (2)生成
Stream<Student> generate = Stream.generate(Student::new);
generate.limit(5).forEach(System.out::println);
```

## 中间操作

中间不会执行任何操作，只有终值操作才会一次性执行全部内容，称之为：惰性求值

### filter 过滤数据

```java
public class ABC {

    private static List<Student> stuList = new ArrayList<>();

    static {
        stuList.add(new Student("1", "Tom", 20));
        stuList.add(new Student("2", "Jerry", 32));
        stuList.add(new Student("3", "Jack", 45));
    }

    public static void main(String[] args) {
        // 过滤年龄大于20岁的学生
        List<Student> collect =
            stuList.stream().filter(student -> student.getAge() > 20).collect(Collectors.toList());
        System.out.println(collect);
    }

    static class Student {
        private String id;
        private String name;
        private Integer age;

        public Student() {
        }

        public Student(String id, String name, Integer age) {
            this.id = id;
            this.name = name;
            this.age = age;
        }

        public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public Integer getAge() {
            return age;
        }

        public void setAge(Integer age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "Student{" +
                    "id='" + id + '\'' +
                    ", name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```

### Limit

```java
public void LimitTest() {
  List<Student> students = Arrays.asList(
    new Student("李白", "10"),
    new Student("杜甫", "13"),
    new Student("李世民", "34"));
  // 取前两个
  students.stream().limit(2).forEach(System.out::println);
}
```

### Skip

```java
public void LimitTest() {
  List<Student> students = Arrays.asList(
    new Student("李白", "10"),
    new Student("杜甫", "13"),
    new Student("李世民", "34"));
  // 跳过前一个，取后俩个
  students.stream().skip(1).forEach(System.out::println);
}
```

### distinct

去重复，对象需要重写 hashcode 和 equals

```java
public void LimitTest() {
  List<Student> students = Arrays.asList(
    new Student("李白", "10"),
    new Student("杜甫", "13"),
    new Student("李世民", "34"));
  students.stream().distinct().forEach(System.out::println);
}
```

### map 映射

将元素转换成其他形式或者提取信息。接受一个函数作为参数，并且将该函数作用每一个元素身上，将其映射成一个新的元素。

```java
public void MapTest() {
  List<Student> students = Arrays.asList(
    new Student("李白", "10"),
    new Student("杜甫", "13"),
    new Student("李世民", "34"));
  // 提取学生姓名
  students.stream().map(student -> student.getName()).forEach(System.out::println);
}
```

### flatMap 映射

接受一个函数作为参数，将流中的每个值转换成另外一个流，然后把所有流合成一个流。

```java
public void FlatMapTest() {
  List<String> strings = Arrays.asList("aa", "bb", "cc");
  // 将字符串集合，拆分成字符集合打印出来
  strings.stream().flatMap(s -> {
    List<Character> characters = new ArrayList<>();
    char[] chars = s.toCharArray();
    for (int i = 0; i < chars.length; i++) {
      characters.add(chars[i]);
    }
    return characters.stream();
  }).forEach(System.out::print);
}
```

### Sorted()和 Sorted(Comparator c)

```java
public void SortedTest() {
        List<String> strings = Arrays.asList("aa", "bb", "cc");
        strings.stream().sorted().forEach(System.out::println);
    }
```

```java
public void SortedTest() {
  List<Student> students = Arrays.asList(
    new Student("李白", "10"),
    new Student("杜甫", "13"),
    new Student("李世民", "34"));
  students.stream().sorted((s1, s2) -> {
    if (s1.getName().equals(s2.getName())) {
      return s1.getAge().compareTo(s2.getAge());
    }
    return s1.getName().compareTo(s2.getName());
  }).forEach(System.out::println);
}
```

## 终值操作

### 查找匹配

```java
public void AllMatchTest() {
        List<Employee> employees = Arrays.asList(
                new Employee("甲", 28, 9000.0, Status.BUSY),
                new Employee("乙", 58, 5000.0, Status.FREE),
                new Employee("丙", 36, 8000.0, Status.HOLIDAY),
                new Employee("丁", 12, 2000.0, Status.BUSY)
        );
        // allMatch:是否匹配所有元素
        boolean allMatch = employees.stream()
                .allMatch(employee -> employee.getStatus().equals(Status.BUSY));
        System.out.println(allMatch);
        // anyMatch:是否至少匹配一个元素
        boolean anyMatch = employees.stream()
                .anyMatch(employee -> employee.getStatus().equals(Status.BUSY));
        System.out.println(anyMatch);
        // noneMatch:检查是否没有匹配所有元素
        boolean noneMatch = employees.stream()
                .noneMatch(employee -> employee.getStatus().equals(Status.BUSY));
        System.out.println(noneMatch);
        // findFirst:返回第一个元素
        Employee employee = employees.stream()
                .sorted(Comparator.comparingDouble(Employee::getSalary)).findFirst().get();
        System.out.println(employee);
        // findAny:返回任意元素
        Employee findAny = employees.stream().findAny().get();
        System.out.println(findAny);
        // count:总数
        long count = employees.stream().count();
        System.out.println(count);
        // max:最大
        Employee max = employees.stream()
                .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())).get();
        System.out.println(max);
        // min:最小
        Employee min = employees.stream()
                .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())).get();
        System.out.println(min);
    }
```

### reduce 归并操作

将流中的元素反复结合，得到一个新的值

```java
public void reduceTest() {
        List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
        // 元素统计求和
        Integer reduce = integers.stream().reduce(0, (x, y) -> x + y);
        System.out.println(reduce);

        // 工资总和
        List<Employee> employees = Arrays.asList(
                new Employee("甲", 28, 9000.0, Status.BUSY),
                new Employee("乙", 58, 5000.0, Status.FREE),
                new Employee("丙", 36, 8000.0, Status.HOLIDAY),
                new Employee("丁", 12, 2000.0, Status.BUSY)
        );
        Double aDouble = employees.stream().map(Employee::getSalary).reduce(Double::sum).get();
        System.out.println(aDouble);
    }
```

### Collect 收集操作

将流转化成其他形式，接收一个 collector 接口的实现，用于给 Stream 中元素做汇总的方法

```java
public void collectTest() {
        List<Employee> employees = Arrays.asList(
                new Employee("甲", 28, 9000.0, Status.BUSY),
                new Employee("乙", 58, 5000.0, Status.FREE),
                new Employee("丙", 36, 8000.0, Status.HOLIDAY),
                new Employee("丁", 12, 2000.0, Status.BUSY));
        // 所有姓名
        List<String> nameList = employees.stream()
                .map(employee -> employee.getName()).collect(Collectors.toList());
        System.out.println(nameList);
        // 总数
        Long count = employees.stream().collect(Collectors.counting());
        System.out.println(count);
        // 计算下工资平均值
        Double averageSalary = employees.stream().collect(Collectors.averagingDouble(Employee::getSalary));
        System.out.println(averageSalary);
        // 工资最高的人
        Optional<Employee> maxSalary = employees.stream()
                .collect(Collectors.maxBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
        System.out.println(maxSalary.get());
        // 工资最低的人
        Optional<Employee> minSalary = employees.stream()
                .collect(Collectors.minBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
        System.out.println(minSalary.get());

  			// 按照状态分组
        Map<Status, List<Employee>> groupByStatus = employees.stream()
                .collect(Collectors.groupingBy(Employee::getStatus));
        System.out.println(groupByStatus);

  			// 多级分组：先按照状态分组再按照工资分组
        Map<Status, Map<Double, List<Employee>>> groupByStatusAndSalary = employees.stream()
                .collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy(Employee::getSalary)));
        System.out.println(groupByStatusAndSalary);

  			// 分区: 工资5k以上的一波，以下的一波
        Map<Boolean, List<Employee>> partingBySalary = employees.stream()
                .collect(Collectors.partitioningBy(employee -> employee.getSalary() > 5000));
        System.out.println(partingBySalary);

  			// 获得统计数据
        DoubleSummaryStatistics summaryStatistics = employees.stream()
                .collect(Collectors.summarizingDouble(Employee::getSalary));
        System.out.println(summaryStatistics.getAverage());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getMax());
        System.out.println(summaryStatistics.getCount());

  			// 链接字符串，指定分隔符
        String names = employees.stream()
                .map(Employee::getName).collect(Collectors.joining(","));
        System.out.println(names);
    }
```

# 场景使用

## 集合去重分组

```java
List<Student> studentList = new ArrayList<>();
studentList.add(new Student("甲", 15));
studentList.add(new Student("乙", 15));
studentList.add(new Student("丙", 16));
studentList.add(new Student("丁", 16));

// 按照年龄去重
Object[] ages = studentList.stream().map(student -> student.getAge()).distinct().toArray();

// 按照年龄分组
Map<Integer, List<Student>> map =
    studentList.stream().collect(Collectors.groupingBy(student -> student.getAge()));
```

相关学习文档：https://developer.ibm.com/zh/articles/j-experience-stream/
