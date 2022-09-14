## list 列表

相当于Java中的LinkedList，插入删除很快是O(1)复杂度，索引很慢，是O(n)复杂度

可以同时当作栈或队列使用，使用命令

### 加入、删除元素

```
rpush key value [value ...] // 右边放入列表
lpush key value [value ...] // 左边放入列表
rpop key 					// 右边排出元素
lpop key 					// 左边排出元素
```

**当列表弹出或删除最后一个元素时，该数据结构自动删除，内存被回收**

### 慢操作

#### lindex

相当于Java链表的get(int index)方法，注意：起始元素的索引是0，性能较差，需要遍历链表，复杂度为O(n)

```
 LINDEX key index
```

#### ltrim

```
LTRIM key start stop
```

保留链表索引start到stop区间的元素，端点的元素也可取，同样索引也是从0开始的

index可以为负数，-1表示倒数第一个元素，-2表示倒数第二个元素

```
127.0.0.1:6379> LTRIM test 1 5
OK
127.0.0.1:6379> lrange test 0 10
1) "2"
2) "3"
3) "4"
4) "5"
5) "6"
```

### quicklist

实际上Redis底层存储的并不是一个简单的LinkedList，而是快速链表quicklist

首先在列表元素较少时，会使用一块**连续的内存存储**，这个结构是ziplist压缩列表，当数据量较多时，才会转换为quicklist，因为普通链表需要的附加空间较大，需要prev和next两个指针，会比较浪费空间，而且会加重内存的碎片化

所以Redis把链表和ziplist组合成了quicklist，将多个ziplist使用双向指针串起来，既提高了快速的插入删除的性能，又不会出现太大的空间冗余

![image-20220910232844517](E:\笔记\Redis的深度历险.assets\image-20220910232844517.png)

## hash 字典

内部结构与Java的HashMap是一致的，同样的数组加链表的二维结构，当一维的数组发生hash碰撞时，就会将碰撞的元素用链表连接起来![image-20220910235413187](E:\笔记\Redis的深度历险.assets\image-20220910235413187.png)

### rehash

Redis的字典的值只能是字符串，而且rehash的方式与Java的HashMap的rehash方式不同。Java的HashMap在字典很大时，是一次性全部rehash，比较耗时，而Redis为了高性能，采用的是 **渐进式rehash**策略。

> rehash有两种功能
>
> - （扩容）扩容防止hash冲突后，形成链表带来的性能下降，时间复杂度提升（5倍容量后才扩容）；
> - （缩容）大量key被回收后，大量的空闲空间，通过rehash节省空间（1/10以下使用量才缩容）；

渐进式rehash会在rehash的同时，保留新旧两个结构，查询时，同时查询两个hash结构，然后在后续的定时任务中以及hash的子指令中，渐渐的将旧的hash的内容一点点迁移到新的hash结构中

![image-20220912235250887](E:\笔记\Redis的深度历险.assets\image-20220912235250887.png)

> 当 hash 移除了最后一个元素之后，该数据结构自动被删除，内存被回收。
> hash 结构也可以用来存储用户信息，不同于字符串一次性需要全部序列化整个对象，
> hash 可以对用户结构中的每个字段单独存储。这样当我们需要获取用户信息时可以进行部分
> 获取。而以整个字符串的形式去保存用户信息的话就只能一次性全部读取，这样就会比较浪
> 费网络流量。

### 缺点

存储要消耗内存比字符串更大

### hincrby

*Hincrby* 命令用于为哈希表中的字段值加上指定增量值

```
127.0.0.1:6379> HSET Test zhangsan 123
(integer) 1
127.0.0.1:6379> HINCRBY Test zhangsan 22
(integer) 145
127.0.0.1:6379> HGET Test zhangsan
"145"
```

## set 集合

