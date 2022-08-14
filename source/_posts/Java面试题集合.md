---
title: Java面试题集合
date: 2022-08-14 21:33:10
categories:
-  编程
tags:
- Java
- 面试题
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
1. 在ConcurrentHashMap中，无论是读操作还是写操作都能保证很高的性能
2. 在进行读操作时(几乎)不需要加锁，而在写操作时通过锁分段技术只对所操作的段加锁而不影响客户端对其它段的访问。
3. 在理想状态下，ConcurrentHashMap 可支持16个线程执行并发写操作，及任意数量线程的读操作。

- 关于它的存储结构
 - JDK 1.7 中使用分段锁（ReentrantLock + Segment + HashEntry），相当于把一个 HashMap 分成多个段，每段分配一把锁，这样支持多线程访问。锁粒度：基于 Segment，包含多个 HashEntry。
 - JDK 1.8 中使用 CAS + synchronized + Node + 红黑树。锁粒度：Node（首结点）（实现 Map.Entry<K,V>）。锁粒度降低了。

# 算法&数据结构&设计模式
## 1. 动态代理有几种实现？
1. java的动态代理技术的实现主要有两种方式：
- JDK原生动态代理
- CGLIB动态代理

2. JDK原生动态代理是Java原生支持的，不需要任何外部依赖，但是它只能基于接口进行代理（需要代理的对象必须实现于某个接口）
3. CGLIB通过继承的方式进行代理（让需要代理的类成为Enhancer的父类），无论目标对象有没有实现接口都可以代理，但是无法处理final的情况。

## 2. 堆和栈的区别?
1. 栈内存
- 栈内存首先是一片内存区域，存储的都是**局部变量**
- 凡是定义在方法中的都是局部变量（方法外的是全局变量），for循环内部定义的也是局部变量
- 是先加载函数才能进行局部变量的定义，所以**方法先进栈**，然后再定义变量，变量有自己的作用域，一旦离开作用域，变量就会被释放。
- 栈内存的更新速度很快，因为局部变量的生命周期都很短。

2. 堆内存
- 存储的是数组和**对象**（其实数组就是对象），凡是**new建立的都是在堆中**
- 堆中存放的都是实体（对象），实体用于封装数据，而且是封装多个（实体的多个属性），如果一个数据消失，这个实体也没有消失，还可以用，所以堆是不会随时释放的
- 但是栈不一样，栈里存放的都是单个变量，变量被释放了，那就没有了。
- 堆里的实体虽然不会被释放，但是会被当成垃圾，Java有垃圾回收机制不定时的收取。

3. 堆与栈的区别
- 栈内存存储的是**局部变量**而堆内存存储的是**实体**；
- 栈内存的更新速度要快于堆内存，因为局部变量的生命周期很短；
- 栈内存存放的变量生命周期一旦结束就会被释放，而堆内存存放的实体会被垃圾回收机制不定时的回收。

## 3.常见的数据结构有哪些？
1. 八大数据结构分类
- 数组
- 栈
- 队列
- 链表
 - 单向链表
 - 双向链表
 - 循环链表
- 树
- 散列表
- 堆
- 图

2. 数据结构简述
- 数组：有下标索引，长度固定
- 栈：先进后出
- 队列：先进先出
- 链表
 - 单链表：链表中的元素的指向只能指向链表中的下一个元素或者为空，元素之间不能相互指向。也就是一种线性链表。
 - 双向链表：一个有序的结点序列，每个链表元素既有指向下一个元素的指针，又有指向前一个元素的指针，其中每个结点都有两种指针，即left和right。left指针指向左边结点，right指针指向右边结点。
 - 循环链表 :是在单向链表和双向链表的基础上，将两种链表的最后一个结点指向第一个结点从而实现循环。
- 树：
 - 二叉树
 - 二叉查找树
 - 平衡二叉树
  - 平衡查找树之AⅥL树
  - 平衡二叉树之红黑树
 - B树
 - B+树
 - B\*\树
- 散列表：散列表（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。
- 堆：堆是一种比较特殊的数据结构，可以被看做一棵树的数组对象，具有以下的性质：
 - 堆中某个节点的值总是不大于或不小于其父节点的值；
 - 堆总是一棵完全二叉树。
- 图：图是由结点的有穷集合V和边的集合E组成，按照顶点指向的方向可分为无向图和有向图

## 4. 常见设计模式有哪些？
- 单例模式(Singleton)
- 构建模式(Builder)
- 抽象工厂模式(Abstract Factory)
- 工厂方法模式(Factory Method)
- 观察者模式(Observer)
- 模板方法模式(Template Method)
- 装饰者模式(Decorator)
- 代理模式(Proxy) 

## 5. 排序算法有哪些？
- 冒泡排序
- 插入排序
- 快速排序
- 选择排序
- 希尔排序
- 归并排序
- 堆排序
- 计数排序
- 桶排序
- 基数排序

## 6. 设计模式分类
1. 创建型的设计模式：
- 单例模式(Singleton)
- 构建模式(Builder)
- 原型模式(Prototype)
- 抽象工厂模式(Abstract Factory)
- 工厂方法模式(Factory Method) 
2. 行为设计模式：策略模式(Strategy) 
- 状态模式(State)
- 责任链模式(Chain of Responsibility)
- 解释器模式(Interpreter)
- 命令模式(Command)
- 观察者模式(Observer)
- 备忘录模式(Memento)
- 迭代器模式(Iterator)
- 模板方法模式(Template Method)
- 访问者模式(Visitor)
- 中介者模式(Mediator) 
3. 结构型设计模式
- 装饰者模式(Decorator)
- 代理模式(Proxy)
- 组合模式(Composite)
- 桥连接模式(Bridge)
- 适配器模式(Adapter)
- 蝇量模式(Flyweight)
- 外观模式(Facade) 

详解：
- 单例模式(Singleton)：确保有且只有一个对象被创建。
- 抽象工厂模式(Abstract Factory)：允许客户创建对象的家族，而无需指定他们的具体类。
- 工厂方法模式(Factory Method)：由子类决定要创建的具体类是哪一个。
- 装饰者模式(Decorator)：包装一个对象，以提供新的行为。
- 状态模式(State)：封装了基于状态的行为，并使用委托在行为之间切换。
- 迭代器模式(Iterator)：在对象的集合之中游走，而不暴露集合的实现。
- 外观模式(Facade)：简化一群类的接口。
- 策略模式(Strategy)：封装可以互换的行为，并使用委托来决定要使用哪一个。
- 代理模式(Proxy)：包装对象，以控制对此对象的访问。
- 适配器模式(Adapter)：封装对象，并提供不同的接口。
- 观察者模式(Observer)：让对象能够在状态改变时被通知。
- 模板方法模式(Template Method)：有子类决定如何实现一个算法中的步骤。
- 组合模式(Composite)：客户用一致的方法处理对象集合和单个对象。
- 命令模式(Command)：封装请求成为对象。

## 7. 单例模式
1. 单列模式有5种常见的写法
- 饿汉式
- 懒汉式
- 双检锁
- 静态内部类，用的最多
- 枚举
2. 单例的四大原则：
- 构造私有。
- 以静态方法或者枚举返回实例。
- 确保实例只有一个，尤其是多线程环境。
- 确保反序列换时不会重新构建对象。

- 饿汉模式
```java
public class SingleTon{
    private static SingleTon  INSTANCE = null;
    private SingleTon(){}
    public static SingleTon getInstance() {  
    if(INSTANCE == null){
       INSTANCE = new SingleTon();
    }
     return INSTANCE；
  }
 }
```
- 双检检查锁
```java
public class SingleTon{
   private static SingleTon  INSTANCE = null;
   private SingleTon(){}
   public static SingleTon getInstance(){if(INSTANCE == null){
    synchronized(SingleTon.class){
      if(INSTANCE == null){
         INSTANCE = new SingleTon();
        }
      }
         return INSTANCE;
    }
  }
 }
```
- 静态内部类，用的最多
```java
public class SingleTon{
   private SingleTon(){}
 
   private static class SingleTonHoler{
      private static SingleTon INSTANCE = new SingleTon();
  }
 
   public static SingleTon getInstance(){
     return SingleTonHoler.INSTANCE;
  }
 }
```

## 8. 设计模式的原则
1. 设计模式的六大原则
- 开闭原则（Open Close Principle）
- 里氏代换原则（Liskov Substitution Principle）
- 依赖倒转原则（Dependence Inversion Principle）
- 接口隔离原则（Interface Segregation Principle）
- 迪米特法则，又称最少知道原则（Demeter Principle）
- 合成复用原则（Composite Reuse Principle）

2. 详解
- 开闭原则（Open Close Principle）：开闭原则的意思是：对扩展开放，对修改关闭。
- 里氏代换原则（Liskov Substitution Principle）：里氏代换原则是面向对象设计的基本原则之一。里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为。里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化，而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。
- 依赖倒转原则（Dependence Inversion Principle）：这个原则是开闭原则的基础，具体内容：**针对接口编程，依赖于抽象**而不依赖于具体。
- 接口隔离原则（Interface Segregation Principle）：这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。它还有另外一个意思是：降低类之间的耦合度。由此可见，其实设计模式就是从大型软件架构出发、便于升级和维护的软件设计思想，它强调降低依赖，降低耦合。
- 迪米特法则：又称最少知道原则（Demeter Principle）最少知道原则是指：一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立。
- 合成复用原则（Composite Reuse Principle）：合成复用原则是指：尽量使用合成/聚合的方式，而不是使用继承。

# 3.多线程&并发
## 1.Java中的锁
1. 乐观锁/悲观锁
2. 共享锁/独享锁
3. 公平锁/非公平锁
4. 互斥锁/读写锁
5. 可重入锁
6. 自旋锁
7. 分段锁
8. 偏向锁/轻量级锁/重量级锁

## 2. Java线程的状态|生命周期
1. Java的线程状态被定义在公共枚举类java.lang.Thread.state中。一种有**六种状态**
- 新建（NEW）：表示线程新建出来还没有被启动的状态，比如：Thread t = new MyThread();
- 就绪/运行（RUNNABLE）：该状态包含了经典线程模型的两种状态：就绪(Ready)、运行(Running)：
- 阻塞（BLOCKED）：通常与锁有关系，表示线程正在获取有锁控制的资源，比如进入synchronized代码块，获取ReentryLock等；发起阻塞式IO也会阻塞，比如字符流字节流操作。
- 等待（WAITING）：线程在等待某种资源就绪。
- 超时等待（TIMED_WAIT）：线程进入条件和等待类似，但是它调用的是带有超时时间的方法。
- 终止（TERMINATED）：线程正常退出或异常退出后，就处于终结状态。也可以叫线程的死亡。

2. 经典线程模型包含5种状态，：新建、就绪、运行、等待、退出
3. 经典线程的就绪、运行，在java里面统一叫RUNNABLE
4. Java的BLOCKED、WAITING、TIMED_WAITING都属于传统模型的等待状态

哪些情况或者方法可以进入等待状态？
- 当一个线程执行了Object.wait()的时候，它一定在等待另一个线程执行Object.notify()或者Object.notifyAll()。
- 一个线程thread，其在主线程中被执行了thread.join()的时候，主线程即会等待该线程执行完成。
- 当一个线程执行了LockSupport.park()的时候，其在等待执行LockSupport.unpark(thread)。

哪些情况或者方法可以进入超时等待状态？
- 该状态不同于WAITING，它可以在指定的时间后自行返回
1. Object.wait(long)
2. Thread.join(long)
3. LockSupport.parkNanos()
4. LockSupport.parkUntil()
5. Thread.sleep(long)

## 3. synchronized 与lock区别？
1. lock是一个接口，而synchronized是java的一个关键字
2. synchronized异常会释放锁，lock异常不会释放，所以一般try catch包起来，finally中写入unlock，避免死锁。
3. Lock可以提高多个线程进行读操作的效率
4. synchronized关键字，可以放代码块，实例方法，静态方法，类上
5. lock一般使用ReentrantLock类做为锁，配合lock()和unlock()方法。在finally块中写unlock()以防死锁。
6. jdk1.6之前synchronized低效。jdk1.6之后synchronized高效。

## 4. synchronized 与ReentrantLock区别？
1. synchronized依赖JVM实现，ReentrantLock是JDK实现的。synchronized是内置锁，只要在代码开始的地方加synchronized，代码结束会自动释放。Lock必须手动加锁，手动释
2. ReenTrantLock比synchronized增加了一些高级功能。synchronized代码量少，自动化，但扩展性低，不够灵活；ReentrantLock扩展性好，灵活，但代码量相对多。
3. 两者都是可重入锁。都是互斥锁。
4. synchronized是非公平锁，ReentrantLock可以指定是公平锁还是非公平锁。

## 5. synchronized 与ThreadLocal区别？
1. 都是为了解决多线程中相同变量的访问冲突问题。
2. Synchronized同步机制，提供一份变量，让不同的线程排队访问。
3. ThreadLocal关键字，为每一个线程都提供了一份变量，因此可以同时访问而互不影响。
4. ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

- ThreadLocal
```java
public class ConnectionUtil {
     private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>();
     private static Connection initConn = null;
     static {
         try {
             initConn = DriverManager.getConnection("url, name and password");
        } catch (SQLException e) {
             e.printStackTrace();
        }
    }
     
     public Connection getConn() {
         Connection c = tl.get();
         if(null == c) tl.set(initConn);
         return tl.get();
    }
 }
```

- synchronized
```java
public class ConnectionUtil {
     private static DBOPool instance=null;
     public static synchronized Connection getInstance(){
         if(instance==null)
             instance=new DBOPool();
         return instance.getConnection();
    }          
 }
```

## 6. synchronized 与volatile区别？
1. volatile是一个类型修饰符（type specifier）。
2. volatile，它能够使变量在值发生改变时能尽快地让其他线程知道。
3. 关键字volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好，并且只能修改变量，而synchronized可以修饰方法，以及代码块。
4. 多线程访问volatile不会发生阻塞，而synchronized会出现阻塞
5. volatile能保证数据的可见性，但不能保证原子性；而synchronized可以保证原子性，也可以间接保证可见性，因为它会将私有内存和公共内存中的数据做同步
6. 关键字volatile解决的下变量在多线程之间的可见性；而synchronized解决的是多线程之间资源同步问题

## 7. Thread类中的start()和run()方法有什么区别?
1. 通过调用线程类的start()方法来启动一个线程，使线程处于就绪状态，即可以被JVM来调度执行，在调度过程中，JVM通过调用线程类的run()方法来完成实际的业务逻辑，当run()方法结束后，此线程就会终止。
2. 如果直接调用线程类的run()方法，会被当作一个普通的函数调用，程序中仍然只有主线程这一个线程。即start()方法能够异步的调用run()方法，但是直接调用run()方法却是同步的，无法达到多线程的目的。
3. 因此，只用通过调用线程类的start()方法才能达到多线程的目的。

## 8. 事务的隔离级别及引发的问题
1. 四个隔离级别
- 读未提交（READ UNCOMMITTED），事务中的修改，即使没有提交，对其它事务也是可见的。
- 读已提交（READ COMMITTED），一个事务能读取已经提交的事务所做的修改，不能读取未提交的事务所做的修改。也就是事务未提交之前，对其他事务不可见。
- 可重复读（REPEATABLE READ），保证在同一个事务中多次读取同样数据的结果是一样的。
- 串行化（SERIALIZABLE），强制事务串行执行。
2. 读已提交是sql server，Oracle的默认隔离级别。
3. 可重复读是mysql的默认隔离级别。
4. 各个隔离级别引起的问题
- 读未提交：可能出现脏读、不可重复度、幻读；
- 读已提交：可能出现不可重复度、幻读；
- 串行化：都没问题；

5. 理解脏读、不可重复读、幻读
- 脏读：读到未提交的数据。
- 不可重复读：重点是修改，同样的条件, 你读取过的数据, 再次读取出来发现值不一样了。
- 幻读：重点在于新增或者删除，同样的条件, 第1次和第2次读出来的记录数不一样。

## 9. 什么是线程安全，Java如何保证线程安全？
1. 在多线程环境中，能永远保证程序的正确性。执行结果不存在二义性。说白了，运行多少次结果都是一致的。
2. 换种说法，当多个线程访问某一个类（对象或方法）时，这个类始终都能表现心正确的行为，那么这个类（对象或方法）就是线程安全的。
3. 使用synchronized关键字和使用锁。

## 10. 线程池介绍
1. 线程池就是预先创建一些线程，它们的集合称为线程池。
2. 线程池可以很好地提高性能，在系统启动时即创建大量空闲的线程，程序将一个task给到线程池，线程池就会启动一条线程来执行这个任务，执行结束后，该线程不会死亡，而是再次返回线程池中成为空闲状态，等待执行下一个任务。
3. 线程的创建和销毁比较消耗时间，线程池可以避免这个问题。
4. Executors是jdk1.5之后的一个新类，提供了一些静态方法，帮助我们方便的生成一些常见的线程池
- newSingleThreadExecutor：创建一个单线程化的Executor。
- newFixedThreadPool：创建一个固定大小的线程池。
- newCachedThreadPool：创建一个可缓存的线程池
- newScheduleThreadPool：创建一个定长的线程池，可以周期性执行任务。
5. 我们还可以使用ThreadPoolExecutor自己定义线程池，弄懂它的构造参数即可
- int corePoolSize，//核心池的大小
- int maximumPoolSize，//线程池最大线程数
- long keepAliveTime，//保持时间/额外线程的存活时间
- TimeUnit unit，//时间单位
- BlockingQueue<Runnable> workQueue，//任务队列
- ThreadFactory threadFactory，//线程工厂
- RejectedExecutionHandler handler //异常的捕捉器

常见的线程池有哪些？
1. Executors是jdk1.5之后的一个新类，提供了一些静态方法，帮助我们方便的生成一些常见的线程池
2. 单线程线程池，通过newSingleThreadExecutor()创建
3. 可缓存的线程池，通过newCachedThreadPool()创建
4. 可周期性执行任务的线程池，通过newScheduleThreadPool()创建

几个线程的区别？
1. newCachedThreadPool
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。特点：
- 工作线程的创建数量几乎没有限制(其实也有限制的,数目为Interger. MAX_VALUE), 这样可灵活的往线程池中添加线程。
- 如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程。
- 在使用CachedThreadPool时，一定要注意控制任务的数量，否则，由于大量线程同时运行，很有会造成系统瘫痪。
2. newFixedThreadPool
创建一个指定工作线程数量的线程池。每当提交一个任务就创建一个工作线程，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中。
- FixedThreadPool是一个典型且优秀的线程池，它具有线程池提高程序效率和节省创建线程时所耗的开销的优点。但是，在线程池空闲时，即线程池中没有可运行任务时，它不会释放工作线程，还会占用一定的系统资源。
3. newSingleThreadExecutor
- 创建一个单线程化的Executor，即只创建唯一的工作者线程来执行任务，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。如果这个线程异常结束，会有另一个取代它，保证顺序执行。单工作线程最大的特点是可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的。
4. newScheduleThreadPool
- 创建一个定长的线程池，而且支持定时的以及周期性的任务执行，支持定时及周期性任务执行

## 11.  同步和异步有何异同？
1. 同步发了指令，会等待返回，然后再发送下一个。
2. 异步发了指令，不会等待返回，随时可以再发送下一个请求
3. 同步可以避免出现死锁，读脏数据的发生
4. 异步则是可以提高效率
5. 实现同步的机制主要有临界区、互斥、信号量和事件

## 12. 哪些集合是线程安全的？
1. Vector：就比Arraylist多了个同步化机制（线程安全）。
2. Hashtable：就比Hashmap多了个线程安全。
3. ConcurrentHashMap:是一种高效但是线程安全的集合。

http://120.24.95.18/javamianshi20200522/