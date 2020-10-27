### limit 函数的使用及优化

#### limit函数的常见写法

```sql
select * from table LIMIT [offset,] rows
```

```sql
select * from table LIMIT rows OFFSET offset
```

1. offset 表示偏移量，注意从0开始
2. rows表示返回记录的行数
3. pgsql不支持```limit offset,rows```这种写法
4. 现在不支持rows为-1的写法

#### 实例

```sql
SELECT * from flow_order ORDER BY create_time LIMIT 10, 2;
```

​	该条语句会从表flow_order中查询offset 10开始之后的2条数据

```sql
SELECT * from flow_order ORDER BY create_time LIMIT 10 OFFSET 2;
```

​	该条语句会从表flow_order中查询offset 2开始之后的10条数据

#### 使用子查询优化分页查询

​	这种方式先定位偏移位置id，然后往后查询，这种方式适用于id递增的情况

```sql
-- 原方式
SELECT * from flow_order ORDER BY id limit 5 OFFSET 20000;
```

```sql
-- 优化后的方式
SELECT * from flow_order WHERE id >=(
	SELECT id from flow_order ORDER BY id limit 1 OFFSET 20000) LIMIT 5;
```

