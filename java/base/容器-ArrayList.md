
# 容器 - ArrayList

### 简介
ArrayList 本质上就是一个数组，另外就是做了一些额外操作，允许动态的修改大小。

所谓的动态，其实不难理解，就是分离了容量和尺寸的概念，容量是总共能容纳的元素数量，尺寸是实际已经存放的元素个数。对应在源码中，有2个重要的变量：

```java
transient Object[] elementData;

private int size;
```

> transient 关键字表示变量不需要被序列化。

elementData 表示实际存放的元素数据，size 就是实际存放的元素个数。

### 源码
下面我们根据一段常用的代码逐步切入到源码实现：
```java
List<String> a = new ArrayList<>();
a.add("sidfate");
```

首先第一句我们直接 new 了一个 ArrayList 对象，没带参数，进入源码就是：

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

可以看到无参构造函数会初始化我们的 elementData 为空数组，那么 size 也是为0。接着进入第一句的源码：

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

关键函数来了，ensureCapacityInternal 函数会去判断当前的数组需不需要扩容：

```java
// 步骤1
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
// 步骤2
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 如果空数组，返回默认容量 DEFAULT_CAPACITY = 10
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
// 步骤3
private void ensureExplicitCapacity(int minCapacity) {
    // 修改次数+1
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
// 步骤4
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

因为我们传递的 minCapacity 是1，而elementData是空数组，所以elementData.length 为0，因此进入 grow 函数去增加数组的容量。
1. 取出原数组容量 oldCapacity，计算 oldCapacity + (oldCapacity >> 1) 的值为新数组容量 newCapacity。
2. 去 newCapacity 和 newCapacity 的最大值作为新数组的容量。如果新的容量超过了允许的最大容量，那么进入 hugeCapacity 流程。
3. 将原数组拷贝到新容量的数组。

> oldCapacity >> 1 相当于 oldCapacity/2

所以如果按照上面的流程走下来，我们的列表 a 中的变量就变成了：
* elementData: Object[10]{"sidfate"}
* size: 1

由于默认容量是10，当我们插入第11个元素的时候，又会进行一次扩容，此时扩容后的容量为 10 + 10/2 = 15。

了解完 ArrayList 的扩容机制后，我们再来看另一个 add 方法：

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                        size - index);
    elementData[index] = element;
    size++;
}
```

指针 index 的 add 方法我们可以称之为插入，可以看到在插入实际元素前，我们又执行了一次 System.arraycopy，即将 index 后的元素全部后移一位。由此你也可以看出 ArrayList 的插入最坏情况下时间复杂度是 O(N)，并不高。同理删除也是，但是获取元素就很快了，直接取下标值就行，这跟 LinkedList 就不同了，具体可以看下一篇讲 LinkedList 的文章。

另外，ArrayList 被设计为非线程安全的，例如在迭代器中遍历同时修改 ArrayList 会报 ConcurrentModificationException 的异常（并发修改异常），这个就是根据 modCount 这个变量来判断的。有点乐观锁的意思，每当 ArrayList 改变，modCount 递增，多线程下如果在遍历时修改 modCount 会判断是不是与起始值不同，不同就会报错。
