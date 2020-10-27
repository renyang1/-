##  反射（Reflection）

### 动态语言

​	一类在运行时可以改变其结构的语言：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其它结构上的变化。通俗点说就是在运行时代码可以根据某些条件改变自身结构。

​	主要动态语言有：Object-C、C#、JavaScript、PHP、Python...

### 静态语言

​	运行时结构不可变的语言就是静态语言。例如Java、C、C++

​	Java不是动态语言，但Java可以称之为“准动态语言”。即Java有一定的动态性，我们可以利用反射机制获得类似动态语言的特性。

### Class类

​	从Class类中可以得到的信息：某个类的属性、方法、构造器、类实现了哪些接口。对每个类而言，JRE都为其保留了一个不变的Class类型的对象。

1. Class本身也是一个类
2. Class对象只能由系统建立对象
3. **一个加载的类在JVM中只会有一个Class实例**
4. 一个Class对象对应的是一个加载到JVM中的一个.class文件
5. 每个类的实例都会记得自己是由哪个Class实例所生成
6. 通过Class可以完整地得到一个类中的所有被加载的结构
7. Class类是Reflection的根源，针对任何你想动态加载、运行的类，唯有先获得相应的Class对象

#### 哪些类型可以有Class对象

1. class:外部类、成员（成员内部类、静态内部类）、局部内部类、匿名内部类
2. interface：接口
3. []:数组  
4. enum：枚举
5. annotation:注解@interface
6. primitive type:基本数据类型
7. void

#### Class类的创建方式

1. 已知具体的类，通过class属性获取，改方法最安全可靠，程序性能最高
2. 已知某个类的实例，调用该类的getClass()方法获取Class对象
3. 已知类的全类名，且该类在类路径下，可以通过Class类的静态方法forName()获取，可能抛出ClassNotFoundException

```java
public class Test01 {

    public static void main(String[] args) throws ClassNotFoundException {

        Person person = new Person();

        // 1.通过对象获取
        Class c1 = person.getClass();
        System.out.println(c1.hashCode());

        // 2.forName获取
        Class c2 = Class.forName("com.ryang.reflection.Person");
        System.out.println(c2.hashCode());

        // 3.类名.class获取
        Class c3 = Person.class;
        System.out.println(c3.hashCode());
    }
}

@Data
class Person {
    private String name;
    private int age;
}
```

#### 获取类相关信息（名称、方法、属性、构造方法...）

```java
public class Test04 {

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {

        // 获得Class对象
        Class c1 = Class.forName("com.ryang.reflection.Dog");

        // 构造对象
        Dog dog = (Dog) c1.newInstance();
        System.out.println(dog);

        /** 通过反射操作方法 */
        // 获取方法
        Method setOrg = c1.getDeclaredMethod("setOrg", String.class);
        setOrg.invoke(dog, "大南高");
        System.out.println(dog.getOrg());

        /** 通过反射操作属性 */
        Dog dog1 = (Dog) c1.newInstance();
        System.out.println(dog1);
        // 获取属性
        Field org = c1.getDeclaredField("org");
        // 不能直接操作私有属性，需要调用属性或方法的setAccessible(true)关闭安全检测
        org.setAccessible(true);
        org.set(dog1, "南充");
        System.out.println(dog1.getOrg());
    }
}
```

#### 调用指定方法

```
Object invoke(Object obj, Object... args)
```

1. Object对应原方法的返回值，若原方法无返回值，此时返回null
2. 若原方法为静态方法，此时形参Object obj可为null
3. 若原方法形参列表为空，则Object[] args为null
4. 若原方法声明为private,则需要在调用此invoke()方法前，显示调用方法对象的setAccessible(true)方法，将可访问private的方法

```java
  public class Test04 {

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {

        // 获得Class对象
        Class c1 = Class.forName("com.ryang.reflection.Dog");

        // 构造对象
        Dog dog = (Dog) c1.newInstance();
        System.out.println(dog);

        /** 通过反射操作方法 */
        // 获取方法
        Method setOrg = c1.getDeclaredMethod("setOrg", String.class);
        setOrg.invoke(dog, "大南高");
        System.out.println(dog.getOrg());

        /** 通过反射操作属性 */
        Dog dog1 = (Dog) c1.newInstance();
        System.out.println(dog1);
        // 获取属性
        Field org = c1.getDeclaredField("org");
        // 不能直接操作私有属性，需要调用属性或方法的setAccessible(true)关闭安全检测
        org.setAccessible(true);
        org.set(dog1, "南充");
        System.out.println(dog1.getOrg());
    }
}
```



  

