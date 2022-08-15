---
title: Java数组基础
categories:
  - 编程
tags:
  - Java
  - 数组基础
  - 面试题
date: 2022-08-15 17:08:18
---

# 数组基础
## 1.ArrayList和LinkedList区别?
1. 都是List接口的实现类
2. ArrayList基于数组，LinkedList基于链表
3. ArrayList
3.1 查询快，增删慢
3.2 往数组尾部添加元素的效率高，也就是调用add(obj)，但是还是比LinkedList慢。
4. LinkedList
4.1 数据添加删除效率高，只需要改变指针指向即可
4.2 查询数据的平均效率低，需要对链表进行遍历

## 2.ArrayList扩容机制怎样？
1. ArrayList每次扩容是原来的1.5倍。
2. 数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的1.5倍。
3. 代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。尽可能，就指定其容量，以避免数组扩容的发生。
4. 创建方式方式不同，容量不同

| 初始化方式                          | 容量                                                        | 数量变化                    |
| ----------------------------------- | ----------------------------------------------------------- | --------------------------- |
| List arrayList = new ArrayList();   | **初始数组容量为10**,当真正对数组进行添加时，才真正分配容量 | 10->15->22->33->49->74->... |
| List arrayList = new ArrayList(4)； | 4                                                           | 4->6->9->13->19->...        |



断点看扩容情况1:

```java
List arrayList = new ArrayList();
public class TestArrayList {
	 public static void main(String[] args) {
	 	List<Integer> list = new ArrayList<>();
	 	
	 	for (int i = 1; i <= 20; i++) {
	 		if (i == 1) {
                System.out.println("断点位置 没添加元素时候容量 0");
            }
            list.add(i);
            if (i == 1) {
                System.out.println("断点位置 第一次添加元素 容量 10");
            }
            if (i == 11) {
                System.out.println("断点位置 第1次扩容 容量 15");
            }
            if (i == 16) {
                System.out.println("断点位置 第2次扩容 容量15*1.5= 22.5,22 or 23 ");
            }
        }
     }
}
            
```

断点看扩容情况2:
```java
public class TestArrayList1  {
	 public static void main(String[] args) {
	 	List<Integer> list = new ArrayList<>(4);
	 	
	 	for (int i = 1; i <= 20; i++) {
	 		if (i == 1) {
                System.out.println("断点位置 没添加元素时候容量 4");
            }
            list.add(i);
            if (i == 1) {
                System.out.println("断点位置 第一次添加元素 容量 10");
            }
            if (i == 5) {
                System.out.println("断点位置 第一次扩容 容量 6 ");
            }
            if (i == 7) {
                System.out.println("断点位置 第二次扩容 容量9 ");
            }
        }
     }
}
```

## 3.HashMap和HashTable的区别
1. HashMap采用了数组+链表的数据结构，能在查询和修改方便继承了数组的线性查找和链表的寻址修改
2. HashMap是非synchronized，所以HashMap比HashTable更快
3. HashMap可以接受null键和值，而Hashtable则不能（原因就是equlas()方法需要对象，因为HashMap是后出的API经过处理才可以）

## 4.HashMap的工作原理（wait publish）
1. HashMap 的存储机制
1.1 在 Java 1.8 中，如果链表的长度超过了 8 ，那么链表将转化为红黑树；链表长度低于6，就把红黑树转回链表;
2. Java 1.8 中 HashMap 的不同
2.1 在 Java 1.8 中，如果链表的长度超过了 8 ，那么链表将转化为红黑树；链表长度低于6，就把红黑树转回链表;
2.2 发生 hash 碰撞时，Java 1.7 会在链表头部插入，而 Java 1.8 会在链表尾部插入
2.3 在 Java 1.8 中，Entry 被 Node 代替（换了一个马甲）
3.put过程
3.1 对Key求Hash值，然后再计算下标
3.2 如果没有碰撞，直接放入桶中（碰撞的意思是计算得到的Hash值相同，需要放到同一个bucket中
3.3 如果碰撞了，以链表的方式链接到后面
3.4 如果链表长度超过阀值( TREEIFY THRESHOLD==8)，就把链表转成红黑树，链表长度低于6，就把红黑树转回链表
3.5 如果节点已经存在就替换旧值
3.6 如果桶满了(容量16乘以加载因子0.75)，就需要 resize（扩容2倍后重排）
4. get过程
4.1 当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置
4.2 找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点
4.3 最终找到要找的值对象


## 5. List、Set、Map三个接口的区别以及常见子类？
1. List、Set单列，Map是双列的键值对
2. List可重复，set不可重复
3. List有序的，set是无序
4. List中最常用的两个子类：ArrayList（基于数组，查询快）和LinkedList（基于链表，增删快
5. Set中最常用的两个子类：HashSet和TreeSet
6. Map中最常用的两个子类：HashMap和TreeMa

- 是否了解ConcurrentHashMap

- 经常了解HashMap和HashTable的区别

- 是否了解LinkedHashMap

- 是否了解CopyOnWriteArrayList

- 是否了解ConcurrentModificationException异常


## 6. 抽象类接口的区别？
- 抽象类

1.抽象类使用abstract修饰；
2.抽象类不能实例化，即不能使用new关键字来实例化对象；
3.含有抽象方法（使用abstract关键字修饰的方法）的类是抽象类，必须使用abstract关键字修饰；
4.抽象类可以含有抽象方法，也可以不包含抽象方法，抽象类中可以有具体的方法；
5.如果一个子类实现了父类（抽象类）的所有抽象方法，那么该子类可以不必是抽象类，否则就是抽象类；
6.抽象类中的抽象方法只有方法体，没有具体实现；

- 接口
1. 接口使用interface修饰；
2. 接口不能被实例化；
3. 一个类只能继承一个类，但是可以实现多个接口；
4. **接口中方法均为抽象方法**；
5. 接口中不能包含实例域或静态方法（静态方法必须实现，接口中方法是抽象方法，不能实现）

- 区别
1. **抽象类可以提供成员方法的实现细节**，而接口中没有；但是JDK1.8之后，在接口里面可以定义default方法，default方法里面是可以具备方法体的。
2. 抽象类中的成员变量可以是**各种类型的**，而接口中的成员变量只能是**public static final**类型；
3. 接口中每一个方法也是隐式指定为 public abstract，不能含有静态代码块以及静态方法，而抽象类可以有**静态代码块和静态方法**；
4. 一个类只能继承**一个**抽象类，而一个类却可以实现**多个**接口。

## 7. ConcurrentHashMap
1. 在ConcurrentHashMap中，无论是读操作还是写操作都能保证很高的性能，是Java中的一个`线程安全且高效`的HashMap实现
2. 在进行读操作时(几乎)不需要加锁，而在写操作时通过锁分段技术只对所操作的段加锁而不影响客户端对其它段的访问。
3. 在理想状态下，ConcurrentHashMap 可支持16个线程执行并发写操作，及任意数量线程的读操作。

- 关于它的存储结构
 - JDK 1.7 中ConcurrentHashMap由Segment 数组+HashEntry 组成，也就是数组+链表。使用分段锁（ReentrantLock + Segment + HashEntry），相当于把一个 HashMap 分成多个段，每段分配一把锁，这样支持多线程访问。锁粒度：基于 Segment，包含多个 HashEntry。
 - JDK 1.8 中使用 CAS + synchronized + Node + 红黑树。锁粒度：Node（首结点）（实现 Map.Entry<K,V>）。锁粒度降低了。将HashEntry改为了Node，和 1.8 HashMap 结构类似，当链表节点数超过指定阈值的话，转换成`红黑树`。