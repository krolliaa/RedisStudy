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

# 3. 常用五大数据类型

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

`String`进阶命令：

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

`List`常用命令：

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
