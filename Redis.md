# 坑

## 常见命令

### 1.过期时间丢失

用set命令设置一个已有过期时间的key，而忘记加过期时间

如

```
127.0.0.1:6379> SET testkey val1 EX 60
OK
127.0.0.1:6379> TTL testkey
(integer) 59
```

```
127.0.0.1:6379> SET testkey val2
OK
127.0.0.1:6379> TTL testkey  // key永远不过期了！
(integer) -1
```

### 2.DEL阻塞Redis

Redis删除一个非String类型的key，这个key的元素越多，执行del耗时越久

因为删除这种key时，Redis需要依次释放每个元素的内存。元素越多，耗时越长

当在删除 List/Hash/Set/ZSet 类型的 key 时，一定要格外注意，不能无脑执行 DEL，而是应该用以下方式删除：

1. 查询元素数量：执行 LLEN/HLEN/SCARD/ZCARD 命令
2. 判断元素数量：如果元素数量较少，可直接执行 DEL 删除，否则分批删除
3. 分批删除：执行 LRANGE/HSCAN/SSCAN/ZSCAN + LPOP/RPOP/HDEL/SREM/ZREM 删除

**删除String类型的key时，如果这个key占用内存很大，也有可能造成阻塞**

### 3.RANDOMKEY也可能造成阻塞

这个命令会从 Redis 中「随机」取出一个 key。

> Redis的过期策略：是采用定时清理 + 懒惰清理 2 种方式结合来做的
