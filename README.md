# redis-learning
Redis 命令学习笔记
[redis-commands.md](https://github.com/user-attachments/files/22744501/redis-commands.md)

# Redis 数据类型操作大全

[TOC]



## 1、String（字符串）操作

### 基础操作命令

#键管理

KEYS pattern                    # 查找所有符合给定模式的key
TYPE key                        # 返回key所存储的值的类型
DEL key [key...]                # 删除一个或多个key
EXISTS key                      # 检查key是否存在
EXPIRE key seconds              # 为key设置过期时间
TTL key                         # 查看key的剩余过期时间

```redis
# 键管理
127.0.0.1:6379> KEYS *
1) "myhash"
2) "mylist"
127.0.0.1:6379> TYPE mylist
list
127.0.0.1:6379> DEL mylist
(integer) 1
127.0.0.1:6379> EXISTS mylist
(integer) 0
```



#字符串操作

SET key value                   # 设置指定key的值
GET key                         # 获取指定key的值
MSET key value [key value...]   # 同时设置多个key-value对
MGET key [key...]               # 获取所有给定key的值

```redis
# 基础字符串操作
127.0.0.1:6379> SET username "john_doe"
OK
127.0.0.1:6379> GET username
"john_doe"
127.0.0.1:6379> MSET key1 "value1" key2 "value2" key3 "value3"
OK
127.0.0.1:6379> MGET key1 key2 key3
1) "value1"
2) "value2"
3) "value3"
```

GETSET key value                # 设置新值并返回旧值
STRLEN key                      # 返回key所储存的字符串值的长度
APPEND key value                # 将value追加到key原来的值的末尾
GETRANGE key start end          # 返回key中字符串值的子字符
SETRANGE key offset value       # 用value参数覆写给定key所储存的字符串值

```redis
# 字符串操作
127.0.0.1:6379> SET message "hello"
OK
127.0.0.1:6379> APPEND message " world"
(integer) 11
127.0.0.1:6379> GET message
"hello world"
127.0.0.1:6379> STRLEN message
(integer) 11
127.0.0.1:6379> GETRANGE message 0 4
"hello"
127.0.0.1:6379> GETSET message "new message"
"hello world"
```



#数值操作

INCR key                        # 将key中储存的数字值增一
DECR key                        # 将key中储存的数字值减一
INCRBY key increment            # 将key所储存的值加上给定的增量值
DECRBY key decrement            # 将key所储存的值减去给定的减量值
INCRBYFLOAT key increment       # 将key所储存的值加上给定的浮点增量值

```
# 数值操作
127.0.0.1:6379> SET counter 10
OK
127.0.0.1:6379> INCR counter
(integer) 11
127.0.0.1:6379> INCRBY counter 5
(integer) 16
127.0.0.1:6379> DECR counter
(integer) 15
127.0.0.1:6379> DECRBY counter 3
(integer) 12
127.0.0.1:6379> INCRBYFLOAT counter 1.5
"13.5"
```



#条件设置

SETEX key seconds value         # 设置值并指定过期时间(秒)
PSETEX key milliseconds value   # 设置值并指定过期时间(毫秒)
SETNX key value                 # 只有在key不存在时设置key的值

```
# 条件设置
127.0.0.1:6379> SETEX session_id 3600 "user_data"
OK
127.0.0.1:6379> TTL session_id
(integer) 3598
127.0.0.1:6379> SETNX unique_key "value"
(integer) 1
127.0.0.1:6379> SETNX unique_key "new_value"
(integer) 0
```



### 字符串操作注意事项

- `SET` 会覆盖已存在的值
- `GET` 只能用于字符串类型，对其他类型会报 `WRONGTYPE` 错误
- `MSET` 是原子操作，要么全部成功要么全部失败
- `KEYS *` 在生产环境慎用，可能阻塞Redis服务
- 执行数值操作的键必须是整数或浮点数格式
- `SETEX` 和 `PSETEX` 会自动创建key并设置过期时间
- SETNX常用在分布式锁



## 2、List（列表）操作

### 列表操作命令

#插入操作

LPUSH key value [value...]      # 将一个或多个值插入到列表头部
RPUSH key value [value...]      # 将一个或多个值插入到列表尾部
LINSERT key BEFORE|AFTER pivot value  # 在列表的元素前或后插入元素

```redis
127.0.0.1:6379> LPUSH mylist "first"
(integer) 1
127.0.0.1:6379> RPUSH mylist "last"
(integer) 2
127.0.0.1:6379> LPUSH mylist "new_first" "another"
(integer) 4
127.0.0.1:6379> LINSERT mylist BEFORE "first" "before_first"
(integer) 5
```



#查询操作

LRANGE key start stop           # 获取列表指定范围内的元素
LLEN key                        # 获取列表长度
LINDEX key index                # 通过索引获取列表中的元素

```
# 查询操作
127.0.0.1:6379> LRANGE mylist 0 -1
1) "another"
2) "new_first"
3) "before_first"
4) "first"
5) "last"
127.0.0.1:6379> LLEN mylist
(integer) 5
127.0.0.1:6379> LINDEX mylist 0
"another"
```



#弹出操作

LPOP key                        # 移除并获取列表的第一个元素
RPOP key                        # 移除并获取列表的最后一个元素
BLPOP key [key...] timeout      # 阻塞式移出并获取列表的第一个元素
BRPOP key [key...] timeout      # 阻塞式移出并获取列表的最后一个元素

```
# 弹出操作
127.0.0.1:6379> LPOP mylist
"another"
127.0.0.1:6379> RPOP mylist
"last"

# 阻塞操作
# 终端1：
127.0.0.1:6379> BLPOP task_queue 30
(阻塞等待...)

# 终端2：
127.0.0.1:6379> LPUSH task_queue "task_data"
(integer) 1

# 终端1返回：
1) "task_queue"
2) "task_data"
```

#删除操作

LREM key count value            # 移除列表元素
LTRIM key start stop            # 对一个列表进行修剪
LSET key index value            # 通过索引设置列表元素的值

### LREM key count value

**说明：** 根据count的值移除列表中与value相等的元素

**count参数的三种情况：**

- `count > 0`：从表头开始向表尾搜索，移除与value相等的元素，数量为count
- `count < 0`：从表尾开始向表头搜索，移除与value相等的元素，数量为count的绝对值
- `count = 0`：移除表中所有与value相等的元素

**LREM注意事项：**

1. **返回值**：返回实际删除的元素数量，而不是删除后的列表长度
2. **元素不存在**：如果要删除的值不存在，返回0，列表不变
3. **性能考虑**：在长列表中删除元素需要遍历，时间复杂度O(N)
4. **部分删除**：如果count大于实际存在的元素数量，只删除实际存在的数量

```
# 准备测试数据
127.0.0.1:6379> RPUSH mylist A B A C A D A
(integer) 7
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "B"
3) "A"
4) "C"
5) "A"
6) "D"
7) "A"

# count > 0：从左往右删除2个A
127.0.0.1:6379> LREM mylist 2 A
(integer) 2
127.0.0.1:6379> LRANGE mylist 0 -1
1) "B"
2) "C"
3) "A"
4) "D"
5) "A"

# count < 0：从右往左删除1个A
127.0.0.1:6379> LREM mylist -1 A
(integer) 1
127.0.0.1:6379> LRANGE mylist 0 -1
1) "B"
2) "C"
3) "A"
4) "D"

# count = 0：删除所有A
127.0.0.1:6379> LREM mylist 0 A
(integer) 1
127.0.0.1:6379> LRANGE mylist 0 -1
1) "B"
2) "C"
3) "D"
```



### LTRIM key start stop

**说明：** 只保留指定区间内的元素，不在指定区间内的元素都将被删除

**索引规则：**

- 0表示第一个元素，1表示第二个元素，以此类推
- -1表示最后一个元素，-2表示倒数第二个元素，以此类推
- 区间包含start和stop位置的元素

**LTRIM注意事项：**

1. **永久删除**：被修剪掉的元素会永久删除，无法恢复
2. **空列表**：如果start > stop，列表会被清空
3. **索引越界**：如果stop超出列表范围，会截取到列表末尾
4. **常用场景**：常用于限制列表长度，如最新消息列表
5. **性能**：时间复杂度O(N)，N为要删除的元素数量

```
# 准备测试数据
127.0.0.1:6379> RPUSH numbers 1 2 3 4 5 6 7 8 9
(integer) 9
127.0.0.1:6379> LRANGE numbers 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
9) "9"

# 保留前5个元素（索引0-4）
127.0.0.1:6379> LTRIM numbers 0 4
OK
127.0.0.1:6379> LRANGE numbers 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"

# 保留最后3个元素（索引-3到-1）
127.0.0.1:6379> RPUSH numbers 1 2 3 4 5 6 7 8 9
(integer) 9
127.0.0.1:6379> LTRIM numbers -3 -1
OK
127.0.0.1:6379> LRANGE numbers 0 -1
1) "7"
2) "8"
3) "9"

# 保留中间部分元素（索引2-5）
127.0.0.1:6379> RPUSH numbers 1 2 3 4 5 6 7 8 9
(integer) 9
127.0.0.1:6379> LTRIM numbers 2 5
OK
127.0.0.1:6379> LRANGE numbers 0 -1
1) "3"
2) "4"
3) "5"
4) "6"
```



### LSET key index value

**说明：** 通过索引设置列表元素的值（严格来说不是删除，是修改）

```
# 准备测试数据
127.0.0.1:6379> RPUSH mylist A B C D
(integer) 4
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "B"
3) "C"
4) "D"

# 修改指定位置的元素
127.0.0.1:6379> LSET mylist 1 "B_updated"
OK
127.0.0.1:6379> LSET mylist -1 "D_updated"  # 使用负数索引修改最后一个元素
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "A"
2) "B_updated"
3) "C"
4) "D_updated"
```

#其他操作

RPOPLPUSH source destination    # 移除列表的最后一个元素，并将该元素添加到另一个列表并返回



### 列表操作注意事项

- `LPUSH` 和 `RPUSH` 可以一次插入多个元素
- `LRANGE 0 -1` 可以获取列表所有元素
- `LREM count 0 value` 会删除所有匹配的元素
- `BLPOP` 超时时间为0表示无限等待
- 列表为空时 `LPOP`/`RPOP` 返回 nil



## 3、Set（集合）操作

### 添加和删除元素

#添加元素（可批量）

SADD key member1 [member2 member3...]

#删除元素（可批量）

SREM key member1 [member2 member3...]

```
127.0.0.1:6379> SADD tags redis mongodb mysql
(integer) 3
127.0.0.1:6379> SADD tags redis  # 重复添加，不会成功
(integer) 0
127.0.0.1:6379> SREM tags mysql
(integer) 1
```

### 查询操作

SMEMBERS key           # 获取所有元素
SISMEMBER key member   # 判断元素是否存在
SCARD key              # 获取元素数量

```
127.0.0.1:6379> SMEMBERS tags
1) "mongodb"
2) "redis"
127.0.0.1:6379> SISMEMBER tags redis
(integer) 1
127.0.0.1:6379> SISMEMBER tags mysql
(integer) 0
127.0.0.1:6379> SCARD tags
(integer) 2
```

### 随机操作

SRANDMEMBER key [count]    # 随机获取元素（不删除）
SPOP key [count]           # 随机弹出元素（会删除）****

```
127.0.0.1:6379> SADD lottery "user1" "user2" "user3" "user4" "user5"
(integer) 5
127.0.0.1:6379> SRANDMEMBER lottery 2    # 随机查看2个，不删除
1) "user3"
2) "user1"
127.0.0.1:6379> SPOP lottery 2           # 随机抽取2个，会删除
1) "user2"
2) "user5"
127.0.0.1:6379> SMEMBERS lottery         # 剩余用户
1) "user1"
2) "user3"
3) "user4"
```

### 集合运算（核心功能）

#### 差集 (Difference)

SDIFF key1 [key2 key3...]     # 在key1中，但不在其他key中的元素

```
127.0.0.1:6379> SADD set1 A B C D
(integer) 4
127.0.0.1:6379> SADD set2 C D E F
(integer) 4
127.0.0.1:6379> SDIFF set1 set2        # 在set1但不在set2
1) "A"
2) "B"
127.0.0.1:6379> SDIFF set2 set1        # 在set2但不在set1
1) "E"
2) "F"
```

#### 交集 (Intersection)

SINTER key1 [key2 key3...]    # 在所有key中都存在的元素

```
127.0.0.1:6379> SADD user:1:tags sports music tech
(integer) 3
127.0.0.1:6379> SADD user:2:tags music food tech
(integer) 3
127.0.0.1:6379> SINTER user:1:tags user:2:tags    # 共同兴趣
1) "music"
2) "tech"
```

#### 并集 (Union)

SUNION key1 [key2 key3...]    # 所有key中的元素（去重）

```
127.0.0.1:6379> SUNION set1 set2
1) "A"
2) "B"
3) "C"
4) "D"
5) "E"
6) "F"
```

### 存储运算结果

SDIFFSTORE destination key1 [key2...]   # 存储差集结果
SINTERSTORE destination key1 [key2...]  # 存储交集结果
SUNIONSTORE destination key1 [key2...]  # 存储并集结果

```
127.0.0.1:6379> SINTERSTORE common_tags user:1:tags user:2:tags
(integer) 2
127.0.0.1:6379> SMEMBERS common_tags
1) "music"
2) "tech"
```

### 元素移动

SMOVE source destination member    # 移动元素到另一个set

```
127.0.0.1:6379> SADD pending_tasks "task1" "task2" "task3"
(integer) 3
127.0.0.1:6379> SMOVE pending_tasks completed_tasks "task1"
(integer) 1
127.0.0.1:6379> SMEMBERS pending_tasks
1) "task2"
2) "task3"
127.0.0.1:6379> SMEMBERS completed_tasks
1) "task1"
```

## set集合实际应用场景

### 1. 标签系统

```
# 为用户添加标签
SADD user:100:tags "python" "redis" "backend"
SADD user:101:tags "python" "frontend" "vue"

# 查找共同标签
SINTER user:100:tags user:101:tags
1) "python"

# 查找用户独有的标签
SDIFF user:100:tags user:101:tags
1) "redis"
2) "backend"
```

### 2. 好友关系

```
# 添加好友
SADD user:123:friends "user:456" "user:789"
SADD user:456:friends "user:123" "user:999"

# 共同好友
SINTER user:123:friends user:456:friends
1) "user:123"  # 注意：这里应该是相互的，实际需要调整数据结构

# 推荐好友（user:456的好友中，user:123还不认识的）
SDIFF user:456:friends user:123:friends
1) "user:999"
```

### 3. 抽奖系统

```
# 参与用户
SADD lottery:2024 "user1" "user2" "user3" "user4" "user5"

# 随机抽取3个中奖者（不删除）
SRANDMEMBER lottery:2024 3

# 随机抽取3个中奖者（删除，避免重复中奖）
SPOP lottery:2024 3
```

### 4. 黑白名单

```
# 黑名单
SADD blacklist "192.168.1.100" "192.168.1.101"

# 检查IP是否在黑名单
SISMEMBER blacklist "192.168.1.100"
(integer) 1

# 从黑名单移除
SREM blacklist "192.168.1.100"
```

### 重要特性总结

| 特性           | 说明             | 应用场景           |
| :------------- | :--------------- | :----------------- |
| **元素唯一**   | 自动去重         | 标签、投票防重复   |
| **无序存储**   | 没有顺序概念     | 不需要顺序的场景   |
| **集合运算**   | 交集、并集、差集 | 好友推荐、共同兴趣 |
| **随机操作**   | SRANDMEMBER/SPOP | 抽奖、随机推荐     |
| **O(1)复杂度** | 大部分操作很快   | 高性能需求         |

## 注意事项

1. **SMEMBERS** 在集合很大时可能阻塞，考虑使用 **SSCAN**
2. **SPOP** 会删除元素，**SRANDMEMBER** 不会
3. 集合运算在多个大集合时可能较慢
4. 集合最大可存储 2³² - 1 个元素



# 4、Hash（哈希）操作

## 什么是 Hash？

Hash 是 field-value 映射表，适合存储对象

- 类似 Java 的 `HashMap` 或 Go 的 `map[string]interface{}`
- 适合存储：用户信息、商品信息、配置项等

## 基础操作

### 设置和获取

HSET key field value [field value ...] #设置字段值（新版可批量）

HGET key field #获取字段值

HGETALL key #获取所有字段和值

```
127.0.0.1:6379> HSET user:1000 name "Alice" age 30 email "alice@example.com"
(integer) 3
127.0.0.1:6379> HGET user:1000 name
"Alice"
127.0.0.1:6379> HGETALL user:1000
1) "name"
2) "Alice"
3) "age"
4) "30"
5) "email"
6) "alice@example.com"
```

### 批量操作

HMGET key field1 [field2 field3...]    # 批量获取多个字段

#tips:HMSET 已不推荐，直接用 HSET 替代

```
127.0.0.1:6379> HMGET user:1000 name age email
1) "Alice"
2) "30"
3) "alice@example.com"
```

### 删除操作

HDEL key field1 [field2 field3...]    # 删除一个或多个字段

```
127.0.0.1:6379> HDEL user:1000 email
(integer) 1
127.0.0.1:6379> HGETALL user:1000
1) "name"
2) "Alice"
3) "age"
4) "30"
```

## 查询和判断

### 字段查询

HEXISTS key field      # 判断字段是否存在
HLEN key              # 获取字段数量
HKEYS key             # 获取所有字段名
HVALS key             # 获取所有字段值
HSTRLEN key field     # 获取字段值的字符串长度

```
127.0.0.1:6379> HEXISTS user:1000 name
(integer) 1
127.0.0.1:6379> HLEN user:1000
(integer) 2
127.0.0.1:6379> HKEYS user:1000
1) "name"
2) "age"
127.0.0.1:6379> HVALS user:1000
1) "Alice"
2) "30"
127.0.0.1:6379> HSTRLEN user:1000 name
(integer) 5
```

## 数值操作

### 增减操作

HINCRBY key field increment        # 整数字段增加
HINCRBYFLOAT key field increment   # 浮点数字段增加
#递减使用负值
HINCRBY key field -5              # 递减5

```
127.0.0.1:6379> HSET product:1001 price 99.99 stock 50
(integer) 2
127.0.0.1:6379> HINCRBY product:1001 stock -5    # 库存减少5
(integer) 45
127.0.0.1:6379> HINCRBYFLOAT product:1001 price -10.50  # 降价10.5
"89.49"
127.0.0.1:6379> HINCRBY user:1000 login_count 1  # 登录次数+1
(integer) 1
```

## 实际应用场景

### 1. 用户信息存储

```
# 存储用户完整信息
127.0.0.1:6379> HSET user:1001 \
  username "bob" \
  password_hash "xxx" \
  email "bob@example.com" \
  age 25 \
  last_login "2024-01-15" \
  login_count 1

# 部分更新
127.0.0.1:6379> HSET user:1001 last_login "2024-01-20"
127.0.0.1:6379> HINCRBY user:1001 login_count 1

# 获取用户基本信息
127.0.0.1:6379> HMGET user:1001 username email login_count
```

### 2. 商品信息缓存

```
# 缓存商品详情
127.0.0.1:6379> HSET product:5001 \
  name "iPhone 15" \
  price 5999 \
  stock 100 \
  category "phone" \
  brand "Apple" \
  description "Latest iPhone"

# 库存管理
127.0.0.1:6379> HINCRBY product:5001 stock -1    # 售出1件
127.0.0.1:6379> HINCRBY product:5001 stock 10    # 进货10件

# 价格调整
127.0.0.1:6379> HINCRBYFLOAT product:5001 price -500.0  # 降价500
```

### 3. 购物车实现

```
# 添加商品到购物车
127.0.0.1:6379> HSET cart:user123 \
  product:5001 2 \
  product:6001 1 \
  product:7001 3

# 修改商品数量
127.0.0.1:6379> HINCRBY cart:user123 product:5001 1    # 数量+1
127.0.0.1:6379> HINCRBY cart:user123 product:6001 -1   # 数量-1

# 删除商品
127.0.0.1:6379> HDEL cart:user123 product:7001

# 获取购物车所有商品
127.0.0.1:6379> HGETALL cart:user123
```

### 4. 配置信息存储

```
# 系统配置
127.0.0.1:6379> HSET config:app \
  max_connections 1000 \
  timeout 30 \
  debug_mode false \
  log_level "info"

# 获取单个配置项
127.0.0.1:6379> HGET config:app timeout
"30"

# 检查配置是否存在
127.0.0.1:6379> HEXISTS config:app debug_mode
(integer) 1
```

## 高级特性

### 字段遍历（大数据量时使用）

HSCAN key cursor [MATCH pattern] [COUNT count]

````
127.0.0.1:6379> HSCAN user:1000 0 MATCH "name*" COUNT 10
````



### Hash 特性

- **字段操作原子性**：单个字段操作是原子的
- **内存高效**：小哈希很节省内存
- **部分更新**：可以只更新单个字段
- **适合对象存储**：用户信息、商品信息等





# 5、Sorted Set (ZSet)（有序集合）

## 什么是 Sorted Set？

Sorted Set 是 **带分数排序** 的 Set

- 元素唯一，但每个元素有一个分数(score)
- 按分数排序，分数可以重复
- 适合：排行榜、延迟队列、范围查询

## 基础操作

### 添加和查询

ZADD key [NX|XX] [GT|LT] [CH] [INCR] score1 member1 [score2 member2...] #添加元素（可批量）

ZRANGE key start stop [WITHSCORES] #按索引范围查询

ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count] #按分数范围查询

````
127.0.0.1:6379> ZADD leaderboard 1000 "player1" 1500 "player2" 800 "player3"
(integer) 3
127.0.0.1:6379> ZRANGE leaderboard 0 -1 WITHSCORES
1) "player3"
2) "800"
3) "player1"
4) "1000"
5) "player2"
6) "1500"
127.0.0.1:6379> ZRANGEBYSCORE leaderboard 900 1600 WITHSCORES
1) "player1"
2) "1000"
3) "player2"
4) "1500"
````

### 反向操作

#反向范围查询

ZREVRANGE key start stop [WITHSCORES]
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

````
127.0.0.1:6379> ZREVRANGE leaderboard 0 -1 WITHSCORES  # 从高到低
1) "player2"
2) "1500"
3) "player1"
4) "1000"
5) "player3"
6) "800"
````

## 查询操作

### 分数和排名

ZSCORE key member          # 获取元素分数
ZCARD key                  # 获取元素数量
ZCOUNT key min max         # 分数范围内的元素数量
ZRANK key member           # 获取元素排名（从低到高）
ZREVRANK key member        # 获取元素排名（从高到低）

````
127.0.0.1:6379> ZSCORE leaderboard "player1"
"1000"
127.0.0.1:6379> ZCARD leaderboard
(integer) 3
127.0.0.1:6379> ZCOUNT leaderboard 900 1600  # 900-1600分的玩家数量
(integer) 2
127.0.0.1:6379> ZRANK leaderboard "player1"   # 正序排名（从0开始）
(integer) 1
127.0.0.1:6379> ZREVRANK leaderboard "player1" # 逆序排名
(integer) 1
````

## 删除操作

ZREM key member [member...]                    # 删除元素
ZREMRANGEBYRANK key start stop                # 按排名范围删除
ZREMRANGEBYSCORE key min max                  # 按分数范围删除

````
127.0.0.1:6379> ZREM leaderboard "player3"
(integer) 1
127.0.0.1:6379> ZREMRANGEBYRANK leaderboard 0 0  # 删除最后一名
(integer) 1
127.0.0.1:6379> ZREMRANGEBYSCORE leaderboard 0 500  # 删除500分以下的
(integer) 0
````

## 数值操作

ZINCRBY key increment member    # 增加元素分数

````
127.0.0.1:6379> ZINCRBY leaderboard 200 "player1"  # player1增加200分
"1200"
127.0.0.1:6379> ZINCRBY leaderboard -100 "player2" # player2减少100分
"1400"
````

## 集合运算

ZINTERSTORE destination numkeys key [key...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
ZUNIONSTORE destination numkeys key [key...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]

````
127.0.0.1:6379> ZADD game1 100 "player1" 200 "player2"
127.0.0.1:6379> ZADD game2 150 "player1" 180 "player3"
127.0.0.1:6379> ZUNIONSTORE total 2 game1 game2 AGGREGATE SUM
(integer) 3
127.0.0.1:6379> ZRANGE total 0 -1 WITHSCORES
1) "player2"
2) "200"
3) "player3"
4) "180"
5) "player1"
6) "250"  # 100 + 150
````

## 实际应用场景

### 1. 游戏排行榜

````
# 初始化排行榜
127.0.0.1:6379> ZADD leaderboard:game1 \
  1000 "player1" \
  1500 "player2" \
  800 "player3" \
  2000 "player4" \
  1200 "player5"

# 获取前十名
127.0.0.1:6379> ZREVRANGE leaderboard:game1 0 9 WITHSCORES
# 玩家得分增加
127.0.0.1:6379> ZINCRBY leaderboard:game1 300 "player1"
# 查看玩家排名
127.0.0.1:6379> ZREVRANK leaderboard:game1 "player1"
# 查看前10%的玩家
127.0.0.1:6379> ZREVRANGE leaderboard:game1 0 2 WITHSCORES  # 前3名
````

### 2. 延迟队列

````
# 添加延迟任务（时间戳作为分数）
127.0.0.1:6379> ZADD delay_queue \
  1642567800 "task1" \    # 2024-01-19 10:50:00
  1642567860 "task2" \    # 2024-01-19 10:51:00
  1642567920 "task3"      # 2024-01-19 10:52:00

# 获取到期的任务
127.0.0.1:6379> ZRANGEBYSCORE delay_queue 0 1642567860 WITHSCORES
````

### 3. 热度排行榜

````
# 文章热度评分（阅读、点赞、评论加权）
127.0.0.1:6379> ZADD hot_articles \
  156.8 "article:1" \
  89.3 "article:2" \
  203.1 "article:3" \
  45.6 "article:4"

# 获取热门文章（前10）
127.0.0.1:6379> ZREVRANGE hot_articles 0 9 WITHSCORES

# 文章热度更新
127.0.0.1:6379> ZINCRBY hot_articles 10.5 "article:1"  # 阅读量增加
````

### 4. 时间轴数据

````
# 用户动态时间轴
127.0.0.1:6379> ZADD user:123:timeline \
  1642567800 "post:1" \
  1642567860 "post:2" \
  1642567920 "post:3"

# 获取最新动态（按时间倒序）
127.0.0.1:6379> ZREVRANGE user:123:timeline 0 9 WITHSCORES

# 分页获取历史动态
127.0.0.1:6379> ZREVRANGE user:123:timeline 10 19 WITHSCORES
````

## 高级用法

### 带权重的集合运算

````
# 多个数据源计算综合排名
127.0.0.1:6379> ZADD scores:math 80 "student1" 90 "student2"
127.0.0.1:6379> ZADD scores:english 85 "student1" 88 "student2"
127.0.0.1:6379> ZUNIONSTORE total_scores 2 scores:math scores:english WEIGHTS 0.6 0.4
````

### 范围删除应用

````
# 清理过期数据
127.0.0.1:6379> ZREMRANGEBYSCORE user_sessions 0 1642567800  # 清理过期会话

# 保留TopN，删除其余
127.0.0.1:6379> ZREMRANGEBYRANK leaderboard 100 -1  # 只保留前100名
````

### Sorted Set 特性

- **自动排序**：按分数自动维护顺序
- **范围查询**：支持按分数、排名范围查询
- **元素唯一**：成员唯一，分数可重复
- **高性能**：大部分操作 O(log(N)) 复杂度



------

目前只更新五种基本的数据类型的操作，后续会更新Geospatial地理位置、Hyperloglog基数统计、Bitmap位图场景、Redis基本的事务操作等
