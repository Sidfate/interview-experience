# Redis 不常用数据结构汇总

引用官网的一段话介绍 Redis：

> Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（string）， 散列（hashe）， 列表（list）， 集合（set）， 有序集合（sorted set） 与范围查询， bitmap， hyperloglog 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

上面一段话其实已经很好的概括了 redis 的数据类型和内置功能。下面我就针对数据结构这一点详细说明下。

### 基础结构

我们常用的数据结构有5种：字符串String、字典Hash、列表List、集合Set、有序集合SortedSet。这几种结构在这里不讲了，之后有文章详细介绍，接下来介绍几种不常用的结构。

### **位图 bitmap**

位图不是实际的数据类型，它的内容其实就是字符串，也就是字节数组。位图的一大优势就是储存数量级大：一个字符串类型的值最多能存储512M字节的内容，位上限：2^(9+10+10+3)=2^32b 。

一个常见的使用场景就是统计用户的一些bool信息，例如月活，因为用户量量级大，每个用户是否活跃可以用一个位0或1表示，这个位图就可以用来统计用户月活数。

位图命令如下：

- [SETBIT](http://redisdoc.com/bitmap/setbit.html) 设置位

    setbit 设置 key 上指定位的值，并返回指定位原先的值

        > setbit usercount:201912 7 1
        (integer) 0

- [GETBIT](http://redisdoc.com/bitmap/getbit.html) 获取位

        > getbit usercount:201912 7
        (integer) 1

- [BITCOUNT](http://redisdoc.com/bitmap/bitcount.html) 位统计

    bitcount 用来统计指定位置范围内 1 的个数。

        > bitcount usercount:201912
        (integer) 1

- [BITPOS](http://redisdoc.com/bitmap/bitpos.html) 位查询

    bitpos 用来查找指定范围内出现的第一个 0 或 1。

- [BITOP](http://redisdoc.com/bitmap/bitop.html)

    对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。

- [BITFIELD](http://redisdoc.com/bitmap/bitfield.html)

    BITFIELD 命令可以在一次调用中同时对多个位范围进行操作： 它接受一系列待执行的操作作为参数， 并返回一个数组作为回复， 数组中的每个元素就是对应操作的执行结果。

### **HyperLogLog**

HyperLogLog 其实本身是一种概率算法，用于基数计数，redis中的HyperLogLog结构正是使用了这个算法。所谓的概率算法，就是在计数要求不需要精确，量级本身又很大的时候，通过一定的概率统计方法预估基数值，大大减少了数据的储存容量。

一个常用的使用场景是统计页面实时 UV 数，UV意味着要去重，当用户量级很大时明显 set 已经不适合用来储存了，HyperLogLog 结构正是一种不精确的去重计数方案。

HyperLogLog 命令如下：

- [PFADD](http://redisdoc.com/hyperloglog/pfadd.html) 增加计数

        > pfadd page:uv user1
        (integer) 1

- [PFCOUNT](http://redisdoc.com/hyperloglog/pfcount.html) 获取计数

        > pfcount page:uv
        (integer) 1

- [PFMERGE](http://redisdoc.com/hyperloglog/pfmerge.html) 计数合并

    将多个 HyperLogLog 合并（merge）为一个 HyperLogLog。

> HyperLogLog 也有个缺点就是计数达到一定量后会稳稳的占据12k的储存空间，所以它不太适合用来统计单个用户的相关计数。

### **GEO**

redis中内置了地理位置距离排序 GeoHash 算法，提供了一套计算附近位置的一套解决方案。

geo其实本身不是一个新的数据类型，它基于 zset 集合。

geo的api指令如下：

- [GEOADD](http://redisdoc.com/geo/geoadd.html) 添加位置

        > geoadd company 120.1916400000 30.1899800000 alibaba
        (integer) 1
        > geoadd company 120.2270060000 30.1897230000 huawei
        (integer) 1
        > geoadd company 120.1923200000 30.1876500000 netease

- [GEOPOS](http://redisdoc.com/geo/geopos.html) 获取位置

        > geopos company alibaba
        1) 1) "120.19163936376571655"
           2) "30.18998013559062343"

- [GEODIST](http://redisdoc.com/geo/geodist.html) 计算距离

        > geodist company alibaba huawei km
        "3.4004"

- [GEORADIUS](http://redisdoc.com/geo/georadius.html) 根据指定位置查询附近地址

    以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

        > georadius company 120 30 200 km WITHCOORD
        1) 1) "netease"
           2) 1) "120.19232064485549927"
              2) "30.18765072684519879"
        2) 1) "alibaba"
           2) 1) "120.19163936376571655"
              2) "30.18998013559062343"
        3) 1) "huawei"
           2) 1) "120.22700697183609009"
              2) "30.18972412875353228"

- [GEORADIUSBYMEMBER](http://redisdoc.com/geo/georadiusbymember.html) 查询附近地址

        > georadiusbymember company alibaba 1 km count 5
        1) "alibaba"
        2) "netease"

- [GEOHASH](http://redisdoc.com/geo/geohash.html) 获取位置hash

        > geohash company alibaba
        1) "wtm7xpbvkh0"