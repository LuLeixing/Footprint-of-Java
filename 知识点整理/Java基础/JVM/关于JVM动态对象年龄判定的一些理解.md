# 关于JVM动态对象年龄判定的一些理解

> 菜鸟面试问题之一：MaxTenuringThreshold(类似于对象从新生代到老年代的年龄阈值如何进行设置？)
>
> 一脸懵逼...忘记了有动态对象年龄判定这个概念，于是搜索了一波。
>
> 其实书上对动态对象年龄判定的讲解很少，只是一笔带过的几句概念性描述，上网搜索了下发现了下面这篇博客，感觉正是我想要的答案！

参考链接：<https://my.oschina.net/xpbob/blog/2221709>

> 虚拟机并不是永远地要求对象的年龄必须达到了MaxTenuringThreshold才能晋升老年代，**如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代**，无须等到MaxTenuringThreshold中要求的年龄。

上面这段话摘自深入理解Java虚拟机，讲的主要就是动态年龄的判定。对于动态的判定的条件是相同年龄所有对象大小的总和大于Survivor空间的一半，然后算出的年龄要和MaxTenuringThreshold的值进行比较，以此保障MaxTenuringThreshold设置太大（默认15），导致对象无法晋升。

## 问题的提出

### 情景假设：

如果说非得相同年龄所有对象大小总和大于Survivor空间的一半才能晋升。那么我们看一下下面的场景：

1.MaxTenuringThreshold为15

2.年龄1的对象占用了33%

3.年龄2的对象占用了33%

4.年龄3的对象占用了34%

### 开始推论

1.按照晋升的标准，首先年龄不满足MaxTenuringThreshold，不会晋升。

2.每个年龄的对象都不满足50%，不会晋升。

### 基于假设得到的结论：

Survivor都占用了100%了，但是对象就不晋升。导致老年代明明有空间，但是对象就是停留在年轻代。但这个结论似乎与jvm的表现不符合，只要老年代有空间，最后还是会晋升。

### 问题的解答：

```java
uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
	//survivor_capacity是survivor空间的大小
  size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
  size_t total = 0;
  uint age = 1;
  while (age < table_size) {
    total += sizes[age];//sizes数组是每个年龄段对象大小
    if (total > desired_survivor_size) break;
    age++;
  }
  uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
	...
}
```

**上面这段代码为晋升年龄的计算代码，其中有一个TargetSurvivorRatio的值。**

> **-XX:TargetSurvivorRatio 目标存活率，默认为50%**

1.通过这个比率来算出一个期望值，desired_survivor_size。

2.然后用一个total计数器，累加每个年龄段对象大小的总和。

3.当total大于desired_survivor_size时停止。

4.然后用当前age和MaxTenuringThreshold对比找出最小值作为结果。

**总体表征就是，年龄从小到大进行累加，当加入某个年龄段后，累加和超过Survivor区域*TargetSurvivorRatio的时候，就从这个年龄段往上的年龄的对象进行晋升。**

### 再次推演：

还是上面的场景。年龄1的占用了33%，年龄2的占用了33%，累加和超过默认的TargetSurvivorRatio(50%)，年龄2和3的对象都要进行晋升。

### 小结：

动态对象年龄判断，主要是被TargetSurvivorRatio这个参数来控制。而且算的是年龄从小到大的累加和，而不是某个年龄段对象的大小。看完后记住这个参数，TargetSurvivorRatio。



## 补充说明：

> **-XX:MaxTenuringThreshold**

晋升年龄最大阈值，默认15。在新生代中对象存活次数（经过YGC的次数）后仍然存活，就会晋升到老年代。每经过一次YGC，年龄加1，当survivor区的对象年龄达到TenuringThreshold时，表示该对象是长存活对象，就会直接晋升到老年代。

> **-XX：TargetSurvivorRatio**

设定survivor区的目标使用率，默认是50，即survivor区对象目标使用率为50%。

JVM会将每个对象的年龄信息，各个年龄段对象的总大小记录在"age table"表中，基于"age table"，survivor区大小、survivor区目标使用率（-XX：TargetSurvivorRatio）、晋升年龄阈值（-XX：MaxTenuringThreshold），JVM会动态地计算tenuring threshold的值。一旦对象年龄达到了tenuring threshold就会晋升到老年代。

### 为什么要进行动态计算？

假设有很多年龄还未达到TenuringThreshold的对象依旧停留在survivor区，这样不利于新对象从eden晋升到survivor。因此设置survivor区的目标使用率，当使用率达到时重新调整TenuringThreshold值，让对象尽早的去old区。

如果希望跟踪每次新生代GC后，survivor区中对象的年龄分布，可在启动参数上增加：

> **-XX：+PrintTenuringDistribution**



