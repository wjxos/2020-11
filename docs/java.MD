## volatile
1. volatile是java虚拟机提供的轻量级的同步机制。
 * 保证可见性：（多线程修改同一个数据）
 * 不保证原子性：证明（for循环家数据到2W，用volatile修饰不会达到）
 * 禁止指令重排：

使用原子包装类可以解决不保证原子性的问题Atomic。

DCL(双端检锁)机制不一定线程安全，原因是指令重排的存在，加入volatile可禁止指令重排
验证（单例模式）

原因是某一线程执行到第一次检测，读到instance不为null时，instance会引用对象可能没有完成初始化。
instance=new SingletonDemo();可以分一下三个步骤完成。
1. memony=allocate();分配对象内存空间。
2. instance(memony);初始化对象。
3. instance=memony;设置instance指向刚分配的内存地址，因此instance!=null

步骤2和3不存在数据依赖关系，而且无论重排前还是重排后，之慈宁宫结果在单线程并没有改变，因此这种重排是被允许的。
 
volatile的实际应用举例：单例模式（多线程：双端检锁）

2. JMM 
  * 可见性
  * 原子性
  * 有序性 
  
## CAS（比较并交换）
它是一条cpu并发原语。 
unsafe+cas（自旋锁）
缺点
* 循环时间长CPU开销大（do while 循环）
* 只能保证一个共享变量的原子性操作。
* 引出ABA的问题。

## ABA (狸猫换太子)
* AtomicReference （CAS）
* AtomicStampedReference （CAS 乐观锁的实现）
* static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
* static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

## 集合的不安全性
1. ArrayList线程不安全。
List add 时会会出现并发修改异常
ConcurrentModificationException
```
1. 故障现象 ：java.util.ConcurrentModificationException
2. 导致原因
3. 解决方案
   3.1 Vector
   3.2 Collections.synchronizedList
   3.3 new CopyOnWriteArrayList() : ReentrantLock
```
2. HashMap线程不安全
```java
    Map<String, String> list = new HashMap();
    for (int i = 0; i < 30; i++) {
        new Thread(()->{
            list.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 8));
            System.out.println(list);
        }, String.valueOf(i)).start();
    }
发生异常：java.util.ConcurrentModificationException
```
# java的锁
1. 公平锁和非公平锁

并发包中```ReentrantLock```创建可以指定构造函数的boolean类型来得到公平锁或非公平锁，默认是非公平锁。
```
公平锁：按照先进先出的规则，从队列中获取。
非公平锁：直接尝试占有锁，如果尝试失败，就在采类似公平锁的方式。
java中ReentrantLock默认是非公平锁
对于synchronize而言，也是一种非公平锁
```
2. 可重入锁（又名递归锁）

可重入锁，又名递归锁：指的是同一线程外层函数获得锁之后，内层递归函数任然能获得该锁的代码。
线程可以进入任意一个它已经拥有的锁同步着的代码块。
```java
ReentrantLock和synchronize默认都是非公平的可重入锁
```
3. 自旋锁
4. 读写锁

5. CountDownLatch
```java
CountDownLatch：
当所有的同学都离开了教室，班长就会关闭走人。
秦灭6国全国统一。
```
6. CyclicBarrier
```
CyclicBarrier
集齐7颗龙珠才能召唤神龙
所有的人到齐了才能开会
```
7. Semaphore（信号灯）
```java

信号量主要用于两个目的，一个是用于多个资源的互斥使用，另一个用于并发线程数的控制。

```
## 队列 （待学习）

## synchronize和lock对的区别
```java
1. 原始构成：
   synchronized是关键字属于JVM层面，底层是monitorenter和monitorexit
   lock是具体的类（JUC）是挨批层面的锁。
2. 使用方法
   synchronized不需要用户手动释放锁，当synchronized方法执行完成之后自动释放对锁的占用
   ReentrantLock需要用户手动释放锁。
3. 等待是否可中断
   synchronized不可中断，除非抛出异常或者运行完成。
   ReentrantLock可中断（1.设置超时方法，2.调用interrupt()）可中断，
4. 加锁是否公平
   synchronized非公平锁
   ReenteantLock默认非公平锁，可以修改
5. 锁绑定多个条件
   synchronized没有
   ReentrantLock用来实现分组唤醒的线程们，可准确唤醒，而不像synchronized要么唤醒一个线程要么全部唤醒。
```
## GC
1. GCRoot 对象
```
虚拟机栈（局部变量表）中引用的对象
方法区中的类静态属性引用的对象
方法区中常量引用的对象
本地方法栈中JNI（native方法）引用的对象
```
JVM参数类型
```java
1. 标配参数
   -version
   -help
   java -showversion
2. X参数（了解）
   -Xint：解释执行
   -Xcomp：第一次使用就编译成本地代码
   -Xmixed：混合模型
3. XX参数
   1>Boolean类型（-XX:+开启或者-关闭，某个属性值）
   举例：1. 查看是否打印GC手机细节（jps 查到线程号，jinfo -flag PrintGCDetails 线程号）
   -XX:+PrintGCDetails  ||  -XX:-PrintGCDetails
        2. -XX:-UseSerialGC -XX:+UseSerialGC
   2>KV设置类型（-XX:属性key=属性value）
        1. -XX:MetaspaceSize=1024M 元空间大小
        2. -XX:Xmx=1024 -XX:Xms=1024
        3. -XX:Xss=默认512K~1024K 单个线程栈的大小
```

## 强引用
```java
//强引用 obj2 不受影响
Object obj1 = new Object();
Object obj2 = obi1;
obj1 = null;
System.gc();
```
## 垃圾回收器
1. 串行垃圾回收器（Serial）：它为单线程环境设计，且只使用一个线程进行垃圾回收，会暂停所有的用户线程。
2. 并行垃圾回收器（Parallel）：多个垃圾收集线程并行工作，此时用户线程是暂停的。
3. 并发垃圾回收器（CMS）：用户线程和垃圾收集线程同时执行（不一定是并行，可能交替执行）
4. G1垃圾回收器：G1垃圾回收器将堆内存分割成不停的区域，然后并发的对其进行垃圾回收。

总结：串行（一个清洁工），并行（多个清洁工），并发（垃圾回收和用户线程交替执行），分割区域回收。 JVM8 默认是G!
```java
java -XX:+PrintCommandLineFlags -version //查看默认的垃圾回收器
并行垃圾回收器 -XX:+UseSerialGC
并行垃圾回收器 -XX:+UseParallelGC
```
## AQS
是用来构建锁或其他同步器组件的重量级基础框架及整个JUC体系的基石。
通过内置的FIFO队列来完成资源获取线程的排队工作，通过一个int类型的变量表示持有锁的状态。
锁：面向锁的使用者，APi层面
同步器：面向锁的实现者，

