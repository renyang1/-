## Optional的使用

### 1. 为什么使用Optional

​	在实际开发中我们会遇到不能确定方法的**入参、出参、对象的属性**是否为null，所以通常需要在使用入参、出参、对象属性时使用if语句进行null判断，Optional可以帮助我们避免进行这种繁杂的判断。我们只需将可能为null的对象使用Optional进行包装，这样我们便能清楚的知道方法入参、出参、对象属性是否可能为null。除此之外，Optional还能帮助我们对null进行更多优雅的处理。

### 2. 应用Optional的几种模式

#### 1. 创建Optional对象

- **Optional.empty()**：创建空对象
- **Optional.of(T)**：创建非空Optional对象，T为空则抛出异常
- **Optional.ofNullable(T)**：可接收null的Optional,如果T为null,则得到的Optional为空对象

```java
User user = null;
// 1.声明一个空的Optional对象
Optional<User> optionalUser = Optional.empty();

// 2.根据一个非空值创建Optional，若值为null则立即抛出NullPointerException
Optional<User> optionalUser1 = Optional.of(user);

// 3.可接收null的Optional,如果user为null,则得到的Optional为空对象
Optional<User> optionalUser2 = Optional.ofNullable(user);
user = new User();
user.setName("任洋爱吃肉").setAge(18);
optionalUser2 = Optional.ofNullable(user);
```

#### 2. 使用map从Optional中提取和转换值

​	map操作会将提供的函数应用于流的每个元素。你可以把Optional对象看成一种特殊的集合数据，它至多包含一个元素。如果Optional包含一个值，那函数就将该值作为参数传递给map，**对该值进行转换**。**如果Optional为空，就什么也不做。**

```java
User user = new User();
user.setName("任洋爱吃肉").setAge(18);
Car car = new Car();
car.setName("BMW").setColor("red");
Optional<Car> optionalCar = Optional.ofNullable(car);
user.setCar(optionalCar);  
      
// map方法返回值也为Optional对象
Optional<String> carNameOptional = user.getCar().map(Car::getName);
System.out.println(carNameOptional.orElse("Unknown"));

optionalCar = Optional.empty();
user.setCar(optionalCar);
carNameOptional = user.getCar().map(Car::getName);
System.out.println(carNameOptional.orElse("Unknown"));
```

#### 3. 使用flatMap链接Optional对象

​	使用流时，flatMap方法接受一个函数作为参数，这个函数的**返回值是另一个流**。这个方法会应用到流中的每一个元素，最终形成一个新的流的流。但是flagMap会**用流的内容替换每个新生成的流**。换句话说，由方法生成的各个流会被**合并或者扁平化为一个单一的流**。这里你希望的结果其实也是类似的，但是你想要的是将两层的optional合并为一个。**对一个空的Optional对象调用flatMap()方法，返回值也是一个空的Optional对象**。

```java
        Insurance insurance = new Insurance();
        insurance.setName("平安");
        Optional<Insurance> insuranceOptional = Optional.ofNullable(insurance);

        Car car = new Car();
        car.setName("奔驰").setColor("black");
        car.setInsurance(insuranceOptional);
        Optional<Car> optionalCar = Optional.ofNullable(car);

        User user = new User();
        user.setName("任洋爱吃肉").setAge(18);
        user.setCar(optionalCar);

        String insuranceName = user.getCar().flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("Unknown");
```

#### 4. 默认行为及解引用Optional对象

1. **get()**

   如果变量存在，它直接返回封装的变量值，否则就抛出一个NoSuchElementException异常。

   ```java
           User user1 = new User("ry", 20);
           User user2 = null;
   
           Optional optional1 = Optional.ofNullable(user1);
           User user = (User) optional1.get();
           System.out.println(user);// 输出：User{name='ry', age=20}
           Optional optional2 = Optional.ofNullable(user2);
           optional2.get();// 抛出异常java.util.NoSuchElementException: No value present
   ```

2. **orElse(T other)**

   在Optional对象不包含值时提供一个默认值，**若方法入参中对象是调用另一个方法创建而来，那么不论是否包含值，创建默认值的方法都会被调用。**如下creatUser()方法会被调用两次。

   ```java
           User user1 = new User("ry", 20);
           User user2 = null;
   
           User u1 = Optional.ofNullable(user1).orElse(creatUser());// creatUser()会执行
           System.out.println(u1);
   
           User u2 = Optional.ofNullable(user2).orElse(creatUser());// creatUser()会执行
           System.out.println(u2);
   ```

3. **orElseGet(Supplier<? extends T> other)**

   ​	是orElse方法的**延迟调用版**，Supplier方法只有在Optional对象不含值时才执行调用。如果创建默认值是件耗时费力的工作，你应该考虑采用这种方式（借此提升程序的性能），或者你需要非常确定某个方法仅Optional为空时才进行调用，也可以考虑该方式。

   ```java
           User user1 = new User("ry", 20);
           User user2 = null;
   
           Optional optional1 = Optional.ofNullable(user1);
           // creatUser()不会执行
           User user3 = (User) optional1.orElseGet(()-> creatUser());
           System.out.println(user3);
   
           Optional optional2 = Optional.ofNullable(user2);
           // creatUser()会执行
           User user4 = (User) optional2.orElseGet(() -> creatUser());
           System.out.println(user4);
   ```

4. **orElseThrow(Supplier<? extends X> exceptionSupplier)**

   ​	和get方法非常类似，它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制希望抛出的异常类型。

   ```java
           User user1 = new User("ry", 20);
           User user2 = null;
   	
           User user3 = Optional.ofNullable(user1).orElseThrow(
                   () -> new RuntimeException("该用户不存在"));
           System.out.println(user3);
   		// 抛出java.lang.RuntimeException: 该用户不存在
           Optional.ofNullable(user2).orElseThrow(
               () -> new RuntimeException("该用户不存在"));
   ```

5. **isPresent()**

   如果值存在就返回 true，否则返回 false

   ```java
           User user1 = new User("ry", 20);
           User user2 = null;
   
           boolean result1 = Optional.ofNullable(user1).isPresent();
           System.out.println(result1);// true
   
           boolean result2 = Optional.ofNullable(user2).isPresent();
           System.out.println(result2);// false
   
           System.out.println(Optional.empty().isPresent());// false
   ```

6. **ifPresent(Consumer<? super T>)**

   让你能在变量值存在时执行一个**作为参数传入**的方法，否则就不进行任何操作。

   ```java
           User user1 = new User("ry", 20);
   		// 输出：20
           Optional.ofNullable(user1).ifPresent(
               user -> System.out.println(user.getAge()));
   
           User user2 = null;
   		// 无如何输出
           Optional.ofNullable(user2).ifPresent(
               user -> System.out.println(user.getAge()));
   ```

#### 5. 使用filter剔除特定值

​	filter方法接受一个谓词作为参数。如果Optional对象的值存在，并且它符合谓词的条件，filter方法就返回其值；否则它就返回一个空的Optional对象。**如果Optional对象为空，它不做任何操作**，反之，它就对Optional对象中包含的值施加谓词操作。**如果该操作的结果为true，它不做任何改变，直接返回该Optional对象，否则就将该值过滤掉，将Optional的值置空**。

```java
        User user1 = new User("ry", 20);
        User user2 = null;

		// 输出：User{name='ry', age=20}
        Optional.ofNullable(user1)
                .filter(user -> user.getName().equals("ry"))
                .ifPresent(user -> System.out.println(user));
		
		//无输出 
        Optional.ofNullable(user2)
                .filter(user -> user.getName().equals("ZhangSan1"))
                .ifPresent(user -> System.out.println(user));
```