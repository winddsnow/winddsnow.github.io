---
title: Java算法基础-数据结构-设计模式
categories:
  - 编程
tags:
  - Java
  - 数据结构
  - 面试题
date: 2022-08-15 17:11:37
---

# Java算法基础-数据结构-设计模式
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