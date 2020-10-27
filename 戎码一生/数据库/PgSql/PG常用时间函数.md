## PG常用时间函数

### PG获取当前时间的几种方式

1. **now()**
      通过now()获取的时间是最完整的时间，包括时区，秒也保留到了6位小数。

   ```sql
   SELECT now();
   ```

   ![image-20200325153958385](https://raw.githubusercontent.com/renyang1/typora/master/image-20200325153958385.png)

2. **CURRENT_TIMESTAMP**效果是和now()一样的

3. **current_time** 
      只显示当前的时间,不包括日期，时区，秒也保留到了6位小数

   ```sql
   select current_time;
   ```

   ![image-20200325154306871](https://raw.githubusercontent.com/renyang1/typora/master/image-20200325154352480.png)

4. **current_date**
       只显示当前的日期，不包括小时等信息

   ```sql
   select current_date;
   ```

   ![image-20200325154352480](https://raw.githubusercontent.com/renyang1/typora/master/image-20200325154306871.png)

### 时间转字符串

​	使用**to_char()**函数，并指定需要转换成的时间格式，可以将时间将转换为字符串

```sql
SELECT to_char(create_time, 'yyyy-MM-dd hh24:mi:ss') 
FROM finance_receipt_record 
WHERE no = '419210815509169208';
```

### 字符串转时间

```sql
select to_date('2018-03-12 18:47:35','yyyy-MM-dd hh24:mi:ss');
select to_timestamp('2020-04-08 10:26:09','yyyy-MM-DD hh24:mi:ss');
```

![image-20200506204224104](https://raw.githubusercontent.com/renyang1/typora/master/image-20200506204224104.png)

### 时间转为时间戳

​	新纪元时间 Epoch 是以 1970-01-01 00:00:00 格林威治为标准的时间，将目标时间与 1970-01-01 00:00:00时间的差值以秒来计算。

```sql
-- 精确到秒
select floor(extract(epoch from now()));
```

![image-20200507113514908](https://raw.githubusercontent.com/renyang1/typora/master/image-20200507113514908.png)

```sql
-- 精确到秒的小数
select extract(epoch from now());
```

![image-20200506211505013](https://raw.githubusercontent.com/renyang1/typora/master/image-20200506211505013.png)

```sql
-- 精确到毫秒
select floor(extract(epoch from((
    current_timestamp - timestamp '1970-01-01 00:00:00')*1000)));
```

![image-20200506211520920](https://raw.githubusercontent.com/renyang1/typora/master/image-20200506211520920.png)

### 时间戳转时间

```sql
-- 精确到秒的时间戳转为时间
SELECT to_timestamp(1586312769);
```

![image-20200506203333682](https://raw.githubusercontent.com/renyang1/typora/master/image-20200506203333682.png)

```sql
-- 精确到毫秒的时间戳转为时间
SELECT to_timestamp(1588822897000/1000);
```

#### 使用EXTRACT()函数截取时间

```sql
SELECT EXTRACT(year from now()), EXTRACT(month from now()), EXTRACT(day from now());
```

![image-20200506205133701](https://raw.githubusercontent.com/renyang1/typora/master/image-20200506205133701.png)

### 时间计算

#### 获取当前时间前一天

```sql
SELECT now() - interval '1d';
-- interval可以不用写，但后面只能跟+，即若想前推的话需要+一个负数
SELECT now() + '-1 d';
-- 十分钟后
select now() + '10 m';
```

**interval可以不写，其值可以是：**

| **Abbreviation** | **Meaning**                |
| ---------------- | -------------------------- |
| Y                | Years                      |
| M                | Months (in the date part)  |
| W                | Weeks                      |
| D                | Days                       |
| H                | Hours                      |
| M                | Minutes (in the time part) |
| S                | Seconds                    |

