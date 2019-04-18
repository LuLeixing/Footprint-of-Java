### Java集合类知识点整理

#### **数组与集合的区别**

**1.长度的区别**

- 数组的长度固定
- 集合的长度可变

**2.内容不同**

- 数组存储的是同一类型的元素
- 集合可以存储不同类型的元素（但是我们一般不这么干...）

**3.元素的数据类型**

- 数组可以存储基本数据类型，也可以存储引用类型
- 集合只能存储引用类型（如果存缩的是简单的int，它会被自动装箱成Integer）

**Collection的由来和功能**

**Collection的由来：**

集合可以存储多个元素，但是我们对多个元素也有不同的需求

- 多个元素，不能有相同的
- 多个元素，能够按照某个规则排序

针对不同的需求：Java就提供了很多集合类，多个集合类的数据结构不同。但是，结构不重要，重要的是能够存储东西，能够判断，获取。

把集合共性的内容不断往上提取，最终形成了集合的继承体系——>Collection

**Collection的大致结构体系是这样的：**

![img](https://user-gold-cdn.xitu.io/2018/4/4/1628e4e1f6a487f2?w=720&h=385&f=jpeg&s=27756)

**精简后，需要重点掌握的为：**

![img](https://user-gold-cdn.xitu.io/2018/4/4/1628e4e1f69d1124?w=1328&h=763&f=png&s=100521)

**基础功能：**

![img](https://user-gold-cdn.xitu.io/2018/4/4/1628e4e1f73ea209?w=1651&h=827&f=png&s=94001)

![img](https://user-gold-cdn.xitu.io/2018/4/4/1628e4e1f75125d8?w=1292&h=578&f=png&s=42527)



#### Collection主要学习的类型有两种：Set和List



#### List集合介绍

**List集合的特点：有序（存储顺序和取出顺序一致），可重复**

1.ArrayList

- 底层数据结构是数组，线程不安全

2.LinkedList

- 底层数据结构是链表，线程不安全

3.Vector

- 底层数据结构是数组，线程安全

#### ArrayList解析

##### 属性：

![img](https://segmentfault.com/img/remote/1460000014240710?w=1323&h=748)

根据上面我们可以清晰的发现：**ArrayList底层其实就是一个数组**，ArrayList中有**扩容**这么一个概念，正因为它扩容，所以它能够**实现“动态”增长**

##### 构造方法：

![img](https://segmentfault.com/img/remote/1460000014240711?w=874&h=717)

##### Add方法：

![img](https://segmentfault.com/img/remote/1460000014240712?w=957&h=642)

**add(E e)**

**步骤：**

- 检查是否需要扩容
- 插入元素

```java
public boolean add(E e) {
    ensureCapapcityInternal(size + 1);
    elementData[size++] = e;
    return true;
}
```

该方法很短，我们可以根据方法名就猜到他是干了什么：

- **确认list容量，尝试容量加1，看看有无必要**
- **添加元素**

接下来我们来看看这个小容量(+1)是否满足我们的需求：

![img](https://segmentfault.com/img/remote/1460000014240713?w=1048&h=237)

随后调用`ensureExplicitCapacity()`来确定明确的容量，我们也来看看这个方法是怎么实现的：

![img](https://segmentfault.com/img/remote/1460000014240714?w=1111&h=260)

所以，接下来看看`grow()`是怎么实现的

![img](https://segmentfault.com/img/remote/1460000014240715?w=1235&h=793)

进去看`copyOf()`方法：

![img](https://segmentfault.com/img/remote/1460000014240716?w=1260&h=290)

到目前为止，我们就可以知道`add(E e)`的基本实现了：

- 首先去检查一下数组的容量是否足够
  - 足够：直接添加
  - 不足够：扩容
    - **扩容到原来的1.5倍**
    - 第一次扩容后，如果容量还是小于minCapacity，就将容量扩充为minCapacity。



##### add(int index, E element)

步骤：

- 检查角标
- 空间检查，如果有需要进行扩容
- 插入元素

![img](https://segmentfault.com/img/remote/1460000014240717?w=1099&h=499)

我们发现，与扩容相关ArrayList的add方法底层其实都是`arraycopy()`来实现的

看到`arraycopy()`，我们可以发现：**该方法是由C/C++来编写的**，是个本地方法，并不是由Java实现：

![img](https://segmentfault.com/img/remote/1460000014240718?w=1024&h=126)

总的来说：`arraycopy()`还是比较可靠高效的一个方法。

##### get方法

- 检查角标
- 返回元素

![img](https://segmentfault.com/img/remote/1460000014240719?w=963&h=337)

##### set方法

- 检查角标
- 替代元素
- 返回旧值

![img](https://segmentfault.com/img/remote/1460000014240720?w=1013&h=453)

##### remove方法

步骤：

- 检查角标
- 删除元素
- 计算出需要移动的个数，并移动
- 设置为null，让GC回收

![img](https://segmentfault.com/img/remote/1460000014240721?w=1163&h=638)

##### 补充说明:

- ArrayList是基于动态数组实现的，在增删的时候，需要数组的拷贝复制。
- ArrayList的默认初始化容量是10，每次扩容时候增加原来容量的一般，也就是变为原来的1.5倍。
- 删除元素时不会减少容量，若希望减少容量则调用trimToSize()。
- 它不是线程安全的，它能存放null值。

![img](https://segmentfault.com/img/remote/1460000014240722?w=1060&h=378)

#### Vector与ArrayList的区别

- Vector是jdk1.2的类，比较老
- Vector底层实现也是数组，最大区别是 “同步”（线程安全）
- Vector是同步的，我们可以从方法上就可以看得出来。
- 在要求非同步的情况下，我们一般都是使用ArrayList来替代Vector了。
- 如果想要ArrayList实现同步，可以使用Collections的方法：`List list = Collections.synchronizedList(new ArrayList(...));`，就可以实现同步了~
- ArrayList在底层数组不够用时在原来的基础上扩展0.5倍，Vector是扩展1倍。

### LinkedList解析

LinkedList底层是**双向链表**

从结构上，我们还看到了**LinkedList实现了Deque接口**，因此，我们可以**操作LinkedList像操作队列和栈一样**。

LinkedList变量就这么几个，因为我们操作单向链表的时候也发现了：有了头结点，其他的数据我们都可以获取得到了。(双向链表也同理)

![img](https://segmentfault.com/img/remote/1460000014240730?w=942&h=458)

##### 构造方法

构造方法有两个

![img](https://segmentfault.com/img/remote/1460000014240731?w=980&h=431)



##### add方法

- **add方法实际上就是往链表最后添加元素**

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

##### remove方法

![img](https://segmentfault.com/img/remote/1460000014240732)

![img](https://segmentfault.com/img/remote/1460000014240733?w=494&h=604)

实际上就是下面那个图的操作：

![img](https://segmentfault.com/img/remote/1460000014240734?w=393&h=167)

##### get方法

可以看到get方法实现就两段代码：

```java
 public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

![img](https://segmentfault.com/img/remote/1460000014240735?w=892&h=581)

##### set方法

set方法和get方法其实差不多，**根据下标来判断是从头遍历还是从尾遍历**

```java
 public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```

##### 参考链接：

- <https://blog.csdn.net/panweiwei1994/article/details/77110354>
- <https://zhuanlan.zhihu.com/p/24730576>
- <https://zhuanlan.zhihu.com/p/28373321>

##### 总结：

ArrayList：

- 底层实现是数组
- ArrayList默认初始化容量是10，每次扩容变成原来的1.5倍
- 增删的时候，需要数组的拷贝复制（native方法由C/C++实现）

LinkedList:

- 底层实现是双向链表【双向链表方便实现往前遍历】

Vector：

底层是数组，现在已少用，被ArrayList代替，原因有两个:

- Vector所有方法都是同步，有性能损失
- Vector初始length是10，超过容量时，扩容100%，相比于ArrayList消耗更多内存

总的来说，查询多用ArrayList，增删多用LinkedList

ArrayList增删慢不是绝对的（在数据量大的情况下



### Set集合介绍

**Set集合的特点是：元素不可重复**

1.HashSet

- 底层数据结构是哈希表（是一个元素为链表的数组）

2.TreeSet

- 底层数据结构是红黑树（是一个自平衡的二叉查找树）
- 保证元素的排序方式（有序）

3.LinkedHashSet

- 底层数据结构由哈希表和链表组成

#### HashSet剖析

![img](https://segmentfault.com/img/remote/1460000014391406?w=1918&h=1920)

要点：

- 实现Set接口
- 不保证迭代顺序
- 允许元素为null
- 底层实际上是一个HashMap实例
- 非同步
- 初始容量非常影响迭代性能

![img](https://segmentfault.com/img/remote/1460000014391407?w=1127&h=191)

方法和属性：

![img](https://segmentfault.com/img/remote/1460000014391408?w=1706&h=734)

我们知道Map是一个映射，有key有value，**既然HashSet底层用的是HashMap，那么value在哪里呢**？？？

![img](https://segmentfault.com/img/remote/1460000014391409?w=940&h=100)

value是一个Object，**所有的value都是它**

所以可以直接总结出：HashSet实际上就是封装了HashMap，**操作HashSet元素实际上就是操作HashMap**。这也是面向对象的一种体现，**重用性贼高**！



#### TreeSet剖析

![img](https://segmentfault.com/img/remote/1460000014391411?w=1918&h=1920)

从顶部注释来看，我们就可以归纳TreeSet的要点了：

- 实现NavigableSet接口
- 可以实现排序功能
- **底层实际上是一个TreeMap实例**
- 非同步

![img](https://segmentfault.com/img/remote/1460000014391412?w=1382&h=704)



#### LinkedHashSet剖析

![img](https://segmentfault.com/img/remote/1460000014391414?w=1918&h=2649)

从顶部注释来看，我们就可以归纳LinkedHashSet的要点了：

- 迭代是有序的
- 允许为null
- **底层实际上是一个HashMap+双向链表实例(其实就是LinkedHashMap)...**
- 非同步
- 性能比HashSet差一丢丢，因为要维护一个双向链表
- 初始容量与迭代无关，LinkedHashSet迭代的是双向链表

总结：

Set集合的底层就是Map

三个常用子类：

HashSet：无序，允许为null，底层是HashMap(散列表+红黑树)，非线程同步。

TreeSet：有序，不允许为null，底层是TreeMap(红黑树)，非线程同步。

LinkedHashSet：迭代有序，允许为null，底层是HashMap+双向链表，非线程同步。

参考资料：

- <https://zhuanlan.zhihu.com/p/29021276>
- <https://blog.csdn.net/panweiwei1994/article/details/76555359>