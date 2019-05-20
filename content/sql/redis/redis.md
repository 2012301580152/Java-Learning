# 4.1 redis

## Redis 数据结构简介

1. STRING
    1. `GET` 获取存储在给定键中的值 `set key value`
    2. `SET` 设置存储在给定键中的值 `get key`
    3. `DEL` 删除存储在给定键中的值（该命令可用于所有类型）`del key`

2. LIST
    1. `RPUSH` 将给定值推入列表右端 `rpush key item`
    2. `LRANGE` 获取列表在给定范围上的所有值 `lrange key 0 -1`
    3. `LINDEX` 获取列表在给定位置上的单个元素 `lindex key 1`
    4. `LPOP` 从列表左端弹出一个值，并返回被弹出的值 `lpop key`

3. SET
    1. `SADD` 将给定元素添加到集合 `sadd key item`
    2. `SMEMBERS` 返回集合包含的所有元素 `smembers key`
    3. `SISMEMBER` 检查给定元素是否存在于集合 `sismember key item`
    4. `SREM` 如果给定元素存在于集合中，那么移除这个元素 `srem key item`
4. HASH
    1. `HSET` 在散列里面关联起给定的键值对 `hset hash-key key value`
    2. `HGET` 获取指定散列键的值 `hget hash-key key`
    3. `HGETALL` 获取散列包含的所有键值对 `hgetall hash-key`
    4. `HDEL` 如果给定键存在与散列里面，那么移除这个键 `hdel hash-key key`
5. ZSET
    1. `ZADD` 将一个带有给定分值的成员添加到有序集合里面 `zadd key 728 member`
    2. `ZRANGE` 根据元素在有序排列中所处的位置，从有序集合里面获取多个元素 `zrange key 0 -1`
    3. `ZRANGEBYSCORE` 获取有序集合在给定分值范围内的所有元素 `zrangebyscore key 0 800 withscores`
    4. `ZREM` 如果给定成员存在于有序集合，那么移除这个成员 `zrange key member`

