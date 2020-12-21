

## Stream对集合常见操作

### 什么是Stream

> **Stream**将要处理的元素集合看作一种流，在流的过程中，借助**Stream API**对流中的元素进行操作，比如：筛选、排序、聚合等。

`Stream`可以由数组或集合创建，对流的操作分为两种：

1. 中间操作，每次返回一个新的流，可以有多个。
2. 终端操作，每个流只能进行一次终端操作，终端操作结束后流无法再次使用。终端操作会产生一个新的集合或值。

另外，`Stream`有几个特性：

1. stream不存储数据，而是按照特定的规则对数据进行计算，一般会输出结果。
2. stream不会改变数据源，通常情况下会产生一个新的集合或一个值。
3. stream具有延迟执行特性，只有调用终端操作时，中间操作才会执行。

```java
@Data
@AllArgsConstructor
class User {
    private Integer id;
    private Integer age;
    private String name;
    private String sex;
}
```

### 匹配（foreach/find/match）

```java
public void streamTerminalOperation() {
    /**
         * 1.anyMatch:检查谓词是否至少匹配一个元素
         * */
    // 检查是否有一个name为'小敏'的user
    boolean isAnyMatch = userList.stream().anyMatch(user -> user.getName().equals("小敏"));
    if (isAnyMatch) {
        System.out.println("有name为小敏的用户");
    }

    /**
         * 2.allMatch:检查谓词是否匹配所有元素
         * */
    boolean isAllMatch = userList.stream().allMatch(user -> user.getName().equals("小敏"));
    if (isAllMatch) {
        System.out.println("所有用户name均为小敏");
    }

    /**
         * 3.noneMatch：确保流中没有任何元素与给定的谓词匹配
         * */
    boolean isNoneMatch = userList.stream().noneMatch(user -> user.getName().equals("张三"));
    if (isNoneMatch) {
        System.out.println("所有用户中没有name为张三的用户");
    }

    // ***流的终端操作之查找
    /**
         * 4. findAny:返回当前流中的任意元素(不一定是集合中符合条件的第一个元素)
         * */
    Optional<User> optionalUser = userList.stream().findAny();
    User user = optionalUser.orElse(null);
    System.out.println("findAny: " + user);

    /**
         * 5. findFirst:查找第一个元素
         * */
    Optional<User> optionalFirstUser = userList.stream().findFirst();
    User userFirst = optionalFirstUser.orElse(null);
    System.out.println("findFirst: " + userFirst);
}
```

### 遍历/筛选（filter）

​	在使用filter时需要注意**NullPointerException**的出现，如下demo中，若age为空，user.getAge() > 20则会出现NullPointerException，这里涉及Integer和int的拆箱问题

```java
    /**
     * 2.filter: 筛选符合条件的元素,注意函数式接口Predicate的使用
     * */
    // 得到年龄大于20的user名称
    List<String> ageMoreThan20UserNameList = userList.stream()
        .filter(user -> user.getAge() > 20).map(User::getName)
        .collect(Collectors.toList());
    System.out.println(ageMoreThan20UserNameList);    
```

### 聚合（max/min/count)

​	使用max、min获取最值的时候，同样需要注意**NullPointerException**，如下在获取String集合最长元素时，若元素有为空，在String::length会出现**NullPointerException**，在获取最大年龄用户demo中若age为空，在调用Comparator.comparingInt(User::getAge)同样会出现**NullPointerException**

#### 获取`String`集合中最长的元素

```
    List<String> listString = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujg");
    // 若有多个最长字符串，返回第一个
    Optional<String> maxString = listString.stream().max(Comparator.comparing(String::length));
    System.out.println("最长的字符串：" + maxString.get());
```

#### 获取`Integer`集合中的最大值。

```java
    List<Integer> listInteger = Arrays.asList(7, 6, 9, 4, 11, 6);
    // 自然排序
    Optional<Integer> maxInteger = listInteger.stream().max(Integer::compareTo);
    // 自定义排序
    Optional<Integer> maxInteger2 = listInteger.stream().max(new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o1.compareTo(o2);
        }
    });
    System.out.println("自然排序的最大值：" + maxInteger.get());
    System.out.println("自定义排序的最大值：" + maxInteger2.get());     
    }
```

#### 获取年龄最大的用户

```java
    // 获取年龄最大的用户
    // 注意null值过滤,否则Comparator.comparingInt(User::getAge)会出现空指针异常
    Optional<User> ageMaxUser = userList.stream()
        .filter(user -> user.getAge() != null)
        .max(Comparator.comparingInt(User::getAge));
    if (ageMaxUser.isPresent()) {
        System.out.println("年龄最大的用户为：" + ageMaxUser);
    }
```

#### 计算`Integer`集合中大于6的元素的个数

```java
    List<Integer> list = Arrays.asList(7, 6, 4, 8, 2, 11, 9);
    long count = list.stream().filter(x -> x > 6).count();
    System.out.println("list中大于6的元素个数：" + count);
```

### 映射(map/flatMap)

映射，可以将一个流的元素按照一定的映射规则映射到另一个流中。分为`map`和`flatMap`：

- `map`：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
- `flatMap`：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

#### 英文字符串数组的元素全部改为大写。整数数组每个元素+3。

```java
    String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
    List<String> strList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());

    List<Integer> intList = Arrays.asList(1, 3, 5, 7, 9, 11);
    List<Integer> intListNew = intList.stream().map(x -> x + 3).collect(Collectors.toList());

    System.out.println("每个元素大写：" + strList);
    System.out.println("每个元素+3：" + intListNew);
```

#### 将学生名字后面加上'test'字符串

```java
	List<User> usersNew = userList.stream().map(user -> {
            User user1 = new User();
            user1.setId(user.getId());
            user1.setName(user.getName() + "test");
            return user1;
        }).collect(Collectors.toList());

	System.out.println(usersNew.stream().map(User::getName).collect(Collectors.joining("、")));

// 输出结果：小明test、小明test、小敏test、小红test、小李test、小李test、小王test
```

#### 将两个字符数组合并成一个新的字符数组。

```java
    List<String> list = Arrays.asList("m,k,l,a", "1,3,5,7");
    List<String> listNew = list.stream().flatMap(s -> {
    // 将每个元素转换成一个stream
    String[] split = s.split(",");
    Stream<String> s2 = Arrays.stream(split);
    return s2;
    }).collect(Collectors.toList());

    System.out.println("处理前的集合：" + list);
    System.out.println("处理后的集合：" + listNew);
```

### 归约(reduce)

​	归约，也称缩减，顾名思义，是把一个流缩减成**一个值**，能实现对集合求和、求乘积和求最值操作。

#### 求`Integer`集合的元素之和、乘积和最大值。

```java
  List<Integer> list = Arrays.asList(1, 3, 2, 8, 11, 4);
  // 求和方式1
  Optional<Integer> sum = list.stream().reduce((x, y) -> x + y);
  // 求和方式2
  Optional<Integer> sum2 = list.stream().reduce(Integer::sum);
  // 求和方式3
  Integer sum3 = list.stream().reduce(0, Integer::sum);

  // 求乘积
  Optional<Integer> product = list.stream().reduce((x, y) -> x * y);

  // 求最大值方式1
  Optional<Integer> max = list.stream().reduce((x, y) -> x > y ? x : y);
  // 求最大值写法2
  Integer max2 = list.stream().reduce(1, Integer::max);

  System.out.println("list求和：" + sum.get() + "," + sum2.get() + "," + sum3);
  System.out.println("list求积：" + product.get());
  System.out.println("list求和：" + max.get() + "," + max2);
```

**求所有员工的年龄之和和最大年龄。**

```java
    // 求年龄和方式1
    Optional<Integer> ageSum = userList.stream().map(User::getAge).reduce(Integer::sum);
    // 求年龄和方式2
    Integer ageSum2 = userList.stream().reduce(0, (sum, user) -> sum += user.getAge(), (sum1, sum2) -> 				sum1 + sum2);
    // 求年龄和方式3
    Integer ageSum3 = userList.stream().reduce(0, (sum, p) -> sum += p.getAge(), Integer::sum);

    // 求最大年龄方式1
    Integer maxAge = userList.stream().reduce(0, (max, p) -> max > p.getAge() ? max : p.getAge(), 					Integer::max);
    // 求最大年龄方式2
    Integer maxAge2 = userList.stream().reduce(0, (max, p) -> max > p.getAge() ? max : p.getAge(),
                                               (max1, max2) -> max1 > max2 ? max1 : max2);
    System.out.println("工资之和：" + ageSum.get() + "," + ageSum2 + "," + ageSum3);
    System.out.println("最高工资：" + maxAge + "," + maxAge2);
```

### 收集(collect)

`	collect`，收集，可以说是内容最繁多、功能最丰富的部分了。从字面上去理解，就是把一个流收集起来，最终可以是收集成一个值也可以收集成一个新的集合。

> `collect`主要依赖`java.util.stream.Collectors`类内置的静态方法。

#### 3.6.1 归集(toList/toSet/toMap)

因为流不存储数据，那么在流中的数据完成处理后，需要将流中的数据重新归集到新的集合里。`toList`、`toSet`和`toMap`比较常用，另外还有`toCollection`、`toConcurrentMap`等复杂一些的用法。

```java
    List<Integer> list = Arrays.asList(1, 6, 3, 4, 6, 7, 9, 6, 20);
    List<Integer> listNew = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toList());
    Set<Integer> set = list.stream().filter(x -> x % 2 == 0).collect(Collectors.toSet());

    Map<Integer, User> userMap = userList.stream().filter(user -> user.getAge() > 10)
        .collect(Collectors.toMap(User::getId, p -> p));
    System.out.println("toList:" + listNew);
    System.out.println("toSet:" + set);
    System.out.println("toMap:" + userMap);
```

#### 3.6.2 统计(count/averaging)

`Collectors`提供了一系列用于数据统计的静态方法：

- 计数：`count`
- 平均值：`averagingInt`、`averagingLong`、`averagingDouble`
- 最值：`maxBy`、`minBy`
- 求和：`summingInt`、`summingLong`、`summingDouble`
- 统计以上所有：`summarizingInt`、`summarizingLong`、`summarizingDouble`

```java
    // 求总数
    Long count = userList.stream().collect(Collectors.counting());
    // 求平均年龄
    Double average = userList.stream().collect(Collectors.averagingDouble(User::getAge));
    // 求最大年龄
    Optional<Integer> max = userList.stream().map(User::getAge)
        .collect(Collectors.maxBy(Integer::compare));
    // 求年龄之和
    Integer sum = userList.stream().collect(Collectors.summingInt(User::getAge));
    // 一次性统计所有信息
    DoubleSummaryStatistics collect = userList.stream()
        .collect(Collectors.summarizingDouble(User::getAge));

    System.out.println("用户总数：" + count);
    System.out.println("用户平均年龄：" + average);
    System.out.println("用户最大年龄：" + max.get());
    System.out.println("用户年龄总和：" + sum);
    System.out.println("用户年龄所有统计：" + collect);
```

#### 3.6.3 分组(partitioningBy/groupingBy)

- 分区：将`stream`按条件分为两个`Map`，比如用户按年龄是否高于40分为两部分。
- 分组：将集合分为多个Map，比如用户按性别分组。有单级分组和多级分组。

**注意**：分组的属性若为null,会报**java.lang.NullPointerException: element cannot be mapped to a null key**

```java
    // 将用户按是否大于40岁分组
    Map<Boolean, List<User>> userPart = userList.stream()
        .collect(Collectors.partitioningBy(user -> user.getAge() > 40));
    // 将用户按性别分组
    Map<String, List<User>> sexMap = userList.stream()
        .collect(Collectors.groupingBy(User::getSex));
    // 将用户先按性别分组，再按年龄分组
    Map<String, Map<Integer, List<User>>> sexAgeMap = userList.stream()
        .collect(Collectors.groupingBy(User::getSex, Collectors.groupingBy(User::getAge)));
    System.out.println("用户年龄是否大于40岁分组情况：" + userPart);
    System.out.println("用户按性别分组情况：" + sexMap);
    System.out.println("用户先按性别分组，再按年龄分组：" + sexAgeMap);
```

### 3.6.4 接合(joining)

`joining`可以将stream中的元素用特定的连接符（没有的话，则直接连接）连接成一个字符串。

```java
    String allName = userList.stream().map(user -> user.getName()).collect(Collectors.joining(","));
    System.out.println("所有用户的姓名：" + allName);

    List<String> list = Arrays.asList("A", "B", "C");
    String string = list.stream().collect(Collectors.joining("-"));
    System.out.println("拼接后的字符串：" + string);
```

### 排序(sorted)

sorted，中间操作。有两种排序：

- sorted()：自然排序，流中元素需实现Comparable接口
- sorted(Comparator com)：Comparator排序器自定义排序

**注意：**排除的属性不能为null,否则会报**java.lang.NullPointerException**

```java
    // 按年龄排序（自然排序）
    List<User> users = userList.stream().sorted(
        Comparator.comparing(User::getAge)).collect(Collectors.toList());
    System.out.println("按年龄排序（自然排序）" + users);

    // 按年龄倒序
    List<User> users1 = users.stream().sorted(Comparator.comparing(
        User::getAge).reversed()).collect(Collectors.toList());
    System.out.println("按年龄倒序" + users1);

    // 先按年龄降序再按id升序
    List<User> users2 = userList.stream().sorted(Comparator.comparing(
        User::getAge).reversed().thenComparing(User::getId)).collect(Collectors.toList());
    System.out.println("先按年龄降序再按id升序:" + users2);

    // 先按年龄降序再按id升序(自定义排序)
    List<User> users3 = userList.stream().sorted((p1, p2) -> {
        if (p1.getAge().intValue() == p2.getAge().intValue()) {
            return p1.getId() - p2.getId();
        }else {
            return p2.getAge() - p1.getAge();
        }
    }).collect(Collectors.toList());
    System.out.println("先按年龄降序再按id升序(自定义排序):" + users3);
```

### 提取/组合

流也可以进行合并(concat)、去重(distinct)、限制(limit)、跳过(skip)等操作。

```java
    String[] arr1 = { "a", "b", "c", "d" };
    String[] arr2 = { "d", "e", "f", "g" };

    Stream<String> stream1 = Stream.of(arr1);
    Stream<String> stream2 = Stream.of(arr2);
    // concat:合并两个流 distinct：去重
    List<String> newList = Stream.concat(stream1, stream2).distinct().collect(Collectors.toList());
    // limit：限制从流中获得前n个数据
    List<Integer> collect = Stream.iterate(1, x -> x + 2).limit(10).collect(Collectors.toList());
    // skip：跳过前n个数据
    List<Integer> collect2 = Stream.iterate(1, x -> x + 2).skip(1).limit(5)
        .collect(Collectors.toList());

    System.out.println("流合并：" + newList);
    System.out.println("limit：" + collect);
    System.out.println("skip：" + collect2);
```

### 常见问题

**1.将集合中的对象转换为Map结构，key为对象的某个属性（重复则覆盖），value为对象**

```java
public void toMap1() {
    // 出现重复id,则直接用后面的覆盖前面的
    Map<Integer, User> userMap = users.stream()
        .collect(Collectors.toMap(User::getId, Function.identity(), (id1, id2) -> id2));
    System.out.println(userMap.size());
}
```

**2.将集合中的对象转换为Map结构，key为对象的某个属性（重复则覆盖），value为另一个属性**

```java
    /**
     * 将list转为Map,value为String
     * 1.如果报map里的value空指针异常，需要在value，也就是toMap()的第二个参数进行空(null)值的判断逻辑
     * 2.toMap()第三个参数是当有重复的Key时，会执行这段逻辑，传入2个参数，第一个参数是已经存在Map的value	 值，第二个是即将	 		覆盖的value值，最终以返回哪个值为准
     * */
  @Test
  public void toMap() {
    Map<Integer, String> nameMap = users.stream().collect(Collectors.toMap(User::getId,
    user -> user.getName() == null ? "" :user.getName(), (id1, id2) -> id2));
    System.out.println(nameMap);
   }
```
