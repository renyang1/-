## Spring事务及@Transactional总结

### 事务传播类型

### 异常分类

![image-20200806152229390](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20200806152229390.png)

**异常分为运行时异常（RuntimeException）和非运行时异常**

- 受检异常（checked Exception）:Exception下除了RuntimeException外的异常
- 非受检异常（unchecked Exception）:Runtime Exception及其子类的错误（Error）

如果不对运行时异常进行处理，那么出现运行时异常之后，要么是线程中止，要么是主程序终止。 如果不想终止，则必须捕获所有的运行时异常，决不让这个处理线程退出。

**队列里面出现异常数据了，正常的处理应该是把异常数据舍弃，然后记录日志**。不应该由于异常数据而影响下面对正常数据的处理。

非运行时异常是RuntimeException以外的异常，类型上都属于Exception类及其子类。如IOException、SQLException等以及用户自定义的Exception异常。

对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch并处理，否则程序就不能编译通过。所以，面对这种异常不管我们是否愿意，只能自己去写一大堆catch块去处理可能的异常。

### @Transactional的用法总结

### 注意

