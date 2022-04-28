# `Redis`

## 1. `NoSQL`数据库简介

### 1.1 技术发展

我们之前学习了许多东西，按照解决的东西进行分类可以分为：

1. 解决功能性问题：`Java Jsp RDBMS Tomcat HTML Linux JDBC SVN`
2. 解决扩展性问题：`Struts Spring SpringMVC Hibernate MyBatis`
3. 解决性能的问题：`NoSQL JavaThread Hadoop Nginx MQ ElasticSearch`

`Web 1.0`时代：数据访问量非常有限，用单点服务器就可以完成大部分工作。想想这种架构有什么缺点？

![](https://img-blog.csdnimg.cn/cad546abc7124f9c9ab94853f182df2a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ3JBY0tlUi0x,size_20,color_FFFFFF,t_70,g_se,x_160)

`Web 2.0`时代：随着访问量的增加，时代快车进入了`Web 2.0`，访问量越大给处理器和内存都造成了前所未有的压力，这就给数据库的访问造成了很大的压力，那这些问题如何解决呢？可以使用`NoSQL`来解决。

![](https://img-blog.csdnimg.cn/5a4e531ebc1646b1a0b86f6460bf5ada.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ3JBY0tlUi0x,size_20,color_FFFFFF,t_70,g_se,x_16)

如何解决处理器和内存的压力？可以将服务器做集群，然后在访问请求和服务器之间加上一个`Nginx`用来做负载均衡。但是这种将服务器做集群的方式会产生一个新的问题，就是用户访问的`session`信息应该放在哪里呢？如果第一次访问，请求发送到了服务器一，那按照之前`session`存储在服务器中的这种做法，此时`session`就会放到第一个服务器上，但是如果后面用户再发了一次请求，这时请求来到第二个服务器上，此时第二台服务器是没有`session`对象，此时就无法证明用户是已登录状态就无法进行后续的操作。

那如何解决`session`存储或者说用户登录信息存储的问题呢？

> 1. 将用户登录信息存储到客户端`cookie`中，每次请求都带上`cookie`，内含用户信息，但是因为该种方式是存储在客户端的，每次请求都将带上，可能某些恶意用户在他人的`cookie`里头获取了一些重要信息或者将一些恶意信息捆绑在`cookie`往服务器中发送导致服务器出现安全性问题，这就使得该种方式变得不是很好，有没有更好的方案呢？
> 2. `session`复制，我们依然还是使用`session`，我们知道`session`含有用户登录信息保存在服务端，那么第一次用户请求，我们获取到`session`对象之后我们将其复制给其它的服务器，这样一来，以后每次访问都可以获取到`session`对象也就解决了之前的问题，但是这样一来每个都复制一遍，有时候请求压根不到某些服务器去，如果说第一个`cookie`方案造成了安全性的问题，那么这个方案就造成了空间的浪费
> 3. 将用户信息存储在文件服务器或者数据库里，这样虽然解决了第一种方案的安全性问题以及第二种方案的大量空间浪费的问题，但是放在文件服务器或者数据库每次用户请求都要访问文件服务器或者数据库这样就产生了效率问题，因为每次都要进行磁盘读写也就是`I/O`操作
> 4. 将用户信息放在`NoSQL`数据库中，因为`NoSQL`中的数据都放在内存中，速度非常快而且结构简单，每次用户请求我们都去`NoSQL`数据库中验证下看下是否是登录状态，如果是就继续后面的请求，如果不是就拦截，使用`NoSQL`数据库不仅很安全解决了第一个`cookie`方案的安全性问题，而且所占空间非常小这就解决了`session复制`该方案的空间浪费问题，除此之外使用`NoSQL`不用进行`I/O磁盘读写`，因为是`NoSQL`数据库的数据是存放到内存中的，这就大大提高了效率

![](https://img-blog.csdnimg.cn/fc68a7dee5a24fc0bd7b80a26ece2dfe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ3JBY0tlUi0x,size_20,color_FFFFFF,t_70,g_se,x_16)

`NoSQL`除了可以解决`session`存储的问题之外，还可以将频繁访问关系型数据库的数据存储到`NoSQL`数据库中，相当于把磁盘的数据放到内存中做了个缓存，可以大大的提高运行效率。`NoSQL`数据库打破了传统关系型数据库以业务逻辑为依据的存储模式，而针对不同数据结构类型改为以性能为最优先的存储方式。

![](https://img-blog.csdnimg.cn/41054c6f9195472586a301c27ca7fed7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ3JBY0tlUi0x,size_20,color_FFFFFF,t_70,g_se,x_16)

### 1.2 `NoSQL`数据库

`NoSQL`表示的是`Not Only SQL`，表示不仅仅是`SQL`，泛指非关系型数据库，`NoSQL`不依赖业务逻辑方式存储，而是以简单的`key-value`模式存储。因此大大的增加了数据库的扩展能力。`NoSQL`不遵循`SQL`标准，也不支持`ACID`原则，同时其性能远超于关系型数据库。

非关系型数据库适用于：对数据高并发的读写、海量数据的读写、对数据要求高可扩展性的【只要是用不着关系型数据库但是又有存储需求的，或者说用了关系型数据库也无法解决的场景就可以使用非关系型数据库】

常见的非关系型数据库：`Memcache Redis MongoDB 行式数据库 列式数据库 HBase Neo4j`

## 2. `Redis`介绍

`Redis`是一个开源的键值对`key-value`存储系统，数据都缓存在内存中，但是`Redis`为了防止一些不可预知的情况会周期性地把数据写入磁盘或者把修改操作写入追加的记录文件。它支持存储的`value`类型很多，包括字符串类型`string`、链表类型`list`、哈希类型`hash`、集合类型`set`、有序集合类型`zset`，这些数据类型都支持`push/pop add/remove`以及取交集并集差集等更丰富的操作，而且这些操作都是原子性的即要么成功要么失败。在此基础上`Redis`支持各种不同方式的排序。`Redis`还实现了主从同步机制`master-slave`。

高频次高并发高可用的数据，大量数据读写，热数据的访问，做分布式架构做`session`共享都可以用`Redis`

1. 通过`List`实现按自然时间排序的数据可以用于最新`N`个数据
2. 使用`zset`有序集合可以用作排行榜做`Top N`
3. 使用`Expire`过期可以做时效性的功能比如短信验证码
4. 使用原子性自增方法`INCR`，自减方法`DECR`可以用作计数器秒杀
5. 使用`Set`集合可以去除大量数据中的重复数据
6. 使用`List`集合可以构建队列
7. 使用`pub/sub`模式可以发布订阅消息系统

`Redis`其实只有`Linux`版本的，微软表示不错所以自己搞了个`windows`版本的`redis`:smiley:

`Redis`使用的技术是：单线程 + 多路`IO`复用

### 2.1 `Redis`端口从哪来？

`Redis`的端口是`6379`，为什么是这个呢？

> `Alessia Merz`是一位意大利舞女、女演员。`Redis`作者`Antirez`早年看电视节目，觉得`Merz`在节目中的一些话愚蠢可笑，`Antirez`喜欢造“梗”用于平时和朋友们交流，于是造了一个词`MERZ`，形容愚蠢，与 `stupid`含义相同。到了给`Redis`选择一个数字作为默认端口号时，`Antirez`没有多想，把`MERZ`在手机键盘上对应的数字`6379`拿来用了。:open_mouth:

`Redis`默认有`16`个数据库，初始默认使用`0`号库，使用`select <dbid>`可以用来切换数据库，例如：`select 8`，所有的库都有同样的密码。常用的一些命令：

> -  `select`：选择某个数据库
>
> - `dbsize`：查看当前数据库`key`的数量
>
> - `flushdb`：清空当前数据库数据
>
> - `flushall`：清空全部数据

## 3. 常用五大数据类型

### 3.1 `Redis`键`(key)`

> - `keys * `：查看当前库的所有`key`匹配，例如：`keys *1`
> - `exists [keyName]`：判断某个`key`是否存在，`1`表示存在，`0`表示不存在
> - `type [keyName]`：查看某个`key`是什么类型，如果不存在某个`key`则返回`none`
> - `del [keyName]`：删除某个`key`
> - `unlink [keyName]`：异步删除，执行时仅仅将`key`从表层删去但其实还没有，真正的删除会在后续异步执行
> - `expire [keyName] [seconds]`：为给定的`key`设置过期时间
> - `ttl [keyName]`：查看还有多少秒过期，`-1`表示永不过期，`-2`表示已过期

### 3.2 字符串`String`

**<font color="red">注：说的存放的是`String`数据类型是针对`value`而言的而不是`key`而言</font>**

`String`是`Redis`最基本的类型，一个`key`对应着一个`value`。`String`类型是二进制安全的。意味着`Redis`的`String`可以包含任何数据。比如图片或者序列化后的对象。`String`类型是`Redis`最基本的数据类型，一个`Redis`中字符串`value`最多可以是`512M`

> - `set [key] [value]`：添加键值对
> - `get [key]`：查询对应键值
> - `append [key] [value]`：将给定的`value`追加到`key`原值的末尾
> - `strlen [key]`：获得值的长度
> - `setnx [key] [value]`：只有在`key`不存在的时候才设置`key`的值
> - `incr [key]`：将`key`中存储的数字值增加`1`，只有是数字才可以这么做，如果为空则新增值为`1`
> - `decr [key]`：将`key`中储存的数字值减`1`，只有是数字才可以这么做，如果为空则新增值为`-1`
> - `incrby/decrby [key] [step]`：将`key`中存储的数字值增减，自定义步长

无论是`incr`还是`decr`执行的都是原子操作，所谓原子操作指的是不会被线程调度机制打断的操作。这样的操作一旦开始，就一直运行到结束，中间不会有任何的`context switch`切换到另外一个线程。

- 在单线程中能够在单条指令中完成的操作都可以认为是原子性操作，因为中断只能发生在指令之间。
- 在多线程中能够不被其它线程打断的操作都可以认为是原子性操作。

`Redis`之所以命令都是原子性的其原因是`Redis`是单线程架构。总之原子性操作就是一旦执行就不会被打断的操作【除非强制性，这里指的是正常情况】。

**问：`java`中`i++`是否是原子性操作？`i=0`两个线程分别对`i`进行`++100`次，值是多少？**

> `Java`是多线程编程语言，所以肯定不是原子性操作，如果两个线程对一个共享变量`i`进行`++100`次的操作，其最后的结果在`2-200`范围之间。比如`A`线程初始化`i=0`，然后一直加到`99`，此时`B`线程初始化`i=0`，打断`A`线程，进行一次`i++`，此时`A`线程抢到`i=1`，然后`B`线程进行`++`一直到`i=100`此时`A`线程再进行最后一次`i=1`对其进行`i++`结果得到的就是：`2`
>
> `200`的情况就是类似同步线程的情况，`++`操作的结果保存在缓存当中，但是取数据的时候是从主内存中取的。所以才会出现可能是`2`的局面，只有执行权被另外一个线程抢去的时候才会写进主内存，所以此时另外一个线程拿到的数据就是另外一个线程保存在主内存中的数据。

**`String`进阶命令：**

> `mset [key1] [value1] [key2] [value2]...`：同时设置多个键值
>
> `mget [key1] [key2] [key3]...`：同时取到多个键值
>
> `msetnx [key1] [value1] [key2] [value2]...`：设置不存在的数据才能成功但凡有一个`key`在`redis`中式存在的都不成功，因为操作是原子性的
>
> `getrange [key] [start] [end]`：获取键值对中范围内中的值比如：`getrange name 0 3[name=lucykroll]`：得到的结果为`lucy`
>
> `setrange [key] [start] [value]`：将键值对中对应键的值的`start`位置开始替换成新的值
>
> `setex [key] [seconds] [value]`：设置值的时候就设置过期时间，结合`ttl [key]`就可以查看键的过期时间
>
> `getset [key] [value]`：以新值换旧值，设置了新值的同时获取旧值，以前不存在的键获取的值为：`(nil)`

**<font color="red">关于`String`字符串在`Redis`中的数据结构：</font>**

> `String`的数据结构为简单动态字符串`SDS:Simple Dynamic String`，表示的是可以修改的字符串，内部结构实现上类似于`Java`的`ArrayList`，采用预分配冗余空间的方式来减少内存的频繁分配。
>
> ![](https://img-blog.csdnimg.cn/c61f108365e04799a7a04f404eea5c87.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ3JBY0tlUi0x,size_20,color_FFFFFF,t_70,g_se,x_16)
>
> 如图所示，内部为当前字符串实际分配空间`capacity`，一般要高于实际字符串长度`len`。当字符串长度小于`1M`时，扩容都是加倍现有的空间，如果超过`1M`扩容时最多只会扩`1M`的空间，需要注意的时字符串最大长度为：`512MB`

### 3.3 列表`List`

单键多值，`Redis`列表时简单的字符串列表，按照插入顺序进行排序。你可以添加一个元素到列表的头部也可以添加元素到列表的尾部。它的底层实际上是一个双向链表，对两端的操作性能要求很高，通过索引下标的操作中间的节点性能会比较差。添加效率较高，查询效率较低。

**`List`常用命令：**

> `lpush/rpush [key1] [value2] [value3]...`：从左边或者右边插入一个或者多个值
>
> `lpop/rpop [key1]`：从左边或者右边取出值，此时键中这个值被弹出，如果取完了这个列表也就不复存在了
>
> `rpoplpush [key1] [key2]`：从`key1`列表右边吐出一个值插入到`key2`列表左边
>
> `lrange [key] [start] [stop]`：按照索引下标从左到右获取元素，下标从`0`开始，若`stop = -1`则表示取所有值
>
> `lindex [key] [index]`：按照索引下标获得元素[从左到右]
>
> `llen [key]`：获得列表长度
>
> `linsert [key] before [value] [newvalue]`：在`value`的后面插入`newvalue`值
>
> `lrem [key] [n] [value]`：从左边删除`n`个`value`[从左到右]
>
> `lset [key] [index] [value]`：将列表`key`下标为`index`的值替换成`value`

**<font color="red">关于`List`列表在`Redis`中的数据结构：</font>**

> `List`的数据结构是快速链表`quickList`，首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构式`zipList`，也就是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成`quickList`。
>
> 因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存储的只是`int`类型的数据，结构上还需要两个额外的指针`prev`和`next`。
>
> `Redis`将链表和`zipList`结合起来组成了`quickList`，也就是将多个`zipLisr`使用双向指针串起来。这样即满足了快速的插入删除性能，又不会出现太大的空间荣冗余。[巧妙的构思~:cow:]
>
> ![](https://img-blog.csdnimg.cn/1ac4e2d4c4054d61bc9bc4250d131983.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQ3JBY0tlUi0x,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3.4 集合`Set`

`Redis Set`对外提供的功能与`List`列表类似，特殊之处在于列表是允许添加重复的`value`值的但是集合`Set`是不允许的，它可以自动排重，当你需要存储一个列表数据但是又不希望出现重复数据的时候就可以使用`Set`，这将是一个很好的选择，并且`Set`提供了判断某个成员是否在一个`Set`集合内的重要接口，这个也是`List`所不能提供的。

`Redis`的`Set`是`String`类型的无序集合。它底层其实是一个`value`为`null`的哈希表，所以添加删除和查找的复杂度都是`O(1)`。

**`Set`常用命令：**

> `sadd [key] [value1] [value2] [value3]...`：将一个或者多个`member`元素加入到集合`key`中，如果已经存在的`member`元素将被忽略
>
> `smembers [key]`：取出该集合的所有值
>
> `sismember [key] [value]`：判断集合`key`是否含有该`value`值，有的话为`1`，没有则为`0`
>
> `scard [key]`：返回该集合的元素个数
>
> `srem [key] [value1] [value2] [value3]...`：删除集合中的某个或某些元素
>
> `spop [key]`：随机从该集合中吐出一个值，会从集合中删除
>
> `srandmemeber [key] [n]`：随机从该集合中取出`n`个值，不会从集合中删除
>
> `smove [source] [desitination] [value]`：把集合中一个值移动到另外一个集合中
>
> `sinter [key1] [key2]`：返回两个集合的交集元素
>
> `sunion [key1] [key2]`：返回两个集合的并集元素
>
> `sdiff [key1] [key2]`：返回两个集合的差集元素[包含`key1`的但是不包含`key2`的]

**<font color="red">关于`Set`列表在`Redis`中的数据结构：</font>**

> `Set`数据结构是`dict`字典，字典是用哈希表实现的。
>
> `Java`中`HashSet`的内部实现使用的是`HashMap`，只不过所有的`value`都指向同一个对象。`Redis`的`set`结构也是一样，它的内部也使用`hash`结构，所有的`value`都指向同一个内部值。

### 3.5 哈希`Hash`

`Redis`哈希是一个键值对集合，特别适合用于存储对象，其格式为：`[key] [field] [value]`，类似`Java`里的`Map<String, Object>`，用户`ID`为查找的`key`，存储的`value`用户对象包含姓名、年龄、生日等信息，如果用普通的`key/value`结构来存储对象主要有以下`2`种存储方式：

1. 例如`key`为`用户ID`，`value`为序列化的对象`姓名数据 + 年龄数据 + 生日数据据`等，这种存储方式每次修改用户的某个属性都需要修改整个数据，先反序列化改好之后再序列化回去，开销比较大
2. 例如`key`为`用户ID + 姓名标签`，`value`为`姓名数据`，再新建一个`key`存储`用户ID + 年龄标签`，`value`为`年龄数据`，在新建一个`key`为`用户ID + 年龄标签`，`value`为`年龄数据`，这样子做完才完整的存储一个对象，这种存储方式非常冗余

现在我们有了`hash`这种存储方式，我们可以将对象的属性和属性值直接存储在`filed value`而`key`代表某个用户`ID`，这样就存储好了一个对象，非常方便，既不需要重复存储数据，也不会带来反序列化和并发修改控制的问题。

**`Hash`常用命令：**

> `hset <key><field><value>`给`<key>`集合中的`<field>`键赋值`<value>`
>
> `hget <key1><field>从<key1>集合<field>`取出`value` 
>
> `hmset <key1><field1><value1><field2><value2>...`批量设置`hash`的值
>
> `hexists<key1><field>`查看哈希表`key`中，给定域`field`是否存在。 
>
> `hkeys <key>`列出该`hash`集合的所有`field`
>
> `hvals <key>`列出该`hash`集合的所有`value`
>
> `hincrby <key><field><increment>`为哈希表`key`中的域`field`的值加上增量`1 -1`
>
> `hsetnx <key><field><value>`将哈希表`key`中的域`field`的值设置为`value`，当且仅当域`field`不存在

**<font color="red">关于`Hash`哈希在`Redis`中的数据结构：</font>**

> `Hash`类型对应的数据结构是两种：`zipList`（压缩列表），`hashTable`（哈希表）。当`field-value`长度较短且个数较少时，使用`zipList`，否则使用`hashTable`。

### 3.6 有序集合`Zset`

`Redis`有序集合`Zset`和普通集合`Set`费城相似，是一个没有重复元素的字符串集合。不同之处是有序集合的每个成员都关联了一个评分`score`，这个评分`score`被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分是可以重复的。因为元素是有序的，所以你可以很快的根据评分`score`或者次序`position`来获取一个范围的元素。

`Zset`常用命令：

> `zadd [key] [score1] [value1] [score2] [value2] [score3] [value3]...`：将一个或者多个`member`元素及其`score`值加入到有序集合`key`当中去，例如：`zadd topn 100 Java 200 C/C++ 300 Python 400 PHP`
>
> `zrange [key] [start] [stop] [WITHSCORES/withscores]`：返回有序集合`key`下标在`start stop`之间的元素，例如：`zrange topn 0 -1`以及`zrange topn 0 -1 WITHSCORES`
>
> `zrevrange [key] [start] [stop] [WITHSCORES/withscores]`：返回有序集合`key`下标在`start stop`之间的元素【倒序排序】，例如：`zrevrange topn 0 -1`以及`zrange topn 0 -1 WITHSCORES`
>
> `zrangebyscore [key] [min] [max] [WITHSCORES/withscores] [limit offset count]`：返回有序集合`key`中，所有的`score`值介于`min`和`max`之间【闭区间】，有序集合成员按`score`递增也就是从小到大的次序排序，例如：`zrangebyscore topn 100 300 withscores`以及`zrangebyscore topn 100 300 withscores limit 0 1`【跟数据库类似】
>
> `zrevrangebyscore [key] [max] [min] [WITHSCORES/withscores] [limit offset count]`：返回有序集合`key`中，所有的`score`值介于`max`和`min`之间【闭区间】，有序集合成员按`score`递减也就是从大到小的次序排序，例如：`zrevrangebyscore topn 300 100 withscores`以及`zrevrangebyscore topn 300 100 withscores limit 0 1`【跟数据库类似】
>
> `zincrby [key] [increment] [value]`：为元素的`score`加上增量，例如：`zincyby topn 100 Java`
>
> `zrem [key] [value]`：删除该集合中指定的元素，例如：`zrem topn PHP`
>
> `zcount [key] [min] [max]`：统计在分数区间的元素个数`zcount topn 100 300` ---> `2`
>
> `zrank [key] [value]`：返回该值在集合中的排名，从`0`开始

案例：如何利用`zset`实现一个文章访问量的排行榜？

> `zadd topn 1000 v1 2000 v2 3000 v3`
>
> `zrevrange topn 0 -1`

**<font color="red">关于`Zset`有序集合在`Redis`中的数据结构：</font>**

> `Zset`的底层结构式`SortedSet`是`Redis`中一个非常特别的数据结构，一方面它等价于`Java`的数据结构`Map<String, Double>`可以给每一个元素`value`赋予一个权重`score`，另一方面它有类似于`TreeSet`，内部的元素会按照权重`score`进行排序，可以得到每一个元素的名词，还可以通过`score`范围来获取元素的列表，`Zset`的底层使用了两个数据结构：
>
> 1. `Hash`，`field`为`value`，`value`为`score`
> 2. 跳跃表，跳跃表的目的在于给元素`value`进行排序，根据`score`的范围获取元素列表，跳跃表可以好好去研究下，非常有意思

## 4. `Redis`配置文件

1. `###Units 单位###`

   > 配置大小单位，开头定义了一些基本的度量单位，只支持`bytes`不支持`bit`，大小写不敏感
   >
   > `Note on units: when memory size is needed, it is possible to specify`：当需要内存大小时，可以指定
   > `it in the usual form of 1k 5GB 4M and so forth`：通常使用`1k 5GB 4M`这种方式
   > `units are case insensitive so 1GB 1Gb 1gB are all the same.`：单位不区分大小写，因此`1GB 1Gb 1gB`都是相同的

2. `###INCLUDES 包含###`

   > ```xml
   > Include one or more other config files here.  This is useful if you have a standard template that goes to all Redis servers but also need to customize a few per-server settings.  Include files can include other files, so use this wisely.
   > 在这里可以包含一个或者多个文件。如果这个对你有用的话，我们给所有的 Redis 服务器提供了一个标准的模板但是需要额外做一些自定义的服务器配置。包含的文件可以包含其它文件所以需要谨慎使用。
   > 
   > Notice option "include" won't be rewritten by command "CONFIG REWRITE" from admin or Redis Sentinel. Since Redis always uses the last processed line as value of a configuration directive, you'd better put includes at the beginning of this file to avoid overwriting config change at runtime.
   > 注意 include 配置不能被 admin 或者 Redis 哨兵重写，因为 Redis 通常使用的是最后解析的配置行为作为配置指令的值。你最好在这个文件的开头配置 include 来避免它在运行时重写配置
   > 
   > If instead you are interested in using includes to override configuration options, it is better to use include as the last line.
   > 如果相反你正好有兴趣使用 includes 去覆盖原来的配置，你最好是在配置文件的最后一行使用 include
   > 
   > 例如：
   > include .\path\to\local.conf
   > include c:\path\to\other.conf
   > ```

3. `###GENERAL 通用###`

   > ```xml
   > # On Windows, daemonize and pidfile are not supported.
   > # However, you can run redis as a Windows service, and specify a logfile.
   > # The logfile will contain the pid. 
   > 在 Windows 中是不支持守护进程和 pidfile 的设置的，然而你仍然可以运行一个 windows 端的 Redis 服务并指定一个日志文件，该日志文件将包含 pid
   > daemonize 其实就是守护进程也就是让 Redis 服务器后台启动
   > pidfile 表示存放 pid 文件的位置，每个实例会产生一个不同的 pid 文件
   > 
   > # Accept connections on the specified port, default is 6379.
   > # If port 0 is specified Redis will not listen on a TCP socket.
   > 接受指定的端口进行连接，Redis 默认端口为 6379。如果指定的端口为 0 则 Redis 将不会监听 TCP 套接字。
   > port 6379
   > 
   > # TCP listen() backlog.
   > #
   > # In high requests-per-second environments you need an high backlog in order
   > # to avoid slow clients connections issues. Note that the Linux kernel
   > # will silently truncate it to the value of /proc/sys/net/core/somaxconn so
   > # make sure to raise both the value of somaxconn and tcp_max_syn_backlog
   > # in order to get the desired effect.
   > 在每秒请求数量较高的环境中，为了确保避免客户端连接缓慢的问题，需要大量的 backlog。请注意 Linux 内核会默默地将其截断为 /proc/sys/net/core/somaxconn 的值，因此要确保提高 somaxconn 和 tcp_max_syn_backlog 的值以获得所需的效果 
   > tcp-backlog 511
   > 注意：tcp-backlog 其实是一个连接队列，backlog 队列总和 = 未完成三次握手队列 + 已经完成三次握手队列
   > 
   > # By default Redis listens for connections from all the network interfaces
   > # available on the server. It is possible to listen to just one or multiple
   > # interfaces using the "bind" configuration directive, followed by one or
   > # more IP addresses.
   > 默认情况下 Redis 侦听来自服务器上所有可用的网络接口的连接。使用 bind 配置指令可以监听一个或者多个接口，后跟一个或者多个 IP 地址
   > #
   > # Examples:
   > 例如：
   > #
   > # bind 192.168.1.100 10.0.0.1
   > # bind 127.0.0.1
   > 
   > 
   > # Specify the path for the Unix socket that will be used to listen for
   > # incoming connections. There is no default, so Redis will not listen
   > # on a unix socket when not specified.
   > 指定用于侦听传入连接的 Unix 套接字的路径，unixsocket 是没有默认值的，所以 Redis 在没有明确指定 unixsocket 配置指令时是不会侦听 unis 套接字的
   > #
   > # unixsocket /tmp/redis.sock
   > # unixsocketperm 700
   > 
   > # Close the connection after a client is idle for N seconds (0 to disable)
   > 客户端空闲 N 秒之后关闭连接（0 表示禁用，也就是除非手动或者意外关闭否则一直处于连接状态）
   > timeout 0
   > 
   > # TCP keepalive.
   > TCP 心跳检测
   > #
   > # If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
   > # of communication. This is useful for two reasons:
   > 如果非零，则在没有通信的情况下发送 SO_KEEPALIVE 向客户端发送 TCP ACKs 将非常有用，原因有两个：
   > #
   > # 1) Detect dead peers.
   > 1） 检测死节点
   > # 2) Take the connection alive from the point of view of network
   > #    equipment in the middle.
   > 2） 从中间网络设备的视角来看，这样做可以使连接处于活动状态
   > #
   > # On Linux, the specified value (in seconds) is the period used to send ACKs.
   > # Note that to close the connection the double of the time is needed.
   > # On other kernels the period depends on the kernel configuration.
   > 在 Linux 上，发送 ACKs 的周期需要明确指定，注意：关闭连接时需要双倍的时间去关闭。在另外的内核上心跳检测的周期取决于内核的配置。
   > #
   > # A reasonable value for this option is 60 seconds.
   > 心跳检测周期的值合理设置为 60 秒
   > tcp-keepalive 0
   > 
   > # Specify the server verbosity level.
   > # This can be one of:
   > # debug (a lot of information, useful for development/testing)
   > # verbose (many rarely useful info, but not a mess like the debug level)
   > # notice (moderately verbose, what you want in production probably)
   > # warning (only very important / critical messages are logged)
   > 指定服务器的详细级别。详细级别可以是以下之一：
   > debug（大量信息，常用于开发/测试环境）
   > verbose（少有有用的信息，但是不想 debug 级别那样混乱）
   > notice（适度冗长，可能是想在生产环境中才使用的级别，默认是 notice 警告级别）
   > warning（该级别下，只有一些非常重要或者关键的信息被记录）
   > loglevel notice
   > 
   > # Specify the log file name. Also 'stdout' can be used to force
   > # Redis to log on the standard output. 
   > 指定日志文件名。stdout 也可以用于强制 Redis 登录标准输出。
   > logfile ""
   > 
   > # To enable logging to the Windows EventLog, just set 'syslog-enabled' to 
   > # yes, and optionally update the other syslog parameters to suit your needs.
   > # If Redis is installed and launched as a Windows Service, this will 
   > # automatically be enabled.
   > 要想使用Windows EventLog只需要将 syslog-enabled 设置为 yes，并可以选择更新其它适合你所需要的的 syslog 参数，如果 Redis 作为 Windows Service 来安装并启动，将默认自动启动【也就是说在 Linux 系统日志记录默认是关闭的】
   > # syslog-enabled no
   > 
   > # Specify the source name of the events in the Windows Application log.
   > 在 Windows 应用程序日志中指定时间的源
   > # syslog-ident redis
   > 
   > # Set the number of databases. The default database is DB 0, you can select
   > # a different one on a per-connection basis using SELECT <dbid> where
   > # dbid is a number between 0 and 'databases'-1
   > 设置数据库的数量。默认选择数据库是 0 号数据库，你可以使用 SELECT <dbid> 命令选择一个不同的数据库，dbid 中的数字在 0 到（数据库的数量 - 1）【跟数组一样，下标从 0 开始】
   > databases 16
   > 
   > 这里应该还有个 protected-mode 保护模式，默认是开启的，如果想远程访问 Redis 需要将其设置为 no 关闭保护模式
   > ```

4. `###SECURITY 安全###`

   > ```xml
   > # Require clients to issue AUTH <PASSWORD> before processing any other
   > # commands.  This might be useful in environments in which you do not trust
   > # others with access to the host running redis-server.
   > 该配置可以让客户端在执行其它命令之前需要客户端用密码做一个身份认证。该配置可能在你对不信任的极其中需要连接 Redis 服务器时有用。
   > #
   > # This should stay commented out for backward compatibility and because most
   > # people do not need auth (e.g. they run their own servers).
   > 为了向后兼容这里将安全认证做了注释也就是默认不适用，因为大多数人不需要身份认证（例如：他们运行他们自己的服务器）
   > # 
   > # Warning: since Redis is pretty fast an outside user can try up to
   > # 150k passwords per second against a good box. This means that you should
   > # use a very strong password otherwise it will be very easy to break.
   > 警告：由于 Redis 服务器运行的速度非常快，所以外部用户可以使用每秒 15 万个密码去尝试暴力破解你的服务器密码。这意味着你应该使用一个非常健壮的密码否则将很容易被攻破。
   > 
   > 设置密码在这里设置：
   > # requirepass foobared
   > 【设置密码也可以通过命令设置，但是这种设置时暂时的，永久设置还是得在配置文件中设置：config get requirepass, config set requirepass "123456",此时再使用 config get requirepass 则需要你做一个身份认证】
   > 
   > # Command renaming.
   > 命令重命名
   > #
   > # It is possible to change the name of dangerous commands in a shared
   > # environment. For instance the CONFIG command may be renamed into something
   > # hard to guess so that it will still be available for internal-use tools
   > # but not available for general clients.
   > 在共享环境中有可能改变一些重要的命令，这是非常危险的。例如：CONFIG 命令被重命名成了难以猜测的名字，虽然这仍然可以被当作内部工具使用，但是不适用于普通用户
   > #
   > # Example:
   > 例如：
   > # rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
   > 重命名 CONFIG 为 b840fc02d524045429941cc15f59e41cb7be6c52
   > #
   > # It is also possible to completely kill a command by renaming it into
   > # an empty string:
   > 当然，你也可以通过将命令重置为空字符串来完全终止命令
   > #
   > # rename-command CONFIG ""
   > #
   > # Please note that changing the name of commands that are logged into the
   > # AOF file or transmitted to slaves may cause problems.
   > 请注意，更改命令的名称登录到 AOF 文件或者传输到 从服务器 都可能出现问题
   > ```

5. `###LIMIT 设置###`

   > ```xml
   > # Set the max number of connected clients at the same time. By default
   > # this limit is set to 10000 clients, however if the Redis server is not
   > # able to configure the process file limit to allow for the specified limit
   > # the max number of allowed clients is set to the current file limit
   > # minus 32 (as Redis reserves a few file descriptors for internal uses).
   > 设置允许同时访问的最大连接数。默认设置的最大连接数为 10000，然后 Redis 是无法做到让设置上的那样的数量去连接服务器，允许的最大客户端数量为当前文件限制减 32 ，因为 Redis 保留了一些文件描述符供内部使用
   > #
   > # Once the limit is reached Redis will close all the new connections sending
   > # an error 'max number of clients reached'.
   > 一旦达到限制，Redis 将关闭所有新连接，发送错误“达到最大客户端数”
   > #
   > # maxclients 10000
   > 
   > # Don't use more memory than the specified amount of bytes.
   > # When the memory limit is reached Redis will try to remove keys
   > # according to the eviction policy selected (see maxmemory-policy).
   > 不要使用超过指定字节数的内存。当达到内存限制的时候，Redis 将尝试删除键，根据选择的驱逐策略
   > #
   > # If Redis can't remove keys according to the policy, or if the policy is
   > # set to 'noeviction', Redis will start to reply with errors to commands
   > # that would use more memory, like SET, LPUSH, and so on, and will continue
   > # to reply to read-only commands like GET.
   > 如果 Redis 不能根据策略删除键，或者如果策略设置为'noeviction'，Redis 将开始以错误回复命令 会使用更多内存的命令，例如SET、LPUSH 等，并将继续回复只读命令，如 GET。
   > #
   > # This option is usually useful when using Redis as an LRU cache, or to set
   > # a hard memory limit for an instance (using the 'noeviction' policy).
   > 当使用 Redis 作为 LRU 缓存或设置实例的硬内存限制（使用“noeviction”策略）时，此选项通常很有用。
   > #
   > # WARNING: If you have slaves attached to an instance with maxmemory on,
   > # the size of the output buffers needed to feed the slaves are subtracted
   > # from the used memory count, so that network problems / resyncs will
   > # not trigger a loop where keys are evicted, and in turn the output
   > # buffer of slaves is full with DELs of keys evicted triggering the deletion
   > # of more keys, and so forth until the database is completely emptied.
   > 警告：如果您将从属连接到启用了 maxmemory 的实例，则从使用的内存计数中减去提供从属所需的输出缓冲区的大小，因此网络问题/重新同步将不会触发密钥所在的循环驱逐，然后输出从属的缓冲区已满，删除的键的 DEL 触发删除更多的键，依此类推，直到数据库完全清空。
   > #
   > # In short... if you have slaves attached it is suggested that you set a lower
   > # limit for maxmemory so that there is some free RAM on the system for slave
   > # output buffers (but this is not needed if the policy is 'noeviction').
   > #
   > # WARNING: not setting maxmemory will cause Redis to terminate with an
   > # out-of-memory exception if the heap limit is reached.
   > 警告：如果达到堆限制，不设置 maxmemory 将导致 Redis 以 out-of-memory 异常终止。
   > #
   > # NOTE: since Redis uses the system paging file to allocate the heap memory,
   > # the Working Set memory usage showed by the Windows Task Manager or by other
   > # tools such as ProcessExplorer will not always be accurate. For example, right
   > # after a background save of the RDB or the AOF files, the working set value
   > # may drop significantly. In order to check the correct amount of memory used
   > # by the redis-server to store the data, use the INFO client command. The INFO
   > # command shows only the memory used to store the redis data, not the extra
   > # memory used by the Windows process for its own requirements. Th3 extra amount
   > # of memory not reported by the INFO command can be calculated subtracting the
   > # Peak Working Set reported by the Windows Task Manager and the used_memory_peak
   > # reported by the INFO command.
   > 注意：由于 Redis 使用系统分页文件来分配堆内存，Windows 任务管理器或其他工具（如 ProcessExplorer）显示的工作集内存使用情况并不总是准确的。例如，在后台保存 RDB 或 AOF 文件后，工作集值可能会显着下降。为了检查 redis-server 用于存储数据的正确内存量，请使用 INFO 客户端命令。 INFO 命令仅显示用于存储 redis 数据的内存，而不是 Windows 进程出于自身需求而使用的额外内存。可以通过减去 Windows 任务管理器报告的 Peak Working Set 和 INFO 命令报告的 used_memory_peak 来计算未报告 INFO 命令的额外内存量。
   > #
   > # maxmemory <bytes>
   > 
   > # MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
   > # is reached. You can select among five behaviors:
   > # 
   > # volatile-lru -> remove the key with an expire set using an LRU algorithm
   > # allkeys-lru -> remove any key according to the LRU algorithm
   > # volatile-random -> remove a random key with an expire set
   > # allkeys-random -> remove a random key, any key
   > # volatile-ttl -> remove the key with the nearest expire time (minor TTL)
   > # noeviction -> don't expire at all, just return an error on write operations
   > volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）
   > allkeys-lru：在所有集合key中，使用LRU算法移除key
   > volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
   > allkeys-random：在所有集合key中，移除随机的key
   > volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
   > noeviction：不进行移除。针对写操作，只是返回错误信息
   > 
   > # Note: with any of the above policies, Redis will return an error on write
   > #       operations, when there are no suitable keys for eviction.
   > #
   > #       At the date of writing these commands are: set setnx setex append
   > #       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
   > #       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
   > #       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
   > #       getset mset msetnx exec sort
   > #
   > # The default is:
   > #
   > # maxmemory-policy noeviction
   > 
   > # LRU and minimal TTL algorithms are not precise algorithms but approximated
   > # algorithms (in order to save memory), so you can select as well the sample
   > # size to check. For instance for default Redis will check three keys and
   > # pick the one that was used less recently, you can change the sample size
   > # using the following configuration directive.
   > 设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。
   > 一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。
   > 
   > # maxmemory-samples 3
   > ```

## 5. `Redis`的发布和订阅

什么是发布订阅？`Redis`发布订阅`pub/sub`是一种消息通信模式：发送者`pub`发送消息，订阅者`sub`接收消息，`Redis`客户端可以订阅任意数量的频道。

1. 客户端可以订阅频道如下图：

   ![](https://img-blog.csdnimg.cn/802251179d704e8e8ce17dc36a20dd8a.png)

2. 当给这个频道发布消息后，消息就会发送给订阅的客户端：

   ![](https://img-blog.csdnimg.cn/a0e9c97480d04424acca3f4b7ee755a5.png)

流程如下：

> 1. 打开一个客户端订阅`channel1 ---> subscribe channel1`
> 2. 打开另外一个客户端，给`channel1`发布消息`hello ---> publish channel1 hello ---> 返回订阅者数量`
> 3. 打开第一个客户端可以看到发送的消息
