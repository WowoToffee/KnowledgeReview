

#### Redis 键(key) 命令

| 命令                                                         | 描述                                                         | 对应代码                                 |
| :----------------------------------------------------------- | :----------------------------------------------------------- | ---------------------------------------- |
| [Redis Type 命令](https://www.redis.net.cn/order/3543.html)  | 返回 key 所储存的值的类型。                                  | redisTemplate.type(key)                  |
| [Redis PEXPIREAT 命令](https://www.redis.net.cn/order/3533.html) | 设置 key 的过期时间亿以毫秒计。                              |                                          |
| [Redis PEXPIREAT 命令](https://www.redis.net.cn/order/3534.html) | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计           |                                          |
| [Redis Rename 命令](https://www.redis.net.cn/order/3541.html) | 修改 key 的名称                                              | redisTemplate.rename(K oldKey, K newKey) |
| [Redis PERSIST 命令](https://www.redis.net.cn/order/3537.html) | 移除 key 的过期时间，key 将持久保持。                        |                                          |
| [Redis Move 命令](https://www.redis.net.cn/order/3536.html)  | 将当前数据库的 key 移动到给定的数据库 db 当中。              |                                          |
| [Redis RANDOMKEY 命令](https://www.redis.net.cn/order/3540.html) | 从当前数据库中随机返回一个 key 。                            |                                          |
| [Redis Dump 命令](https://www.redis.net.cn/order/3529.html)  | 序列化给定 key ，并返回被序列化的值。                        |                                          |
| [Redis TTL 命令](https://www.redis.net.cn/order/3539.html)   | 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |                                          |

#### Redis 字符串(String) 命令

| 命令                                                         | 描述                                                         | 对应代码 |
| :----------------------------------------------------------- | :----------------------------------------------------------- | -------- |
| [Redis Setnx 命令](https://www.redis.net.cn/order/3552.html) | 只有在 key 不存在时设置 key 的值。                           |          |
| [Redis Getrange 命令](https://www.redis.net.cn/order/3546.html) | 返回 key 中字符串值的子字符                                  |          |
| [Redis Mset 命令](https://www.redis.net.cn/order/3555.html)  | 同时设置一个或多个 key-value 对。                            |          |
| [Redis Setex 命令](https://www.redis.net.cn/order/3551.html) | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。 |          |
| [Redis SET 命令](https://www.redis.net.cn/order/3544.html)   | 设置指定 key 的值                                            |          |
| [Redis Get 命令](https://www.redis.net.cn/order/3545.html)   | 获取指定 key 的值。                                          |          |
| [Redis Getbit 命令](https://www.redis.net.cn/order/3548.html) | 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。         |          |
| [Redis Setbit 命令](https://www.redis.net.cn/order/3550.html) | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。   |          |
| [Redis Decr 命令](https://www.redis.net.cn/order/3561.html)  | 将 key 中储存的数字值减一。                                  |          |

#### Redis 哈希(Hash) 命令

| 命令                                                         | 描述                                                     |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| [Redis Hmset 命令](https://www.redis.net.cn/order/3573.html) | 同时将多个 field-value (域-值)对设置到哈希表 key 中。    |
| [Redis Hmget 命令](https://www.redis.net.cn/order/3572.html) | 获取所有给定字段的值                                     |
| [Redis Hset 命令](https://www.redis.net.cn/order/3574.html)  | 将哈希表 key 中的字段 field 的值设为 value 。            |
| [Redis Hgetall 命令](https://www.redis.net.cn/order/3567.html) | 获取在哈希表中指定 key 的所有字段和值                    |
| [Redis Hget 命令](https://www.redis.net.cn/order/3566.html)  | 获取存储在哈希表中指定字段的值/td>                       |
| [Redis Hexists 命令](https://www.redis.net.cn/order/3565.html) | 查看哈希表 key 中，指定的字段是否存在。                  |
| [Redis Hincrby 命令](https://www.redis.net.cn/order/3568.html) | 为哈希表 key 中的指定字段的整数值加上增量 increment 。   |
| [Redis Hlen 命令](https://www.redis.net.cn/order/3571.html)  | 获取哈希表中字段的数量                                   |
| [Redis Hdel 命令](https://www.redis.net.cn/order/3564.html)  | 删除一个或多个哈希表字段                                 |
| [Redis Hvals 命令](https://www.redis.net.cn/order/3576.html) | 获取哈希表中所有值                                       |
| [Redis Hincrbyfloat 命令](https://www.redis.net.cn/order/3569.html) | 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。 |
| [Redis Hkeys 命令](https://www.redis.net.cn/order/3570.html) | 获取所有哈希表中的字段                                   |
| [Redis Hsetnx 命令](https://www.redis.net.cn/order/3575.html) | 只有在字段 field 不存在时，设置哈希表字段的值。          |

#### Redis 列表(List) 命令

| 命令                                                         | 描述                                                         | 对应代码                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| [Redis Lindex 命令](https://www.redis.net.cn/order/3580.html) | 通过索引获取列表中的元素                                     |                                                              |
| [Redis Rpush 命令](https://www.redis.net.cn/order/3592.html) | 在列表中添加一个或多个值                                     |                                                              |
| [Redis Lrange 命令](https://www.redis.net.cn/order/3586.html) | 获取列表指定范围内的元素                                     | redisTemplate.opsForList().range(key, 0, -1) // -1 表示查全部 |
| [Redis Rpoplpush 命令](https://www.redis.net.cn/order/3591.html) | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回     |                                                              |
| [Redis Blpop 命令](https://www.redis.net.cn/order/3577.html) | 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |                                                              |
| [Redis Brpop 命令](https://www.redis.net.cn/order/3578.html) | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |                                                              |
| [Redis Brpoplpush 命令](https://www.redis.net.cn/order/3579.html) | 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |                                                              |
| [Redis Lrem 命令](https://www.redis.net.cn/order/3587.html)  | 移除列表元素                                                 |                                                              |
| [Redis Llen 命令](https://www.redis.net.cn/order/3582.html)  | 获取列表长度                                                 |                                                              |

#### Redis 集合(Set) 命令

| 命令                                                         | 描述                                                |
| :----------------------------------------------------------- | :-------------------------------------------------- |
| [Redis Sunion 命令](https://www.redis.net.cn/order/3606.html) | 返回所有给定集合的并集                              |
| [Redis Scard 命令](https://www.redis.net.cn/order/3595.html) | 获取集合的成员数                                    |
| [Redis Srandmember 命令](https://www.redis.net.cn/order/3604.html) | 返回集合中一个或多个随机数                          |
| [Redis Smembers 命令](https://www.redis.net.cn/order/3601.html) | 返回集合中的所有成员                                |
| [Redis Sinter 命令](https://www.redis.net.cn/order/3598.html) | 返回给定所有集合的交集                              |
| [Redis Srem 命令](https://www.redis.net.cn/order/3605.html)  | 移除集合中一个或多个成员                            |
| [Redis Smove 命令](https://www.redis.net.cn/order/3602.html) | 将 member 元素从 source 集合移动到 destination 集合 |
| [Redis Sadd 命令](https://www.redis.net.cn/order/3594.html)  | 向集合添加一个或多个成员                            |
| [Redis Sismember 命令](https://www.redis.net.cn/order/3600.html) | 判断 member 元素是否是集合 key 的成员               |
| [Redis Sdiffstore 命令](https://www.redis.net.cn/order/3597.html) | 返回给定所有集合的差集并存储在 destination 中       |
| [Redis Sdiff 命令](https://www.redis.net.cn/order/3596.html) | 返回给定所有集合的差集                              |
| [Redis Sscan 命令](https://www.redis.net.cn/order/3608.html) | 迭代集合中的元素                                    |
| [Redis Sinterstore 命令](https://www.redis.net.cn/order/3599.html) | 返回给定所有集合的交集并存储在 destination 中       |
| [Redis Sunionstore 命令](https://www.redis.net.cn/order/3607.html) | 所有给定集合的并集存储在 destination 集合中         |
| [Redis Spop 命令](https://www.redis.net.cn/order/3603.html)  | 移除并返回集合中的一个随机元素                      |

#### Redis 有序集合(sorted set) 命令

| 命令                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Redis Zrevrank 命令](https://www.redis.net.cn/order/3625.html) | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| [Redis Zlexcount 命令](https://www.redis.net.cn/order/3614.html) | 在有序集合中计算指定字典区间内成员数量                       |
| [Redis Zunionstore 命令](https://www.redis.net.cn/order/3627.html) | 计算给定的一个或多个有序集的并集，并存储在新的 key 中        |
| [Redis Zremrangebyrank 命令](https://www.redis.net.cn/order/3621.html) | 移除有序集合中给定的排名区间的所有成员                       |
| [Redis Zcard 命令](https://www.redis.net.cn/order/3610.html) | 获取有序集合的成员数                                         |
| [Redis Zrem 命令](https://www.redis.net.cn/order/3619.html)  | 移除有序集合中的一个或多个成员                               |
| [Redis Zinterstore 命令](https://www.redis.net.cn/order/3613.html) | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 |
| [Redis Zrank 命令](https://www.redis.net.cn/order/3618.html) | 返回有序集合中指定成员的索引                                 |
| [Redis Zincrby 命令](https://www.redis.net.cn/order/3612.html) | 有序集合中对指定成员的分数加上增量 increment                 |
| [Redis Zrangebyscore 命令](https://www.redis.net.cn/order/3617.html) | 通过分数返回有序集合指定区间内的成员                         |
| [Redis Zrangebylex 命令](https://www.redis.net.cn/order/3616.html) | 通过字典区间返回有序集合的成员                               |
| [Redis Zscore 命令](https://www.redis.net.cn/order/3626.html) | 返回有序集中，成员的分数值                                   |
| [Redis Zremrangebyscore 命令](https://www.redis.net.cn/order/3622.html) | 移除有序集合中给定的分数区间的所有成员                       |
| [Redis Zscan 命令](https://www.redis.net.cn/order/3628.html) | 迭代有序集合中的元素（包括元素成员和元素分值）               |
| [Redis Zrevrangebyscore 命令](https://www.redis.net.cn/order/3624.html) | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| [Redis Zremrangebylex 命令](https://www.redis.net.cn/order/3620.html) | 移除有序集合中给定的字典区间的所有成员                       |
| [Redis Zrevrange 命令](https://www.redis.net.cn/order/3623.html) | 返回有序集中指定区间内的成员，通过索引，分数从高到底         |
| [Redis Zrange 命令](https://www.redis.net.cn/order/3615.html) | 通过索引区间返回有序集合成指定区间内的成员                   |
| [Redis Zcount 命令](https://www.redis.net.cn/order/3611.html) | 计算在有序集合中指定区间分数的成员数                         |
| [Redis Zadd 命令](https://www.redis.net.cn/order/3609.html)  | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |

#### Redis 连接 命令

| 命令                                                         | 描述               |
| :----------------------------------------------------------- | :----------------- |
| [Redis Echo 命令](https://www.redis.net.cn/order/3650.html)  | 打印字符串         |
| [Redis Select 命令](https://www.redis.net.cn/order/3653.html) | 切换到指定的数据库 |
| [Redis Ping 命令](https://www.redis.net.cn/order/3651.html)  | 查看服务是否运行   |
| [Redis Quit 命令](https://www.redis.net.cn/order/3652.html)  | 关闭当前连接       |
| [Redis Auth 命令](https://www.redis.net.cn/order/3649.html)  | 验证密码是否正确   |

#### Redis 服务器 命令

| 命令                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Redis Client Pause 命令](https://www.redis.net.cn/order/3659.html) | 在指定时间内终止运行来自客户端的命令                         |
| [Redis Debug Object 命令](https://www.redis.net.cn/order/3672.html) | 获取 key 的调试信息                                          |
| [Redis Flushdb 命令](https://www.redis.net.cn/order/3675.html) | 删除当前数据库的所有key                                      |
| [Redis Save 命令](https://www.redis.net.cn/order/3680.html)  | 异步保存数据到硬盘                                           |
| [Redis Showlog 命令](https://www.redis.net.cn/order/3683.html) | 管理 redis 的慢日志                                          |
| [Redis Lastsave 命令](https://www.redis.net.cn/order/3677.html) | 返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示 |
| [Redis Config Get 命令](https://www.redis.net.cn/order/3667.html) | 获取指定配置参数的值                                         |
| [Redis Command 命令](https://www.redis.net.cn/order/3662.html) | 获取 Redis 命令详情数组                                      |
| [Redis Slaveof 命令](https://www.redis.net.cn/order/3682.html) | 将当前服务器转变为指定服务器的从属服务器(slave server)       |
| [Redis Debug Segfault 命令](https://www.redis.net.cn/order/3673.html) | 让 Redis 服务崩溃                                            |
| [Redis Flushall 命令](https://www.redis.net.cn/order/3674.html) | 删除所有数据库的所有key                                      |
| [Redis Dbsize 命令](https://www.redis.net.cn/order/3671.html) | 返回当前数据库的 key 的数量                                  |
| [Redis Bgrewriteaof 命令](https://www.redis.net.cn/order/3654.html) | 异步执行一个 AOF（AppendOnly File） 文件重写操作             |
| [Redis Cluster Slots 命令](https://www.redis.net.cn/order/3661.html) | 获取集群节点的映射数组                                       |
| [Redis Config Set 命令](https://www.redis.net.cn/order/3669.html) | 修改 redis 配置参数，无需重启                                |
| [Redis Command Info 命令](https://www.redis.net.cn/order/3666.html) | 获取指定 Redis 命令描述的数组                                |
| [Redis Shutdown 命令](https://www.redis.net.cn/order/3681.html) | 异步保存数据到硬盘，并关闭服务器                             |
| [Redis Sync 命令](https://www.redis.net.cn/order/3684.html)  | 用于复制功能(replication)的内部命令                          |
| [Redis Client Kill 命令](https://www.redis.net.cn/order/3656.html) | 关闭客户端连接                                               |
| [Redis Role 命令](https://www.redis.net.cn/order/3679.html)  | 返回主从实例所属的角色                                       |
| [Redis Monitor 命令](https://www.redis.net.cn/order/3678.html) | 实时打印出 Redis 服务器接收到的命令，调试用                  |
| [Redis Command Getkeys 命令](https://www.redis.net.cn/order/3664.html) | 获取给定命令的所有键                                         |
| [Redis Client Getname 命令](https://www.redis.net.cn/order/3658.html) | 获取连接的名称                                               |
| [Redis Config Resetstat 命令](https://www.redis.net.cn/order/3670.html) | 重置 INFO 命令中的某些统计数据                               |
| [Redis Command Count 命令](https://www.redis.net.cn/order/3663.html) | 获取 Redis 命令总数                                          |
| [Redis Time 命令](https://www.redis.net.cn/order/3665.html)  | 返回当前服务器时间                                           |
| [Redis Info 命令](https://www.redis.net.cn/order/3676.html)  | 获取 Redis 服务器的各种信息和统计数值                        |
| [Redis Config rewrite 命令](https://www.redis.net.cn/order/3668.html) | 对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写    |
| [Redis Client List 命令](https://www.redis.net.cn/order/3657.html) | 获取连接到服务器的客户端连接列表                             |
| [Redis Client Setname 命令](https://www.redis.net.cn/order/3660.html) | 设置当前连接的名称                                           |
| [Redis Bgsave 命令](https://www.redis.net.cn/order/3655.html) | 在后台异步保存当前数据库的数据到磁盘                         |

#### Redis 脚本 命令

| 命令                                                         | 描述                                                     |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| [Redis Script kill 命令](https://www.redis.net.cn/order/3647.html) | 杀死当前正在运行的 Lua 脚本。                            |
| [Redis Script Load 命令](https://www.redis.net.cn/order/3648.html) | 将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。 |
| [Redis Eval 命令](https://www.redis.net.cn/order/3643.html)  | 执行 Lua 脚本。                                          |
| [Redis Evalsha 命令](https://www.redis.net.cn/order/3644.html) | 执行 Lua 脚本。                                          |
| [Redis Script Exists 命令](https://www.redis.net.cn/order/3645.html) | 查看指定的脚本是否已经被保存在缓存当中。                 |
| [Redis Script Flush 命令](https://www.redis.net.cn/order/3646.html) | 从脚本缓存中移除所有脚本。                               |

#### Redis 事务 命令

| 命令                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Redis Exec 命令](https://www.redis.net.cn/order/3639.html)  | 执行所有事务块内的命令。                                     |
| [Redis Watch 命令](https://www.redis.net.cn/order/3642.html) | 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |
| [Redis Discard 命令](https://www.redis.net.cn/order/3638.html) | 取消事务，放弃执行事务块内的所有命令。                       |
| [Redis Unwatch 命令](https://www.redis.net.cn/order/3641.html) | 取消 WATCH 命令对所有 key 的监视。                           |
| [Redis Multi 命令](https://www.redis.net.cn/order/3640.html) | 标记一个事务块的开始。                                       |

#### Redis HyperLogLog 命令

| 命令                                                         | 描述                                      |
| :----------------------------------------------------------- | :---------------------------------------- |
| [Redis Pgmerge 命令](https://www.redis.net.cn/order/3631.html) | 将多个 HyperLogLog 合并为一个 HyperLogLog |
| [Redis Pfadd 命令](https://www.redis.net.cn/order/3629.html) | 添加指定元素到 HyperLogLog 中。           |
| [Redis Pfcount 命令](https://www.redis.net.cn/order/3630.html) | 返回给定 HyperLogLog 的基数估算值。       |

#### Redis 发布订阅 命令

| 命令                                                         | 描述                               |
| :----------------------------------------------------------- | :--------------------------------- |
| [Redis Unsubscribe 命令](https://www.redis.net.cn/order/3637.html) | 指退订给定的频道。                 |
| [Redis Subscribe 命令](https://www.redis.net.cn/order/3636.html) | 订阅给定的一个或多个频道的信息。   |
| [Redis Pubsub 命令](https://www.redis.net.cn/order/3633.html) | 查看订阅与发布系统状态。           |
| [Redis Punsubscribe 命令](https://www.redis.net.cn/order/3635.html) | 退订所有给定模式的频道。           |
| [Redis Publish 命令](https://www.redis.net.cn/order/3634.html) | 将信息发送到指定的频道。           |
| [Redis Psubscribe 命令](https://www.redis.net.cn/order/3632.html) | 订阅一个或多个符合给定模式的频道。 |

#### Redis 地理位置(geo) 命令

| 命令                                                         | 描述                                                      |
| :----------------------------------------------------------- | :-------------------------------------------------------- |
| [Redis GEOHASH 命令](https://www.redis.net.cn/order/3687.html) | 返回一个或多个位置元素的 Geohash 表示                     |
| [Redis GEOPOS 命令](https://www.redis.net.cn/order/3688.html) | 从key里返回所有给定位置元素的位置（经度和纬度）           |
| [Redis GEODIST 命令](https://www.redis.net.cn/order/3686.html) | 返回两个给定位置之间的距离                                |
| [Redis GEORADIUS 命令](https://www.redis.net.cn/order/3689.html) | 以给定的经纬度为中心， 找出某一半径内的元素               |
| [Redis GEOADD 命令](https://www.redis.net.cn/order/3685.html) | 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中 |
| [Redis GEORADIUSBYMEMBER 命令](https://www.redis.net.cn/order/3690.html) | 找出位于指定范围内的元素，中心点是由给定的位置元素决定    |

参考：
[Redis命令列表](https://www.redis.net.cn/order/)

