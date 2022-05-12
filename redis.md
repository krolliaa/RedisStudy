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

## 6. `Redis`新数据类型

### 6.1 `Bitmaps`

现代计算机使用二进制位作为信息的基础单位，`1`个字节等于`8`位，例如：`abc`字符串是由`3`个字节组成的，但实际在计算机存储时使用的是二进制存储。`a b c`分别对应的`ASCII`码分别是：`97 98 99`对应的二进制分别为：`01100001 01100010 01100011`，合理地使用操作位能够有效地提高内存使用率和开发效率。

`Redis`提供了`Bitmaps`这个数据类型就是用于对位进行操作的。关于`Bitmaps`有两点需要说明的：

1. `Bitmaps`本身不是一种数据类型，实际上它就是字符串`key-value`，但是它可以对字符串的位进行操作

2. `Bitmaps`单独提供了一套命令，所以在`Redis`中使用`Bitmaps`和使用字符串的方式不相同。可以把`Bitmaps`想象成一个以位为单位的数组，数组中的每个单元只能存储`0 1`，数组下标在`Bitmaps`中，为了让它特殊点，我们给它叫做偏移量，以便给数组区分开来

   ![](https://img-blog.csdnimg.cn/890b95d2945c4a06a51020fbf7bc05c1.png)

`Bitmaps`常用命令：

> - `setbit [key] [offset] [value]`：设置`Bitmaps`中某个偏移量的值`0 1`
>
>   - 实例：每个独立用户是否访问过网站存放在`Bitmaps`中， 将访问的用户记做`1`， 没有访问的用户记做`0`， 用偏移量作为用户的`id`
>
>     设置键的第`offset`个位的值（从`0`算起） ， 假设现在有`20`个用户，`userid=1`， `6`， `11`， `15`， `19`的用户对网站进行了访问， 那么当前`Bitmaps`初始化结果如图
>
>     ![](https://img-blog.csdnimg.cn/34b0652859b04880bca5f8ff641d8be4.png)
>
>     - `setbit users:20201106 1 1`
>
>     - `setbit users:20201106 11 1`
>
>     - `setbit users:20201106 15 1`
>
>     - `setbit users:20201106 19 1`
>
>   因为在第一次初始化`Bitmaps`初始化的时候，如果偏移量`offset`非常大，那么整个初始化过程执行会比较慢，可能造成`Redis`的阻塞。
>
>   又因为很多应用的用户`id`基本都是以一个指定的数字，比如：`10000`开头的，这样如果直接按照上面的例子，直接将用户`id`和`Bitmaps`的偏移量对应势必会造成一定程度的浪费，所以通常的做法就是每次做`setbit`操作的时候都将用户`id`减去这个指定的数字
>
> - `getbit [key] [offset]`：获取`Bitmaps`中某个偏移量的值
>
>   实例：获取`id=8`的用户是否在`2020-11-06`这天访问过， 返回`0`说明没有访问过：`getbit users:20201106 8`
>
> - `bitcount [key] [start] [end]`：统计字符串被设置为`1`的`bit`数。一般情况下，给定的整个字符串都会被惊醒计数，通过指定额外的`start`或者`end`参数，可以让计数只在特定的位上进行。`start end`参数设置可以是负数，比如`-1`代表最后一个位，`-2`代表的是倒数第二个位，`start end`值的是`bit`组的字节的下标数，二者皆包含
>
>   实例：计算`2022-11-06`这天的独立访问用户数量
>
>   `bitcount users:20201106`
>
>   一定要注意的是：`start end`代表的是起始和结束的字节数，下面操作计算用户`id`在第`1`个字节到第`3`个字节之间的独立访问用户数，对应的用户是：`11 15 19`
>
>   比如，这里有个`Bitmaps`对应：`key1 01000001 01000000 00000000 00100001`这里一共有`4`个字节，对应`0 1 2 3`，此时如果输入命令：`bitcount k1  1 2`则代表的是统计下标`1 2`字节组中`bit = 1`的个数，结果为`1`：`bitcount k1 1 2 ---> 1`
>
>   统计下标`1 2 3`字节组中`bit = 1`的个数：`bitcount k1 1 3 ---> 3`
>
>   统计下标`0` 到下标倒数第二的字节组：`bitcount k1 0 -2 ---> 3`
>
> - `bitop [and/or/not/xor] [destkey] [key...]`：`bitop`是一个符合操作，它可以将多个`Bitmaps`进行交并非以及异或操作并将其结果保存在`destkey`中。
>
>   实例：设置`2022-11-04` 访问网站的`userId`为`1 2 5 9`，设置`2022-11-03`访问网站的`userId`为`0 1 4 9`，计算出两天都访问过网站的用户数量：
>
>   `bitop and users:20221103:and:20221104 users:20221103 users:20221104`
>
>   `bitcount users:20221103:and:20221104`
>
>   计算出任意一天都访问过网站的用户数量，例如月活跃就是类似这种，可以使用`or`求并集：
>
>   `bitop or users:or:fourth:month users:20221103 users:20221104` 

`Bitmaps`和`Set`的对比，假设网站有`1`亿用户，每天独立访问的用户有`5`千万，如果每天用集合类型和`Bitmaps`分别存储活跃用户可以得到：

| 数据类型  | 每个用户`id`占用空间 | 需要存储的用户量 | 全部内存量                  |
| --------- | -------------------- | ---------------- | --------------------------- |
| 集合类型  | `64bit`              | `50000000`       | `64bit *50000000 = 400MB`   |
| `Bitmaps` | `1bit`               | `100000000`      | `1bit * 100000000 = 12.5MB` |

很明显可以看到这种情况使用`Bitmaps`可以节省大量的内存空间，尤其是随着时间的推移节省的内存空间是非常可观的：

| 数据类型  | 一天     | 一个月  | 一年    |
| --------- | -------- | ------- | ------- |
| 集合类型  | `400MB`  | `12GB`  | `144GB` |
| `Bitmaps` | `12.5MB` | `375MB` | `4.5GB` |

但是`Bitmaps`并不是万能的，不是什么情况都适合用`Bitmaps`，比方说现在的独立访问用户很少只有`10`万。但是有大量的僵尸用户，总访问为`1`亿。那么二者对比的表如下，此时使用`Bitmaps`就不太合适了，因为基本大部分位都是`0`：

| 数据类型  | 每个`userid`占用空间 | 需要存储的用户量 | 全部内存量                  |
| --------- | -------------------- | ---------------- | --------------------------- |
| 集合类型  | `64bit`              | `100000`         | `64bit * 100000 = 800KB`    |
| `Bitmaps` | `1bit`               | `100000000`      | `1bit * 100000000 = 12.5MB` |

### 6.2 `HyperLogLog`

在工作当中，经常会遇到与统计相关的功能需求，比如统计网站`PV(PageView 页面访问量)`，可以使用`Redis`的`incr incrby`轻松实现。但是像`UV(UniqueVisitor 独立访客)`、独立`IP`数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。

解决基数问题有很多种解决方案：

1. 数据存储在`MySQL`表中，使用`distinct count`计算不重复个数
2. 使用`Redis`提供的`hash set bitmaps`等数据结构来处理

以上的方案结果精确，但随着数据不断增加，导致占用空间会越来越大，对于非常大的数据集是不切实际的。有什么方案可以平衡存储空间吗？通过降低一定的精度？

`Redis`推出了`HyperLogLog`通过降低一定的精度平衡存储空间。`Redis HyperLogLog`是用来做技术统计的算法，其优点是：在输入元素的数量或者体积非常非常大的时候，计算基数所需的空间总是很小的、固定的。

在`Redis`里面，每个`HyperLogLog`键只需要花费`12KB`内存就可以计算将近`2^64`个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是`HyperLogLog`由于只会根据输入元素来计算基数，而不会存储输入元素本身，所以`HyperLogLog`不能像集合那样返回输入的各个元素。

那么到底什么是基数？比如数据集：`{1, 3, 5, 7, 5, 7, 8}`那么这个数据集的基数集为：`{1, 3, 5, 7, 8}`，基数，也就是不重复元素的个数为`5`。基数估计就是在误差可接受的范围内，快速计算基数。
`HyperLogLog`常用命令：

> - `pfadd [key] [element] [element...]`：添加指定元素到`HyperLogLog`中，例如：`pfadd hyperloglog1 "redis" --- pfadd hyperloglog1 "mysql" pfadd hyperloglog1 "redis"` ---> 将所有的元素添加到指定`HyperLogLog`新数据结构中，如果执行命令之后`HyperLogLog`估计的近似基数发生了变化，则返回`1`否则返回`0`
>
> - `pfcount [key] [key...]`计算`HyperLogLog`的近似基数，可以计算多个，比如用`HyperLogLog`存储每天的独立访客数，计算一周的独立访客数`UV`可以使用`7`天的独立访客数进行合并计算。例如：`pfadd hll2 "redis" --- pfadd hll2 "mongodb" --- pfcount hll1 hll2 ---> 3`
>
> - `pfmerge [destkey] [sourcekey] [sourcekey...]`：将一个或者多个`HyperLogLog`合并之后的结果存储在另外一个`HyperLogLog`中，比如每月活跃用户可以使用每天的活跃用户来合并计算。例如：`pfmerge hll3 hll1 hll2`

### 6.3 `Geospatial`

`Redis 3.2`开始增加了对`GEO`类型的支持。`GEO Geographic`，地理信息的缩写，该类型就是元素的二维坐标，在地图上就是经纬度。`Redis`基于该类型，提供了经纬度设置，查询，范围查询、距离查询、经纬度`Hash`等常见操作。

`Geopatial`常见命令：

> - `geoadd [key] [longtiude] [latitude] [member] [longtitude latitude member..]`：添加地理位置（经度、纬度、名称）
>
>   `geoadd china:city 114.05 22.52 shenzhen 116.38 39.90 beijing 121.47 31.23 shanghai 106.50 29.53 chongqing `
>
>   两极无法直接添加，一般会下载城市数据，直接通过`Java`程序一次性导入。
>
>   有效的经度从`-180`度到`180`度。有效的纬度从`-85.05112878`度到`85.05112878`度。
>
>   当坐标位置超出指定范围时，该命令将会返回一个错误。
>
>   已经添加的数据，是无法再次往里面添加的。
>
> - `geopos [key] [member] [member...]`：获得指定地区的坐标值
>
>   `geopos china:city shanghai`
>
> - `geodist [key] [member1] [member2]`：获取两个位置之间的直线距离
>
>   `geodist china:city beijing shanghai km`
>
>   单位：
>
>   `m`表示单位为米[默认值]。
>
>   `km`表示单位为千米。
>
>   `mi`表示单位为英里。
>
>   `ft`表示单位为英尺。
>
>   如果用户没有显式地指定单位参数， 那么`GEODIST`默认使用米作为单位
>
> - `georadius [key] [longtitude] [latitude] radius m|km|ft|mi`：以给定的经纬度为中心，找出某一径内的元素`georadius china:city 110 30 1000 km`

## 7. `Jedis`

> 引入依赖：
>
> ```xml
> <dependency>
> 	<groupId>redis.clients</groupId>
> 	<artifactId>jedis</artifactId>
> 	<version>3.2.0</version>
> </dependency>
> ```
>
> 如果是在`Linux`中运行的`Redis`需要注意的事情：
>
> - 禁用`Linux`的防火墙，以`CentOS7`为例，执行的命令为：`systemctl stop/disable firewalld.service`，`redis.conf`中注释掉`bind 127.0.0.1`，然后`protected-mode on`

初步使用程序如下：

```java
import redis.clients.jedis.Jedis;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        String pong = jedis.ping();
        System.out.println("连接成功 >>> " + pong);
        jedis.close();
    }
}
```

`Jedis-API`测试程序 --- `Key`：

```java
import redis.clients.jedis.Jedis;

import java.util.Set;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //Key 测试
        jedis.flushDB();
        jedis.set("k1", "v1");
        jedis.set("k2", "v2");
        jedis.set("k3", "v3");
        Set<String> keys = jedis.keys("*");
        for (String key : keys) System.out.println("key >>> " + key);
        System.out.println("jedis-exists >>> " + jedis.exists("k1"));
        System.out.println("jedis-ttl >>> " + jedis.ttl("k1"));
        System.out.println("jedis-get >>> " + jedis.get("k1"));
        jedis.close();
    }
}
```

`Jedis-API`测试程序 --- `String`：

```java
import redis.clients.jedis.Jedis;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //String 测试
        jedis.flushDB();
        jedis.set("k1", "v1");
        jedis.set("k2", "v2");
        jedis.set("k3", "v3");
        jedis.mset("str1", "v1", "str2", "v2", "str3", "v3");
        System.out.println("jedis-get >>> " + jedis.get("k1"));
        System.out.println("jedis-mget >>> " + jedis.mget("str1", "str2", "str3"));
        jedis.close();
    }
}
```

`Jedis-API`测试程序 --- `List`：

```java
import redis.clients.jedis.Jedis;

import java.util.List;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //List 测试
        jedis.flushDB();
        jedis.lpush("myList", "1", "2", "3", "4", "5", "6", "7", "8");
        List<String> list = jedis.lrange("myList", 0, -1);
        for (String element : list) System.out.println(element);
        jedis.close();
    }
}
```

`Jedis-API`测试程序 --- `Set`：

```java
import redis.clients.jedis.Jedis;

import java.util.Set;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //Set 测试
        jedis.flushDB();
        jedis.sadd("orders", "order01");
        jedis.sadd("orders", "order02");
        jedis.sadd("orders", "order03");
        jedis.sadd("orders", "order04");
        jedis.sadd("orders", "order05");
        jedis.sadd("orders", "order06");
        jedis.sadd("orders", "order07");
        jedis.sadd("orders", "order08");
        jedis.sadd("orders", "order09");
        jedis.sadd("orders", "order10");
        jedis.sadd("orders", "testOrder");
        Set<String> orders = jedis.smembers("orders");
        for (String order : orders) System.out.println("Set order >>> " + order);
        jedis.srem("orders", "testOrder");
		jedis.close();
    }
}
```

`Jedis-API`测试程序 --- `Hash`：

```java
import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //Set 测试
        jedis.flushDB();
        jedis.hset("hash1", "username", "ZhangSan");
        jedis.hset("hash1", "age", "18");
        System.out.println(jedis.hget("hash1", "username"));
        Map<String, String> map = new HashMap<>();
        map.put("telPhone", "18888888888");
        map.put("address", "ShenZhen");
        map.put("email", "aaa@188.com");
        jedis.hmset("hash2", map);
        List<String> result = jedis.hmget("hash2", "telPhone", "address", "email");
        for (String element : result) System.out.println(element);
        jedis.close();
    }
}
```

`Jedis-API`测试程序 --- `ZSet`：

```java
import redis.clients.jedis.Jedis;

import java.util.Set;

public class JedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //ZSet 测试
        jedis.flushDB();
        jedis.zadd("zSet01", 100d, "z3");
        jedis.zadd("zSet01", 90d, "l4");
        jedis.zadd("zSet01", 80d, "w5");
        jedis.zadd("zSet01", 70d, "z6");
        jedis.zadd("zSet01", 60d, "z7");
        Set<String> zRange = jedis.zrange("zSet01", 0, -1);
        for (String element : zRange) System.out.println(element);
        jedis.close();
    }
}
```

### 7.1 手机验证码

1. 完成一个手机验证码功能，要求：
   - 输入手机号码，点击发送后随机生成`6`位数字码，`2`分钟有效
   
     ```java
     //获取随机验证码
     public static String getCode() {
         Random random = new Random();
         String code = "";
         for (int i = 0; i < 6; i++) {
             int rand = random.nextInt(10);
             code += rand;
         }
         return code;
     }
     ```
   
   - 输入验证码，点击验证，返回成功或者失败
   
   - 每个手机号码每天只能输入三次
   
     ```java
     //获取验证码：2分钟内有效 + 每个手机一天只有 3 次
     public static void sendCode(String phone) {
         String codeKey = "VerifyCode_" + phone + "_Code";
         String countKey = "VerifyCode_" + phone + "_Count";
         Jedis jedis = new Jedis("10.0.0.155", 6379);
         String count = jedis.get(countKey);
         if (count == null) {
             //设置过期时间 ---> 1 天
             jedis.setex(countKey, 24 * 60 * 60, "1");
         } else if (Integer.parseInt(count) <= 2) {
             //小于 3 次可以发送验证码
             jedis.incr(countKey);
         } else if (Integer.parseInt(count) > 2) {
             //大于 2 次无法发送验证码
             System.out.println("今日发送验证码已达 3 次！！！");
             jedis.close();
         }
         String code = getCode();
         jedis.setex(codeKey, 2 * 60, code);
         jedis.close();
     }
     ```
   
     ```java
     //验证验证码
     public static void verifyCode(String phone, String code) {
         Jedis jedis = new Jedis("10.0.0.155", 6379);
         String codeKey = "VerifyCode_" + phone + "_Code";
         String dbCode = jedis.get(codeKey);
         if (dbCode != null && dbCode.equals(code)) {
             System.out.println("验证通过！");
         } else {
             System.out.println("验证码错误！");
         }
     }
     ```
   
   - 全部代码如下：
   
     ```java
     import redis.clients.jedis.Jedis;
     
     import java.util.Random;
     
     public class JedisRandomTest {
         public static void main(String[] args) {
             verifyCode("13888888888", "504775");
         }
     
         //创建随机验证码
         public static String getCode() {
             Random random = new Random();
             String code = "";
             for (int i = 0; i < 6; i++) {
                 int rand = random.nextInt(10);
                 code += rand;
             }
             return code;
         }
     
         //【须符合条件】获取验证码：2分钟内有效 + 每个手机一天只有 3 次
         public static String sendCode(String phone) {
             String codeKey = "VerifyCode_" + phone + "_Code";
             String countKey = "VerifyCode_" + phone + "_Count";
             Jedis jedis = new Jedis("10.0.0.155", 6379);
             String count = jedis.get(countKey);
             if (count == null) {
                 //设置过期时间 ---> 1 天
                 jedis.setex(countKey, 24 * 60 * 60, "1");
             } else if (Integer.parseInt(count) <= 2) {
                 //小于 3 次可以发送验证码
                 jedis.incr(countKey);
             } else if (Integer.parseInt(count) > 2) {
                 //大于 2 次无法发送验证码
                 System.out.println("今日发送验证码已达 3 次！！！");
                 jedis.close();
                 return "今日发送验证码已达 3 次！！！";
             }
             String code = getCode();
             jedis.setex(codeKey, 120, code);
             jedis.close();
             return code;
         }
     
         //验证验证码
         public static void verifyCode(String phone, String code) {
             Jedis jedis = new Jedis("10.0.0.155", 6379);
             String codeKey = "VerifyCode_" + phone + "_Code";
             String dbCode = jedis.get(codeKey);
             if (dbCode != null && dbCode.equals(code)) {
                 System.out.println("验证通过！");
             } else {
                 System.out.println("验证码错误！");
             }
         }
     }
     ```

## 8. `Redis`与`SpringBoot`整合

1. 在`pom.xml`文件中引入`redis`相关依赖

   ```xml
   <!-- redis -->
   <dependency>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   
   <!-- spring2.X集成redis所需common-pool2-->
   <dependency>
   	<groupId>org.apache.commons</groupId>
   	<artifactId>commons-pool2</artifactId>
   	<version>2.6.0</version>
   </dependency>
   ```

2. `application.properties`配置`redis`配置

   ```properties
   spring.redis.host=10.0.0.155
   # Redis使用的端口（默认就是 6379）
   spring.redis.port=6379
   # Redis使用的数据库索引（默认就是 0）
   spring.redis.database=0
   # 连接超时时间
   spring.redis.timeout=1800000
   # 连接池最大连接数（使用负值表示没有限制）
   spring.redis.lettuce.pool.max-active=20
   # 最大阻塞等待时间（使用负值表示没有限制）
   spring.redis.lettuce.pool.max-wait=-1
   # 连接池中最大的空闲连接
   spring.redis.lettuce.pool.max-idle=5
   # 连接池中最小的空闲连接
   spring.redis.lettuce.pool.min-idle=0
   
   ```

3. 添加`RedisTemplate`配置类

   ```java
   package com.example.demo;
   
   import com.fasterxml.jackson.annotation.JsonAutoDetect;
   import com.fasterxml.jackson.annotation.PropertyAccessor;
   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.springframework.cache.CacheManager;
   import org.springframework.cache.annotation.CachingConfigurerSupport;
   import org.springframework.cache.annotation.EnableCaching;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.data.redis.cache.RedisCacheConfiguration;
   import org.springframework.data.redis.cache.RedisCacheManager;
   import org.springframework.data.redis.connection.RedisConnectionFactory;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
   import org.springframework.data.redis.serializer.RedisSerializationContext;
   import org.springframework.data.redis.serializer.RedisSerializer;
   import org.springframework.data.redis.serializer.StringRedisSerializer;
   
   import java.time.Duration;
   
   @EnableCaching
   @Configuration
   public class RedisConfig extends CachingConfigurerSupport {
       @Bean
       public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
           RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
           RedisSerializer<String> redisSerializer = new StringRedisSerializer();
           Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
           ObjectMapper objectMapper = new ObjectMapper();
           objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
           objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
           jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
           redisTemplate.setConnectionFactory(redisConnectionFactory);
           redisTemplate.setKeySerializer(redisSerializer);
           redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
           redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
           return redisTemplate;
       }
   
       @Bean
       public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
           RedisSerializer<String> redisSerializer = new StringRedisSerializer();
           Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
           ObjectMapper objectMapper = new ObjectMapper();
           objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
           objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
           jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
           RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofSeconds(600)).serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer)).serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer)).disableCachingNullValues();
           RedisCacheManager cacheManager = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(redisCacheConfiguration).build();
           return cacheManager;
       }
   }
   ```

4. 测试`RedisTemplate`

   ```java
   package com.example.demo;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   @RequestMapping(value = "/redis")
   public class RedisControllerTest {
       @Autowired
       private RedisTemplate redisTemplate;
   
       @GetMapping(value = "/getKey")
       public String testRedis() {
           redisTemplate.opsForValue().set("name", "Lucky");
           String name = (String) redisTemplate.opsForValue().get("name");
           return name;
       }
   }
   ```


## 9. `Redis`事务_锁机制

`All the commands in a transaction are seriallized and executed sequentiallu.It can never happen that a request issued by another client is served in the middle of the execution of a Redis transaction.This guarantees that the commands are executed as a single isolated operation.`

在`Redis`中所有关于事务的命令都会序列化、按顺序地执行。事务在执行过程中，不会被其他客户端发送来的命令的请求所打断。`Redis`事务是一个单独的隔离操作。

`Redis`事务【或者说事务都如此】的主要作用就是串联多个命令防止别的命令插队。

关于`Redis`事务的命令主要有：`Multi Exec discard`

从输入`Multi`命令开始，输入的命令都会依次进入命令队列中，但是不会执行，直到输入`Exec`之后，`Redis`会将之前的命令队列中的命令依次执行。

组队的过程中可以通过`discard`来放弃组队。

组队过程中如果发生错误则表示组队不成功，不欢而散。整个队列都会被取消，这就是事务的错误处理之组队过程的中发生的错误，也就是`multi`发生的错误。这个很好理解。

如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其它的命令都会执行，不会回滚。而且只是错误的不会进行，后面的命令不被影响，举个例子：

```powershell
127.0.0.1:6379> flushAll
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) (error) ERR value is not an integer or out of range
4) OK
5) OK
```



![](https://img-blog.csdnimg.cn/c51d3086b8bb44d395f483740e02a052.png)

```powershell
> redis-cli
> flushAll
> multi
> set k1 v1
> set k2 v2
> set k3 v3
> set k4 v4
> exec
```

### 9.1 事务冲突

想想一个场景：有很多人有你的账户,同时去参加双十一抢购，你的账户一共只有`10000`元，有一个人想买`8000`元商品，另外一个人想买5000元商品，还有一个想买1000元商品。这就会导致事务冲突的问题。导致最后的账户为：`-4000`事务冲突问题。

![](https://img-blog.csdnimg.cn/f9545658da9c4fb984bd114cf0014e64.png)

### 9.2 锁机制

要想解决掉这个问题，需要用到锁机制：

1. 悲观锁

   > 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次拿数据的时候都会上锁，这样别人想拿这个数据就会`block`【也就是等待，阻塞态】，直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如：行锁，表锁，读锁，写锁等，都是在做操作之前先上锁。
   >
   > 这种方式的缺点就是效率低。
   >
   > ![](https://img-blog.csdnimg.cn/8a818c9fb59b4618a2be0bf7eb25833c.png)

2. 乐观锁

   > 说明下版本号机制，比如当前数据定义为是`v1.0`版本，想获取的都可以获取到，但是如果要修改数据的话肯定是有先后顺序的，先修改的会将版本更改，比如更改为：`v1.1`版本，后修改的会先检查`check-and-set`机制检查版本是否一致，如果版本不一致，会先将数据更改到`v1.1`版本的，然后再做改变，此时又做一次版本升级，最终的版本这里就是`v1.2`
   >
   > 顾名思义就是很乐观，每次去拿数据的时候都会认为别人不会修改，所以不会上锁，但是在更新数据的时候会判断一下在此期间别人有没有去更新这个数据，就可以使用上述所说的版本号机制。乐观锁适用于多读的应用类型也就是多人同时读这种情况，这样就可以提高吞吐量。`Redis`就是用的版本号机制，使用`check-and-set`机制实现事务。
   >
   > 乐观锁的应用场景很广泛，最经典的就是**抢票**。
   >
   > ![](https://img-blog.csdnimg.cn/9c95e32743474f4fbe46ab7ef7ab5838.png)

### 9.3 `Redis`中使用乐观锁

如何在`Redis`中使用锁机制 ---> 乐观锁：`watch [key...]`，在执行`multi`也就是开始事务之前，使用了`watch`，那么在事务执行当这个`key`的`value`被改动了则事务将被打断。

如果想要取消对所有`key`的监视，可以使用`unwatch`，如果`exec/discard`命令先执行了的话就不需要取消监视使用`unwatch`了

### 9.4 `Redis`事务三大特性

- 单独的隔离操作
  - 事务中的所有命令都会序列化按照顺序执行。事务在执行过程中，不会被其他的客户端发送来的命令请求所打断。
- 没有隔离级别的概念
  - 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行
- 不保证原子性
  - 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

### 9.5 `Redis`事务秒杀案例

#### 9.5.1 模拟秒杀操作【无并发】【第一版】

1. 解决计数器和人员记录的事务操作 ---> 重点就是用户的`id`即`uid`还有商品`id`即`prodid`

   > 商品库存：`sk:prodid:qt`
   >
   > 秒杀成功者清单：`sk:prodid:user`

2. 做一个简单的页面，当用户点击时发送秒杀请求

   ```jsp
   <%@ page language="java" contentType="text/html; charset=UTF-8"
            pageEncoding="UTF-8"%>
   <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
   <html>
   <head>
       <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
       <title>Insert title here</title>
   </head>
   <body>
   <h1>iPhone 100 Pro !!!  1元秒杀！！！
   </h1>
   
   
   <form id="msform" action="${pageContext.request.contextPath}/doseckill" enctype="application/x-www-form-urlencoded">
       <input type="hidden" id="prodId" name="prodId" value="888">
       <input type="button"  id="miaosha_btn" name="seckill_btn" value="秒杀点我"/>
   </form>
   
   </body>
   <%--<script  type="text/javascript" src="${pageContext.request.contextPath}/script/jquery/jquery-3.1.0.js"></script>--%>
   <script src="https://code.jquery.com/jquery-3.1.0.min.js"></script>
   <script  type="text/javascript">
       $(function(){
           $("#miaosha_btn").click(function(){
               var url=$("#msform").attr("action");
               $.post(url,$("#msform").serialize(),function(data){
                   if(data=="false"){
                       alert("抢光了" );
                       $("#miaosha_btn").attr("disabled",true);
                   }
               } );
           })
       })
   </script>
   </html>
   ```

3. 创建`Servlet`处理用户发送的`post`请求

   ```java
   @Override
   protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       //这里是模拟，创建随机的 userId 即可
       String userId = new Random().nextInt(5000) + ":";
       //从前端获取商品 id
       String prodId = req.getParameter("prodId");
       //完成秒杀操作 ---> 返回一个布尔值表示是否秒杀成功
       boolean isSuccess = secondKillRedis(userId, prodId);
       resp.getWriter().print(isSuccess);
   }
   ```

4. 秒杀处理

   ```java
   public boolean secondKillRedis(String userId, String prodId) {
       //如果 userId 和 prodId 为null，直接返回 false
       if (userId == null || prodId == null) return false;
       Jedis jedis = new Jedis("10.0.0.155", 6379);
       //用户是否秒杀成功
       String userKey = "sk:" + prodId + ":user";
       //剩余商品数量
       String productCountKey = "sk:" + prodId + ":qt";
       //查询有无该商品，使用 exists 判断是否存在，如果不存在表示秒杀活动尚未开始，如果存在则判断数量够不够，不够表示秒杀活动结束
       if (!jedis.exists(productCountKey)) {
           System.out.println("秒杀活动尚未开始！请耐心等待哦~");
           jedis.close();
           return false;
       } else if (Integer.parseInt(jedis.get(productCountKey)) <= 0) {
           //如果查到了该商品但是数量不够表示秒杀活动已经结束了
           System.out.println("秒杀活动已经结束了哦~");
           jedis.close();
           return false;
       } else if (jedis.sismember(userKey, userId)) {
           //如果商品有，但是秒杀成功的名单上已经有该用户就无法继续进行秒杀了
           System.out.println("您已经秒杀过了哦~");
           jedis.close();
           return false;
       }
       //参与秒杀
       //商品数量减1
       jedis.decr(productCountKey);
       //用户加入秒杀成功的名单【防止添加重复人员，可以使用 sadd】
       jedis.sadd(userKey, userId);
       jedis.close();
       System.out.println("恭喜你！秒杀成功！");
       return true;
   }
   ```

5. `web.xml`如下

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
       <servlet>
           <servlet-name>sk</servlet-name>
           <servlet-class>SecondKillServlet</servlet-class>
       </servlet>
       <servlet-mapping>
           <servlet-name>sk</servlet-name>
           <url-pattern>/doseckill</url-pattern>
       </servlet-mapping>
   </web-app>
   ```

6. `pom.xml`如下：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>org.example</groupId>
       <artifactId>untitled1</artifactId>
       <version>1.0-SNAPSHOT</version>
       <packaging>war</packaging>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <maven.compiler.source>1.8</maven.compiler.source>
           <maven.compiler.target>1.8</maven.compiler.target>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>redis.clients</groupId>
               <artifactId>jedis</artifactId>
               <version>3.2.0</version>
           </dependency>
           <dependency>
               <groupId>javax.servlet</groupId>
               <artifactId>servlet-api</artifactId>
               <version>2.5</version>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.11</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   
       <build>
           <resources>
               <resource>
                   <filtering>false</filtering>
                   <includes>
                       <include>**/*.properties</include>
                       <include>**/*.xml</include>
                   </includes>
                   <directory>src/main/java</directory>
               </resource>
           </resources>
       </build>
   </project>
   ```

7. `redis`中设置：

   ```powershell
   127.0.0.1:6379> set sk:888:qt 10
   OK
   127.0.0.1:6379> get sk:888:qt
   "10"
   ```

#### 9.5.2 模拟秒杀操作【有并发】

使用`Linux`中的`ab`工具可以模拟并发测试，`Centos6`默认安装，`Centos7`需要手动安装

- 安装：`yum install httpd-tools`

- 没有网络时：
  1. 进入`cd /run/media/root/CentOS 7 x86_64/Packages`（路径跟`Centos6`不同）
  2. `apr-1.4.8-3.el7.x86_64.rpm`
  3. `apr-util-1.5.2-6.el7.x86_64.rpm`
  4. `httpd-tools-2.4.6-67.el7.centos.x86_64.rpm`

- `vim postfile 模拟表单提交参数,以&符号结尾`

- 存放当前目录，内容为：`prodid=0101&`

- `ab -n 2000 -c 200 -k -p ~/postfile -T application/x-www-form-urlencoded http://10.0.0.155:8080/doseckill`

可以看到`IDEA`后台中秒杀的数据，然后可以使用`smembers`查看秒杀成功的用户，但是如果秒杀的用户够大，就会产生超卖问题，本质就是之前提到的事务冲突问题。

#### 9.5.3 解决超卖问题【乐观锁】【第二版】

![](https://img-blog.csdnimg.cn/6707c181071a4756bb3d2edd4007b190.png)

按照之前的学习，可以使用悲观锁或者乐观锁解决超卖的问题，但是悲观锁的效率比较低下，因此更建议使用乐观锁【版本号机制】来解决超卖的问题。

![](https://img-blog.csdnimg.cn/13ef9c58c3b741bea34f722224872f2e.png)

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;
import java.util.Random;

public class SecondKillServlet extends HttpServlet {
    public SecondKillServlet() {
        super();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //这里是模拟，创建随机的 userId 即可
        String userId = new Random().nextInt(5000) + ":";
        //从前端获取商品 id
        String prodId = req.getParameter("prodId");
        //完成秒杀操作 ---> 返回一个布尔值表示是否秒杀成功
        boolean isSuccess = secondKillRedis(userId, prodId);
        resp.getWriter().print(isSuccess);
    }

    public boolean secondKillRedis(String userId, String prodId) {
        //如果 userId 和 prodId 为null，直接返回 false
        if (userId == null || prodId == null) return false;
        Jedis jedis = new Jedis("10.0.0.155", 6379);
        //用户是否秒杀成功
        String userKey = "sk:" + prodId + ":user";
        //剩余商品数量
        String productCountKey = "sk:" + prodId + ":qt";
        jedis.watch(productCountKey);
        //查询有无该商品，使用 exists 判断是否存在，如果不存在表示秒杀活动尚未开始，如果存在则判断数量够不够，不够表示秒杀活动结束
        if (!jedis.exists(productCountKey)) {
            System.out.println("秒杀活动尚未开始！请耐心等待哦~");
            jedis.close();
            return false;
        } else if (Integer.parseInt(jedis.get(productCountKey)) <= 0) {
            //如果查到了该商品但是数量不够表示秒杀活动已经结束了
            System.out.println("秒杀活动已经结束了哦~");
            jedis.close();
            return false;
        } else if (jedis.sismember(userKey, userId)) {
            //如果商品有，但是秒杀成功的名单上已经有该用户就无法继续进行秒杀了
            System.out.println("您已经秒杀过了哦~");
            jedis.close();
            return false;
        }
        //参与秒杀 ---> 添加事务
        Transaction transaction = jedis.multi();
        //商品数量减1
        jedis.decr(productCountKey);
        //用户加入秒杀成功的名单【防止添加重复人员，可以使用 sadd】
        jedis.sadd(userKey, userId);
        //执行事务
        List<Object> result = transaction.exec();
        //如果没有执行，result == null 或者 result.size() == 0 表示事务没有执行成功，此时声明商品被抢光了
        if(result == null || result.size() == 0) {
            System.out.println("商品被抢光啦！！！");
            jedis.close();
            return false;
        }
        jedis.close();
        System.out.println("恭喜你！秒杀成功！");
        return true;
    }
}
```

#### 9.5.4 秒光但仍有库存【连接池】【第三版】

因为用的是乐观锁，所以会导致许多连接`Redis`的请求失败，先点的可能没有秒到但是后点的可能秒到了，所以需要使用到连接池来解决这个问题。

使用连接池可以通过管理参数，节省每次连接`Redis`服务所带来的的消耗，可以把连接好的实例反复进行利用。常见参数如下：

> `MaxTotal`：控制一个`pool`可分配多少个`jedis`实例，通过`pool.getResource()`来获取；如果赋值为`-1`，则表示不限制；如果`pool`已经分配了`MaxTotal`个`jedis`实例，则此时`pool`的状态为`exhausted`。【表示筋疲力尽的意思】
>
> `maxIdle`：控制一个`pool`最多有多少个状态为`idle`(空闲)的`jedis`实例；
>
> `MaxWaitMillis`：表示当`borrow`一个`jedis`实例时，最大的等待毫秒数，如果超过等待时间，则直接抛`JedisConnectionException`
>
> `testOnBorrow`：获得一个`jedis`实例的时候是否检查连接可用性`（ping()）`；如果为`true`，则得到的`jedis`实例均是可用的；

打造`Jedis`连接池工具：

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class JedisPoolUtil {

    private static volatile JedisPool jedisPool = null;

    public JedisPoolUtil() {
    }

    public static JedisPool getJedisPoolInstance() {
        if (null == jedisPool) {
            synchronized (JedisPool.class) {
                JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
                jedisPoolConfig.setMaxTotal(200);
                jedisPoolConfig.setMaxIdle(32);
                jedisPoolConfig.setMaxWaitMillis(100 * 1000);
                jedisPoolConfig.setBlockWhenExhausted(true);
                jedisPoolConfig.setTestOnBorrow(true);
                jedisPool = new JedisPool(jedisPoolConfig, "10.0.0.155", 6379, 60000);
            }
        }
        return jedisPool;
    }

    public static void release(JedisPool jedisPool, Jedis jedis) {
        if (null != jedis) {
            jedisPool.getResource();
        }
    }
}
```

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Transaction;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;
import java.util.Random;

public class SecondKillServlet extends HttpServlet {
    public SecondKillServlet() {
        super();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //这里是模拟，创建随机的 userId 即可
        String userId = new Random().nextInt(5000) + ":";
        //从前端获取商品 id
        String prodId = req.getParameter("prodId");
        //完成秒杀操作 ---> 返回一个布尔值表示是否秒杀成功
        boolean isSuccess = secondKillRedis(userId, prodId);
        resp.getWriter().print(isSuccess);
    }

    public boolean secondKillRedis(String userId, String prodId) {
        //如果 userId 和 prodId 为null，直接返回 false
        if (userId == null || prodId == null) return false;
        JedisPool jedisPoolInstance = JedisPoolUtil.getJedisPoolInstance();
        Jedis jedis =jedisPoolInstance.getResource();
        //Jedis jedis = new Jedis("10.0.0.155", 6379);
        //用户是否秒杀成功
        String userKey = "sk:" + prodId + ":user";
        //剩余商品数量
        String productCountKey = "sk:" + prodId + ":qt";
        jedis.watch(productCountKey);
        //查询有无该商品，使用 exists 判断是否存在，如果不存在表示秒杀活动尚未开始，如果存在则判断数量够不够，不够表示秒杀活动结束
        if (!jedis.exists(productCountKey)) {
            System.out.println("秒杀活动尚未开始！请耐心等待哦~");
            jedis.close();
            return false;
        } else if (Integer.parseInt(jedis.get(productCountKey)) <= 0) {
            //如果查到了该商品但是数量不够表示秒杀活动已经结束了
            System.out.println("秒杀活动已经结束了哦~");
            jedis.close();
            return false;
        } else if (jedis.sismember(userKey, userId)) {
            //如果商品有，但是秒杀成功的名单上已经有该用户就无法继续进行秒杀了
            System.out.println("您已经秒杀过了哦~");
            jedis.close();
            return false;
        }
        //参与秒杀 ---> 添加事务
        Transaction transaction = jedis.multi();
        //商品数量减1
        jedis.decr(productCountKey);
        //用户加入秒杀成功的名单【防止添加重复人员，可以使用 sadd】
        jedis.sadd(userKey, userId);
        //执行事务
        List<Object> result = transaction.exec();
        //如果没有执行，result == null 或者 result.size() == 0 表示事务没有执行成功，此时声明商品被抢光了
        if(result == null || result.size() == 0) {
            System.out.println("商品被抢光啦！！！");
            jedis.close();
            return false;
        }
        jedis.close();
        System.out.println("恭喜你！秒杀成功！");
        return true;
    }
}
```

#### 9.5.5 库存遗留问题【`Lua`脚本】【第四版】

`Redis 2.6`以上版本可以编写`Lua`脚本解决超卖问题

```java
package com.xiaoqiu;

import java.io.IOException;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.slf4j.LoggerFactory;

import ch.qos.logback.core.joran.conditional.ElseAction;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisCluster;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;
import redis.clients.jedis.ShardedJedisPool;
import redis.clients.jedis.Transaction;

public class SecKill_redisByScript {
	
	private static final  org.slf4j.Logger logger =LoggerFactory.getLogger(SecKill_redisByScript.class) ;

	public static void main(String[] args) {
		JedisPool jedispool =  JedisPoolUtil.getJedisPoolInstance();
 
		Jedis jedis=jedispool.getResource();
		System.out.println(jedis.ping());
		
		Set<HostAndPort> set=new HashSet<HostAndPort>();
		
	}
	
	static String secKillScript ="local userid=KEYS[1];\r\n" + 
			"local prodid=KEYS[2];\r\n" + 
			"local qtkey='sk:'..prodid..\":qt\";\r\n" + 
			"local usersKey='sk:'..prodid..\":usr\";\r\n" + 
			"local userExists=redis.call(\"sismember\",usersKey,userid);\r\n" + 
			"if tonumber(userExists)==1 then \r\n" + 
			"   return 2;\r\n" + 
			"end\r\n" + 
			"local num= redis.call(\"get\" ,qtkey);\r\n" + 
			"if tonumber(num)<=0 then \r\n" + 
			"   return 0;\r\n" + 
			"else \r\n" + 
			"   redis.call(\"decr\",qtkey);\r\n" + 
			"   redis.call(\"sadd\",usersKey,userid);\r\n" + 
			"end\r\n" + 
			"return 1" ;
			 
	static String secKillScript2 = 
			"local userExists=redis.call(\"sismember\",\"{sk}:0101:usr\",userid);\r\n" +
			" return 1";

	public static boolean doSecKill(String uid,String prodid) throws IOException {

		JedisPool jedispool =  JedisPoolUtil.getJedisPoolInstance();
		Jedis jedis=jedispool.getResource();

		 //String sha1=  .secKillScript;
		String sha1=  jedis.scriptLoad(secKillScript);
		Object result= jedis.evalsha(sha1, 2, uid,prodid);

		  String reString=String.valueOf(result);
		if ("0".equals( reString )  ) {
			System.err.println("已抢空！！");
		}else if("1".equals( reString )  )  {
			System.out.println("抢购成功！！！！");
		}else if("2".equals( reString )  )  {
			System.err.println("该用户已抢过！！");
		}else{
			System.err.println("抢购异常！！");
		}
		jedis.close();
		return true;
	}
}
```

## 10. `Redis`持久化

`Redis`提供了两种不同形式的持久化方式：`RDB`和`AOF`

### 10.1 `RDB`持久化

- 什么是`RDB`持久化？

  > 在指定的**<font color="red">时间间隔</font>**内将内存中的**数据集<font color="red">快照</font>**写入磁盘中，恢复时将快照文件直接读到内存里。

- 备份是如何执行的？

  > `Redis`会单独创建`fork`一个子进程进行持久化，**<font color="red">会先将数据写入到一个临时文件中</font>**，待持久化过程都结束了，**<font color="red">再用这个临时文件替换上次持久化好的文件</font>**。整个过程中，主进程是不进行任何`IO`操作的，这就确保了极高的性能，如果需要进行大规模数据的恢复，并且对于数据恢复的完整性不是很敏感，那么`RDB`方式会比`AOF`方式更加的高效。`RDB`的缺点是最后一次持久化后的数据可能丢失。
  >
  > **<font color="black">`RDB`持久化缺点：</font><font color="red">最后一次持久化后的数据可能丢失</font>**

- 关于`Fork`子进程？

  > - `Fork`子进程的作用就是复制一个与当前主进程一样的进程。新进程的所有数据包括变量、环境变量、程序计数器等数值都和原来的进程一致，**<font color="red">但是`fork`是一个全新的进程，并作为原来进程的子进程</font>**。
  >
  > - 在`Linux`程序中，`fork()`会产生一个和父进程完全相同的子进程，但是子进程在此之后一般会`exec`系统调用，出于效率的考虑，`Linux`引入了**<font color="red">”写时复制技术“</font>**
  >
  > - 一般来说父进程和子进程会共用一段物理内存，只有进程空间的各段内容要发生变化的时候，才会将父进程的内容复制一份给子进程

- 关于持久化流程？

  > ![](https://img-blog.csdnimg.cn/30a52b246aff4e0cae0b7f45e9225e3b.png)

- 关于配置文件中`RDB`相关配置信息？

  >- 使用`vim /usr/local/redis-7.0.0/redis.conf`打开配置文件，使用`?SNAPSHOTTING`搜索。
  >
  >  ![](https://img-blog.csdnimg.cn/784082c15416432eb91d9a27e2e73e1a.png)
  >
  >  在`3600s`中至少有一个`key`发生了改变则做`RDB`持久化，在`300s`中至少`100`个`key`发生了改变则做`RDB`持久化，在`60s`中至少有`10000`个`key`发生了改变则做`RDB`持久化。
  >
  >  - `save 秒数 写操作次数`：`RDB`是整个内存压缩过的快照`SnapShot`，若想禁用`save`可以将其注释还可以将`save`传入空字符串。
  >
  >  - `stop-writes-on-bgsave-error`：当`Redis`发生错误无法写入磁盘时，直接关闭`Redis`的写操作，推荐`yes`。
  >
  >    ```shell
  >    # However if you have setup your proper monitoring of the Redis server
  >    # and persistence, you may want to disable this feature so that Redis will
  >    # continue to work as usual even if there are problems with disk,
  >    # permissions, and so forth.
  >    stop-writes-on-bgsave-error yes
  >    ```
  >
  >  - `rdbcompression`压缩文件：对于存储到磁盘中的快照，可以设置是否进行压缩存储，如果是的话，`Redis`会采用`LZF`算法进行压缩。如果你不想消耗`CPU`进行压缩的话，可以将其关闭，`Redis`该项压缩功能默认是打开的。
  >
  >    ```shell
  >    # Compress string objects using LZF when dump .rdb databases?
  >    # By default compression is enabled as it's almost always a win.
  >    # If you want to save some CPU in the saving child set it to 'no' but
  >    # the dataset will likely be bigger if you have compressible values or keys.
  >    rdbcompression yes
  >    ```
  >
  >  - `rdbchecksum`检查完整性：在存储快照后，可以让`Redis`使用`CRC64`算法进行数据校验，但是这样做会增加约`10%`的性能消耗，如果需要获取最大的性能提升可以将该项校验数据完整性的功能关闭。`Redis`该项检查数据完整性的功能默认是开启的。
  >
  >    ```shell
  >    # RDB files created with checksum disabled have a checksum of zero that will
  >    # tell the loading code to skip the check.
  >    rdbchecksum yes
  >    ```
  >
  
- 关于如何进行`RDB`备份？

  > 通过查看`redis.conf`可以看到`dump.rdb`默认是保存在`redis`目录中的，我们需要先使用`save 20 3`表示如果在`20s`之内修改了`3`个`key`那么可以就需要做`RDB`持久化。
  >
  > 配置文件弄好了之后，重启服务端，去到客户端，更改至少三个`key`，然后等待一下，可以看到`redis/src`目录中出现了`dump.rdb`文件。
  >
  > ![](https://img-blog.csdnimg.cn/13048a4f51a24c4a927767e377095523.png)
  >
  > 将`dump.rdb`文件进行备份，`cp dump.rdb ./d.rdb`
  >
  > 然后删除`dump.rdb`，在客户端中进行`flushdb`，因为`flushdb`也会产生`dump.rdb`所以还要回去删除一次`dump.rdb`文件，再回到客户端使用`keys *`检查一下，确保没有`key`
  >
  > 将`d.rdb`改回`dump.rdb`然后重启`redis-server`，打开`redis-cli`使用`keys *`可以看到`key`复原了。

- `RDB`的优点？

  > 1. 适合大规模的数据恢复，但不适用于对数据完整性和一致性有高要求的数据恢复
  > 2. 节省磁盘空间，又因为进行的是写时拷贝然后用子进程生成的备份文件，所以效率很高，但是如果数据特别特别大，还是比较消耗性能的
  > 3. 恢复速度快

- `RDB`的缺点？

  > 1. 虽然使用了`Fork`子进程提高了效率，但是由于内存中的数据克隆了一份那么就需要考虑大致`2`倍膨胀性
  > 2. 因为是在某时间间隔做的数据备份，那么如果在这个时间间隔之内`Redis`服务`Down`掉的话，则上次备份到`Down`掉的这一段时间内的数据将会丢失。

- 如何停止`RDB`?

  > 只需要将`redis.conf`中的`save 20 3`该行注释掉即可

- `RDB`总结

  > 1. 生成的`RDB`文件会在`redis`服务启动时候恢复数据
  > 2. `RDB`文件时非常紧凑的文件
  > 3. `RDB`在保存`RDB`文件的时候，父进程唯一要做的就是`fork`出一个子进程，接下来所有的工作全部都交由父进程来完成，父进程不需要再做其它的`I/O`操作，所以`RDB`可以最大化`redis`的性能，与后面的`AOF`相比，`RDB`恢复数据会更快一些
  > 4. `Redis`数据丢失风险比较大，虽然是`fork`子进程在做事，但是如果数据特别庞大的话，还是挺耗时的，可能会导致`Redis`在一些毫秒级层级无法响应客户端的请求

### 10.2 `AOF`持久化

- `AOF Append Only File`是什么？

  > 以日志的形式来记录每个写操作（增量保存），将`Redis`执行过的所有写指令记录下来（**<font color="red">不记录读操作</font>**），只允许追加文件但是不可以改写文件，`Redis`启动之初会读取该文件重新构建数据，换言之，`Redis`重启的时候就会根据日志文件的内容 将写指令从前到后执行一次，以便完成数据的恢复工作。

- `AOF`持久化流程？

  > 1. 客户端的请求写命令会被`append`追加到`AOF`缓冲区中
  > 2. `AOF`缓冲区根据`AOF`持久化策略`[always everysec no]`将操作`sync`同步到磁盘的`AOF`文件中
  > 3. `AOF`文件大小超过重写策略或手动重写时，会对`AOF`文件`rewrite`重写，压缩`AOF`文件容量
  > 4. `Redis`服务重启时，会重新`Load`加载`AOF`文件中的写操作达到数据恢复的目的
  >
  > ![](https://img-blog.csdnimg.cn/c0a653111d034fa59e64f9165318ab7b.png)

- `AOF`启动流程？

  > `AOF`默认是不开启的，如果要开启需要在`redis.conf`中配置文件名称，默认为`appendonly no`
  >
  > `AOF`文件的保存路径，同`RDB`的路径一致，都是在`Redis`安装文件下，将`no`改为`yes`就是开启了，日志保存在`appendonly.aof`，然后重启`Redis`服务

- `AOF`和`RDB`同时开启，`Redis`听谁的？

  > `AOF`和`RDB`同时开启，系统默认取`AOF`的数据【数据不会存在丢失，可以保持完整性】

- `AOF`恢复文件？

  > `AOF`的备份机制和性能虽然和`RDB`不同, 但是备份和恢复的操作同`RDB`一样，都是拷贝备份文件，需要恢复时再拷贝到`Redis`工作目录下，启动系统即加载。
  >
  > 如果`appendonly.aof`文件有异常，可以使用`/usr/local/redis-check-aof --fix appendonly.aof`进行修复然后再按正常的流程重启`Redis`服务器进行备份即可

- `AOF`同步频率设置？

  > 1. `appendfsync always`：始终同步，每次对`Redis`进行写操作的时候都会立刻记入日志，性能较差但是数据完整性保存的比较好。
  > 2. `appendfsync everysec`：每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。
  > 3. `appendfsync no`：`Redis`不主动进行同步，把同步时机交给操作系统

- `AOF`重写机制？

  > `AOF`采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当`AOF`文件的大小超过所设定的阈值时，`Redis`就会启动`AOF`文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令`bgrewriteaof`
  >
  > **<font color="red">原理</font>**：`AOF`文件持续增长而过大的时候，会`fork`出一条新的子进程来将文件进行重写【先写临时文件然后再`rename`】，<font color="red">redis4.0版本后的重写，是指上就是把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作。</font>
  >
  > - 如果`no-appendfsync-on-rewrite=yes` ,不写入`aof`文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）
  > - 如果`no-appendfsync-on-rewrite=no`, 还是会把数据往磁盘里刷，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）
  >
  > 默认配置是当`AOF`文件大小是上次重写之后`rewrite`后大小的一倍并且文件大于`64M`的时候触发。
  >
  > **<font color="black">重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。 </font>**
  >
  > `auto-aof-rewrite-percentage`：设置重写的基准值，文件达到`100%`时开始重写（文件是原来重写后文件的2倍时触发）
  >
  > `auto-aof-rewrite-min-size`：设置重写的基准值，最小文件`64MB`。达到这个值开始重写。
  >
  > 例如：文件达到`70MB`开始重写，降到`65MB`，下次什么时候开始重写？`130MB`
  >
  > 系统载入时或者上次重写完毕时，`Redis`会记录此时`AOF`大小，设为`base_size`,如果`Redis`的`AOF`当前大小`>= base_size +base_size*100%`(默认)且当前大小`>=64mb`(默认)的情况下，`Redis`会对`AOF`进行重写。 

- `AOF`重写流程？

  > 1. `bgrewriteaof`触发重写，判断是否当前有`bgsave`或`bgrewriteaof`在运行，如果有，则等待该命令结束后再继续执行。
  > 2. 主进程`fork`出子进程执行重写操作，保证主进程不会阻塞。
  > 3. 进程遍历`redis`内存中数据到临时文件，客户端的写请求同时写入`aof_buf`缓冲区和`aof_rewrite_buf`重写缓冲区保证原`AOF`文件完整以及新`AOF`文件生成期间的新的数据修改动作不会丢失。
  > 4. 子进程写完新的`AOF`文件后，向主进程发信号，父进程更新统计信息
  > 5. 主进程把`aof_rewrite_buf`中的数据写入到新的`AOF`文件
  > 6. 使用新的`AOF`文件覆盖旧的`AOF`文件，完成`AOF`重写
  >
  > ![](https://img-blog.csdnimg.cn/d2442f0ae99e43cd83a5292d66cec400.png)

- `AOF`持久化优势

  > 客户端 ----命令请求---> 服务器 ---网络协议格式的命令内容--->`AOF`文件
  >
  > - 备份机制更加文件，丢失数据的概率更低
  > - 可读的日志文本，通过操作`AOF`稳健，可以处理错误操作【`redis-aof-check --fix`】

- `AOF`持久化劣势

  > - 比起`RDB`占用更多的磁盘空间
  > - 恢复备份数据的速度比较慢
  > - 每次读写都同步的话有一定的性能压力
  > - 存在个别`bug`，造成无法恢复

- `AOF`总结

  > - `AOF`文件实质上操作就是对日志文件进行数据追加
  > - `Redis`可以在`AOF`文件体积变得过大的时候，自动地在后台对`AOF`进行重写，默认是`2`倍原来的大小并且大于`65M`
  > - `AOF`文件有序地保存了对`Redis`进行的所有写入操作，遵循`Redis`协议，格式更容易读懂，分析起来比较轻松
  > - 对于相同数据集来说，`AOF`文件比`RDB`文件体积大
  > - 根据`AOF appendfsync`追加策略不同，`AOF`速度可能略慢于`RDB`

### 10.3 `RDB`和`AOF`的选择与比较

- 用`RDB`还是`AOF`？

  > 官方推荐两个都开启，如果对数据不敏感可以单独使用`RDB`，不建议单独使用`AOF`，因为可能会出现bug`，如果只是做纯内存缓存，可以都不适用，追求最大性能效率

- 官方性能建议

  > **<font color="red">因为`RDB`文件只用作后备用途，建议只在`Slave`上持久化`RDB`文件，而且只要`15`分钟备份一次就够了，只保留`save 900 1`这条规则。</font>**

  - `RDB`持久化方式能够在指定的时间间隔能对你的数据进行快照存储
  - `AOF`持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据，`AOF`命令以`redis`协议追加保存每次写的操作到文件末尾. 
  - `Redis`还能对`AOF`文件进行后台重写,使得`AOF`文件的体积不至于过大
  - 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
  - 同时开启两种持久化方式
  - 在这种情况下，当`redis`重启的时候会优先载入`AOF`文件来恢复原始的数据, 因为在通常情况下`AOF`文件保存的数据集要比`RDB`文件保存的数据集要完整
  - `RDB`的数据不实时，同时使用两者时服务器重启也只会找`AOF`文件。那要不要只使用`AOF`呢？ 
  - 建议不要，因为`RDB`更适合用于备份数据库(`AOF`在不断变化不好备份)， 快速重启，而且不会有`AOF`可能潜在的`bug`，适合留着作为一个万一的手段。

  > **<font color = "red">如果使用`AOF`，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单`Load`自己的`AOF`文件就可以了。</font>**
  >
  > **<font color = "red">代价：一是带来了持续的`IO`，二是`AOF rewrite`重写的最后将`rewrite`过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。</font>**
  >
  > **<font color = "red">只要硬盘许可，应该尽量减少`AOF rewrite`的频率，`AOF`重写的基础大小默认值`64M`太小了，可以设到`5G`以上。
  > 默认超过原大小`100%`大小时重写可以改到适当的数值</font>**

## 11. `Redis`主从复制

- 主从复制是什么？

  > 主机数据更新后根据配置和策略，自动同步到备用机器的`master/slaver`机制，`Master`以写为主，`Slave`以读为主

- 主从复制可以干什么？

  > 读写分离，性能扩展
  >
  > 容灾快速恢复
  >
  > ![](https://img-blog.csdnimg.cn/0ecc1be16f8548959ff99445437b67d5.png)

- 主从复制如何操作？

  > 设置多个`redis.conf`配置文件：
  >
  > 1. `redis6379.conf`
  >
  >    ```
  >    include /usr/local/redis-7.0.0/redis.conf
  >    pidfile /var/run/redis_6379.pid
  >    port 6379
  >    dbfilename dump6379.rdb
  >    ```
  >
  > 2. `redis6380.conf`
  >
  >    ```
  >    include /usr/local/redis-7.0.0/redis.conf
  >    pidfile /var/run/redis_6380.pid
  >    port 6380
  >    dbfilename dump6380.rdb
  >    ```
  >
  > 3. `redis6381.conf`
  >
  >    ```
  >    include /usr/local/redis-7.0.0/redis.conf
  >    pidfile /var/run/redis_6381.pid
  >    port 6381
  >    dbfilename dump6381.rdb
  >    ```
  >
  > `slave-priority 10`：设置从机的优先级，值越小，优先级越高，用于选举主机时的使用。默认为：`100`
  >
  > 启动三台`Redis`服务器：
  >
  > ```
  > redis-server redis6379.conf
  > redis-server redis6380.conf
  > redis-server redis6381.conf
  > ```
  >
  > 查看系统进程，确认是否启动：
  >
  > ```
  > ps -ef | grep redis
  > ```
  >
  > 查看主机的运行情况：
  >
  > ```
  > redis-cli -p 6379
  > info replication
  > 
  > redis-cli -p 6380
  > info replication
  > 
  > redis-cli -p 6381
  > info replication
  > ```
  >
  > 配置从机：
  >
  > `slave <ip> <port>`称为某个实例的从服务器
  >
  > 1. 在`6380 6381`上执行：`slaveof 127.0.0.1 6379`[`redis-cli`中执行]
  > 2. 查看信息：`info replication`
  >
  > **<font color="red">主机写，从机读，在从机上写数据会报错</font>**
  >
  > 主机挂掉直接重启即可，从机挂掉需重设主从关系：`slaveof 127.0.0.1 6379`，如果嫌麻烦可以将配置增加到配置文件中，永久生效

- 常用三招

  > 第一招：一主二仆
  >
  > 切入点问题？`slave1 slave2`是从头开始赋值还是从切入点开始赋值？比如从`k4`进来，之前的`k1 k2 k3`是否也可以复制？
  >
  > 从机是否可以写？`set`可否？ 
  >
  > 主机`shutdown`后情况如何？从机是上位还是原地待命？
  >
  > 主机又回来了后，主机新增记录，从机还能否顺利复制？ 
  >
  > 其中一台从机`down`后情况如何？依照原有它能跟上大部队吗？
  >
  >  
  >
  > 第二招：薪火相传
  >
  >  上一个`slave`可以是下一个`slave`的`Master`，`slave`同样可以接收其他`slaves`的连接和同步请求，那么该`slave`作为了链条中下一个的`master`, 可以有效减轻`master`的写压力,去中心化降低风险。
  >
  > 用`slaveof <ip> <port>`
  >
  > 中途变更转向:会清除之前的数据，重新建立拷贝最新的
  >
  > 风险是一旦某个`slave`宕机，后面的`slave`都没法备份
  >
  > 主机挂了，从机还是从机，无法写数据了
  >
  > 
  >
  > 第三招：反客为主
  >
  > 当一个`master`宕机后，后面的`slave`可以立刻升为`master`，其后面的`slave`不用做任何修改。
  >
  > 用`slaveof  no one`将从机变为主机。

- 主从复制原理

  > - `slave`启动成功连接到`master`后会发送一个`sync`命令
  > - `Master`接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，`master`将传送整个数据文件到`slave`，以完成一次完全同步
  > - 全量复制：而`slave`服务在接收到数据库文件数据后，将其存盘并加载到内存中
  > - 增量复制：`master`继续将新的所有收集到的修改命令依次传给`slave`，完成同步
  > - 但是只要是重新连接`master`，一次完全同步（全量复制)将被自动执行
  
- 哨兵模式

  > 哨兵模式是反客为主模式的进化版本，它可以不用手动`slaveof no one`将从服务器升为主服务器，可以后台监控主机是否故障，如果故障了可以根据投票数自动将从服务器转成主服务器。
  >
  > ![](https://img-blog.csdnimg.cn/f796c68e06354df9823fcfdc76d8651a.png)
  >
  > 如何操作？
  >
  > - 调整为一主二仆模式，在`/myredis`目录下新建：`sentinel.conf`，名字不能出错。
  >
  > - 配置哨兵填写内容：`sentinel monitor mymaster 127.0.0.1 6379 1` ---> 哨兵监控端口为`6379`的服务器，当出错了，只需一个哨兵同意迁移即可更换服务器。
  >
  > - 启动哨兵：`/usr/local/bin` --> `redis`做压力测试可以用自带的`redis-benchmark`工具。
  >
  > - 执行`redis-sentinel /myredis/sentinel.conf`启动哨兵。
  >
  > 当主机挂掉的时候可以在哨兵窗口看到日志，某台从服务器切换成了新的主服务器。那么是根据什么东西选择某台从服务器切换成主服务器呢？
  >
  > **<font color="red">选择的条件依次为：</font>**
  >
  > 1. **<font color="black">选择优先级`slave-priority`靠前的</font>**
  > 2. **<font color="black">选择偏移量最大的【所谓偏移量就是跟主服务器数据同步最多数据的那个】</font>**
  > 3. **<font color="black">选择`runid`最小的从服务器，`runid`是创建服务器的时候就创建的，每个`redis`实例启动后都会随机生成一个`40`位的`runid`</font>**
  >
  > **<font color="red">若原先的主服务器再次启动，则切换成从服务器角色</font>**
  >
  > **<font color="black">优先级在`redis.conf`中默认：`slave-priority 100`，值越小优先级越高</font>**
  >
  > ```java
  > private static JedisSentinelPool jedisSentinelPool=null;
  > public static  Jedis getJedisFromSentinel(){
  > 	if(jedisSentinelPool==null){
  > 		Set<String> sentinelSet=new HashSet<>();
  > 		sentinelSet.add("192.168.11.103:26379");
  > 		JedisPoolConfig jedisPoolConfig =new JedisPoolConfig();
  > 		jedisPoolConfig.setMaxTotal(10); //最大可用连接数
  > 		jedisPoolConfig.setMaxIdle(5); //最大闲置连接数
  > 		jedisPoolConfig.setMinIdle(5); //最小闲置连接数
  > 		jedisPoolConfig.setBlockWhenExhausted(true); //连接耗尽是否等待
  > 		jedisPoolConfig.setMaxWaitMillis(2000); //等待时间
  > 		jedisPoolConfig.setTestOnBorrow(true); //取连接的时候
  > 	    //进行一下测试 ping pong
  > 		jedisSentinelPool=new JedisSentinelPool("mymaster",sentinelSet,jedisPoolConfig);
  >         return jedisSentinelPool.getResource();
  > 	}else{
  >         return jedisSentinelPool.getResource();
  > 	}
  > }
  > ```

## 12. `Redis`集群

### 12.1 问题

容量不够的时候，`Redis`服务器要如何进行扩容？并发写操作，`Redis`如何分摊？

另外，主从模式、新货相传、反客为主、哨兵模式，如果主机宕机，导致主服务器的`IP`地址发生了变化，应用程序中配置需要修改对应的主机地址、端口等信息。

之前通过代理主机来解决，但是`Redis 3.0`中提供了解决方案。就是无中心化集群配置。

### 12.2 什么是集群？

`Redus`集群实现了对`Redis`的水平扩容，即启动了`N`个`Redis`节点，将整个数据库分布存储在这`N`个节点中，每个节点存储总数据的`1/N`。

`Redis`集群通过分区`(partition)`来提供一定程度的可用性`(availability)`：即使集群中有一部分节点失效或者无法进行通讯，集群也可以继续处理命令请求。

## 13. `Redis`应用问题解决

在生产环境中，会因为很多的原因造成访问请求绕过了缓存，都需要访问数据库持久层，虽然对`Redis`缓存服务器不会造成影响，但是数据库的负载就会增大，使得缓存的作用降低

### 13.1 缓存穿透

缓存穿透指的是**<font color="red">查询一个根本不存在的数据</font>**，缓存层和持久层都不会命中。在日常工作中处于容错的考虑，如果从持久层查不到数据则不写入缓存层，缓存穿透导致不存在的数据每次请求都要到持久层去查询，失去了缓存保护后端持久层的意义。【一般都是恶意攻击】

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9xcWFkYXB0LnFwaWMuY24vdHhkb2NwaWMvMC81ZWQzMTk3MDYzMjVjOTgyN2M0NDU5MDZmOWQ4YzIyOS8w?x-oss-process=image/format,png)

缓存穿透问题可能会使后端存储负载加大，由于很多后端持久层不具备高并发性，甚至可能造成后端存储宕机。通常可以再程序中统计总调用数、缓存层命中数、如果同一个`key`的缓存命中率很低，可能就是出现了缓存穿透的问题。

造成缓存穿透的基本原因有两个。第一：自身业务代码或者数据出现问题【例如：`set get`的`key`不一样】，第二：一些恶意攻击、爬虫等造成大量空命中（爬取线上商城商品数据，超大循环递增商品的`ID`）

解决方案：

1. **<font color="red">对空值做缓存。</font>**就算数据访问不到，查询返回的数据为空（不管数据是否存在），我们也做缓存，把这个空结果`null`进行缓存，并且设置空结果的的过期时间会很短，最长不超过五分钟。【临时应急方案】
2. **<font color="red">设置可访问的名单（白名单）。</font>**使用`bitmaps`类型定义一个可以访问的名单，名单`id`作为`bitmaps`的偏移量，每次访问和`bitmaps`里面的`id`进行比较，如果访问`id`不在`bitmaps`里面，则进行拦截，不允许访问。【效率比较低，因为还要访问`bitmaps`】
3. **<font color="red">采用布隆过滤器。</font>**`Bloom Filter 1970由布隆提出`，它实际上底层还是`bitmaps`但是对其做了优化使其效率更高，实际上布隆过滤器就是一个很长的二进制向量（位图）和一系列随机映射函数（哈希函数）。【命中率可能不是很准确】
4. **<font color="red">对`Redis`进行实时监控。</font>**当发现`Redis`的命中率开始急速降低时，需要排查访问对象和访问的数据，设置黑名单限制服务。

### 13.2 缓存击穿

某个`key`存在并且是一个热点数据，大量并发请求不断访问该数据，在该`key`数据失效的一瞬间，会大量访问后端持久层数据库，并回设到缓存，这个时候的大并发请求可能会在这一瞬间把数据库压垮。

![](https://img-blog.csdnimg.cn/3ec06bb3338f49789bcdc3ad3c54333d.png)

解决方案：

1. **<font color="red">预先设置热门数据。</font>**在`Redis`高峰访问之前，把一些热门数据提前存入到`Redis`里面，加大这些热门数据的`key`时长。

2. **<font color="red">适时调整。</font>**现场监控哪些数据热门，实时调整`key`的过期时长。

3. **<font color="red">使用锁。</font>**使用锁必定会降低效率，如果可以忍受的话还是可以用的。

   - 就是在缓存失效的时候【判断拿出来的值为空】，不是立即去数据库拿数据，而是先使用缓存工具的某些带成功操作返回值的操作比如`Redis`中的`SETNX`，去设置一个`mutex key`。【设置排它锁，相当于其他人拿到的是空数据】
     - 如果操作返回成功的时候，再去数据库拿数据，然后回设到缓存，最后删除`mutex key`
     - 如果操作返回失败，证明由线程在向数据库拿数据，那么将当前线程睡眠一段时间再重新`get`缓存

   ![](https://img-blog.csdnimg.cn/707ec8bea866433a9c60d828d6f0dcc3.png)

### 13.3 缓存雪崩

雪崩问题产生原因在于：在集中的某段极短时间内，有大量的数据`key`过期了。这时候大量数据访问底层数据库，就造成了缓存雪崩。数据库直接崩了。对底层系统的冲击非常可怕！

解决方案：

1. **<font color="red">构建多级缓存结构。</font>**`nginx + redis + 其它缓存如 ehcache 等`
2. **<font color="red">使用锁或者队列。</font>**用加锁或者队列的方式来保证不会有大量的线程对数据库一次性进行读写，从而避免失效的时候大量的并发请求落到底层存储系统上。不适用高并发情况，效率较低。
3. **<font color="red">设置过期标志更新缓存。</font>**记录缓存数据是否过期，提前预知一下，然后如果要过期了，就通知另外的线程在后台去更新实际`key`的缓存。
4. **<font color="red">将缓存失效时间分散开。</font>**比如我们在原有的失效时间基础上增加一个随机值，比如`1-5`分钟随机，这样每一个缓存的过期时间的重复率就会大大降低，减小缓存雪崩的可能性。

![](https://img-blog.csdnimg.cn/0e21fc6386cc4179a34c34a001af2fa9.png)
