---
title: 减少 JAVA 代码中的 if-else
categories: java
tags:
  - java
abbrlink: c3d44e35
---



![](https://tva4.sinaimg.cn/large/e444d1c7ly1gginkmw30jj20e806vmxd.jpg)

当项目中的代码数量越来越多，就会有各种各样的 if-else 组合，这就会导致代码有点“乱七八糟”的，尤其是没有注释的话，那可麻烦了 😡

所以需 要项目刚开始写的时候就定义一些方法，但是刚开始都是先注重功能也业务，所以这部分很容易忽略掉，所以整理记录下解决多

个 if-else 的优化方案 😄

---

# 提前 return

提前 return ,减少 else 的出现

```java
if(A==null){
	//do somethings
}else{
	//do somethings
	return;
}
//----------优化-------------
if(A!=null){
	//do somethings
	return;
}
//do somethings
```

# Optional

java8 引入主要是为了解决尴尬的 NPE 问题。Optional 类既可以含有对象也可以为空。既然可以判断对象是为空，从而减少 if-else 的判断。

创建 Optional 对象，需要注意以下问题。

```java
//创建一个空的,在get的时候会报错NoSuchElementException: No value present
Optional<Object> empty = Optional.empty();
//用这种方式创建，如果传入的对象A是空，在get的时候依然会报错空指针
Optional<Object> obj = Optional.of(A);
//用这种方式创建，如果传入的对象A是空，在get的时候会报错NoSuchElementException: No value present
Optional<Object> obj1 = Optional.ofNullable(A);
```

应该使用 ofNullable 创建 Optional 然后再利用**isPresent()**;判断是否为空再继续使用对象,

```java
Optional<Object> obj = Optional.ofNullable(new Object());
if(obj.isPresent()){
    System.out.println(obj.getClass());
}
```

这样的话可能还的需要 if 和文章的主题不符合了,所以还有另外的选择**ifPresent()**;该方法除了执行检查，还接收一个参数，如果对象不是空的，就对执行传入的 Lambda 表达式 。

```java
Optional<AA> obj = Optional.ofNullable(new AA("Jom"));
obj.ifPresent(o -> o.setName("Tom"));
System.out.println(obj.toString());
```

其他返回默认值的方法

orElse: 有值就返回，么有就返回传入的参数值。

```java
AA a1 = null;
AA a2 = new AA("tom");
AA aa = Optional.ofNullable(a1).orElse(a2);
System.out.println(aa.toString());
```

orElseGet: 有值就返回，没有就使用传入的方法返回值

```java
public static void main(String[] args) {
    AA a1 = null;
    AA a2 = new AA("tom");
    AA aa = Optional.ofNullable(a1).orElseGet(() -> getAA());
    System.out.println(aa);
}

public static AA getAA() {
    return new AA("Jerry");
}
```

# 策略模式

**策略模式**是一种行为设计模式， 它能让你定义一系列算法， 并将每种算法分别放入独立的类中，使其算法可以在运行时候根据条件进行更改。

## 多态策略

定义一个策略接口。然后实现不同的策略，根据不同的条件，走不同的策略

```java
public interface Strategy {
    void run();
}
public class FastStrategy implements Strategy {
    @Override
    public void run() {
        System.out.println("fast");
    }
}
public class SlowStrategy implements Strategy {
    @Override
    public void run() {
        System.out.println("slow");
    }
}
```

测试下：

```java
public class Test {
    private static HashMap<Integer, Strategy> map = new HashMap<>();

    static {
        map.put(1, new FastStrategy());
        map.put(2, new SlowStrategy());
    }

    public static void main(String[] args) {
        Strategy strategy = map.get(1);
        strategy.run();
    }
}
--
fast
```

## 枚举

枚举是一种特殊的抽象类，里面每个枚举可以看成是枚举类的一个对象。

```java
public enum CODE {
    SUCCESS,ERROR
}
```

当然可以添加一些属性

```java
 SUCCESS(1,"OK"),
 ERROR(2,"error");
```

枚举里面可以定义抽象方法，在每个枚举中实现该方法，然后根据 code 去做不同的实现，这样就像 if-else 判断一样了

```java
public enum CODE {

    SUCCESS(1, "OK") {
        @Override
        public String codeMsg() {
            return "result is ok";
        }
    },
    ERROR(2, "error") {
        @Override
        public String codeMsg() {
            return "result is error";
        }
    };

    private int code;
    private String msg;

    CODE(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public static CODE valueOf(int code){
        for (CODE c:values()){
            if(c.getCode()==code){
                return c;
            }
        }
        return null;
    }

    abstract public String codeMsg();
}

```

测试一下：

```java
public class test {
    public static void main(String[] args) {
        String msg = CODE.valueOf(1).codeMsg();
        System.out.println(msg);
    }
}
--
result is ok
```

## 枚举+switch-case

枚举中定义一个 getByValue 方法，名称可以随便的。这种方式也是多条件判断，但是比 if-else 看起来更容易理解。

```java
public static DirEnum getByValue(String value) {
    for (DirEnum dirEnum : values()) {
        if (dirEnum.getCode().equals(value)) {
            return dirEnum;
        }
    }
    return null;
}
```

switch-case 中使用枚举判断。

```java
---省略
switch (DirEnum.getByValue(enumFkIdType)) {
    case DIR_OWNER_ORG:
        obj.setDirOwnerId("AAA");
        break;
    case DIR_OWNER_DEPT:
        obj.setDirOwnerId("BBB");
        break;
    case DIR_OWNER_USER:
        obj.setDirOwnerId("CCC");
        break;
}
---省略
```

# 利用数组

一种编程思想，叫做表驱动法，意思是从表中查询结果来代替逻辑判断语句。

例如：获取每个班级对应的学生数，下标代表班级号码，例如：一班 59 人，二班 45 人......

```java
public class Test {
    private static int students[] = {49, 45, 49, 33};

    public static int studentsOfClassRoom(int m) {
        return students[--m];
    }

    public static void main(String[] args) {
        int i = studentsOfClassRoom(2);
        System.out.println(i);
    }
}
```

---

纸上得来终觉浅，绝知此事要躬行 🎈

参考文档：https://www.zhihu.com/question/344856665/answer/816270460
