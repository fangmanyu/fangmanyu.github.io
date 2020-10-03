### NoSQL特点
- 易扩展
- 大数据量,高性能
- 灵活的数据模型
- 高可用

---
### Redis应用场景
- 缓存
- 任务队列
- 应用排行榜
- 网站访问统计
- 数据过期处理
- 分布式集群架构中的Session分离

---
- 开启Redis服务
```
redis-server.exe redis-windows.conf
```

- Redis安全

查看客户端密码。如果为空则表示客户端没有设置密码
```
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) ""
```
设置密码
```
127.0.0.1:6379> config set requirepass [password]
OK
```
密码设置后连接Redis客户端就需要进行密码验证,
否则会出现错误提示```(error) NOAUTH Authentication required.```
```
127.0.0.1:6379> auth [password]
OK
```

- 连接客户端
```
D:\Redis\Redis-x64-3.2.100>redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> auth [password]
OK
```

---

### 数据类型
```string```, ```hash```, ```list```, ```set```, ```zset```(sorted set: 有序集合)

#### 通用Key操作
- __keys__ 通过匹配获取key
```
127.0.0.1:6379> keys *
 1) "mysore"
 2) "mylist"
 3) "myset1"
 4) "myset"
 5) "mylish"
 6) "db"
 7) "info2"
 8) "info"
 9) "naem"
10) "num3"
11) "myset2"
12) "name"
13) "num2"
14) "mysort"
15) "age"
16) "num"
17) "myset3"
127.0.0.1:6379> keys my*
1) "mysore"
2) "mylist"
3) "myset1"
4) "myset"
5) "mylish"
6) "myset2"
7) "mysort"
8) "myset3"
```

- __select__ 切换数据库
```
127.0.0.1:6379> select 1
OK
```

- __type__ 获取key的类型
```
127.0.0.1:6379> type db
list
127.0.0.1:6379> type name
string
```

- __expire__ 设置key有效时间(秒)
```
127.0.0.1:6379> expire db 100
(integer) 1
```
当key过期时,Redis会自动删除该key

- __ttl__ 查看key的有效时间
```
127.0.0.1:6379> ttl db
(integer) 91
```

- __exists__ 查看key是否存在
```
127.0.0.1:6379> exists db
(integer) 0
```

- __del__ 删除key

```shell
127.0.0.1:6379> del "j:start_urls" "jobbole:requests" "jobbole2:start_urls"
(integer) 3
```


---

#### String(字符串)
string类型是Redis最基本的数据类型，一个key对应一个value，value的最大存储空间为512MB。

string类型是二进制安全的，可以包含任何数据,如:图片或序列化的对象。

##### string命令
- **set** 设置键和值
```
127.0.0.1:6379> set name fmy
OK
```

- **get** 获取值
```
127.0.0.1:6379> get name
"fmy"
127.0.0.1:6379> get age
(nil)
```
返回```(nil)``` 说明```age```不存在

- **getset** 返回原来的值并设置新的值

```
127.0.0.1:6379> getset name fjy
"fmy"
```

- **incr** 将key自增1
```
127.0.0.1:6379> incr num
(integer) 1
```
如果key不存在,则将key初始设为0并进行自增

- **incrby** key增加一个增量值
```
127.0.0.1:6379> incrby num 3
(integer) 4
```

- **decr** 将key自减1(与```incr```相同)
```
127.0.0.1:6379> decr num2
(integer) -1
```

- **decrby** key减少一个增量值
```
127.0.0.1:6379> decrby num3 3
(integer) -3
```

- **append** 追加字符串,返回字符串的总长度
```
127.0.0.1:6379> append name " love"
(integer) 8
```

---

#### Hash
Redis的hash是一个string类型的field和value的映射表,比较适合存储对象.

##### hash 命令
- **hset** 设置一个字段和值
```
127.0.0.1:6379> hset info name fmy
(integer) 1
```

- **hmset** 设置多个字段和值

```
127.0.0.1:6379> hmset info name "fmy" age 21 school "ndxy"
OK
```

- **hgetall** 获取指定key所有的字段和值

```
127.0.0.1:6379> hgetall info
1) "name"
2) "fmy"
3) "age"
4) "21"
5) "school"
6) "ndxy"
```

- **hexists** 指定key中的字段是否存在

```
127.0.0.1:6379> hexists info name
(integer) 1
```

- **hdel** 删除指定key中的字段

```
127.0.0.1:6379> hdel info school
(integer) 1
```

- **hget** 获取指定key的字段值

```
127.0.0.1:6379> hget info age
"21"
```

- **hmget** 获取所有给定字段的值
 
```
127.0.0.1:6379> hmget info age name school
1) "21"
2) "fmy"
3) (nil)
```

- **hkeys** 获取指定key中所有字段名称

```
127.0.0.1:6379> hkeys info
1) "name"
2) "age"
```

- **hlen** 获取指定key中所有字段的数量

```
127.0.0.1:6379> hlen info
(integer) 2
```

- **hset** 设置key中指定字段的值

```
127.0.0.1:6379> hset info name asd
(integer) 0
127.0.0.1:6379> hset info school aaa
(integer) 1
```
返回1时表示添加字段;返回0时表示修改字段

- **hsetnx** 当key中字段不存在时添加字段的值

```
127.0.0.1:6379> hsetnx info school aaa
(integer) 1
```

- **hvals** 获取key中所有的值

```
127.0.0.1:6379> hvals info
1) "fmy"
2) "21"
3) "play"
4) "aaa"
```

-  **hscan key cursor [MATCH pattern] [COUNT count]** 迭代哈希表中的键值对

```
127.0.0.1:6379> hscan info 0 count 100
1) "0"
2) 1) "name"
   2) "fmy"
   3) "age"
   4) "21"
   5) "hobby"
   6) "play"
   7) "school"
   8) "aaa"
```

---

#### List
列表是简单的字符串列表,按照顺序排序。可以从列表的头部或尾部添加元素(相当于队列)

##### list 命令
- **lpush/rpush** 将元素依次添加到列表的头部(左侧)/尾部(右侧),返回列表长度

```
127.0.0.1:6379> lpush mylish a b c d e f g
(integer) 7
127.0.0.1:6379> lpush mylist h
(integer) 1
```

- **lpushx/rpushx** 当列表存在时在头部/尾部插入元素
```
127.0.0.1:6379> lpushx mylist 1
(integer) 0
127.0.0.1:6379> lrange mylist 0 -1
(empty list or set)
```

- **lrange** 获取列表元素(负数表示倒数)
```
127.0.0.1:6379> lrange mylish 0 -1
1) "g"
2) "f"
3) "e"
4) "d"
5) "c"
6) "b"
7) "a"
```

- **lpop/rpop** 左/右弹出元素
```
127.0.0.1:6379> lpop mylist
"h"
127.0.0.1:6379> rpop mylish
"a"
```

- **llen** 获取列表长度
```
127.0.0.1:6379> llen mylish
(integer) 6
```

- **lrem** 删除列表元素
```
127.0.0.1:6379> lpush mylist 1 2 3
(integer) 3
127.0.0.1:6379> lpush mylist 1 2 3
(integer) 6
127.0.0.1:6379> lpush mylist 1 2 3
(integer) 9
127.0.0.1:6379> lrange mylist 0 -1
1) "3"
2) "2"
3) "1"
4) "3"
5) "2"
6) "1"
7) "3"
8) "2"
9) "1"
127.0.0.1:6379> lrem mylist 1 3
(integer) 1
127.0.0.1:6379> lrem mylist -1 3
(integer) 1
127.0.0.1:6379> lrange mylist 0 -1
1) "2"
2) "1"
3) "3"
4) "2"
5) "1"
6) "2"
7) "1"
```
参数```count```: count>0,从头部删除count个指定值;count&lt;0,从尾部删除count个指定值;count=0,删除全部指定值

- **lset** 在列表下标位置(从0开始)插入元素
```
127.0.0.1:6379> lrange mylish 0 -1
1) "g"
2) "f"
3) "e"
4) "d"
5) "c"
6) "b"
127.0.0.1:6379> lset mylish 2 qwe
OK
127.0.0.1:6379> lrange mylish 0 -1
1) "g"
2) "f"
3) "qwe"
4) "d"
5) "c"
6) "b"
```

- **linsert** 在元素之前/之后插入元素
```
127.0.0.1:6379> linsert mylish before qwe 111
(integer) 7
127.0.0.1:6379> linsert mylish after qwe 222
(integer) 8
127.0.0.1:6379> lrange mylish 0 -1
1) "g"
2) "f"
3) "111"
4) "qwe"
5) "222"
6) "d"
7) "c"
8) "b"
```

#### Set
set集合中不允许出现重复的元素,是一个无序集合.

##### 使用场景
- 跟踪一些唯一性数据
- 用于维护数据对象之间的关联关系

##### set 命令
- **sadd** 添加集合元素(不可重复)
```
127.0.0.1:6379> sadd myset a b c d e
(integer) 5
127.0.0.1:6379> sadd myset a
(integer) 0
```

- **srem** 删除集合元素
```
127.0.0.1:6379> srem myset a
(integer) 1
```

- **smembers** 查看集合中所有元素
```
127.0.0.1:6379> smembers myset
1) "d"
2) "c"
3) "e"
4) "b"
```

- **sismember** 查看元素是否在集合中
```
127.0.0.1:6379> sismember myset a
(integer) 0
127.0.0.1:6379> sismember myset b
(integer) 1
```
存在返回1;否则返回0

- **sdiff** 返回集合的差集
```
127.0.0.1:6379> sadd myset1 a b c
(integer) 3
127.0.0.1:6379> sadd myset2 d e
(integer) 2
127.0.0.1:6379> sdiff myset myset1
1) "e"
2) "d"
127.0.0.1:6379> sdiff myset myset2
1) "c"
2) "b"
127.0.0.1:6379> sdiff myset myset1 myset2
(empty list or set)
```

- **sinter** 返回集合的交集
```
127.0.0.1:6379> sinter myset myset2
1) "e"
2) "d"
```

- **sunion** 返回集合的并集
```
127.0.0.1:6379> sunion myset myset1
1) "b"
2) "d"
3) "c"
4) "e"
5) "a"
```

- **sdiffstore/sinterstore/sunionstore** 将差集/交集/并集结果存储到新的集合中
```
127.0.0.1:6379> sdiffstore myset3 myset myset1
(integer) 2
127.0.0.1:6379> smembers myset3
1) "e"
2) "d"
127.0.0.1:6379> sinterstore myset3 myset myset2
(integer) 2
127.0.0.1:6379> smembers myset3
1) "e"
2) "d"
127.0.0.1:6379> sunionstore myset3 myset myset1
(integer) 5
127.0.0.1:6379> smembers myset3
1) "b"
2) "d"
3) "c"
4) "e"
5) "a"
```

- **scard** 返回集合元素的个数
```
127.0.0.1:6379> scard myset
(integer) 4
```

- **srandmember** 随机获取集合中的元素
```
127.0.0.1:6379> srandmember myset
"c"
127.0.0.1:6379> srandmember myset
"b"
127.0.0.1:6379> srandmember myset 2
1) "b"
2) "d"
```
```count``` 参数表示随机返回元素的个数,默认为1

---

#### Sorted-Set
Sorted-Set中的成员在集合中的位置是有序的.每一个成员都有唯一的一个分数;成员不可重复,但分数可以重复

##### Sorted-Set使用场景
- 大型在线游戏积分排行榜
- 构建索引数据

##### Sorted-Set 指令
-  __sadd__  添加有序集合元素
```
127.0.0.1:6379> zadd mysort 10 a 20 b 30 c
(integer) 3
127.0.0.1:6379> zadd mysort 40 a
(integer) 0
```
返回0表示修改元素

- __zscore__ 获取元素分数
```
127.0.0.1:6379> zscore mysort a
"40"
```

- __zcard__ 获取集合元素个数
```
127.0.0.1:6379> zcard mysort
(integer) 3
```

- __zcount__ 获取在分数范围内的元素个数
```
127.0.0.1:6379> zcount mysore 40 52
(integer) 4
```

- __zrem__ 删除集合元素
```
127.0.0.1:6379> zrem mysort a b
(integer) 2
```

- __zremrangebyrank__ 按照范围删除元素
```
127.0.0.1:6379> zremrangebyrank mysort 0 2
(integer) 3
127.0.0.1:6379> zrange mysort 0 -1 withscores
1) "d"
2) "50"
3) "e"
4) "51"
5) "f"
6) "52"
```
按照升序排名删除元素

- __zremrangebyscore__ 按照分数范围删除元素
```
127.0.0.1:6379> zremrangebyscore mysort 50 51
(integer) 2
127.0.0.1:6379> zrange mysort 0 -1 withscores
1) "f"
2) "52"
```

- __zrange__ 升序获取元素
```
127.0.0.1:6379> zrange mysort 0 -1
1) "c"
2) "h"
3) "g"
4) "d"
5) "e"
6) "f"
127.0.0.1:6379> zrange mysort 0 -1 withscores
 1) "c"
 2) "30"
 3) "h"
 4) "40"
 5) "g"
 6) "41"
 7) "d"
 8) "50"
 9) "e"
10) "51"
11) "f"
12) "52"
```
添加```withscores```参数可以获取分数 

- __zrangebyscore__ 按照分数范围查找元素
```
127.0.0.1:6379> zadd mysore 30 a 40 b 41 c 42 d 51 e 60 g 70 q
(integer) 7
127.0.0.1:6379> zrangebyscore mysore 40 52 withscores
1) "b"
2) "40"
3) "c"
4) "41"
5) "d"
6) "42"
7) "e"
8) "51"
127.0.0.1:6379> zrangebyscore mysore 40 52 limit 0 2
1) "b"
2) "c"
```

- __zrevrange__ 降序获取元素
```
127.0.0.1:6379> zrevrange mysort 0 -1
1) "f"
2) "e"
3) "d"
4) "g"
5) "h"
6) "c"
```

---

### Redis特性
#### 多数据库
Redis最多支持16个数据库,默认使用第0个数据库.

- __select__  切换数据库
```
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *
(empty list or set)
```

- __move__ 移动key到其他数据库
```
127.0.0.1:6379> move num2 1
(integer) 1
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *
1) "num2"
```

#### 支持事务
Redis支持事务.```multi```开启事务. ```exec```提交事务.```discard```取消事务

>单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。
事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

比如:
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a 1
QUEUED
127.0.0.1:6379> set b 2
QUEUED
127.0.0.1:6379> set c 3
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) OK
```
如果```set b 2``` 出现错误,则```set a 1```不会回滚,```set c 3```也还会继续执行.

---

### Redis 持久化 
[参考文章](https://mp.weixin.qq.com/s?__biz=MzI4NTA1MDEwNg==&mid=2650769300&idx=1&sn=49a11efa1a6ee605fceaddf240a55c40&chksm=f3f93201c48ebb175fa76053d95e315b621485b0e65e42d8b41fe91b8f859c9278f3adec7ca9&mpshare=1&scene=23&srcid=0731SR4C94CRM0Mljym0oEI3%23rd)
#### RDB
RDB持久化是将当前进程中的数据生成快照保存到硬盘(因此也称作快照持久化),保存的文件后缀名rdb;当Redis重新启动时,可以读取快照文件恢复数据.

##### 触发方式
- 手动触发
- 自动触发

###### 手动触发
1. save __不推荐使用__
```
127.0.0.1:6379> save
OK
```
```save```命令会阻塞Redis服务器进程,直到RDB文件创建完毕为止.
在Redis服务器阻塞期间,服务器不能处理任何命令请求.

2. __bgsave__
```
127.0.0.1:6379> bgsave
Background saving started
```
```bgsave```命令会创建一个子进程,由子进程负责创建RDB文件,父进程(主进程)则继续处理请求

######  自动触发
在redis.config(windows下是redis.windows.config)文件中设置触发条件:

__save m n__:  在m秒内至少有n个key被改动则触发(使用```bgsave```)
```
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```

设置RDB名称
```
# The filename where to dump the DB
dbfilename dump.rdb
```

设置保存路径
```
# Note that you must specify a directory here, not a file name.
dir ./
```

####  AOF
RDB是将进程数据写入文件,而AOF持久化(Append Only File持久化),
则是将Redis执行的每次写命令记录到单独的日志文件;
当Redis重启时再次执行AOF文件中的命令来恢复数据.

AOF的实时性更好,因此已成为主流的持久化方法.

##### 开启AOF
Redis默认开启RDB,关闭AOF.开启AOF,需要在配置文件:

```
appendonly no
```

设置成 ```appendonly yes```。
开启AOF后，Redis会优先根据AOF恢复文件，只有当AOF被关闭时才会用RDB文件。

##### 执行流程
AOF记录Redis每一条写记录,因此AOF不需要触发.

AOF的执行流程:
- 命令追加(append):将Redis的写命;令追加到缓冲区 ```aof_buf``` 
- 文件写入(wirte)和文件同步(sync):根据不同的同步策略将```aof_buf```中的内容同步到硬盘中
- 文件重写(rewirte):定期重写AOF文件,达到压缩的目的

1. **命令追加(append)**:
Redis先将命令追加到缓冲区,而不是直接写入文件(避免每次命令直接写入硬盘,使硬盘IO成为Redis负载的瓶颈)

2. **文件写入(wirte)和文件同步(sync)**:
为了提高文件写入效率,用户在调用write函数时写入文件时,操作系统通常先将数据写入缓冲区,
当缓冲区满或达到一定时间限制后才会将缓冲区内容写入文件中.这样提高了效率,可是造成了一定的安全隐患:
出现突然事件导致计算机宕机,缓冲区内数据还没写入文件,造成缓冲数据丢失.
因此系统提供了fsync.fdatasync等同步方法,可以强制让缓冲区数据写入文件中.

    AOF缓存区的同步文件策略在配置文件中设置:
    ```
    # appendfsync always
    appendfsync everysec
    # appendfsync no
    ```
    各策略含义:
    -  ```always```: 命令写入aof_buf后立即调用系统fsync操作同步到AOF文件
    -  ```everysec```: fsync同步文件操作通过专门线程每秒调用一次(默认)
    -  ```no```: 不使用fsync同步方法;同步由操作系统负责,不过同步时间不确定,数据安全无法保证
    
3. **文件重写(rewirte)**:
文件重写是指定期重写AOF文件,减小AOF文件体积.

    减小体积原因:
    * 过期数据不再写入文件
    * 无效命令不再写入文件:key被重复赋值、key被删除等
    * 多条命令可以合并为一个：如sadd a 1, sadd a 2...可以合并为sadd a 1 2...
    
    文件重写触发
    - 手动触发：调用```bgrewriteaof```命令，与```bgsave```类似。
    - 自动触发：根据```auto-aof-rewrite-min-size```和```auto-aof-rewrite-percentage```参数，以及```aof_current_size```和```aof_base_size```状态确定触发时机。

---

## java中使用Redis
Redis Java官方推荐使用Jedis。
pom.xml文件中引用Jedis.
```
<!--Jedis-->
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
</dependency>
```

在连接中如果出现连接超时的错误
```
Exception in thread "main" redis.clients.jedis.exceptions.JedisConnectionException: java.net.SocketTimeoutException: connect timed out
	at redis.clients.jedis.Connection.connect(Connection.java:207)
	at redis.clients.jedis.BinaryClient.connect(BinaryClient.java:93)
	at redis.clients.jedis.Connection.sendCommand(Connection.java:126)
	at redis.clients.jedis.BinaryClient.set(BinaryClient.java:110)
	at redis.clients.jedis.Client.set(Client.java:47)
	at redis.clients.jedis.Jedis.set(Jedis.java:120)
	at xin.stxkfzx.jedis.Demo.test1(Demo.java:22)
	at xin.stxkfzx.jedis.Demo.main(Demo.java:16)
Caused by: java.net.SocketTimeoutException: connect timed out
	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
	at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
	at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
	at java.net.Socket.connect(Socket.java:589)
	at redis.clients.jedis.Connection.connect(Connection.java:184)
	... 7 more
```
- 可能是服务器防火墙没有开放Redis的端口号.
Ubuntu防火墙设置推荐使用:```ufw```

```
root@ubuntu:/lib# ufw default deny
```
开启ufw(默认不激活)

```
root@ubuntu:/lib# ufw status
状态： 激活
```
查看防火墙状态

```
root@ubuntu:/lib# ufw allow 6379/tcp
规则已添加
规则已添加 (v6)
```
添加Redis端口

- Redis配置文件中没有设置远程连接

```
# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all the network interfaces available on the server.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only into
# the IPv4 lookback interface address (this means Redis will be able to
# accept connections only from clients running into the same computer it
# is running).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1 ::1
```
可以添加远程Ip
```
bind 10.0.0.109
```














