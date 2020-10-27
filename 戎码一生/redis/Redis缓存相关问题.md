## Redis缓存相关问题

### 缓存穿透

#### 什么是缓存穿透?

​	当用户在查询一条数据的时候，而此时数据库和缓存却没有关于这条数据的任何记录，而这条数据在缓存中没找到就会向数据库请求获取数据。它拿不到数据时，是会一直查询数据库，这样会对数据库的访问造成很大的压力。

​	如：用户查询一个 id = -1 的商品信息，一般数据库 id 值都是从 1 开始自增，很明显这条信息是不在数据库中，当没有信息返回时，会一直向数据库查询，给当前数据库的造成很大的访问压力。

#### 解决方案

1. **缓存空对象**

   ​	缓存空对象它就是指一个请求发送过来，如果此时缓存中和数据库都不存在这个请求所要查询的相关信息，那么数据库就会返回一个空对象，并将这个空对象和请求关联起来存到缓存中并设置一个相对较短的过期时间，当下次还是这个请求过来的时候，这时缓存就会命中，就直接从缓存中返回这个空对象，这样可以减少访问数据库的压力，提高当前数据库的访问性能。

   ![image-20201009183012912](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201009183012912.png)

2. **布隆过滤器**

   布隆过滤器的原理是在你存入数据的时候，会通过散列函数将它映射为一个位数组中的K个点，同时把他们置为1。

   这样当用户再次来查询A，而A在布隆过滤器值为0，直接返回，就不会产生击穿请求打到DB了。

### 缓存击穿

#### 什么是缓存击穿？

​	单个key在缓存的过期时间失效的时候或者这是个冷门key时，这时候突然有大量有关这个key的访问请求，这样会导致大并发请求直接穿透缓存，请求数据库，瞬间对数据库的访问压力增大。归纳起来以前两个原因：

1. 一个“冷门”key，突然被大量用户请求访问。
2. 一个“热门”key，在缓存中时间恰好过期，这时有大量用户来进行访问。

![image-20201009183847027](C:\Users\renyang\AppData\Roaming\Typora\typora-user-images\image-20201009183847027.png)

#### 解决方案

1. 设置热点数据永不过期
2. 加锁更新，比如请求查询A，发现缓存中没有，对A这个key加锁，同时去数据库查询数据，写入缓存，再返回给用户，这样后面的请求就可以从缓存中拿到数据了。

```java
static Lock reenLock = new ReentrantLock();
 
    public List<String> getData04() throws InterruptedException {
        List<String> result = new ArrayList<String>();
        // 从缓存读取数据
        result = getDataFromCache();
        if (result.isEmpty()) {
            if (reenLock.tryLock()) {
                try {
                    System.out.println("我拿到锁了,从DB获取数据库后写入缓存");
                    // 从数据库查询数据
                    result = getDataFromDB();
                    // 将查询到的数据写入缓存
                    setDataToCache(result);
                } finally {
                    reenLock.unlock();// 释放锁
                }
 
            } else {
                result = getDataFromCache();// 先查一下缓存
                if (result.isEmpty()) {
                    System.out.println("我没拿到锁,缓存也没数据,先小憩一下");
                    Thread.sleep(100);// 小憩一会儿
                    return getData04();// 重试
                }
            }
        }
        return result;
    }
```



### 缓存雪崩

​	缓存雪崩是指在某一个时间段内，缓存**集中过期**失效，如果这个时间段内有大量请求，而查询数据量巨大，所有的请求都会达到存储层，存储层的调用量会暴增，引起数据库压力过大甚至宕机。

#### 原因：

1. Redis突然宕机
2. 大部分数据失效

#### 解决方案

1. **redis高可用**
2. **限流降级**
3. **不同的过期时间**