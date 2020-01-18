## Stream的常见使用场景

```java
@Data
    public static class User {

        /** 姓名 */
        private String name;

        /** 年龄 */
        private Integer age;

        /** 压岁钱 */
        private Integer money;

    }
```

### 自定义实体属性进行对象去重

```java
Set<String> nameSet = new HashSet<>();
List<User> users1 = users.stream().filter(user -> {
	boolean flag = !nameSet.contains(user.getName());
	if (flag) {
		nameSet.add(user.getName());
		return true;
	}else {
		return false;
	}
}).collect(Collectors.toList());
```



