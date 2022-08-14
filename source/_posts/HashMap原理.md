---
title: HashMap原理
date: 2022-08-14 19:58:26
categories:
-  编程
tags:
- Java
- 原理
---

# HashMap原理
- HashMap内部是基于哈希表实现的键值对存储，继承 AbstractMap 并且实现了 Map 接口。
- HashMap基于hashing原理，我们通过put()和get()方法储存和获取对象。
- 当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，让后找到bucket位置来储存值对象。
- 当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。
- HashMap使用LinkedList来解决碰撞问题，当发生碰撞了，对象将会储存在LinkedList的下一个节点中。 
- 当两个不同的键对象的hashcode相同时会发生什么？ 它们会储存在同一个bucket位置的LinkedList中，键对象的equals()方法用来找到键值对。

## 1. 什么是哈希表
### 哈希基础操作
在讨论哈希表之前，我们先大概了解下其他数据结构在新增，查找等基础操作执行性能

- **数组**：采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)

- **链表**：是一种物理存储单元上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)

- **二叉树**：对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。

- **哈希表**：相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)，接下来我们就来看看哈希表是如何实现达到惊艳的常数阶O(1)的。

	我们知道，数据结构的物理存储结构只有两种：顺序存储结构和链式存储结构（像栈，队列，树，图等是从逻辑结构去抽象的，映射到内存中）。
	而在上面我们提到过，在数组中根据下标查找某个元素，一次定位就可以达到，哈希表利用了这种特性，哈希表的主干就是数组。
	比如我们要新增或查找某个元素，我们通过把当前元素的关键字 通过某个函数映射到数组中的某个位置，通过数组下标一次定位就可完成操作。
		**存储位置 = f(关键字)**
	其中，这个函数f一般称为哈希函数，这个函数的设计好坏会直接影响到哈希表的优劣。举个例子，比如我们要在哈希表中执行插入操作：
	
	|      |          Kobe           |      |      |          James          |
	| :--: | :---------------------: | :--: | :--: | :---------------------: |
	|      | 哈希函数<br />f（Kobe） |      |      | 哈希函数<br />f（Kobe） |
	|      |   存储地址<br />（1）   |      |      |   存储地址<br />（4）   |
	|  0   |            1            |  2   |  3   |            4            |
	|      |          Kobe           |      |      |          james          |
	
	查找操作同理，先通过哈希函数计算出实际存储地址，然后从数组中对应地址取出即可。
	
### 哈希冲突
- 然而万事无完美，如果两个不同的元素，通过哈希函数得出的实际存储地址相同怎么办？也就是说，当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞。前面我们提到过，哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？

1. 链地址法：将哈希表的每个单元作为链表的头结点，所有哈希地址为 i 的元素构成一个同义词链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。
2. 开放定址法：即发生冲突时，去寻找下一个空的哈希地址。只要哈希表足够大，总能找到空的哈希地址。
3. 再哈希法：即发生冲突时，由其他的函数再计算一次哈希值。
4. 建立公共溢出区：将哈希表分为基本表和溢出表，发生冲突时，将冲突的元素放入溢出表



## 2. HashMap实现原理
以下是 HashMap 源码里面的一些关键成员变量以及知识点。在后面的源码解析中会遇到，所以我们有必要先了解下。
##### hash简单原理
- initialCapacity：初始容量。指的是 HashMap 集合初始化的时候自身的容量。可以在构造方法中指定；如果不指定的话，总容量默认值是 16 。需要注意的是初始容量必须是 2 的幂次方。

- size：当前 HashMap 中已经存储着的键值对数量，即 HashMap.size()

- loadFactor：加载因子。所谓的加载因子就是 HashMap (当前的容量/总容量) 到达一定值的时候，HashMap 会实施扩容。加载因子也可以通过构造方法中指定，默认的值是 0.75 。举个例子，假设有一个 HashMap 的初始容量为 16 ，那么扩容的阀值就是 0.75 * 16 = 12 。也就是说，在你打算存入第 13 个值的时候，HashMap 会先执行扩容。

- threshold：扩容阀值。即 扩容阀值 = HashMap 总容量 * 加载因子。当前 HashMap 的容量大于或等于扩容阀值的时候就会去执行扩容。扩容的容量为当前 HashMap 总容量的两倍。比如，当前 HashMap 的总容量为 16 ，那么扩容之后为 32 。

- table：Entry 数组。我们都知道 HashMap 内部存储 key/value 是通过 Entry 这个介质来实现的。而 table 就是 Entry 数组。

- 在 Java 1.7 中，HashMap 的实现方法是数组 + 链表的形式。上面的 table 就是数组，而数组中的每个元素，都是链表的第一个结点。即如下图所示：
##### 数组+链表的形式

| 0                     | 1                  | 2                  | 3                  | 4                  | 5                  | ....      |
| --------------------- | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ | --------- |
| key=null<br />value=1 | key=a<br />value=1 | key=d<br />value=1 | key=e<br />value=1 | key=f<br />value=1 | key=h<br />value=1 | Entry数组 |
|                       | key=b<br />value=2 |                    |                    | key=g<br />value=2 |                    |           |
|                       | key=c<br />value=3 |                    |                    |                    |                    |           |

  HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。
```java
//HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}，主干数组的长度一定是2的次幂，至于为什么这么做，后面会有详细分析。
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```
 Entry是HashMap中的一个静态内部类。代码如下
```java
static class Entry<K,V> implements Map.Entry<K,V> {
        final K key;
        V value;
        Entry<K,V> next;//存储指向下一个Entry的引用，单链表结构
        int hash;//对key的hashcode值进行hash运算后得到的值，存储在Entry，避免重复计算

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
```
##### HashMap的整体结构：

|                       0                       |                       1                       |  2   |                       3                       |                       4                       |                       5                       |  6   |
| :-------------------------------------------: | :-------------------------------------------: | :--: | :-------------------------------------------: | :-------------------------------------------: | :-------------------------------------------: | :--: |
| entry<br />key<br />value<br />hash<br />next | entry<br />key<br />value<br />hash<br />next | null | entry<br />key<br />value<br />hash<br />next | entry<br />key<br />value<br />hash<br />next | entry<br />key<br />value<br />hash<br />next | null |
|                     null                      | entry<br />key<br />value<br />hash<br />next |      | entry<br />key<br />value<br />hash<br />next |                     null                      | entry<br />key<br />value<br />hash<br />next |      |
|                                               | entry<br />key<br />value<br />hash<br />next |      |                     null                      |                                               |                     null                      |      |
|                                               | entry<br />key<br />value<br />hash<br />next |      |                                               |                                               |                                               |      |
|                                               |                     null                      |      |                                               |                                               |                                               |      |


**简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的。**

- 如果定位到的数组位置不含链表（当前entry的next指向null）：那么对于查找，添加等操作很快，仅需一次寻址即可；
- 如果定位到的数组包含链表：对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。

所以，性能考虑，HashMap中的链表出现越少，性能才会越好。

HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值,initialCapacity默认为16，loadFactory默认为0.75,我们看下其中一个:
```java
public HashMap(int initialCapacity, float loadFactor) {　　　　　//此处对传入的初始容量进行校验，最大不能超过MAXIMUM_CAPACITY = 1<<30(2
30
)
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;　　　　　
        init();//init方法在HashMap中没有实际实现，不过在其子类如 linkedHashMap中就会有对应实现
    }
```
从上面这段代码我们可以看出，**在常规构造器中，没有为数组table分配内存空间（有一个入参为指定Map的构造器例外），而是在执行put操作的时候才真正构建table数组**

##### put操作的实现

```java
public V put(K key, V value) {
        //如果table数组为空数组{}，进行数组填充（为table分配实际内存空间），入参为threshold，此时threshold为initialCapacity 默认是1<<4(2
4
=16)
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
       //如果key为null，存储位置为table[0]或table[0]的冲突链上
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);//对key的hashcode进一步计算，确保散列均匀
        int i = indexFor(hash, table.length);//获取在table中的实际位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        //如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;//保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        addEntry(hash, key, value, i);//新增一个entry
        return null;
    }
```

##### inflateTable方法

```java
private void inflateTable(int toSize) {
        int capacity = roundUpToPowerOf2(toSize);//capacity一定是2的次幂
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);//此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity);
    }
```

　inflateTable这个方法用于为主干数组table在内存中分配存储空间，通过roundUpToPowerOf2(toSize)可以确保capacity为大于或等于toSize的最接近toSize的二次幂，比如toSize=13,则capacity=16;to_size=16,capacity=16;to_size=17,capacity=32.

```java
private static int roundUpToPowerOf2(int number) {
        // assert number >= 0 : "number must be non-negative";
        return number >= MAXIMUM_CAPACITY
                ? MAXIMUM_CAPACITY
                : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
    }
```

roundUpToPowerOf2中的这段处理使得数组长度一定为2的次幂，Integer.highestOneBit是用来获取最左边的bit（其他bit位为0）所代表的数值.



##### hash函数

```java
//这是一个神奇的函数，用了很多的异或，移位等运算，对key的hashcode进一步进行计算以及二进制位的调整等来保证最终获取的存储位置尽量分布均匀
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

以上hash函数计算出的值，通过indexFor进一步处理来获取实际的存储位置

```java
/**
     * 返回数组下标
     */
    static int indexFor(int h, int length) {
        return h & (length-1);
    }
```



h&（length-1）保证获取的index一定在数组范围内，举个例子，默认容量16，length-1=15，h=18,转换成二进制计算为

```java
 	1  0  0  1  0
&   0  1  1  1  1
 __________________
    0  0  0  1  0    = 2
```

最终计算出的index=2。有些版本的对于此处的计算会使用 取模运算，也能保证index一定在数组范围内，不过位运算对计算机来说，性能更高一些（HashMap中有大量位运算）

所以最终存储位置的确定流程是这样的：

| key  | ---> hashCode() ---> | hashCode | -->hash() | h    | --->indexFor()--->  | 存储下标 |
| ---- | -------------------- | -------- | --------- | ---- | ------------------- | -------- |
|      |                      |          |           |      | --->h&(length-1)__> |          |

##### addEntry的实现：

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

通过以上代码能够得知，当发生哈希冲突并且size大于阈值的时候，需要进行数组扩容，扩容时，需要新建一个长度为之前数组2倍的新的数组，然后将当前的Entry数组中的元素全部传输过去，扩容后的新数组长度为之前的2倍，所以扩容相对来说是个耗资源的操作。



##  3.Java 1.8 中 HashMap 的不同

- 在 Java 1.8 中，如果链表的长度超过了 8 ，那么链表将转化为红黑树；
- 发生 hash 碰撞时，Java 1.7 会在链表头部插入，而 Java 1.8 会在链表尾部插入；
- 在 Java 1.8 中，Entry 被 Node 代替（换了一个马甲）