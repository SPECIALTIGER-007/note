## 为什么使用JwtToken + redis

jwt是无状态的，jwt的body部分包含用户Id，单纯的jwt无法实现手动过期失效的功能，即使用户下线，该token在有效期内仍然可以使用，不安全，存在风险。加上redis功能，将token存储在redis中，（以"token_" + token为键，查询到的SysUser对象为值）可以实现手动过期的功能。该做法与session相类似，但是由于前后端无法分离，故采用该技术方案。

![image-20220803201852394](C:\Users\98449\AppData\Roaming\Typora\typora-user-images\image-20220803201852394.png)

## 主流JSON引擎性能比较（GSON，FASTJSON，JACKSON，JSONSMART）

https://blog.csdn.net/Sword52888/article/details/81062575

## 密码安全问题

前端使用md5加密，后端获取数据库中该账户对应的盐，将传入的密码进行加盐md5运算后的值与数据库的值进行比对，成功则返回token

每次修改密码应重新选择随机盐

## ThreadLocal内存泄漏

![1855493-c1a3ca8e0fe6f67d](E:\BaiduNetdiskDownload\博客\博客项目\05\img\1855493-c1a3ca8e0fe6f67d.webp)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d725e66b598d4f64a08ed22377f2406f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54y_5aeL5aSn54yp54yp,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

```java
/**
 * The table, resized as necessary.
 * table.length MUST always be a power of two.
 */
private Entry[] table;
```

```java
/**
 * The entries in this hash map extend WeakReference, using
 * its main ref field as the key (which is always a
 * ThreadLocal object).  Note that null keys (i.e. 	entry.get()
 * == null) mean that the key is no longer referenced, so the
 * entry can be expunged from table.  Such entries are referred to
 * as "stale entries" in the code that follows.
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
      super(k);
      value = v;
       }
    }

```

```java
/**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue){
	table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

每个Thread维护一个ThreadLocalMap，与Thread的生命周期相同。

ThreadLocalMap有一个内部类，Entry。并且维护一个Entry数组，命名为table，entry的key是ThreadLocal类型，并且声明为弱引用，value是Object类型。

弱引用：每次gc回收时，无论是否有空间，都会没回收掉

因此，entry的key(ThreadLocal类型)每次gc都会被回收，但是value是强引用，不会被回收，若是不采取其他操作，那么value引用的Object将会发生内存泄漏。

所以，每次用完ThreadLocal的变量，需要使用remove()方法。（采用set()，get()，remove()都会清理过期节点）

## 出现TargetException

```
Throwable targetException = var6.getTargetException
```

原因：向数据库插入数据时，如时间属性，未设置正确的值时，会抛出异常，需要设置属性

## Gson.fromJson报错（json转换为javaBean对象报错）

```
java.lang.IllegalStateException: Expected BEGIN_OBJECT but was STRING at line 1 column 1 path $
	at com.google.gson.internal.bind.ReflectiveTypeAdapterFactory$Adapter.read
```

因为：redis存入缓存时，过期时间设置错误，没有写时间单元，

导致redis的value中出现很多\x00，导致gson将json转换为javaBean对象时出现上列异常。

奇怪的是，在使用错误redis设置value方法时，使用阿里的fastjson，不会出现上面的类似异常。

```java
// 错误的
redisTemplate.opsForValue().set(redisKey, 				                       redisValue,CacheUtils.randomTime(expire));	
```

```java
// 正确的
redisTemplate.opsForValue().set(redisKey, redisValue,
            CacheUtils.randomTime(expire), TimeUnit.MILLISECONDS);
```



## RocketMq发送消息异常

```
2022-08-18 11:58:00.015 ERROR 12404 --- [dblog-rocketmq1] .a.i.SimpleAsyncUncaughtExceptionHandler : Unexpected exception occurred invoking async method: public void com.dudigest.ddblog.service.RocketMqThreadService.refreshHotArticles()
```

原因：消息对象传入的字符串为""，空字符，引起异常

```java
// 错误的  
MqMsg mqMsg = new MqMsg();
mqMsg.setTopic(MqConstant.TOPIC_ARTICLE);
mqMsg.setTag(MqConstant.TAG_REFRESH_HOT_ARTICLES_);
mqMsg.setContent("");
rocketMqService.asyncSend(mqMsg)
```

```java
// 正确的
MqMsg mqMsg = new MqMsg();
mqMsg.setTopic(MqConstant.TOPIC_ARTICLE);
mqMsg.setTag(MqConstant.TAG_REFRESH_HOT_ARTICLES_);
mqMsg.setContent("refresh");
rocketMqService.asyncSend(mqMsg)
```

