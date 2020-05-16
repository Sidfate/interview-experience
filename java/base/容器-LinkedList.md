# 容器 - LinkedList


### 简介
LinkedList 顾名思义其本质是一个链表，具体来说是一个双向链表，同时还有2个指针分别对应链表的头和尾。

![](https://sidfate.oss-cn-hangzhou.aliyuncs.com/upic/20200222142056-4Abbjm.jpg)

### 源码

源码还是跟 ArrayList 一样，从我们常用的代码出发：
```java
List<String> a = new LinkedList<>();
a.add("sidfate");
```

进入初始化源码：
```java
// LinkedList 长度
transient int size = 0;

// 指向头结点
transient Node<E> first;

// 指向尾节点
transient Node<E> last;

public LinkedList() {
}
```

可以看到默认的构造函数空空如也，需要注意的是链表是以 Node 为基础连接起来的，Node 的结构如下：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

Node 结构也很简单，prev 和 next 说明它是一个双向的链表，保存前后 Node 的指针。接下来进入 add 的过程：

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

简单来说，添加一个元素就是往链表尾插入一个新的 Node。那么如果是往链表中插入一个元素呢：

```java
public void add(int index, E element) {
    // 检查index合法性
    checkPositionIndex(index);

    if (index == size)
        // 如果index就是尾部则直接添加
        linkLast(element);
    else
        // 在 index 所在的 Node 前插入
        linkBefore(element, node(index));
}

Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

node 函数用来查找对应 index 的 Node 节点，比较有意思的是 2 将遍历降低成了链表长度的一半：
* 如果 index 在头和中点之间，那么就从头开始遍历到中点。
* 如果 index 在中点和尾之间，那么就从尾开始遍历到中点。

双向链表的优势就凸显出来的，相对而言插入的效率比 ArrayList 高，毕竟 ArrayList 需要拷贝 index 后的数组。但是查询效率就低了，ArrayList 根据下标能直接取出元素，而且因为是数组所以内存空间上是连续的，不像 LinkedList是根据指针串联，内存地址上可能是不连续的。

LinkedList 的还有个优势时它既能当队列也能当栈来使用。

