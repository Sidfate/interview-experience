# 容器 - HashMap

### 前言
在讲技术前有必要讲一下这篇文章的由来。写java的朋友，无论是客户端还是服务端，HashMap基本上都最常用的java容器了，正因为最常用，所以我们需要去了解的更深，对代码优化和规范都有好处。网上关于 hashmap 的讲解也铺天盖地多的是，那为什么我还要写一篇这个呢。原因主要在于你可以看网上任何的一篇讲 hashmap 的文章，远远没有这篇文章带给你的清晰和完整，甚至可以让你靠近人家在开发 hashmap 时用到的思维模式。

> 说明：以下汇总下jdk1.8以及之后版本中的 HashMap 实现。

### 简介
首先用一句话说明 HashMap 的结构：

数组+链表，即数组中存放的是指向链表的指针，当链表中数据大于阈值（默认为8），用红黑树代替链表。


![](https://sidfate.oss-cn-hangzhou.aliyuncs.com/upic/20200221123400-xUCvKU.jpg)


上面出现了3个结构：数组、链表和红黑树。那么下面依次说明各个结构在 HashMap 中的具体应用。


### 数组

HashMap 中的数组也被叫做哈希桶，本质上是一个 Node 类型的数组。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        ...
}
```

可以看到 Node 中存放着 key，value 以及一个 hash 值。这个 hash 值的作用就是为了确定指定的 key 存放在数组中的哪个桶，它的具体实现：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

从这个 hash 函数中我们可以看到2点： 
1. key 允许为null，为 null 时存放在数组的第一个桶（下标为0）中。 
2. 用到了 key 对象的 hashCode 方法获取对象的 hash 值，然后运用了一些位运算计算下标。

java 中 int 4个字节，`h>>>16` 相当于获取 h 的高位部分，之后的位运算是将 key 的 hashCode 与其高位异或操作，相当于将高位和低位综合一下，为了减少 hash 碰撞的记录。最终这个扰动函数计算出来的 hash 值会跟数组长度进行求余操作来获取 key 存放的桶下标：

```java
// & 可以用来取余，i 就是计算出的数组的下标
i = (n-1) & hash
```

> 为什么需要将高低位异或？因为hashCode()是int类型，取值范围是40多亿，只要哈希函数映射的比较均匀松散，碰撞几率是很小的。但是由于HashMap的哈希桶的长度远比hash取值范围小，默认是16，所以当对hash值以桶的长度取余，以找到存放该key的桶的下标时，由于取余是通过与操作完成的，会忽略hash值的高位。因此只有hashCode()的低位参加运算，发生不同的hash值，但是得到的index相同的情况的几率会大大增加。

### 链表

hash 函数总是有可能会存在冲突的，当两个不同的 key 计算的 hash 值相等时，此时它们对应数组的下标也就相等了。因此我们就需要将数组中的元素以链表的方式串联起来，这就是 Node 结构存在一个指向下一个节点的 next 字段的原因。其中新建的 Node 会插入到链表的尾部。

```java
// 取一段 put 方法中插入到链表尾的代码
Node<K,V> e; K k;
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        // 到达链表尾部
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

> 思考一下为什么要插入到链表尾部，而不是头插法（插入到头部）呢？
让我们来算下时间复杂度，尾插法需要遍历整个链表，时间复杂度为O(N)，头插法只需要将数组下标中值存放成新 Node，然后将新 Node 的 next 指向旧的头，复杂度为O(1)，明明头插更快啊，为什么会变成尾插呢？答案请看扩容章节。

### 红黑树

上面的代码中我们还能看到一个常量 `TREEIFY_THRESHOLD`，它指代的是链表的最大长度，超过这个长度后链表需要被转化为红黑树，默认这个变量为 8。

```java
static final int TREEIFY_THRESHOLD = 8;
```

> 什么是红黑树？请看我之后的数据结构篇。

为什么要把长的链表转换成红黑树呢，因为你们可以看到上面的插入操作在查找的过程中用了一个 for 循环找到尾部，时间复杂度是O(n)，而红黑树可以让搜索的复杂度降到O(logn)，对于数据量大的情况，效率下降的比链表慢。

下面是链表转换为红黑树的过程：
```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 将链表 Node 替换成 TreeNode
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

### 扩容

当我们不断的往hash桶放数据，整个桶会变得越来越臃肿，操作效率大幅降低，这时候我们需要去给 hashmap 扩容。

假定 hashmap 的容量是 capacity，目前存放着 size 个元素，当 size > threshold 时，hashmap 需要扩容，`threshold = capacity * load factor`。load factor 是负载因子，默认是 0.75。

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 很多地方通过如下判断进入 resize 流程
if(size > threshold) resize();
```

关于容量，大家都知道 hashmap 的默认容量是 16。你有想过为什么是16吗？这其实还是要回过头看到我们计算数组下标的代码 `(n-1) & hash`， n 指代容量，只要n是2的幂，那么 n-1肯定是一个二进制值全为1的数字.例如 n = 16，那么 15 的二进制就是 1111，1111 与其他数 & 操作后的值就是后4位本身的值，所以达到最终目的，均匀的 hash（你可以对比下用 5，7等数跟别的数 & 的结果，碰撞几率明显上升）。而 16 恰巧是那个合适的默认值，不大不小。这个道理跟负载因子是 0.75一样，为啥是 0.75 呢，也是折中过后取的值，如果因子为1，那么允许不扩容的大小就越大，hash 碰撞发生几率高，如果小了，动不动就扩容也不行，内存消耗大了（我猜测 0.75 应该是专家实验统计出来的结果）。

hashmap 中还存在一个最大容量的常量，值是 2^30 次方，当容量到了这个级别就不会在扩容了。正常情况下，扩容时会先创建一个长度为原来2倍的数组，然后经过rehash 把原先的Node放进新数组中。

下面看下 resize 的具体过程：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 达到最大容量后不再扩容
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        // 如果之前的容量是0，初始化为默认的容量
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // rehash到新的数组
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

大家可以看到 rehash 的部分也是尾插的形式重新生成链表，由此它保证了 Node 的顺序依然是之前链表的顺序，如果是头插呢，顺序就变了。顺序变了会产生什么样的问题呢：可能会导致 resize 死循环。


```
尾插法:
原链表 1->2->3->4
rehash后链表可能是  
    1->2->3
    4

头插法:
原链表 
    1->2->3->4
rehash后链表可能是 
    3->2->1
    4   
```

**头插法中死循环是怎么发生的？**

假设现在是头插法有2个线程，线程A resize 执行遍历到了1，然后线程B执行，执行完了整个 resize，链表变成了 3->2->1。然后线程A回来继续执行，那么它在rehash 1后继续rehash 2，这时候发现2的next是1，这就形成了一个环，所以会一直循环下去直到资源耗尽。而尾插法很好的避免的这个问题，所以即使牺牲了一点点搜索的性能（也没有差很多），解决了这个多线程的问题。

由此你也能看到，hashmap 是线程不安全的。想要用线程安全的 hashmap 就用 ConcurrentHashMap，它给很多操作都加了同步锁，具体细节之后请关注多线程篇。
