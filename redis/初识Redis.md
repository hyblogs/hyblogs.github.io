# 初识 Redis



## 一、什么是Redis?

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。
Redis 与其他 key - value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

## 二、Redis的优点

- 性能十分优越 – Redis支持读的速度大约是110000次/s,写的速度大约是81000次/s 。
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子性 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 其他特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## 三、Redis数据类型

Redis 总共支持 5 种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

下面将对每个类型进行单独描述：

### 1、String字符串类型：

- string 是 redis 最基本的类型，一个 key 对应一个 value。
- string 是二进制安全的。对应的意思是说 Redis 中的 string 可以包含任何数据。比如jpg图片或者序列化的对象等。
- string 类型的值在Redis中最大能存储 512MB。

#### 1.1 常见用法

```
127.0.0.1:6379>SET exampleKey "exampleValue"
OK
127.0.0.1:6379>GET exampleKey
exampleValue
127.0.0.1:6379>
```

在上面的示例中使用了 Redis 的 SET 和 GET 命令。<br/>
其对应的键(key)为： exampleKey，对应的值(value)为：exampleValue 。<br/>
++注：Redis中一个键最大能存储 512MB。++

#### 1.2 理解

你可以理解成与 Memcached 一模一样的类型，一个 key 对应一个 value。<br/>
你还可以当作Java中的Map一样，一个 key 对应着一个 value。

### 2、Hash(哈希)类型：

- 在Redis中 hash 是一个键值(key=>value)对集合。
- Redis的 hash 是一个 string 类型的 field 和 value 的映射表，因此hash 类型特别适合用于存储对象。

#### 2.1 和String的区别

> string 是 一个 key - value 键值对，而 hash 是多个 key - value 键值对。

#### 2.2 常见用法

```
127.0.0.1:6379> HMSET hashKey fieldOne "valueOne" fieldTwo "valueTwo"
OK
127.0.0.1:6379> HGET hashKey fieldOne
"valueOne"
127.0.0.1:6379> HGET hashKey fieldTwo
"valueTwo"
127.0.0.1:6379> HGETALL hashKey
1) "fieldOne"
2) "valueOne"
3) "fieldTwo"
4) "valueTwo"
127.0.0.1:6379> HDEL hashKey fieldOne
(integer) 1
127.0.0.1:6379> HGETALL hashKey
1) "fieldTwo"
2) "valueTwo"
127.0.0.1:6379> 
```

在上面的示例中我们使用了 HMSET, HGET, HGETALL, HDEL 命令:<br/> 

> ++HMSET++ 设置了两个 field=>value 对;<br/> 
> ++HGET++ 获取对应 field 对应的 value;<br/> 
> ++HGETALL++ 获取 hashKey 这个 hash 里面的所有键值对;<br/> 
> ++HDEL++ 命令用于删除hashKey里面的 fieldOne 键值对。

#### 2.3 理解

你可以将 hash 看成一个 key - value 的集合。也可以将其想成一个 hash 对应着多个 string。
![hash.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-31_1605539626039.png)

++注：每个 hash 可以存储 2^32 -1 个键值对（40多亿）。++

### 3、List(列表)类型：

- List 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

#### 3.1 常见用法

```
127.0.0.1:6379> LPUSH listKey valueOne
(integer) 1
127.0.0.1:6379> LPUSH listKey valueTwo
(integer) 2
127.0.0.1:6379> LPUSH listKey valueThree
(integer) 3
127.0.0.1:6379> LRANGE listKey 0 10
1) "valueThree"
2) "valueTwo"
3) "valueOne"
127.0.0.1:6379> LINDEX listKey 0
"valueThree"
127.0.0.1:6379> LPOP listKey
"valueThree"
127.0.0.1:6379> LRANGE listKey 0 10
1) "valueTwo"
2) "valueOne"
127.0.0.1:6379> 
```

在上面的示例中我们使用了 LPUSH, LRANGE, LINDEX, LPOP 命令: <br/>

> ++LPUSH++ 将三个值插入了名为 listKey 的列表当中;<br/> 
> ++LRANGE++ 返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。 其中 0 表示列表的第一个元素， 10 表示列表的第十一个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推<br/> 
> ++LINDEX++ 命令用于通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推;<br/> 
> ++LPOP++ 命令用于移除并返回列表的第一个元素;<br/>

#### 3.2 理解

由上面的示例中我们可以看出 list 就是一个简单的字符串集合，和 Java 中的 list 相差不大，区别就是这里的 list 存放的是字符串。**list 内的元素是可重复的。**
![list.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-23_1605539621224.png)

++注：每个列表最多可存储 2^32(即4294967295) - 1 个元素 (每个列表可存储40多亿个元素)。++

### 4、SET(集合)类型：

- 在 Redis 中的 Set 是 string 类型的无序集合。
- SET 集合是通过哈希表实现的，由此可见 ++添加++，++删除++，++查找++ 的复杂度都是 O(1)。

#### 4.1 常见用法

```
127.0.0.1:6379> SADD setKey redis
(integer) 1
127.0.0.1:6379> SADD setKey memcached
(integer) 1
127.0.0.1:6379> SADD setKey Java
(integer) 1
127.0.0.1:6379> SADD setKey web
(integer) 1
127.0.0.1:6379> SMEMBERS setKey
1) "web"
2) "Java"
3) "memcached"
4) "redis"
127.0.0.1:6379> SADD setKey Java
(integer) 0
127.0.0.1:6379> SMEMBERS setKey
1) "web"
2) "Java"
3) "memcached"
4) "redis"
127.0.0.1:6379> SISMEMBER setKey Java
(integer) 1
127.0.0.1:6379> SISMEMBER setKey python
(integer) 0
127.0.0.1:6379> SCARD setKey
(integer) 4
127.0.0.1:6379> SREM setKey web
(integer) 1
127.0.0.1:6379> SMEMBERS setKey
1) "Java"
2) "memcached"
3) "redis"
127.0.0.1:6379> 
```

在上面的示例中，我们使用了 SADD, SMEMBERS, SISMEMBER, SCARD, SREM 命令：

> ++SADD++ 命令将一个或多个成员元素加入到集合中，已经存在于集合的成员元素将被忽略。假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合，如果元素已经在集合中返回 0。当集合 key 不是集合类型时，返回一个错误。<br/>
> ++SMEMBERS++ 命令返回集合中的所有的成员。 不存在的集合 key 被视为空集合。<br/>
> ++SISMEMBER++ 命令判断成员元素是否是集合的成员。<br/>
> ++SCARD++ 命令返回集合中元素的数量。<br/>
> ++SREM++ 命令用于移除集合中的一个或多个成员元素，不存在的成员元素会被忽略。当 key 不是集合类型，返回一个错误。<br/>

*注意：在 Redis 2.4 版本以前，SREM 只接受单个成员值; SADD 只接受单个成员值。以上实例中 Java 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。*

#### 4.2 理解

redis 的 set 与 java 中的 set 还是有点区别的。<br/>
redis 的 set 是一个 key 对应着多个字符串类型的 value，也是一个字符串类型的集合。<br/>
但是和 redis 的 list 不同的是 set 中的字符串集合元素不能重复，但是 list 可以。<br/>
![set.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-33_1605539628403.png)

++注：集合中最大的成员个数为 2^32 - 1(即每个集合可存储40多亿个成员)。++

### 5、ZSet(sorted set：有序集合)类型：

- zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。
  不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。<br/>
- zset的成员是唯一的,但分数(score)却可以重复。

#### 5.1 常见用法

```
127.0.0.1:6379> ZADD zsetKey 1 redis
(integer) 1
127.0.0.1:6379> ZADD zsetKey 1 redis
(integer) 0
127.0.0.1:6379> ZADD zsetKey 1 java
(integer) 1
127.0.0.1:6379> ZADD zsetKey 2 web
(integer) 1
127.0.0.1:6379> ZADD zsetKey 100 js
(integer) 1
127.0.0.1:6379> ZRANGEBYSCORE zsetKey 0 10
1) "java"
2) "redis"
3) "web"
127.0.0.1:6379> ZRANGE zsetKey 0 -1 WITHSCORES
1) "java"
2) "1"
3) "redis"
4) "1"
5) "web"
6) "2"
7) "js"
8) "100"
127.0.0.1:6379> ZRANGEBYSCORE zsetKey 0 10 WITHSCORES
1) "java"
2) "1"
3) "redis"
4) "1"
5) "web"
6) "2"
127.0.0.1:6379> ZREM zsetKey java
(integer) 1
127.0.0.1:6379> ZREM zsetKey java
(integer) 0
127.0.0.1:6379> ZRANGEBYSCORE zsetKey 0 10 WITHSCORES
1) "redis"
2) "1"
3) "web"
4) "2"
127.0.0.1:6379> ZADD zsetKey 10 redis
(integer) 0
127.0.0.1:6379> ZRANGEBYSCORE zsetKey 0 10 WITHSCORES
1) "web"
2) "2"
3) "redis"
4) "10"
127.0.0.1:6379> 
```

在上面的示例中，我们使用了 ZADD, ZRANGEBYSCORE, ZRANGE - WITHSCORES, ZRANGEBYSCORE - WITHSCORES, ZREM 命令：

> ++ZADD++：添加元素到集合，元素在集合中存在则更新对应score。<br/>
> ++ZRANGEBYSCORE++：返回有序集合中指定分数区间的成员列表。有序集合成员按分数值递增(从小到大)次序排列。具有相同分数值的成员按字典序来排列(该属性是有序集提供的，不需要额外的计算)。默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 左括号(符号：'(') 来使用可选的开区间 (小于或大于)。<br/>
> ++ZRANGE - WITHSCORES++：返回有序集中，指定区间内的成员。其中成员的位置按分数值递增(从小到大)来排序。具有相同分数值的成员按字典序(lexicographical order )来排列。<br/>
> ++ZRANGEBYSCORE - WITHSCORES,++：返回有序集合中指定分数区间的成员列表。有序集成员按分数值递增(从小到大)次序排列。具有相同分数值的成员按字典序来排列(该属性是有序集提供的，不需要额外的计算)。默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 左括号(符号：'(') 来使用可选的开区间 (小于或大于)。<br/>
> ++ZREM++：命令用于移除有序集中的一个或多个成员，不存在的成员将被忽略。当 key 存在但不是有序集类型时，返回一个错误。<br/>

注意： 在 Redis 2.4 版本以前， ZREM 每次只能删除一个元素。

![ZSet.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-29_1605539626037.png)

## 四、总结

| 类型               | 简介                                 | 特性                                                                          | 场景                                                         |
| ---------------- | ---------------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------- |
| String(字符串)      | 二进制安全                              | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M                                       | ---                                                        |
| Hash(字典)         | 键值对集合,即编程语言中的Map类型                 | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) | 存储、读取、修改用户属性                                               |
| List(列表)         | 链表(双向链表)                           | 增删快,提供了操作某一段元素的API                                                          | 1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列。                             |
| Set(集合)          | 哈希表实现,元素不重复                        | 1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作                                   | 1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐。 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 | 数据插入集合时,已经进行天然排序。                                                           | 1、排行榜 2、带权重的消息队。                                           |
