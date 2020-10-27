## Redis常用命令

### key

```bash
select 0 选择指定库
keys(pattern) 返回满足给定pattern的所有key,key *返回所有数据
randomkey	随机返回一个key   
rename(oldkey newkey)	重命名key  
exists(key)	确认一个key是否存在
del(key)	删除指定key
type(key)   返回key值类型
expire(key seconds)	设定一个key的活动时间（s）    
ttl(key) 获得一个key的活动时间   
dbsize:	返回当前数据库中key的数目    
flushdb	清除当前选择数据库中的所有key
flushall	清除所有数据库中的所有key    
```

### String

​	 Redis的字符串是动态字符串，是可以修改的字符串，内部表示就是一个字符数组，结构类似Java中的ArrayList,采用预分配冗余空间的方式来减少内存的频繁分配。字符串分配的实际空间一般要高于实际字符串长度len。当字符串长度小于1MB时，扩容都是加倍现有空间。如果字符串长度超过1MB,扩容时一次只会多扩1MB的空间。

```bash
-- sting的插入赋值为覆盖式，即只要key已存在的情况下能插入成功的话，就是使用新的value覆盖旧的value
set(key value)	给名为key的string赋值value
set key value [EX seconds|PX milliseconds] [NX|XX]	
	给名为key的string赋值为value, ex、px、nx、xx均为可选参数
	EX seconds：将键的过期时间设置为seconds秒。执行SET key value EX seconds的效果等同于执行SETEX key seconds value
	PX milliseconds：将键的过期时间设置为 milliseconds 毫秒。 执行 SET key value PX milliseconds 的效果等同于执行 						 SETEX key milliseconds value
	NX ： 只在键不存在时，才对键进行设置操作。执行 SET key value NX 的效果等同于执行 SETNX key value
	XX ： 只在键已经存在时，才对键进行设置操作
setnx(key value)	不存在就插入（not exist）
setex key seconds value	插入并设置过期时间
mset(key1 value key2 value...)	批量设置
mset(key1 value key2 value)	批量设置，所有key都不存在才插入成功
get(key)	返回数据库中名称为key的value
getset(name value)	设置值，返回旧值
mget(key1 key2 key3...) 获取多个string的value
incr(key)	对名称为key的string增1操作
incrby(key integer) 对名称为key的string增加integer
decr(key)	对名称为key的string减1操作
decrby(key integer)	对名称为key的string减少integer
append(key value)	名称为key的string的值附加value，若key不存在，会插入值为value的key
substr(key start end)	返回名称为key的string的value字串，start、end均为闭区间
```

### list

​	  **Redis的列表相当于Java里面的LinkedList,由于是链表而不是数组。这意味着list的插入和删除操作非常快，时间复杂度**

**为O(1),但索引定位很慢，时间复杂度为O(n)。当列表弹出了最后一个元素后，该数据结构被自动删除，内存被回收。**

```sql
-- list的添加为增量，若key不存在则是创建key并添加值，存在则是在原有list值的基础上进行增量式添加
rpush(key value1 value2...)	在名称为key的lsit尾部依次添加值
lpush(key value1 value2...)	在名称为key的lsit头部依次添加值
lrange(key start end)	返回名称为key的list中start至end之间的元素，0 -1为返回所有数据
llen(key)	返回名称为key的list的长度
lpop(key)	弹出首元素
rpop(key)	弹出尾元素
ltrim(key start end) 截取名称为key的list(start、end均为闭区间)
lindex(key)	返回指定索引位置的值
lset(key index value)	给名称为key的list的index位置元素赋值
rpoplpush(srckey detkey)	返回并删除名称为srckey的list尾元素，并将该元素添加到名称为dstkey的头部
```

### hash

  Redis里的hash(字典)相当于Java语言里面的HashMap，是无序字典，内部存储了很多键值对。

实现结构上与Java的HashMap也是一样的，都是数组+链表的二维结构。第一维hash的数组发生碰撞时，

就会将碰撞的元素使用链表串接起来。redis字典的值只能是字符串。当hash移除了最后一个元素后，

该数据结构被自动删除，内存被回收。

```bash
hset(key field value field value...)	向名称为key的hash中添加field键
hget(key field)	返回名称为key的hash中field对应的value
hmget(key field1 field2...)	 返回名称为key的hash中field对应的value
hincrby(key field integer) 将名称为key的hash中field的value增加integer
hexist(key field)	名称为key的hash中是否存在名为field的键
hdel(key field)	删除名称为key的hash中名为field的键
hlen(key)	返回名称为key的hash中键的个数
hkeys(key)	返回名称为key的hash中所有的键
hvals(key)	返回名称为key的hash中所有键对应的value
hgetall(key)	返回名称为key的hash中所有键及其对应的value
```

### set

​	  Redis 的集合相当于 Java 语言里面的 HashSet，它内部的键值对是无序的唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。当集合中最后一个元素移除之后，数据结构自动删除，内存被回收。

```bash
sadd(key member1 member2...)	向名称为key的set中添加元素member
smembers(key)	返回名称为key的set的所有元素
spop(key)	随机返回并删除名称为key的set中的一个member
scard(key)	返回名称为key的set的member的个数
srem(key member1 member2...)	删除名称为key的set中的member
smove(srckey dstkey member)	移到集合元素
sismember(key member)	member是否是名称为key的set中的元素
sinter(key1 key2 key3)	求交集
sinterstore(dstkey key1 key2...)	求交集并将交集保存到dstkey的集合，若dstkey已存在则会覆盖已有元素
sunion(key1 key2...) 求并集
sunionstore(dstkey key1 key2...)	求并集并将并集保存到dstkey的集合，若dstkey已存在则会覆盖已有元素
sdiff(key1 key2...)	求差集
sdiffstore(dstkey key1 key2...)	求差集并将差集保存到dstkey的集合，若dstkey已存在则会覆盖已有元素
srandmember(key)	随机返回名称为key的set的一个元素
```

### zset

  它类似于 Java 的 SortedSet 和 HashMap 的结合体，一方面它是一个 set，保证了内部value 的唯一性，另一方面它可以给每个 value 赋予一个 score，代表这个 value 的排序权重。它的内部实现用的是一种叫着「跳跃列表」的数据结构。zset 中最后一个 value 被移除后，数据结构自动删除，内存被回收。

```bash
zadd(key score member score member)	将带有给定分值的成员添加到有序列表里
zscore(key member)	返回成员member的分值
zrem(key member1 member2...) 移除给定的成员
zcard(key)	返回集合包含的成员数量
zcount(key min max)	返回分值介于min和max之间的成员数量，包括min和max在内
zincrby(key integer member)	将member成员的分值加上integer
zrank(key member)	返回成员member在有序集合中的排名，按照成员分值从小到大排列，分值相同按字典顺序，即分值相同排名不一样
zrevrank(key member)	返回成员member在有序集合中的排名，按照成员分值从大到小排列，分值相同按字典顺序，即分值相同排名不一样
zrange(key start end [withscores])	返回有序集合中排名介于start和end之间的成员，若指定了可选的withscores选项，将分值一									  并返回，成员按照分值从小到大排列
zrevrange(key start end [withscores])	返回有序集合中排名介于start和end之间的成员，若指定了可选的withscores选项，将分										值一并返回，成员按照分值从大到小排列
zrangebyscore(key min max [withscores] [limit offset count])	返回有序集合中分值介于min和max之间的所有成员，包括																min和max,按照分值从小到大返回
zrevrangebyscore(key min max [withscores] [limit offset count])	返回有序集合中分值介于min和max之间的所有成员，包括																min和max,按照分值从大到小返回
```

