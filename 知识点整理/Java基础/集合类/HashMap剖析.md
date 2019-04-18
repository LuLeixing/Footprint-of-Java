### HashMap剖析

![img](https://segmentfault.com/img/remote/1460000014293377?w=941&h=810)

成员属性：

![img](https://segmentfault.com/img/remote/1460000014293378?w=512&h=156)

内部类：

![img](https://segmentfault.com/img/remote/1460000014293379?w=1076&h=764)

我们知道Hash的底层是散列表，而在Java中散列表的实现是通过数组+链表的~

再来简单看看put方法就可以印证我们的说法了：**数组+链表-->散列表**

![img](https://segmentfault.com/img/remote/1460000014293380?w=820&h=86)

简单总结：

- 无序，允许为null，非同步
- 底层由散列表（哈希表）实现
- 初始容量和装载因子对HashMap影响挺大的，设置小了不好，设置大了也不好。

#### HashMap构造方法

![img](https://segmentfault.com/img/remote/1460000014293381?w=480&h=104)

![img](https://segmentfault.com/img/remote/1460000014293382?w=1380&h=581)

在上面的构造方法最后一行，我们会发现调用了`tableSizeFor()`，我们进去看看：

![img](https://segmentfault.com/img/remote/1460000014293383?w=1147&h=479)

这是位运算算法，具体流程可参考：

- <https://www.cnblogs.com/loading4/p/6239441.html>
- <https://blog.csdn.net/fan2012huan/article/details/51097331>

看完上面可能会感到奇怪的是：**为啥是将2的整数幂的数赋给threshold**？

- threshold这个成员变量是阈值，决定了是否要将散列表再散列。它的值应该是：`capacity * load factor`才对的。

其实这里仅仅是一个初始化，当创建哈希表的时候，它会重新赋值的：

![img](https://segmentfault.com/img/remote/1460000014293384?w=1021&h=150)

别的构造方法

![img](https://segmentfault.com/img/remote/1460000014293385?w=1164&h=716)

put方法

put方法是HashMap的核心

![img](https://segmentfault.com/img/remote/1460000014293386?w=1199&h=193)

哈希值计算：

![img](https://segmentfault.com/img/remote/1460000014293387?w=1387&h=579)

![img](https://segmentfault.com/img/remote/1460000014293388?w=1076&h=434)

#### 我们是根据key的哈希值来保存在散列表中的，我们表默认的初始容量是16，要放到散列表中，就是0-15的位置上。也就是`tab[i = (n - 1) & hash]`。可以发现的是：在做`&`运算的时候，仅仅是**后4位有效**~那如果我们key的哈希值高位变化很大，低位变化很小。直接拿过去做`&`运算，这就会导致计算出来的Hash值相同的很多。

而设计者**将key的哈希值的高位也做了运算(与高16位做异或运算，使得在做&运算时，此时的低位实际上是高位与低位的结合)，这就增加了随机性**，减少了碰撞冲突的可能性！

![img](https://segmentfault.com/img/remote/1460000014293389?w=1918&h=1575)

#### resize()

接下来我们看看`resize()`方法，在初始化的时候要调用这个方法，当散列表元素大于`capacity * load factor`的时候也是调用`resize()`

![img](https://segmentfault.com/img/remote/1460000014293391?w=1918&h=2682)

get方法

![img](https://segmentfault.com/img/remote/1460000014293392?w=1106&h=321)

接下来我们看看`getNode()`是怎么实现的：

![img](https://segmentfault.com/img/remote/1460000014293393?w=1453&h=714)

#### remove方法

![img](https://segmentfault.com/img/remote/1460000014293394?w=1060&h=270)

再来看看`removeNode()`的实现：

![img](https://segmentfault.com/img/remote/1460000014293395?w=1918&h=1758)

### HashMap与HashTable的对比

从存储结构和实现来讲基本上都是相同的。它和HashMap的最大的不同是它是线程安全的，**另外它不允许key和value为null。**Hashtable是个过时的集合类，不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

![img](https://segmentfault.com/img/remote/1460000014293396?w=999&h=535)

Hashtable具体阅读源码可参考：

- <https://blog.csdn.net/panweiwei1994/article/details/77427010>
- <https://blog.csdn.net/panweiwei1994/article/details/77428710>

**总结：**

在JDK8中HashMap的底层是：数组+链表（散列表）+红黑树

在散列表中由装载因子这么一个属性，当装载因子*初始容量小于散列表元素时，该散列表会再散列，扩容2倍。

装载因子默认值是0.75，无论是初始大了还是初始小了对我们HashMap的性能都不好。

- 装载因子初始值大了，可以减少散列表再散列（扩容次数），但同时会导致散列冲突的可能性变大（散列冲突也是耗性能的一个操作，要得操作链表（红黑树）。
- 装载因子初始值小了，可以减少散列冲突的可能性，但是同时扩容的次数可能会变多。

初始容量的默认值是16，它也一样，无论是大了还是小了，对我们的HashMap都是有影响的。

- 初始容量过大，那么遍历时速度会有影响。
- 初始容量过小，散列表再散列（扩容的次数）可能就变得多，扩容也是一件耗性能的事情。

从源码上我们可以发现：HashMap并不是直接拿key的哈希值来用的，它会将key的哈希值的高16位进行异或操作，使得我们将元素放入哈希表的时候**增加了一定的随机性**。

还要值得注意的是：**并不是桶子上有8位元素的时候它就能变成红黑树，它得同时满足我们的散列表容量大于64才行的**~

![img](https://segmentfault.com/img/remote/1460000014293397?w=810&h=481)

![img](https://segmentfault.com/img/remote/1460000014293398?w=1113&h=607)



> 参考博客：<https://segmentfault.com/a/1190000014293372>

