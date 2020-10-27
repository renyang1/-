## SQL常用函数

#### lpad(str, length, padstr)

​	对字符串左边进行某类字符自动填充，即不足某一长度，则在左边自动补上指定的字符串，直至达到指定长度，可同时指定多个自动填充的字符。

str：目标字符串

length: 指定长度

padstr: 填充的字符串

```mysql
-- id类型为int，在mysql中会自动处理
SELECT lpad(id,4,'abcde') from a;
```

```sql
-- pgsql中需要将int转为字符串
SELECT lpad(id::VARCHAR, 12, '0') FROM course_mini_class;
```

#### 字符串聚合连接

​	**group_concat**:mysql

​	**string_agg**:pgsql

```mysql
SELECT group_concat(id ORDER BY id DESC separator '|') FROM a;
```

```sql
SELECT
string_agg('''' || id || '''' ,',' ORDER BY id desc)
FROM base_user
```

​	ORDER BY非必须

