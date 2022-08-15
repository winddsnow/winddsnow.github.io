---
title: Java多线程-并发
categories:
  - 编程
tags:
  - Java
  - 多线程
  - 并发
  - 面试题
date: 2022-08-15 17:14:31
---
# 多线程-并发①
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

## 13. 如何异步获取多线程返回的数据？
- 说一下Callable这个接口的理解？
- 说一下Future接口的理解？
- 说一下FutureTask类的理解？
- 说一下CompletionService接口的理解？

1. 通过Callable+Future，Callable负责执行返回，Future负责接收。Callable接口对象可以交给ExecutorService的submit方法去执行。
2. 通过Callable+FutureTask，Callable负责执行返回，FutureTask负责接收。FutureTask同时实现了Runnable和Callable接口，可以给到ExecutorService的submit方法和Thread去执行。
3. 通过CompletionService，jdk1.8之后提供了完成服务CompletionService，可以实现这样的需求。
4. 注意，实现Runnable接口任务执行结束后无法获取执行结果。Callable有返回值，Runnable没有返回值

## 14. 如何自定义线程池？
1. 通过ThreadPoolExecutor创建
2. 弄懂它的7个构造参数即可
- int corePoolSize，//核心池的大小
 - 默认情况下，在创建了线程池之后，线程池中的线程数为0
 - 当有任务到来后，如果线程池中存活的线程数小于corePoolSize，则创建一个线程。
- int maximumPoolSize，//线程池最大线程数
 - 线程池中允许的最大线程数，这个参数表示了线程池中最多能创建的线程数量。
 - 当任务数量比corePoolSize大时，任务添加到workQueue
 - 当workQueue满了，将继续创建线程以处理任务。
 - maximumPoolSize表示当wordQueue满了，线程池中最多可以创建的线程数量。
- long keepAliveTime，//保持时间/额外线程的存活时间
 - 当线程池处于空闲状态时，超过keepAliveTime时间之后，空闲的线程会被终止。
 - 只有当线程池中的线程数大于corePoolSize时，这个参数才会起作用，但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；
 - 当线程数大于corePoolSize时，如果一个线程的空闲时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。
- TimeUnit unit，//时间单位
 - TimeUnit.DAYS; //天
 - TimeUnit.HOURS; //小时
 - TimeUnit.MINUTES; //分钟
 - TimeUnit.SECONDS; //秒
 - TimeUnit.MILLISECONDS; //毫秒
 - TimeUnit.MICROSECONDS; //微妙
 - TimeUnit.NANOSECONDS; //纳秒
- BlockingQueue<Runnable> workQueue，//任务队列-阻塞队列，存储提交的等待任务。常见子类有
 - ArrayBlockingQueue;
 - LinkedBlockingQueue
 - SynchronousQueue;
- ThreadFactory threadFactory，//线程工厂
- RejectedExecutionHandler handler //异常的捕捉器- 任务队列添加异常的捕捉器，当任务超出线程池范围和队列容量时，采取何种拒绝策略。参考 RejectedExecutionHandler，常见实现类。
 - ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
 - ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
 - ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
 - ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
 
 
## 15. 工作中哪些地方使用了多线程？
1. 一般业务，web层--> service层 -->dao --> sql基本用不到多线程
2. 数据量很大（1000w级别、TB级别）的I/O操作，可以考虑多线程
3. 例子：
 - 自己做并发测试的时候，假如想写想模拟3000个并发请求。
 - 多线程下单抢单，假如支持5000人的并发下单。
 - 多线程写入mysql，假如有1000w条数据要入库。
 - 多线程写入redis，假如有1000w的数据要存入redis。
 - 多线程导入ES索引，假如有1000w的数据要添加到ES索引。
 - poi多线程导出，假如xls里面有10w的数据需要导出。
 - poi多线程导入，假如有10w条数据需要导入到xls。
 - 多线程发送邮件，假如有10w用户需要发送邮件。
 - 多线程发送短信，假如有10w用户需要发送邮件。
 - 多线程备份日志，假如10tb日志文件要备份。
 - 多线程验证数据，比如验证url是否存在，假如有100w个url。




