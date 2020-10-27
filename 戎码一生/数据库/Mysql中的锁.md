## Mysql中的锁

### 全局锁

### 表锁

### 行锁

​	InnoDB行锁是通过给索引上的索引项加锁来实现的，InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！**

### 排他锁

### 间隙锁

```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `c` (`c`)
) ENGINE=InnoDB;

insert into t values(0,0,0),(5,5,5),
(10,10,10),(15,15,15),(20,20,20),(25,25,25);
```



![image-20201010163212252](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201010163212252.png)

​	为了解决幻读问题，InnoDB引入了间隙锁，锁的就是两个值之间的空隙。当使用**范围条件或者使用相等条件但相等的记录不存在**时，InnoDB会对这个“间隙”加锁。跟间隙锁存在冲突关系的是往这个间隙中插入一个记录这个操作，间隙锁之间不存在冲突关系。

![image-20201010163416971](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201010163416971.png)

| session1                                                     | session2                                                     |
| ------------------------------------------------------------ | :----------------------------------------------------------- |
| BEGIN;                                                       | BEGIN;                                                       |
| 对不存在的记录行加for update锁<br />SELECT * FROM t WHERE id = 30 for update; |                                                              |
|                                                              | 对不存在的记录行加for update锁<br />SELECT * FROM t WHERE id = 40 for update;<br />执行成功，不会出现锁等待 |
| INSERT INTO t VALUES(30, 30, 30);<br />锁等待                |                                                              |
|                                                              | INSERT INTO t VALUES(40, 40, 40);<br />Deadlock found when trying to get lock; try restarting transaction。<br />插入报错释放锁，此时session1插入成功 |

