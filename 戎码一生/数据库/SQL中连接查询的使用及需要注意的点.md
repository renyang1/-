## SQL中连接查询的使用及需要注意的点

### join执行过程

​	join执行时，首先将驱动表和关联表做一个笛卡尔积，on后面的条件是对这个笛卡尔积做一个过滤形成一张临时表，where条件是对生成的临时表再进行过滤。在inner join中过滤条件放在on和where后面没有区别，但在left join、right join中过滤条件放在on和where后面查询结果将可能不一样。

### 表数据

表a

![image-20201012172708003](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012172708003.png)

表b

![image-20201012172933395](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012172933395.png)

### inner join

#### 条件均放在on后

```mysql
select * from a inner join b on a.id = b.a and a.id = 3;
```

![image-20201012173126832](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012173126832.png)

#### 条件放在where后

```mysql
select * from a inner join b on a.id = b.a where a.id = 3;
```

![image-20201012173636716](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012173636716.png)

### left join

```mysql
select * from a left join b on a.id = b.a;
```

![image-20201012175752286](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012175752286.png)

```mysql
select * from a left join b on a.id = b.a and a.id = 3;
```

![image-20201012174324291](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012174324291.png)

```mysql
select * from a left join b on a.id = b.a where a.id = 3;
```

![image-20201012175529840](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012175529840.png)

### right join

```mysql
select * from a right join b on a.id = b.a;
```

![image-20201012180654324](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012180654324.png)

```mysql
select * from a right join b on a.id = b.a and a.id = 3;
```

![image-20201012180534415](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012180534415.png)

```mysql
select * from a right join b on a.id = b.a where a.id = 3;
```

![image-20201012180145071](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201012180145071.png)

### 结果分析

1. 在left join中，不管on中的条件是否满足，左表中的记录都会返回，对于满足条件的记录，两个表中的数据会连接起来，对于不满足条件的记录，右表字段全为null
2. 在right join与left join类似，只不过全部返回右表的数据
3. 在left join的第1、2个例子中，第一个on后的条件为a.id = b.a，第2个on后的条件为a.id = b.a and a.id = 3。对于a表中id=1的记录:在1中，on后的条件为真，所以该记录会出现两次，且a,b均有数据；在2中，on后的条件不成立，所以左表id=1的记录会出现，但右表中数据为null。所以当on后的条件为真时，左表的数据可能会出现多次，但若条件不成立左表的数据只会出现一次，且右表数据为null。