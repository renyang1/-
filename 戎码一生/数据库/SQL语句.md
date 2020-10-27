# SQL语句

 **SQL语言分为3种: DDL, DML, DCL**

​	**DML**（data manipulation language）是数据操纵语言：它们是SELECT、UPDATE、INSERT、DELETE，就象它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言。

​	**DDL**（data definition language）是数据定义语言：DDL比DML要多，主要的命令有CREATE、ALTER、DROP等，DDL主要是用在定义或改变表（TABLE）的结构，数据类型，表之间的链接和约束等初始化工作上，他们大多在建立表时使用。
​	**DCL**（DataControlLanguage）是数据库控制语言：是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。

### DDL语句

#### 表结构



#### 索引

```sql
-- 查看索引
-- mysql
show index from finance_order_item;
-- pgsql
select * from pg_indexes WHERE tablename = 'course_mini_class_student';

-- 创建索引
CREATE index idx_order_no on finance_order_item(order_no);

-- 创建联合唯一索引
CREATE unique index idx_student_mini_class on course_mini_class_student(student_no,mini_class_no);

```



