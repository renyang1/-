## MySQL常用脚本

### 1.分析SQL执行时间相关脚本

1. **查看profiling是否开启**

   开启它可以让 MySQL 收集在 SQL 执行时所使用的资源情况，profiling=0 代表关闭，我们需要把 profiling 打开，即设置为 1：

```mysql
-- 查询是否开启
select @@profiling;

-- 开启
set profiling=1;
```

2. 查看当前会话产生的profiles

```mysql
-- 查询所有profiles
show profiles;

-- 查询上一次查询的执行时间
show profile;

-- 通过指定Query ID查询
show profile for query 2;
```

![image-20200727142958449](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20200727142958449.png)

![image-20200727142811640](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20200727142811640.png)

### 2.查询数据库版本

```mysql
SELECT VERSION();
```



