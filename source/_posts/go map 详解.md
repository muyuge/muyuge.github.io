---
title: go详解
date: 2020-12-25 00:39:29
tags:
---

## go map详解

**
**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUusfcAq5hCK7rsyTmqGH6yrEqo7aQbsStmOwUWdDyMba3FTiby2z3n2iar8QCxHL1N8umcWRfU3BUyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**什么是Map**

- **维基百科的定义**

*In computer science, an associative array, map, symbol table, or dictionary is an abstract data type composed of a collection of (key, value) pairs, such that each possible key appears at most once in the collection.*

说明：在计算机科学中，包含键值对（key-value）集合的抽象数据结构（关联数组、符号表或字典），其每个可能的键在该集合中最多出现一次，这样的数据结构就是一种Map。



**01**

**操作**

对Map的操作主要是增删改查：

- 在集合中增加键值对

- 在集合中移除键值对

- 修改某个存在的键值对

- 根据特定的键寻找对应的值

  





**02**

**实现**

Map的实现主要有两种方式：哈希表（hash table）和搜索树（search tree）。例如Java中的hashMap是基于哈希表实现，而C++中的Map是基于一种平衡搜索二叉树——红黑树而实现的。以下是不同实现方式的时间复杂度对比。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUtUb8P4XltZLgRwISYwA9K7iaMxwGaX6MbhZ8aYar6X8Vr6aZ2wqjJR3MhHvpYEfXZFplKicWDTxrsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到，对于元素查找而言，二叉搜索树的平均和最坏效率都是O(log n)，哈希表实现的平均效率是O(1)，但最坏情况下能达到O(n)，不过如果哈希表设计优秀，最坏情况基本不会出现（所以，读者不想知道Go是如何设计的Map吗）。另外二叉搜索树返回的key是有序的，而哈希表则是乱序。





**哈希表**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 

由于Go中map的基于哈希表（也被称为散列表）实现，本文不探讨搜索树的map实现。以下是Go官方博客对map的说明。

*One of the most useful data structures in computer science is the hash table. Many hash table implementations exist with varying properties, but in general they offer fast lookups, adds, and deletes. Go provides a built-in map type that implements a hash table.*

学习哈希表首先要理解两个概念：哈希函数和哈希冲突。



**01**

**哈希函数**

哈希函数（常被称为散列函数）是可以用于将任意大小的数据映射到固定大小值的函数，常见的包括MD5、SHA系列等。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUtUb8P4XltZLgRwISYwA9K7kD2Cjht6V545ibeDelo4cDIyyg5pqk0qibosIZnNUqldXF3HA6icvibzmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一个设计优秀的哈希函数应该包含以下特性：

- **均匀性**：一个好的哈希函数应该在其输出范围内尽可能均匀地映射，也就是说，应以大致相同的概率生成输出范围内的每个哈希值。
- **效率高**：哈希效率要高，即使很长的输入参数也能快速计算出哈希值。
- **可确定性**：哈希过程必须是确定性的，这意味着对于给定的输入值，它必须始终生成相同的哈希值。
- **雪崩效应**：微小的输入值变化也会让输出值发生巨大的变化。
- **不可逆**：从哈希函数的输出值不可反向推导出原始的数据。





**02**

**哈希冲突**

重复一遍，哈希函数是将任意大小的数据映射到固定大小值的函数。那么，我们可以预见到，即使哈希函数设计得足够优秀，几乎每个输入值都能映射为不同的哈希值。但是，当输入数据足够大，大到能超过固定大小值的组合能表达的最大数量数，冲突将不可避免！（参见抽屉原理）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUtUb8P4XltZLgRwISYwA9K76UYWdqvlq1WTPDYMQq1t7IuUU37CqORswiahpuNLfkfO5jBDJsyZ56A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

抽屉原理：桌上有十个苹果，要把这十个苹果放到九个抽屉里，无论怎样放，我们会发现至少会有一个抽屉里面放不少于两个苹果。抽屉原理有时也被称为鸽巢原理。

- ##### 如何解决哈希冲突

比较常用的Has冲突解决方案有链地址法和开放寻址法。

在讲链地址法之前，先说明两个概念。

1. 哈希桶。哈希桶（也称为槽，类似于抽屉原理中的一个抽屉）可以先简单理解为一个哈希值，所有的哈希值组成了哈希空间。
2. 装载因子。装载因子是表示哈希表中元素的填满程度。它的计算公式：装载因子=填入哈希表中的元素个数/哈希表的长度。装载因子越大，填入的元素越多，空间利用率就越高，但发生哈希冲突的几率就变大。反之，装载因子越小，填入的元素越少，冲突发生的几率减小，但空间浪费也会变得更多，而且还会提高扩容操作的次数。装载因子也是决定哈希表是否进行扩容的关键指标，在java的HashMap的中，其默认装载因子为0.75；Python的dict默认装载因子为2/3。



**A**

**链地址法**

链地址法的思想就是将映射在一个桶里的所有元素用链表串起来。

下面以一个简单的哈希函数 `H(key) = key MOD 7`（除数取余法）对一组元素 `[50, 700, 76, 85, 92, 73, 101]` 进行映射，通过图示来理解链地址法处理Hash冲突的处理逻辑。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

链地址法解决冲突的方式与图的邻接表存储方式在样式上很相似，发生冲突，就用单链表组织起来。



**B**

**开放寻址法**

###### 

对于链地址法而言，槽位数m与键的数目n是没有直接关系的。但是对于开放寻址法而言，所有的元素都是存储在Hash表当中的，所以无论任何时候都要保证哈希表的槽位数m大于或等于键的数据n（必要时，需要对哈希表进行动态扩容）。

开放寻址法有多种方式：线性探测法、平方探测法、随机探测法和双重哈希法。这里以线性探测法来帮助读者理解开放寻址法思想。

- **线性探测法**

设 `Hash(key)` 表示关键字 `key` 的哈希值， 表示哈希表的槽位数（哈希表的大小）。

线性探测法则可以表示为：

如果 `Hash(x) % M` 已经有数据，则尝试 `(Hash(x) + 1) % M` ;

如果 `(Hash(x) + 1) % M` 也有数据了，则尝试 `(Hash(x) + 2) % M` ;

如果 `(Hash(x) + 2) % M` 也有数据了，则尝试 `(Hash(x) + 3) % M` ;

……

我们同样以哈希函数 `H(key) = key MOD 7` （除数取余法）对`[50, 700, 76, 85, 92, 73, 101]` 进行映射，通过图示来理解线性探测法处理 Hash 碰撞。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

其中，empty代表槽位为空，occupied代表槽位已被占（后续映射到该槽，则需要线性向下继续探测），而lazy delete则代表将槽位里面的数据清除，并不释放存储空间。



**C**

**两种解决方案比较**

对于开放寻址法而言，它只有数组一种数据结构就可完成存储，继承了数组的优点，对CPU缓存友好，易于序列化操作。但是它对内存的利用率不如链地址法，且发生冲突时代价更高。当数据量明确、装载因子小，适合采用开放寻址法。

链表节点可以在需要时再创建，不必像开放寻址法那样事先申请好足够内存，因此链地址法对于内存的利用率会比开方寻址法高。链地址法对装载因子的容忍度会更高，并且适合存储大对象、大数据量的哈希表。而且相较于开放寻址法，它更加灵活，支持更多的优化策略，比如可采用红黑树代替链表。但是链地址法需要额外的空间来存储指针。

值得一提的是，在Python中dict在发生哈希冲突时采用的开放寻址法，而java的HashMap采用的是链地址法。





**Go Map 实现**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

同python与java一样，Go语言中的map是也基于哈希表实现的，它解决哈希冲突的方式是链地址法，即通过使用数组+链表的数据结构来表达map。

注意：本文后续出现的map统一代指Go中实现的map类型。



**01**

**map数据结构**

map中的数据被存放于一个数组中的，数组的元素是桶（bucket），每个桶至多包含8个键值对数据。**哈希值低位（low-order bits）用于选择桶，哈希值高位（high-order bits）用于在一个独立的桶中区别出键****。**哈希值高低位示意图如下

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

本文基于go 1.15.2 darwin/amd64分析，源码位于`src/runtime/map.go`.

- **map的结构体为hmap**

```
 1// A header for a Go map.
 2type hmap struct {
 3    count     int // 代表哈希表中的元素个数，调用len(map)时，返回的就是该字段值。
 4    flags     uint8 // 状态标志，下文常量中会解释四种状态位含义。
 5    B         uint8  // buckets（桶）的对数log_2（哈希表元素数量最大可达到装载因子*2^B）
 6    noverflow uint16 // 溢出桶的大概数量。
 7    hash0     uint32 // 哈希种子。
 8
 9    buckets    unsafe.Pointer // 指向buckets数组的指针，数组大小为2^B，如果元素个数为0，它为nil。
10    oldbuckets unsafe.Pointer // 如果发生扩容，oldbuckets是指向老的buckets数组的指针，老的buckets数组大小是新的buckets的1/2。非扩容状态下，它为nil。
11    nevacuate  uintptr        // 表示扩容进度，小于此地址的buckets代表已搬迁完成。
12
13    extra *mapextra // 这个字段是为了优化GC扫描而设计的。当key和value均不包含指针，并且都可以inline时使用。extra是指向mapextra类型的指针。
```



- **mapextra的结构体**

```
 1// mapextra holds fields that are not present on all maps.
 2type mapextra struct {
 3    // 如果 key 和 value 都不包含指针，并且可以被 inline(<=128 字节)
 4    // 就使用 hmap的extra字段 来存储 overflow buckets，这样可以避免 GC 扫描整个 map
 5    // 然而 bmap.overflow 也是个指针。这时候我们只能把这些 overflow 的指针
 6    // 都放在 hmap.extra.overflow 和 hmap.extra.oldoverflow 中了
 7    // overflow 包含的是 hmap.buckets 的 overflow 的 buckets
 8    // oldoverflow 包含扩容时的 hmap.oldbuckets 的 overflow 的 bucket
 9    overflow    *[]*bmap
10    oldoverflow *[]*bmap
11
12    // 指向空闲的 overflow bucket 的指针
13    nextOverflow *bmap
14}
```



- **bmap结构体**

```
1// A bucket for a Go map.
2type bmap struct {
3    // tophash包含此桶中每个键的哈希值最高字节（高8位）信息（也就是前面所述的high-order bits）。
4    // 如果tophash[0] < minTopHash，tophash[0]则代表桶的搬迁（evacuation）状态。
5    tophash [bucketCnt]uint8
6}
```

bmap也就是bucket（桶）的内存模型图解如下（相关代码逻辑可查看`src/cmd/compile/internal/gc/reflect.go`中的`bmap()`函数）。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在以上图解示例中，该桶的第7位cell和第8位cell还未有对应键值对。需要注意的是，key和value是各自存储起来的，并非想象中的key/value/key/value…的形式。这样做虽然会让代码组织稍显复杂，但是它的好处是能让我们消除例如map[int64]int所需要的填充（padding）。此外，在8个键值对数据后面有一个overflow指针，因为桶中最多只能装8个键值对，如果有多余的键值对落到了当前桶，那么就需要再构建一个桶（称为溢出桶），通过overflow指针链接起来。

- **重要常量标志**

```
 1const (
 2    // 一个桶中最多能装载的键值对（key-value）的个数为8
 3    bucketCntBits = 3
 4    bucketCnt     = 1 << bucketCntBits
 5
 6  // 触发扩容的装载因子为13/2=6.5
 7    loadFactorNum = 13
 8    loadFactorDen = 2
 9
10    // 键和值超过128个字节，就会被转换为指针
11    maxKeySize  = 128
12    maxElemSize = 128
13
14    // 数据偏移量应该是bmap结构体的大小，它需要正确地对齐。
15  // 对于amd64p32而言，这意味着：即使指针是32位的，也是64位对齐。
16    dataOffset = unsafe.Offsetof(struct {
17        b bmap
18        v int64
19    }{}.v)
20
21
22  // 每个桶（如果有溢出，则包含它的overflow的链桶）在搬迁完成状态（evacuated* states）下，要么会包含它所有的键值对，要么一个都不包含（但不包括调用evacuate()方法阶段，该方法调用只会在对map发起write时发生，在该阶段其他goroutine是无法查看该map的）。简单的说，桶里的数据要么一起搬走，要么一个都还未搬。
23  // tophash除了放置正常的高8位hash值，还会存储一些特殊状态值（标志该cell的搬迁状态）。正常的tophash值，最小应该是5，以下列出的就是一些特殊状态值。
24    emptyRest      = 0 // 表示cell为空，并且比它高索引位的cell或者overflows中的cell都是空的。（初始化bucket时，就是该状态）
25    emptyOne       = 1 // 空的cell，cell已经被搬迁到新的bucket
26    evacuatedX     = 2 // 键值对已经搬迁完毕，key在新buckets数组的前半部分
27    evacuatedY     = 3 // 键值对已经搬迁完毕，key在新buckets数组的后半部分
28    evacuatedEmpty = 4 // cell为空，整个bucket已经搬迁完毕
29    minTopHash     = 5 // tophash的最小正常值
30
31    // flags
32    iterator     = 1 // 可能有迭代器在使用buckets
33    oldIterator  = 2 // 可能有迭代器在使用oldbuckets
34    hashWriting  = 4 // 有协程正在向map写人key
35    sameSizeGrow = 8 // 等量扩容
36
37    // 用于迭代器检查的bucket ID
38    noCheck = 1<<(8*sys.PtrSize) - 1
39)
```

综上，我们以B等于4为例，展示一个完整的map结构图。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUtUb8P4XltZLgRwISYwA9K7iamm1g0XhhrpEic2YJ4boOaNBHZpndezNgx1EyHVoTAP823FEmTUpfjA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 

**02**

**创建map**

map初始化有以下两种方式

```
1make(map[k]v)
2// 指定初始化map大小为hint
3make(map[k]v, hint)
```

对于不指定初始化大小，和初始化值hint<=8（bucketCnt）时，go会调用`makemap_small`函数（源码位置`src/runtime/map.go`），并直接从堆上进行分配。

```
1func makemap_small() *hmap {
2    h := new(hmap)
3    h.hash0 = fastrand()
4    return h
5}
```

当hint>8时，则调用`makemap`函数

```
 1// 如果编译器认为map和第一个bucket可以直接创建在栈上，h和bucket可能都是非空
 2// 如果h != nil，那么map可以直接在h中创建
 3// 如果h.buckets != nil，那么h指向的bucket可以作为map的第一个bucket使用
 4func makemap(t *maptype, hint int, h *hmap) *hmap {
 5  // math.MulUintptr返回hint与t.bucket.size的乘积，并判断该乘积是否溢出。
 6    mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
 7// maxAlloc的值，根据平台系统的差异而不同，具体计算方式参照src/runtime/malloc.go
 8    if overflow || mem > maxAlloc {
 9        hint = 0
10    }
11
12// initialize Hmap
13    if h == nil {
14        h = new(hmap)
15    }
16  // 通过fastrand得到哈希种子
17    h.hash0 = fastrand()
18
19    // 根据输入的元素个数hint，找到能装下这些元素的B值
20    B := uint8(0)
21    for overLoadFactor(hint, B) {
22        B++
23    }
24    h.B = B
25
26    // 分配初始哈希表
27  // 如果B为0，那么buckets字段后续会在mapassign方法中lazily分配
28    if h.B != 0 {
29        var nextOverflow *bmap
30    // makeBucketArray创建一个map的底层保存buckets的数组，它最少会分配h.B^2的大小。
31        h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
32        if nextOverflow != nil {
33    h.extra = new(mapextra)
34            h.extra.nextOverflow = nextOverflow
35        }
36    }
37
38    return h
39}
```

分配buckets数组的`makeBucketArray`函数

```
 1// makeBucket为map创建用于保存buckets的数组。
 2func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap) {
 3    base := bucketShift(b)
 4    nbuckets := base
 5  // 对于小的b值（小于4），即桶的数量小于16时，使用溢出桶的可能性很小。对于此情况，就避免计算开销。
 6    if b >= 4 {
 7    // 当桶的数量大于等于16个时，正常情况下就会额外创建2^(b-4)个溢出桶
 8        nbuckets += bucketShift(b - 4)
 9        sz := t.bucket.size * nbuckets
10        up := roundupsize(sz)
11        if up != sz {
12            nbuckets = up / t.bucket.size
13        }
14    }
15
16 // 这里，dirtyalloc分两种情况。如果它为nil，则会分配一个新的底层数组。如果它不为nil，则它指向的是曾经分配过的底层数组，该底层数组是由之前同样的t和b参数通过makeBucketArray分配的，如果数组不为空，需要把该数组之前的数据清空并复用。
17    if dirtyalloc == nil {
18        buckets = newarray(t.bucket, int(nbuckets))
19    } else {
20        buckets = dirtyalloc
21        size := t.bucket.size * nbuckets
22        if t.bucket.ptrdata != 0 {
23            memclrHasPointers(buckets, size)
24        } else {
25            memclrNoHeapPointers(buckets, size)
26        }
27}
28
29  // 即b大于等于4的情况下，会预分配一些溢出桶。
30  // 为了把跟踪这些溢出桶的开销降至最低，使用了以下约定：
31  // 如果预分配的溢出桶的overflow指针为nil，那么可以通过指针碰撞（bumping the pointer）获得更多可用桶。
32  // （关于指针碰撞：假设内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”）
33  // 对于最后一个溢出桶，需要一个安全的非nil指针指向它。
34    if base != nbuckets {
35        nextOverflow = (*bmap)(add(buckets, base*uintptr(t.bucketsize)))
36        last := (*bmap)(add(buckets, (nbuckets-1)*uintptr(t.bucketsize)))
37        last.setoverflow(t, (*bmap)(buckets))
38    }
39    return buckets, nextOverflow
40}
```

根据上述代码，我们能确定在正常情况下，正常桶和溢出桶在内存中的存储空间是连续的，只是被`hmap` 中的不同字段引用而已。



**03**

**哈希函数**

在初始化go程序运行环境时（`src/runtime/proc.go`中的`schedinit`），就需要通过`alginit`方法完成对哈希的初始化。

```
 1func schedinit() {
 2    lockInit(&sched.lock, lockRankSched)
 3
 4    ...
 5
 6    tracebackinit()
 7    moduledataverify()
 8    stackinit()
 9    mallocinit()
10    fastrandinit() // must run before mcommoninit
11    mcommoninit(_g_.m, -1)
12    cpuinit()       // must run before alginit
13    // 这里调用alginit()
14    alginit()       // maps must not be used before this call
15    modulesinit()   // provides activeModules
16    typelinksinit() // uses maps, activeModules
17    itabsinit()     // uses activeModules
18
19    ...
20
21    goargs()
22    goenvs()
23    parsedebugvars()
24    gcinit()
25
26  ...
27}
```

对于哈希算法的选择，程序会根据当前架构判断是否支持AES，如果支持就使用AES hash，其实现代码位于`src/runtime/asm_{386,amd64,arm64}.s`中；若不支持，其hash算法则根据xxhash算法（https://code.google.com/p/xxhash/）和cityhash算法（https://code.google.com/p/cityhash/）启发而来，代码分别对应于32位（`src/runtime/hash32.go`）和64位机器（`src/runtime/hash32.go`）中，对这部分内容感兴趣的读者可以深入研究。

```
 1func alginit() {
 2    // Install AES hash algorithms if the instructions needed are present.
 3    if (GOARCH == "386" || GOARCH == "amd64") &&
 4        cpu.X86.HasAES && // AESENC
 5        cpu.X86.HasSSSE3 && // PSHUFB
 6        cpu.X86.HasSSE41 { // PINSR{D,Q}
 7        initAlgAES()
 8        return
 9    }
10    if GOARCH == "arm64" && cpu.ARM64.HasAES {
11        initAlgAES()
12        return
13    }
14    getRandomData((*[len(hashkey) * sys.PtrSize]byte)(unsafe.Pointer(&hashkey))[:])
15    hashkey[0] |= 1 // make sure these numbers are odd
16    hashkey[1] |= 1
17    hashkey[2] |= 1
18    hashkey[3] |= 1
19}
```

上文在创建map的时候，我们可以知道map的哈希种子是通过`h.hash0 = fastrand()`得到的。它是在以下`maptype`中的`hasher`中被使用到，在下文内容中会看到hash值的生成。

```go
 1type maptype struct {
 2    typ    _type
 3    key    *_type
 4    elem   *_type
 5    bucket *_type
 6  // hasher的第一个参数就是指向key的指针，h.hash0 = fastrand()得到的hash0，就是hasher方法的第二个参数。
 7  // hasher方法返回的就是hash值。
 8    hasher     func(unsafe.Pointer, uintptr) uintptr
 9    keysize    uint8  // size of key slot
10    elemsize   uint8  // size of elem slot
11    bucketsize uint16 // size of bucket
12    flags      uint32
13}
14
```





**04**

**map操作**

假定key经过哈希计算后得到64bit位的哈希值。如果B=5，buckets数组的长度，即桶的数量是32（2的5次方）。

例如，现要置一key于map中，该key经过哈希后，得到的哈希值如下：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUtUb8P4XltZLgRwISYwA9K7Umg4kaUq45vcRpeZGFuTnniapQkibEIU8gX14rCBWshors43rua0TqrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

前面我们知道，哈希值低位（`low-order bits`）用于选择桶，哈希值高位（`high-order bits`）用于在一个独立的桶中区别出键。当B等于5时，那么我们选择的哈希值低位也是5位，即01010，它的十进制值为10，代表10号桶。再用哈希值的高8位，找到此key在桶中的位置。最开始桶中还没有key，那么新加入的key和value就会被放入第一个key空位和value空位。

注意：对于高低位的选择，该操作的实质是取余，但是取余开销很大，在实际代码实现中采用的是位操作。以下是tophash的实现代码。

```
1func tophash(hash uintptr) uint8 {
2    top := uint8(hash >> (sys.PtrSize*8 - 8))
3    if top < minTopHash {
4        top += minTopHash
5    }
6    return top
7}
```

当两个不同的key落在了同一个桶中，这时就发生了哈希冲突。go的解决方式是链地址法（这里为了让读者更好理解，只描述非扩容且该key是第一次添加的情况）：在桶中按照顺序寻到第一个空位并记录下来，后续在该桶和它的溢出桶中均未发现存在的该key，将key置于第一个空位；否则，去该桶的溢出桶中寻找空位，如果没有溢出桶，则添加溢出桶，并将其置溢出桶的第一个空位（因为是第一次添加，所以不描述已存在该key的情况）。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

上图中的B值为5，所以桶的数量为32。通过哈希函数计算出待插入key的哈希值，低5位哈希00110，对应于6号桶；高8位10010111，十进制为151，由于桶中前6个cell已经有正常哈希值填充了(遍历)，所以将151对应的高位哈希值放置于第7位cell（第8个cell为empty Rest，表明它还未使用），对应将key和value分别置于相应的第七个空位。

如果是查找key，那么我们会根据高位哈希值去桶中的每个cell中找，若在桶中没找到，并且overflow不为nil，那么继续去溢出桶中寻找，直至找到，如果所有的cell都找过了，还未找到，则返回key类型的默认值（例如key是int类型，则返回0）。



**A**

**查找Key**

对于map的元素查找，其源码实现如下

```
 1func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
 2  // 如果开启了竞态检测 -race
 3    if raceenabled && h != nil {
 4        callerpc := getcallerpc()
 5        pc := funcPC(mapaccess1)
 6        racereadpc(unsafe.Pointer(h), callerpc, pc)
 7        raceReadObjectPC(t.key, key, callerpc, pc)
 8    }
 9  // 如果开启了memory sanitizer -msan
10    if msanenabled && h != nil {
11        msanread(key, t.key.size)
12    }
13  // 如果map为空或者元素个数为0，返回零值
14    if h == nil || h.count == 0 {
15        if t.hashMightPanic() {
16            t.hasher(key, 0) // see issue 23734
17        }
18        return unsafe.Pointer(&zeroVal[0])
19    }
20  // 注意，这里是按位与操作
21  // 当h.flags对应的值为hashWriting（代表有其他goroutine正在往map中写key）时，那么位计算的结果不为0，因此抛出以下错误。
22  // 这也表明，go的map是非并发安全的
23    if h.flags&hashWriting != 0 {
24        throw("concurrent map read and map write")
25    }
26  // 不同类型的key，会使用不同的hash算法，可详见src/runtime/alg.go中typehash函数中的逻辑
27    hash := t.hasher(key, uintptr(h.hash0))
28    m := bucketMask(h.B)
29  // 按位与操作，找到对应的bucket
30    b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
31  // 如果oldbuckets不为空，那么证明map发生了扩容
32  // 如果有扩容发生，老的buckets中的数据可能还未搬迁至新的buckets里
33  // 所以需要先在老的buckets中找
34    if c := h.oldbuckets; c != nil {
35        if !h.sameSizeGrow() {
36            m >>= 1
37        }
38        oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
39    // 如果在oldbuckets中tophash[0]的值，为evacuatedX、evacuatedY，evacuatedEmpty其中之一
40    // 则evacuated()返回为true，代表搬迁完成。
41    // 因此，只有当搬迁未完成时，才会从此oldbucket中遍历
42        if !evacuated(oldb) {
43            b = oldb
44        }
45    }
46  // 取出当前key值的tophash值
47    top := tophash(hash)
48  // 以下是查找的核心逻辑
49  // 双重循环遍历：外层循环是从桶到溢出桶遍历；内层是桶中的cell遍历
50  // 跳出循环的条件有三种：第一种是已经找到key值；第二种是当前桶再无溢出桶；
51  // 第三种是当前桶中有cell位的tophash值是emptyRest，这个值在前面解释过，它代表此时的桶后面的cell还未利用，所以无需再继续遍历。
52bucketloop:
53    for ; b != nil; b = b.overflow(t) {
54        for i := uintptr(0); i < bucketCnt; i++ {
55      // 判断tophash值是否相等
56            if b.tophash[i] != top {
57                if b.tophash[i] == emptyRest {
58                    break bucketloop
59                }
60                continue
61      }
62      // 因为在bucket中key是用连续的存储空间存储的，因此可以通过bucket地址+数据偏移量（bmap结构体的大小）+ keysize的大小，得到k的地址
63      // 同理，value的地址也是相似的计算方法，只是再要加上8个keysize的内存地址
64            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
65            if t.indirectkey() {
66                k = *((*unsafe.Pointer)(k))
67            }
68      // 判断key是否相等
69            if t.key.equal(key, k) {
70                e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
71                if t.indirectelem() {
72                    e = *((*unsafe.Pointer)(e))
73                }
74                return e
75            }
76        }
77    }
78  // 所有的bucket都未找到，则返回零值
79    return unsafe.Pointer(&zeroVal[0])
80}
```

以下是`mapaccess1`的查找过程图解

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

map的元素查找，对应go代码有两种形式

```
1    // 形式一
2    v := m[k]
3    // 形式二
4    v, ok := m[k]
```

形式一的代码实现，就是上述的`mapaccess1`方法。此外，在源码中还有个`mapaccess2`方法，它的函数签名如下。

```
1func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {}
```

与`mapaccess1`相比，`mapaccess2`多了一个bool类型的返回值，它代表的是是否在map中找到了对应的key。因为和`mapaccess1`基本一致，所以详细代码就不再贴出。

同时，源码中还有mapaccessK方法，它的函数签名如下。

```
1func mapaccessK(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, unsafe.Pointer) {}
```

与`mapaccess1`相比，`mapaccessK`同时返回了key和value，其代码逻辑也一致。



**B**

**赋值Key**

对于写入key的逻辑，其源码实现如下

```
  1func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
  2  // 如果h是空指针，赋值会引起panic
  3  // 例如以下语句
  4  // var m map[string]int
  5    // m["k"] = 1
  6    if h == nil {
  7        panic(plainError("assignment to entry in nil map"))
  8    }
  9  // 如果开启了竞态检测 -race
 10    if raceenabled {
 11        callerpc := getcallerpc()
 12        pc := funcPC(mapassign)
 13        racewritepc(unsafe.Pointer(h), callerpc, pc)
 14        raceReadObjectPC(t.key, key, callerpc, pc)
 15    }
 16  // 如果开启了memory sanitizer -msan
 17    if msanenabled {
 18        msanread(key, t.key.size)
 19    }
 20  // 有其他goroutine正在往map中写key，会抛出以下错误
 21    if h.flags&hashWriting != 0 {
 22        throw("concurrent map writes")
 23    }
 24  // 通过key和哈希种子，算出对应哈希值
 25    hash := t.hasher(key, uintptr(h.hash0))
 26
 27  // 将flags的值与hashWriting做按位或运算
 28  // 因为在当前goroutine可能还未完成key的写入，再次调用t.hasher会发生panic。
 29    h.flags ^= hashWriting
 30
 31    if h.buckets == nil {
 32        h.buckets = newobject(t.bucket) // newarray(t.bucket, 1)
 33}
 34
 35again:
 36  // bucketMask返回值是2的B次方减1
 37  // 因此，通过hash值与bucketMask返回值做按位与操作，返回的在buckets数组中的第几号桶
 38    bucket := hash & bucketMask(h.B)
 39  // 如果map正在搬迁（即h.oldbuckets != nil）中,则先进行搬迁工作。
 40    if h.growing() {
 41        growWork(t, h, bucket)
 42    }
 43  // 计算出上面求出的第几号bucket的内存位置
 44  // post = start + bucketNumber * bucketsize
 45    b := (*bmap)(unsafe.Pointer(uintptr(h.buckets) + bucket*uintptr(t.bucketsize)))
 46    top := tophash(hash)
 47
 48    var inserti *uint8
 49    var insertk unsafe.Pointer
 50    var elem unsafe.Pointer
 51bucketloop:
 52    for {
 53    // 遍历桶中的8个cell
 54        for i := uintptr(0); i < bucketCnt; i++ {
 55      // 这里分两种情况，第一种情况是cell位的tophash值和当前tophash值不相等
 56      // 在 b.tophash[i] != top 的情况下
 57      // 理论上有可能会是一个空槽位
 58      // 一般情况下 map 的槽位分布是这样的，e 表示 empty:
 59      // [h0][h1][h2][h3][h4][e][e][e]
 60      // 但在执行过 delete 操作时，可能会变成这样:
 61      // [h0][h1][e][e][h5][e][e][e]
 62      // 所以如果再插入的话，会尽量往前面的位置插
 63      // [h0][h1][e][e][h5][e][e][e]
 64      //          ^
 65      //          ^
 66      //       这个位置
 67      // 所以在循环的时候还要顺便把前面的空位置先记下来
 68      // 因为有可能在后面会找到相等的key，也可能找不到相等的key
 69            if b.tophash[i] != top {
 70        // 如果cell位为空，那么就可以在对应位置进行插入
 71                if isEmpty(b.tophash[i]) && inserti == nil {
 72                    inserti = &b.tophash[i]
 73                    insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
 74                    elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
 75                }
 76                if b.tophash[i] == emptyRest {
 77                    break bucketloop
 78                }
 79                continue
 80            }
 81      // 第二种情况是cell位的tophash值和当前的tophash值相等
 82            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
 83            if t.indirectkey() {
 84                k = *((*unsafe.Pointer)(k))
 85            }
 86      // 注意，即使当前cell位的tophash值相等，不一定它对应的key也是相等的，所以还要做一个key值判断
 87            if !t.key.equal(key, k) {
 88                continue
 89            }
 90            // 如果已经有该key了，就更新它
 91            if t.needkeyupdate() {
 92                typedmemmove(t.key, k, key)
 93            }
 94      // 这里获取到了要插入key对应的value的内存地址
 95      // pos = start + dataOffset + 8*keysize + i*elemsize
 96            elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
 97      // 如果顺利到这，就直接跳到done的结束逻辑中去
 98            goto done
 99        }
100    // 如果桶中的8个cell遍历完，还未找到对应的空cell或覆盖cell，那么就进入它的溢出桶中去遍历
101        ovf := b.overflow(t)
102    // 如果连溢出桶中都没有找到合适的cell，跳出循环。
103        if ovf == nil {
104            break
105        }
106        b = ovf
107    }
108
109    // 在已有的桶和溢出桶中都未找到合适的cell供key写入，那么有可能会触发以下两种情况
110  // 情况一：
111  // 判断当前map的装载因子是否达到设定的6.5阈值，或者当前map的溢出桶数量是否过多。如果存在这两种情况之一，则进行扩容操作。
112  // hashGrow()实际并未完成扩容，对哈希表数据的搬迁（复制）操作是通过growWork()来完成的。
113  // 重新跳入again逻辑，在进行完growWork()操作后，再次遍历新的桶。
114    if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
115        hashGrow(t, h)
116        goto again // Growing the table invalidates everything, so try again
117    }
118
119  // 情况二：
120// 在不满足情况一的条件下，会为当前桶再新建溢出桶，并将tophash，key插入到新建溢出桶的对应内存的0号位置
121    if inserti == nil {
122        // all current buckets are full, allocate a new one.
123        newb := h.newoverflow(t, b)
124        inserti = &newb.tophash[0]
125        insertk = add(unsafe.Pointer(newb), dataOffset)
126        elem = add(insertk, bucketCnt*uintptr(t.keysize))
127    }
128
129  // 在插入位置存入新的key和value
130    if t.indirectkey() {
131        kmem := newobject(t.key)
132        *(*unsafe.Pointer)(insertk) = kmem
133        insertk = kmem
134    }
135    if t.indirectelem() {
136        vmem := newobject(t.elem)
137        *(*unsafe.Pointer)(elem) = vmem
138    }
139    typedmemmove(t.key, insertk, key)
140    *inserti = top
141  // map中的key数量+1
142    h.count++
143
144done:
145    if h.flags&hashWriting == 0 {
146        throw("concurrent map writes")
147    }
148    h.flags &^= hashWriting
149    if t.indirectelem() {
150        elem = *((*unsafe.Pointer)(elem))
151    }
152    return elem
153}
```

通过对`mapassign`的代码分析之后，发现该函数并没有将插入key对应的value写入对应的内存，而是返回了value应该插入的内存地址。为了弄清楚value写入内存的操作是发生在什么时候，分析如下map.go代码。

```
1package main
2
3func main() {
4    m := make(map[int]int)
5    for i := 0; i < 100; i++ {
6        m[i] = 666
7    }
8}
```

`m[i] = 666`对应的汇编代码

```
 1$ go tool compile -S map.go
 2...
 3        0x0098 00152 (map.go:6) LEAQ    type.map[int]int(SB), CX
 4        0x009f 00159 (map.go:6) MOVQ    CX, (SP)
 5        0x00a3 00163 (map.go:6) LEAQ    ""..autotmp_2+184(SP), DX
 6        0x00ab 00171 (map.go:6) MOVQ    DX, 8(SP)
 7        0x00b0 00176 (map.go:6) MOVQ    AX, 16(SP)
 8        0x00b5 00181 (map.go:6) CALL    runtime.mapassign_fast64(SB) // 调用函数runtime.mapassign_fast64，该函数实质就是mapassign（上文示例源代码是该mapassign系列的通用逻辑）
 9        0x00ba 00186 (map.go:6) MOVQ    24(SP), AX 24(SP), AX // 返回值，即 value 应该存放的内存地址
10        0x00bf 00191 (map.go:6) MOVQ    $666, (AX) // 把 666 放入该地址中
11...        
```

赋值的最后一步实际上是编译器额外生成的汇编指令来完成的，可见靠 runtime 有些工作是没有做完的。所以，在go中，编译器和 runtime 配合，才能完成一些复杂的工作。同时说明，在平时学习go的源代码实现时，必要时还需要看一些汇编代码。



**C**

**删除Key**

#### 

理解了赋值key的逻辑，删除key的逻辑就比较简单了。本文就不再讨论该部分内容了，读者感兴趣可以自行查看`src/runtime/map.go`的`mapdelete`方法逻辑。



**D**

**遍历map**

**结论：迭代 map 的结果是无序的**

```
1    m := make(map[int]int)
2    for i := 0; i < 10; i++ {
3        m[i] = i
4    }
5    for k, v := range m {
6        fmt.Println(k, v)
7    }
```

运行以上代码，我们会发现每次输出顺序都是不同的。

map遍历的过程，是按序遍历bucket，同时按需遍历bucket中和其overflow bucket中的cell。但是map在扩容后，会发生key的搬迁，这造成原来落在一个bucket中的key，搬迁后，有可能会落到其他bucket中了，从这个角度看，遍历map的结果就不可能是按照原来的顺序了（详见下文的map扩容内容）。

但其实，go为了保证遍历map的结果是无序的，做了以下事情：map在遍历时，并不是从固定的0号bucket开始遍历的，每次遍历，都会从一个**随机值序号的bucket**，再从其中**随机的cell**开始遍历。然后再按照桶序遍历下去，直到回到起始桶结束。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

上图的例子，是遍历一个处于未扩容状态的map。如果map正处于扩容状态时，需要先判断当前遍历bucket是否已经完成搬迁，如果数据还在老的bucket，那么就去老bucket中拿数据。

注意：在下文中会讲解到增量扩容和等量扩容。当发生了增量扩容时，一个老的bucket数据可能会分裂到两个不同的bucket中去，那么此时，如果需要从老的bucket中遍历数据，例如1号，则不能将老1号bucket中的数据全部取出，仅仅只能取出老 1 号 bucket 中那些在裂变之后，分配到新 1 号 bucket 中的那些 key（这个内容，请读者看完下文map扩容的讲解之后再回头理解）。

鉴于篇幅原因，本文不再对map遍历的详细源码进行注释贴出。读者可自行查看源码`src/runtime/map.go`的`mapiterinit()`和`mapiternext()`方法逻辑。

这里注释一下`mapiterinit()`中随机保证的关键代码

```
1// 生成随机数
2r := uintptr(fastrand())
3if h.B > 31-bucketCntBits {
4   r += uintptr(fastrand()) << 31
5}
6// 决定了从哪个随机的bucket开始
7it.startBucket = r & bucketMask(h.B)
8// 决定了每个bucket中随机的cell的位置
9it.offset = uint8(r >> h.B & (bucketCnt - 1))
```







**05**

**map扩容**

在文中讲解装载因子时，我们提到装载因子是决定哈希表是否进行扩容的关键指标。在go的map扩容中，除了装载因子会决定是否需要扩容，溢出桶的数量也是扩容的另一关键指标。

为了保证访问效率，当map将要添加、修改或删除key时，都会检查是否需要扩容，扩容实际上是以空间换时间的手段。在之前源码`mapassign`中，其实已经注释map扩容条件，主要是两点:

1. 判断已经达到装载因子的临界点，即元素个数 >= 桶（bucket）总数 * 6.5，这时候说明大部分的桶可能都快满了（即平均每个桶存储的键值对达到6.5个），如果插入新元素，有大概率需要挂在溢出桶（overflow bucket）上。

```
1func overLoadFactor(count int, B uint8) bool {
2    return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
3}
```



1. 判断溢出桶是否太多，当桶总数 < 2 ^ 15 时，如果溢出桶总数 >= 桶总数，则认为溢出桶过多。当桶总数 >= 2 ^ 15 时，直接与 2 ^ 15 比较，当溢出桶总数 >= 2 ^ 15 时，即认为溢出桶太多了。

```
1func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
2    if B > 15 {
3        B = 15
4    }
5    return noverflow >= uint16(1)<<(B&15)
6}
```

对于第2点，其实算是对第 1 点的补充。因为在装载因子比较小的情况下，有可能 map 的查找和插入效率也很低，而第 1 点识别不出来这种情况。表面现象就是计算装载因子的分子比较小，即 map 里元素总数少，但是桶数量多（真实分配的桶数量多，包括大量的溢出桶）。

在某些场景下，比如不断的增删，这样会造成overflow的bucket数量增多，但负载因子又不高，未达不到第 1 点的临界值，就不能触发扩容来缓解这种情况。这样会造成桶的使用率不高，值存储得比较稀疏，查找插入效率会变得非常低，因此有了第 2 点判断指标。这就像是一座空城，房子很多，但是住户很少，都分散了，找起人来很困难。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如上图所示，由于对map的不断增删，以0号bucket为例，该桶链中就造成了大量的稀疏桶。

两种情况官方采用了不同的解决方案

- 针对 1，将 B + 1，新建一个buckets数组，新的buckets大小是原来的2倍，然后旧buckets数据搬迁到新的buckets。该方法我们称之为**增量扩容**。
- 针对 2，并不扩大容量，buckets数量维持不变，重新做一遍类似增量扩容的搬迁动作，把松散的键值对重新排列一次，以使bucket的使用率更高，进而保证更快的存取。该方法我们称之为**等量扩容**。

对于 2 的解决方案，其实存在一个极端的情况：如果插入 map 的 key 哈希都一样，那么它们就会落到同一个 bucket 里，超过 8 个就会产生 overflow bucket，结果也会造成 overflow bucket 数过多。移动元素其实解决不了问题，因为这时整个哈希表已经退化成了一个链表，操作效率变成了 `O(n)`。但 Go 的每一个 map 都会在初始化阶段的 makemap时定一个随机的哈希种子，所以要构造这种冲突是没那么容易的。

在源码中，和扩容相关的主要是`hashGrow()`函数与`growWork()`函数。`hashGrow()` 函数实际上并没有真正地“搬迁”，它只是分配好了新的 buckets，并将老的 buckets 挂到了 oldbuckets 字段上。真正搬迁 buckets 的动作在 `growWork()` 函数中，而调用 `growWork()`函数的动作是在`mapassign()` 和 `mapdelete()` 函数中。也就是插入（包括修改）、删除 key 的时候，都会尝试进行搬迁 buckets 的工作。它们会先检查 oldbuckets 是否搬迁完毕（检查 oldbuckets 是否为 nil），再决定是否进行搬迁工作。

`hashGrow()`函数

```
 1func hashGrow(t *maptype, h *hmap) {
 2  // 如果达到条件 1，那么将B值加1，相当于是原来的2倍
 3  // 否则对应条件 2，进行等量扩容，所以 B 不变
 4    bigger := uint8(1)
 5    if !overLoadFactor(h.count+1, h.B) {
 6        bigger = 0
 7        h.flags |= sameSizeGrow
 8    }
 9  // 记录老的buckets
10    oldbuckets := h.buckets
11  // 申请新的buckets空间
12    newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)
13  // 注意&^ 运算符，这块代码的逻辑是转移标志位
14    flags := h.flags &^ (iterator | oldIterator)
15    if h.flags&iterator != 0 {
16        flags |= oldIterator
17    }
18    // 提交grow (atomic wrt gc)
19    h.B += bigger
20    h.flags = flags
21    h.oldbuckets = oldbuckets
22    h.buckets = newbuckets
23  // 搬迁进度为0
24    h.nevacuate = 0
25  // overflow buckets 数为0
26    h.noverflow = 0
27
28  // 如果发现hmap是通过extra字段 来存储 overflow buckets时
29    if h.extra != nil && h.extra.overflow != nil {
30        if h.extra.oldoverflow != nil {
31            throw("oldoverflow is not nil")
32        }
33        h.extra.oldoverflow = h.extra.overflow
34        h.extra.overflow = nil
35    }
36    if nextOverflow != nil {
37        if h.extra == nil {
38            h.extra = new(mapextra)
39        }
40        h.extra.nextOverflow = nextOverflow
41    }
42}
```

`growWork()`函数

```
 1func growWork(t *maptype, h *hmap, bucket uintptr) {
 2  // 为了确认搬迁的 bucket 是我们正在使用的 bucket
 3  // 即如果当前key映射到老的bucket1，那么就搬迁该bucket1。
 4    evacuate(t, h, bucket&h.oldbucketmask())
 5
 6    // 如果还未完成扩容工作，则再搬迁一个bucket。
 7    if h.growing() {
 8        evacuate(t, h, h.nevacuate)
 9    }
10}
```

从`growWork()`函数可以知道，搬迁的核心逻辑是`evacuate()`函数。这里读者可以思考一个问题：为什么每次至多搬迁2个bucket？这其实是一种性能考量，如果map存储了数以亿计的key-value，一次性搬迁将会造成比较大的延时，因此才采用逐步搬迁策略。

在讲解该逻辑之前，需要读者先理解以下两个知识点。

- **知识点1：bucket序号的变化**

前面讲到，增量扩容（条件1）和等量扩容（条件2）都需要进行bucket的搬迁工作。对于等量扩容而言，由于buckets的数量不变，因此可以按照序号来搬迁。例如老的的0号bucket，仍然搬至新的0号bucket中。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

但是，对于增量扩容而言，就会有所不同。例如原来的B=5，那么增量扩容时，B就会变成6。那么决定key值落入哪个bucket的低位哈希值就会发生变化（从取5位变为取6位），取新的低位hash值得过程称为rehash。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

因此，在增量扩容中，某个 key 在搬迁前后 bucket 序号可能和原来相等，也可能是相比原来加上 2^B（原来的 B 值），取决于低 hash 值第倒数第B+1位是 0 还是 1。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUtUb8P4XltZLgRwISYwA9K7aemCH2sPibe87lbNAmnAWX6umic9cjib5tOhpC2HLz4lLfY9InNudSHlg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，当原始的B = 3时，旧buckets数组长度为8，在编号为2的bucket中，其2号cell和5号cell，它们的低3位哈希值相同（不相同的话，也就不会落在同一个桶中了），但是它们的低4位分别是0010、1010。当发生了增量扩容，2号就会被搬迁到新buckets数组的2号bucket中去，5号被搬迁到新buckets数组的10号bucket中去，它们的桶号差距是2的3次方。

- **知识点2：确定搬迁区间**

在源码中，有bucket x 和bucket y的概念，其实就是增量扩容到原来的 2 倍，桶的数量是原来的 2 倍，前一半桶被称为bucket x，后一半桶被称为bucket y。一个 bucket 中的 key 可能会分裂到两个桶中去，分别位于bucket x的桶，或bucket y中的桶。所以在搬迁一个 cell 之前，需要知道这个 cell 中的 key 是落到哪个区间（而对于同一个桶而言，搬迁到bucket x和bucket y桶序号的差别是老的buckets大小，即2^old_B）。

思考：为什么确定key落在哪个区间很重要？

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

确定了要搬迁到的目标 bucket 后，搬迁操作就比较好进行了。将源 key/value 值 copy 到目的地相应的位置。设置 key 在原始 buckets 的 tophash 为 `evacuatedX` 或是 `evacuatedY`，表示已经搬迁到了新 map 的bucket x或是bucket y，新 map 的 tophash 则正常取 key 哈希值的高 8 位。

下面正式解读搬迁核心代码`evacuate()`函数。

`evacuate()`函数

```
  1func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
  2  // 首先定位老的bucket的地址
  3    b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
  4  // newbit代表扩容之前老的bucket个数
  5    newbit := h.noldbuckets()
  6  // 判断该bucket是否已经被搬迁
  7    if !evacuated(b) {
  8    // 官方TODO，后续版本也许会实现
  9        // TODO: reuse overflow buckets instead of using new ones, if there
 10        // is no iterator using the old buckets.  (If !oldIterator.)
 11
 12    // xy 包含了高低区间的搬迁目的地内存信息
 13    // x.b 是对应的搬迁目的桶
 14    // x.k 是指向对应目的桶中存储当前key的内存地址
 15    // x.e 是指向对应目的桶中存储当前value的内存地址
 16        var xy [2]evacDst
 17        x := &xy[0]
 18        x.b = (*bmap)(add(h.buckets, oldbucket*uintptr(t.bucketsize)))
 19        x.k = add(unsafe.Pointer(x.b), dataOffset)
 20        x.e = add(x.k, bucketCnt*uintptr(t.keysize))
 21
 22    // 只有当增量扩容时才计算bucket y的相关信息（和后续计算useY相呼应）
 23        if !h.sameSizeGrow() {
 24            y := &xy[1]
 25            y.b = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.bucketsize)))
 26            y.k = add(unsafe.Pointer(y.b), dataOffset)
 27            y.e = add(y.k, bucketCnt*uintptr(t.keysize))
 28        }
 29
 30    // evacuate 函数每次只完成一个 bucket 的搬迁工作，因此要遍历完此 bucket 的所有的 cell，将有值的 cell copy 到新的地方。
 31    // bucket 还会链接 overflow bucket，它们同样需要搬迁。
 32    // 因此同样会有 2 层循环，外层遍历 bucket 和 overflow bucket，内层遍历 bucket 的所有 cell。
 33
 34    // 遍历当前桶bucket和其之后的溢出桶overflow bucket
 35    // 注意：初始的b是待搬迁的老bucket
 36        for ; b != nil; b = b.overflow(t) {
 37            k := add(unsafe.Pointer(b), dataOffset)
 38            e := add(k, bucketCnt*uintptr(t.keysize))
 39      // 遍历桶中的cell，i，k，e分别用于对应tophash，key和value
 40            for i := 0; i < bucketCnt; i, k, e = i+1, add(k, uintptr(t.keysize)), add(e, uintptr(t.elemsize)) {
 41                top := b.tophash[i]
 42        // 如果当前cell的tophash值是emptyOne或者emptyRest，则代表此cell没有key。并将其标记为evacuatedEmpty，表示它“已经被搬迁”。
 43                if isEmpty(top) {
 44                    b.tophash[i] = evacuatedEmpty
 45                    continue
 46                }
 47        // 正常不会出现这种情况
 48        // 未被搬迁的 cell 只可能是emptyOne、emptyRest或是正常的 top hash（大于等于 minTopHash）
 49                if top < minTopHash {
 50                    throw("bad map state")
 51                }
 52                k2 := k
 53        // 如果 key 是指针，则解引用
 54                if t.indirectkey() {
 55                    k2 = *((*unsafe.Pointer)(k2))
 56                }
 57                var useY uint8
 58        // 如果是增量扩容
 59                if !h.sameSizeGrow() {
 60          // 计算哈希值，判断当前key和vale是要被搬迁到bucket x还是bucket y
 61                    hash := t.hasher(k2, uintptr(h.hash0))
 62                    if h.flags&iterator != 0 && !t.reflexivekey() && !t.key.equal(k2, k2) {
 63            // 有一个特殊情况：有一种 key，每次对它计算 hash，得到的结果都不一样。
 64            // 这个 key 就是 math.NaN() 的结果，它的含义是 not a number，类型是 float64。
 65            // 当它作为 map 的 key时，会遇到一个问题：再次计算它的哈希值和它当初插入 map 时的计算出来的哈希值不一样！
 66            // 这个 key 是永远不会被 Get 操作获取的！当使用 m[math.NaN()] 语句的时候，是查不出来结果的。
 67            // 这个 key 只有在遍历整个 map 的时候，才能被找到。
 68            // 并且，可以向一个 map 插入多个数量的 math.NaN() 作为 key，它们并不会被互相覆盖。
 69            // 当搬迁碰到 math.NaN() 的 key 时，只通过 tophash 的最低位决定分配到 X part 还是 Y part（如果扩容后是原来 buckets 数量的 2 倍）。如果 tophash 的最低位是 0 ，分配到 X part；如果是 1 ，则分配到 Y part。
 70                        useY = top & 1
 71                        top = tophash(hash)
 72          // 对于正常key，进入以下else逻辑  
 73                    } else {
 74                        if hash&newbit != 0 {
 75                            useY = 1
 76                        }
 77                    }
 78                }
 79
 80                if evacuatedX+1 != evacuatedY || evacuatedX^1 != evacuatedY {
 81                    throw("bad evacuatedN")
 82                }
 83
 84        // evacuatedX + 1 == evacuatedY
 85                b.tophash[i] = evacuatedX + useY
 86        // useY要么为0，要么为1。这里就是选取在bucket x的起始内存位置，或者选择在bucket y的起始内存位置（只有增量同步才会有这个选择可能）。
 87                dst := &xy[useY]
 88
 89        // 如果目的地的桶已经装满了（8个cell），那么需要新建一个溢出桶，继续搬迁到溢出桶上去。
 90                if dst.i == bucketCnt {
 91                    dst.b = h.newoverflow(t, dst.b)
 92                    dst.i = 0
 93                    dst.k = add(unsafe.Pointer(dst.b), dataOffset)
 94                    dst.e = add(dst.k, bucketCnt*uintptr(t.keysize))
 95                }
 96                dst.b.tophash[dst.i&(bucketCnt-1)] = top
 97        // 如果待搬迁的key是指针，则复制指针过去
 98                if t.indirectkey() {
 99                    *(*unsafe.Pointer)(dst.k) = k2 // copy pointer
100        // 如果待搬迁的key是值，则复制值过去  
101                } else {
102                    typedmemmove(t.key, dst.k, k) // copy elem
103                }
104        // value和key同理
105                if t.indirectelem() {
106                    *(*unsafe.Pointer)(dst.e) = *(*unsafe.Pointer)(e)
107                } else {
108                    typedmemmove(t.elem, dst.e, e)
109                }
110        // 将当前搬迁目的桶的记录key/value的索引值（也可以理解为cell的索引值）加一
111                dst.i++
112        // 由于桶的内存布局中在最后还有overflow的指针，多以这里不用担心更新有可能会超出key和value数组的指针地址。
113                dst.k = add(dst.k, uintptr(t.keysize))
114                dst.e = add(dst.e, uintptr(t.elemsize))
115            }
116        }
117    // 如果没有协程在使用老的桶，就对老的桶进行清理，用于帮助gc
118        if h.flags&oldIterator == 0 && t.bucket.ptrdata != 0 {
119            b := add(h.oldbuckets, oldbucket*uintptr(t.bucketsize))
120      // 只清除bucket 的 key,value 部分，保留 top hash 部分，指示搬迁状态
121            ptr := add(b, dataOffset)
122            n := uintptr(t.bucketsize) - dataOffset
123            memclrHasPointers(ptr, n)
124        }
125    }
126
127  // 用于更新搬迁进度
128    if oldbucket == h.nevacuate {
129        advanceEvacuationMark(h, t, newbit)
130    }
131}
132
133func advanceEvacuationMark(h *hmap, t *maptype, newbit uintptr) {
134  // 搬迁桶的进度加一
135    h.nevacuate++
136  // 实验表明，1024至少会比newbit高出一个数量级（newbit代表扩容之前老的bucket个数）。所以，用当前进度加上1024用于确保O(1)行为。
137    stop := h.nevacuate + 1024
138    if stop > newbit {
139        stop = newbit
140    }
141  // 计算已经搬迁完的桶数
142    for h.nevacuate != stop && bucketEvacuated(t, h, h.nevacuate) {
143        h.nevacuate++
144    }
145  // 如果h.nevacuate == newbit，则代表所有的桶都已经搬迁完毕
146    if h.nevacuate == newbit {
147    // 搬迁完毕，所以指向老的buckets的指针置为nil
148        h.oldbuckets = nil
149    // 在讲解hmap的结构中，有过说明。如果key和value均不包含指针，则都可以inline。
150    // 那么保存它们的buckets数组其实是挂在hmap.extra中的。所以，这种情况下，其实我们是搬迁的extra的buckets数组。
151    // 因此，在这种情况下，需要在搬迁完毕后，将hmap.extra.oldoverflow指针置为nil。
152        if h.extra != nil {
153            h.extra.oldoverflow = nil
154        }
155    // 最后，清除正在扩容的标志位，扩容完毕。
156        h.flags &^= sameSizeGrow
157    }
158}
```

代码比较长，但是文中注释已经比较清晰了，如果对map的扩容还不清楚，可以参见以下图解。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

针对上图的map，其B为3，所以原始buckets数组为8。当map元素数变多，加载因子超过6.5，所以引起了增量扩容。

以3号bucket为例，可以看到，由于B值加1，所以在新选取桶时，需要取低4位哈希值，这样就会造成cell会被搬迁到新buckets数组中不同的桶（3号或11号桶）中去。注意，在一个桶中，搬迁cell的工作是有序的：它们是依序填进对应新桶的cell中去的。

当然，实际情况中3号桶很可能还有溢出桶，在这里为了简化绘图，假设3号桶没有溢出桶，如果有溢出桶，则相应地添加到新的3号桶和11号桶中即可，如果对应的3号和11号桶均装满，则给新的桶添加溢出桶来装载。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

对于上图的map，其B也为3。假设整个map中的overflow过多，触发了等量扩容。注意，等量扩容时，新的buckets数组大小和旧buckets数组是一样的。

以6号桶为例，它有一个bucket和3个overflow buckets，但是我们能够发现桶里的数据非常稀疏，等量扩容的目的就是为了把松散的键值对重新排列一次，以使bucket的使用率更高，进而保证更快的存取。搬迁完毕后，新的6号桶中只有一个基础bucket，暂时并不需要溢出桶。这样，和原6号桶相比，数据变得紧密，使后续的数据存取变快。

最后回答一下上文中留下的问题：为什么确定key落在哪个区间很重要？因为对于增量扩容而言，原本一个bucket中的key会被分裂到两个bucket中去，它们分别处于bucket x和bucket y中，但是它们之间存在关系 bucket x + 2^B = bucket y （其中，B是老bucket对应的B值）。假设key所在的老bucket序号为n，那么如果key落在新的bucket x，则它应该置入 bucket x起始位置 + n*bucket 的内存中去；如果key落在新的bucket y，则它应该置入 bucket y起始位置 + n*bucket的内存中去。因此，确定key落在哪个区间，这样就很方便进行内存地址计算，快速找到key应该插入的内存地址。





**map 总结和使用建议**

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/HR9EgRcLqUusfcAq5hCK7rsyTmqGH6yrBVa9Z5GZM7TTlJPelkNwwsPc1QnzX1ayRnBgBLq9ibibgISrsLgAVEhQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**01**

**总结**

### 

Go语言的map，底层是哈希表实现的，通过链地址法解决哈希冲突，它依赖的核心数据结构是数组加链表。

map中定义了2的B次方个桶，每个桶中能够容纳8个key。根据key的不同哈希值，将其散落到不同的桶中。哈希值的低位（哈希值的后B个bit位）决定桶序号，高位（哈希值的前8个bit位）标识同一个桶中的不同 key。

当向桶中添加了很多 key，造成元素过多，超过了装载因子所设定的程度，或者多次增删操作，造成溢出桶过多，均会触发扩容。

扩容分为增量扩容和等量扩容。增量扩容，会增加桶的个数（增加一倍），把原来一个桶中的 keys 被重新分配到两个桶中。等量扩容，不会更改桶的个数，只是会将桶中的数据变得紧凑。不管是增量扩容还是等量扩容，都需要创建新的桶数组，并不是原地操作的。

扩容过程是渐进性的，主要是防止一次扩容需要搬迁的 key 数量过多，引发性能问题。触发扩容的时机是增加了新元素， 桶搬迁的时机则发生在赋值、删除期间，每次最多搬迁两个 桶。查找、赋值、删除的一个很核心的内容是如何定位到 key 所在的位置，需要重点理解。一旦理解，关于 map 的源码就可以看懂了。



**02**

**使用建议**

从map设计可以知道，它并不是一个并发安全的数据结构。同时对map进行读写时，程序很容易出错。因此，要想在并发情况下使用map，请加上锁（sync.Mutex或者sync.RwMutex）。其实，Go标准库中已经为我们实现了并发安全的map——sync.Map，我之前写过文章对它的实现进行讲解，详情可以查看本公众号《深入理解sync.Map》一文。

遍历map的结果是无序的，在使用中，应该注意到该点。

通过map的结构体可以知道，它其实是通过指针指向底层buckets数组。所以和slice一样，尽管go函数都是值传递，但是，当map作为参数被函数调用时，在函数内部对map的操作同样会影响到外部的map。

另外，有个特殊的key值math.NaN，它每次生成的哈希值是不一样的，这会造成m[math.NaN]是拿不到值的，而且多次对它赋值，会让map中存在多个math.NaN的key。不过这个基本用不到，知道有这个特殊情况就可以了。