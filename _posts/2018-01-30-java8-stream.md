---
layout: post
title: Java8里的stream使用总结
categories: 杂项
tags: java
author: 朋也
---

* content
{:toc}

> java8里新特性之一 stream，非常的好用，就是容易忘了怎么写了，下面来总结一下




## 案例

在项目中的一些需求的实现总结一下

#### Q：一个书籍的列表，按照类型分组

```java
@Data
@ToString
@AllArgsConstructor
class Book {
    int id;
    String name;
    String catetory;
}

@Test
public void test1() {
    List<Book> books = Arrays.asList(new Book(1, "Java入门", "编程"),
            new Book(2, "NodeJS入门", "编程"),
            new Book(3, "红楼梦", "名著"),
            new Book(4, "斗罗大陆", "修仙"),
            new Book(5, "十万个为什么", "科普"),
            new Book(6, "西游记", "名著"),
            new Book(7, "名人名言", "鸡汤"));
    Map<String, List<Book>> bookGroupByCategories = books.stream().collect(Collectors.groupingBy(Book::getCatetory));
    System.out.println(bookGroupByCategories);
}
```

打印结果

```log
{鸡汤=[DemoTest.Book(id=7, name=名人名言, catetory=鸡汤)], 名著=[DemoTest.Book(id=3, name=红楼梦, catetory=名著), DemoTest.Book(id=6, name=西游记, catetory=名著)], 科普=[DemoTest.Book(id=5, name=十万个为什么, catetory=科普)], 修仙=[DemoTest.Book(id=4, name=斗罗大陆, catetory=修仙)], 编程=[DemoTest.Book(id=1, name=Java入门, catetory=编程), DemoTest.Book(id=2, name=NodeJS入门, catetory=编程)]}
```

### Q：将集合中的集合收集在一起，`用户 —> 角色 -> 权限` 一个用户有多个角色，一个角色有多个权限

```java
@Data
@ToString
@AllArgsConstructor
class User {
    int id;
    String name;
    List<Role> roles;
}

@Data
@ToString
@AllArgsConstructor
class Role {
    int id;
    String name;
    List<Permission> permissions;
}

@Data
@ToString
@AllArgsConstructor
class Permission {
    int id;
    String name;
}

@Test
public void test2() {
    Permission permission1 = new Permission(1, "用户列表");
    Permission permission2 = new Permission(2, "话题列表");
    Permission permission3 = new Permission(3, "评论列表");
    Permission permission4 = new Permission(4, "收藏列表");
    Permission permission5 = new Permission(5, "日志列表");

    Role role1 = new Role(1, "普通用户", Arrays.asList(permission1, permission2));
    Role role2 = new Role(2, "付费用户", Arrays.asList(permission3, permission4, permission5));
    Role role3 = new Role(3, "管理员", Arrays.asList(permission1, permission2, permission3, permission4, permission5));

    User adminUser = new User(1, "admin", Arrays.asList(role1, role2, role3));
    User vipUser = new User(2, "vip", Arrays.asList(role1, role2));
    User normalUser = new User(3, "normal", Arrays.asList(role1));

    // 统计adminUser的所有权限
    List<List<Permission>> allPermissions = adminUser.getRoles().stream().map(Role::getPermissions).collect(Collectors.toList());
    System.out.println(allPermissions);
}
```

打印结果

```log
[[DemoTest.Permission(id=1, name=用户列表), DemoTest.Permission(id=2, name=话题列表)], [DemoTest.Permission(id=3, name=评论列表), DemoTest.Permission(id=4, name=收藏列表), DemoTest.Permission(id=5, name=日志列表)], [DemoTest.Permission(id=1, name=用户列表), DemoTest.Permission(id=2, name=话题列表), DemoTest.Permission(id=3, name=评论列表), DemoTest.Permission(id=4, name=收藏列表), DemoTest.Permission(id=5, name=日志列表)]]
```

可以看到打印出来的结果是role1 + role2 + role3，里面有重复的，要想不重复，在收集的时候用set来接收就可以了 `collect(Collectors.toSet())`

#### Q: 对集合中对象按照日期排序, 比如一个用户的列表，按照创建时间排序

```java
@Data
@ToString
@AllArgsConstructor
class User {
    int id;
    String name;
    Date createDate;
}

@Test
public void test3() throws ParseException {
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");

    User user1 = new User(1, "user1", simpleDateFormat.parse("2019-01-01"));
    User user2 = new User(2, "user2", simpleDateFormat.parse("2019-04-20"));
    User user3 = new User(3, "user3", simpleDateFormat.parse("2019-02-27"));
    User user4 = new User(4, "user4", simpleDateFormat.parse("2019-08-15"));
    User user5 = new User(5, "user5", simpleDateFormat.parse("2019-06-30"));

    List<User> users = Arrays.asList(user1, user5, user2, user4, user3);
    // 正序
    List<User> users1 = users.stream().sorted(Comparator.comparing(User::getCreateDate)).collect(Collectors.toList());
    System.out.println(users1);
    // 倒序
    List<User> users2 = users.stream().sorted((u1, u2) -> u2.getCreateDate().compareTo(u1.getCreateDate())).collect(Collectors.toList());
    System.out.println(users2);
}
```

打印结果

```log
[DemoTest.User(id=1, name=user1, createDate=Tue Jan 01 00:00:00 CST 2019), DemoTest.User(id=3, name=user3, createDate=Wed Feb 27 00:00:00 CST 2019), DemoTest.User(id=2, name=user2, createDate=Sat Apr 20 00:00:00 CST 2019), DemoTest.User(id=5, name=user5, createDate=Sun Jun 30 00:00:00 CST 2019), DemoTest.User(id=4, name=user4, createDate=Thu Aug 15 00:00:00 CST 2019)]
[DemoTest.User(id=4, name=user4, createDate=Thu Aug 15 00:00:00 CST 2019), DemoTest.User(id=5, name=user5, createDate=Sun Jun 30 00:00:00 CST 2019), DemoTest.User(id=2, name=user2, createDate=Sat Apr 20 00:00:00 CST 2019), DemoTest.User(id=3, name=user3, createDate=Wed Feb 27 00:00:00 CST 2019), DemoTest.User(id=1, name=user1, createDate=Tue Jan 01 00:00:00 CST 2019)]
```

----


**声明：代码来自尚硅谷官网上下载的java8视频教程**

## 创建Stream

```java
//1. 创建 Stream
@Test
public void test1() {
  //1. Collection 提供了两个方法  stream() 与 parallelStream()
  List<String> list = new ArrayList<>();
  Stream<String> stream = list.stream(); //获取一个顺序流
  Stream<String> parallelStream = list.parallelStream(); //获取一个并行流

  //2. 通过 Arrays 中的 stream() 获取一个数组流
  Integer[] nums = new Integer[10];
  Stream<Integer> stream1 = Arrays.stream(nums);

  //3. 通过 Stream 类中静态方法 of()
  Stream<Integer> stream2 = Stream.of(1, 2, 3, 4, 5, 6);

  //4. 创建无限流
  //迭代
  Stream<Integer> stream3 = Stream.iterate(0, (x) -> x + 2).limit(10);
  stream3.forEach(System.out::println);
  //生成
  Stream<Double> stream4 = Stream.generate(Math::random).limit(2);
  stream4.forEach(System.out::println);
}
```

## 对Stream进行中间操作

```java
//2. 中间操作
List<Employee> emps = Arrays.asList(
    new Employee(102, "李四", 59, 6666.66),
    new Employee(101, "张三", 18, 9999.99),
    new Employee(103, "王五", 28, 3333.33),
    new Employee(104, "赵六", 8, 7777.77),
    new Employee(104, "赵六", 8, 7777.77),
    new Employee(104, "赵六", 8, 7777.77),
    new Employee(105, "田七", 38, 5555.55)
);

/*
  筛选与切片
  filter——接收 Lambda ， 从流中排除某些元素。
  limit——截断流，使其元素不超过给定数量。
  skip(n) —— 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补
  distinct——筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素
  */

//内部迭代：迭代操作 Stream API 内部完成
@Test
public void test2() {
  //所有的中间操作不会做任何的处理
  Stream<Employee> stream = emps.stream()
      .filter((e) -> {
        System.out.println("测试中间操作");
        return e.getAge() <= 35;
      });

  //只有当做终止操作时，所有的中间操作会一次性的全部执行，称为“惰性求值”
  stream.forEach(System.out::println);
}

//外部迭代
@Test
public void test3() {
  Iterator<Employee> it = emps.iterator();

  while (it.hasNext()) {
    System.out.println(it.next());
  }
}

@Test
public void test4() {
  emps.stream()
      .filter((e) -> {
        System.out.println("短路！"); // &&  ||
        return e.getSalary() >= 5000;
      }).limit(3)
      .forEach(System.out::println);
}

@Test
public void test5() {
  emps.parallelStream()
      .filter((e) -> e.getSalary() >= 5000)
      .skip(2)
      .forEach(System.out::println);
}

@Test
public void test6() {
  emps.stream()
      .distinct()
      .forEach(System.out::println);
}
```

## 映射, 排序

```java
//2. 中间操作
/*
  映射
  map——接收 Lambda ， 将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
  flatMap——接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
  */
@Test
public void test1() {
  Stream<String> str = emps.stream()
      .map((e) -> e.getName());

  System.out.println("-------------------------------------------");

  List<String> strList = Arrays.asList("aaa", "bbb", "ccc", "ddd", "eee");

  Stream<String> stream = strList.stream()
      .map(String::toUpperCase);

  stream.forEach(System.out::println);

  Stream<Stream<Character>> stream2 = strList.stream()
      .map(TestStreamAPI1::filterCharacter);

  stream2.forEach((sm) -> {
    sm.forEach(System.out::println);
  });

  System.out.println("---------------------------------------------");

  Stream<Character> stream3 = strList.stream()
      .flatMap(TestStreamAPI1::filterCharacter);

  stream3.forEach(System.out::println);
}

public static Stream<Character> filterCharacter(String str) {
  List<Character> list = new ArrayList<>();

  for (Character ch : str.toCharArray()) {
    list.add(ch);
  }

  return list.stream();
}

/*
  sorted()——自然排序
  sorted(Comparator com)——定制排序
  */
@Test
public void test2() {
  emps.stream()
      .map(Employee::getName)
      .sorted()
      .forEach(System.out::println);

  System.out.println("------------------------------------");

  emps.stream()
      .sorted((x, y) -> {
        if (x.getAge() == y.getAge()) {
          return x.getName().compareTo(y.getName());
        } else {
          return Integer.compare(x.getAge(), y.getAge());
        }
      }).forEach(System.out::println);
}
```

## 查找与匹配

```java
List<Employee> emps = Arrays.asList(
    new Employee(102, "李四", 59, 6666.66, Status.BUSY),
    new Employee(101, "张三", 18, 9999.99, Status.FREE),
    new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
    new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
    new Employee(104, "赵六", 8, 7777.77, Status.FREE),
    new Employee(104, "赵六", 8, 7777.77, Status.FREE),
    new Employee(105, "田七", 38, 5555.55, Status.BUSY)
);

//3. 终止操作
/*
  allMatch——检查是否匹配所有元素
  anyMatch——检查是否至少匹配一个元素
  noneMatch——检查是否没有匹配的元素
  findFirst——返回第一个元素
  findAny——返回当前流中的任意元素
  count——返回流中元素的总个数
  max——返回流中最大值
  min——返回流中最小值
  */
@Test
public void test1() {
  boolean bl = emps.stream()
      .allMatch((e) -> e.getStatus().equals(Status.BUSY));

  System.out.println(bl);

  boolean bl1 = emps.stream()
      .anyMatch((e) -> e.getStatus().equals(Status.BUSY));

  System.out.println(bl1);

  boolean bl2 = emps.stream()
      .noneMatch((e) -> e.getStatus().equals(Status.BUSY));

  System.out.println(bl2);
}

@Test
public void test2() {
  Optional<Employee> op = emps.stream()
      .sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
      .findFirst();

  System.out.println(op.get());

  System.out.println("--------------------------------");

  Optional<Employee> op2 = emps.parallelStream()
      .filter((e) -> e.getStatus().equals(Status.FREE))
      .findAny();

  System.out.println(op2.get());
}

@Test
public void test3() {
  long count = emps.stream()
      .filter((e) -> e.getStatus().equals(Status.FREE))
      .count();

  System.out.println(count);

  Optional<Double> op = emps.stream()
      .map(Employee::getSalary)
      .max(Double::compare);

  System.out.println(op.get());

  Optional<Employee> op2 = emps.stream()
      .min((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()));

  System.out.println(op2.get());
}

//注意：流进行了终止操作后，不能再次使用
@Test
public void test4() {
  Stream<Employee> stream = emps.stream()
      .filter((e) -> e.getStatus().equals(Status.FREE));

  long count = stream.count();

  stream.map(Employee::getSalary)
      .max(Double::compare);
}
```

## 归约和收集

```java
List<Employee> emps = Arrays.asList(
    new Employee(102, "李四", 79, 6666.66, Status.BUSY),
    new Employee(101, "张三", 18, 9999.99, Status.FREE),
    new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
    new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
    new Employee(104, "赵六", 8, 7777.77, Status.FREE),
    new Employee(104, "赵六", 8, 7777.77, Status.FREE),
    new Employee(105, "田七", 38, 5555.55, Status.BUSY)
);

//3. 终止操作
/*
  归约
  reduce(T identity, BinaryOperator) / reduce(BinaryOperator) ——可以将流中元素反复结合起来，得到一个值。
  */
@Test
public void test1() {
  List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

  Integer sum = list.stream()
      .reduce(0, (x, y) -> x + y);

  System.out.println(sum);

  System.out.println("----------------------------------------");

  Optional<Double> op = emps.stream()
      .map(Employee::getSalary)
      .reduce(Double::sum);

  System.out.println(op.get());
}

//需求：搜索名字中 “六” 出现的次数
@Test
public void test2() {
  Optional<Integer> sum = emps.stream()
      .map(Employee::getName)
      .flatMap(TestStreamAPI1::filterCharacter)
      .map((ch) -> {
        if (ch.equals('六'))
          return 1;
        else
          return 0;
      }).reduce(Integer::sum);

  System.out.println(sum.get());
}

//collect——将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法
@Test
public void test3() {
  List<String> list = emps.stream()
      .map(Employee::getName)
      .collect(Collectors.toList());

  list.forEach(System.out::println);

  System.out.println("----------------------------------");

  Set<String> set = emps.stream()
      .map(Employee::getName)
      .collect(Collectors.toSet());

  set.forEach(System.out::println);

  System.out.println("----------------------------------");

  HashSet<String> hs = emps.stream()
      .map(Employee::getName)
      .collect(Collectors.toCollection(HashSet::new));

  hs.forEach(System.out::println);
}

@Test
public void test4() {
  Optional<Double> max = emps.stream()
      .map(Employee::getSalary)
      .collect(Collectors.maxBy(Double::compare));

  System.out.println(max.get());

  Optional<Employee> op = emps.stream()
      .collect(Collectors.minBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));

  System.out.println(op.get());

  Double sum = emps.stream()
      .collect(Collectors.summingDouble(Employee::getSalary));

  System.out.println(sum);

  Double avg = emps.stream()
      .collect(Collectors.averagingDouble(Employee::getSalary));

  System.out.println(avg);

  Long count = emps.stream()
      .collect(Collectors.counting());

  System.out.println(count);

  System.out.println("--------------------------------------------");

  DoubleSummaryStatistics dss = emps.stream()
      .collect(Collectors.summarizingDouble(Employee::getSalary));

  System.out.println(dss.getMax());
}

//分组
@Test
public void test5() {
  Map<Status, List<Employee>> map = emps.stream()
      .collect(Collectors.groupingBy(Employee::getStatus));

  System.out.println(map);
}

//多级分组
@Test
public void test6() {
  Map<Status, Map<String, List<Employee>>> map = emps.stream()
      .collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((e) -> {
        if (e.getAge() >= 60)
          return "老年";
        else if (e.getAge() >= 35)
          return "中年";
        else
          return "成年";
      })));

  System.out.println(map);
}

//分区
@Test
public void test7() {
  Map<Boolean, List<Employee>> map = emps.stream()
      .collect(Collectors.partitioningBy((e) -> e.getSalary() >= 5000));

  System.out.println(map);
}

//
@Test
public void test8() {
  String str = emps.stream()
      .map(Employee::getName)
      .collect(Collectors.joining(",", "----", "----"));

  System.out.println(str);
}

@Test
public void test9() {
  Optional<Double> sum = emps.stream()
      .map(Employee::getSalary)
      .collect(Collectors.reducing(Double::sum));

  System.out.println(sum.get());
}
```




