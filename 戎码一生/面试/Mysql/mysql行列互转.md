### MySql行列互转

#### 建表语句
```
create table t_score(
id int primary key auto_increment,
name varchar(20) not null,  #名字
Subject varchar(10) not null, #科目
Fraction double default 0  #分数
);
```
#### 添加数据
```
INSERT INTO `t_score`(name,Subject,Fraction) VALUES
         ('王海', '语文', 86),
        ('王海', '数学', 83),
        ('王海', '英语', 93),
        ('陶俊', '语文', 88),
        ('陶俊', '数学', 84),
        ('陶俊', '英语', 94),
        ('刘可', '语文', 80),
        ('刘可', '数学', 86),
        ('刘可', '英语', 88),
        ('李春', '语文', 89),
        ('李春', '数学', 80),
        ('李春', '英语', 87);
```
#### 表中数据

![1565667230009](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\1565667230009.png)

####  需要的查询结果

![1565668101922](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\1565668101922.png)

#### 实现方式：

##### 方式一：使用if

```
	SELECT name,
			SUM(IF(SUBJECT = '语文', fraction,0)) as '语文',
			SUM(IF(SUBJECT = '数学', fraction,0)) as '数学',
			SUM(IF(SUBJECT = '英语', fraction,0)) as '英语'
			FROM t_score
			GROUP BY `name`
```

##### 方式二：使用case when

```
SELECT name,
			SUM(CASE WHEN SUBJECT = '语文' THEN fraction end) as '语文',
			SUM(CASE WHEN SUBJECT = '数学' THEN fraction end) as '数学',
			SUM(CASE WHEN SUBJECT = '英语' THEN fraction end) as '英语'
			FROM t_score
			GROUP BY `name`;
```









