### Stream对集合常见操作

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

1. **将集合中的对象转换为Map结构，key为对象的某个属性（重复则覆盖），value为对象**

   ```java
   public void toMap1() {
   // 出现重复id,则直接用后面的覆盖前面的
   Map<Integer, User> userMap = users.stream().collect(Collectors.toMap(User::getId, Function.identity(), (id1, id2) -> id2));
   System.out.println(userMap.size());
   }
   ```

2. **将集合中的对象转换为Map结构，key为对象的某个属性（重复则覆盖），value为另一个属性**

   ```java
   /**
   * 将list转为Map,value为String
   * 1.如果报map里的value空指针异常，需要在value，也就是toMap()的第二个参数进行空(null)值的判断逻辑
   * 2.toMap()第三个参数是当有重复的Key时，会执行这段逻辑，传入2个参数，第一个参数是已经存在Map的value	 值，第二个是即将覆盖的value值，最终以返回哪个值为准
   * */
   @Test
   public void toMap() {
   
   Map<Integer, String> nameMap = users.stream().collect(Collectors.toMap(User::getId,
   user -> user.getName() == null ? "" :user.getName(), (id1, id2) -> id2));
   System.out.println(nameMap);
   }
   ```

3. 