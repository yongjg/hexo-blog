---
title: 从Swing EDT到多线程以及线程安全
date: 2018-12-25 13:12:03
tags: [EDT,Thread Safety]
toc: true
thumbnail: gallery/celebration-christmas-decorate.jpeg
---

### Swing的线程模型

在Swing GUI开发过程中我们一直强调不能在EDT中调用Blocking api(一般包括传统的网络、文件的IO操作，可预知的耗时计算等)，以免导致UI失去响应。那么为什么长时间占用EDT会导致UI失去响应呢？因为EDT负责重绘UI和响应用户事件。如果一个长任务占用了EDT，那么这段时间内UI刷新和用户事件必然无法处理。故而我们要保证EDT中的操作时长要极短。另外也要避免EDT任务之间的循环触发依赖，这会导致EDT被无穷无尽的EDT任务风暴累死。

Swing线程模型图：
{% asset_img swing-thread-concurrent-edt-eventqueue.svg %}


EDT不停的循环从EventQueue中获取任务执行，当EventQueue没有任务时，EDT就处于waiting状态休息片刻。而各种AWTEvent会被不定时的加入EventQueue，例如一个鼠标点击事件，一个SwingWorker的finish事件等。
<!-- more -->
那么为什么Swing会采用单一线程EDT来负责UI控件的更新和访问呢？根本原因在于保证UI组件的线程安全。

假设有一个控件显示一段文本，ThreadA对文本进行插入“Hello” ，ThreadB对文本进行插入“World”，如果没有合理同步策略，结果可能是“HeWollorld”这样一个莫名其妙的值。而通过将UI控件相关的操作（组件的更新、绘制）限制在EDT这单一线程，保证了所有操作是被有序执行，避免了并发冲突。这也是为什么实际开发中我们不能在后台线程中直接触发某些组件的更新操作,而要通过类似SwingUtilities#invokeLater方法来提交更新操作。绝大部分GUI库都是基于上述线程模型，通过限制性的单一线程来达到线程安全的目的。

### 多线程

现代CPU大多是多核，给人印象就是快。而多线程一方面追求快另一方面追求多。多线程的快和多来自于合理规划从而充分利用系统资源（基本因子CPU、内存、网络）。

那么如果一台机器是单核的，我们还需要多线程吗？需要！在现代计算机系统中，对网络等的IO操作在等待数据的过程中并不消耗CPU时间。如果只采用一个线程，那么该线程在等待IO操作的过程中只能无所事事白白浪费时间。我们可以通过多线程调度IO操作以避免主线程盲目等待。即使全部都是CPU密集型操作，我们也可以恰当的使用多线程，从而相对公平分配不同任务需要的CPU时间。
{%asset_img swing-thread-concurrent-single-core.svg%}


如上面对单核机器的多线程调用示意图，虽然开启了多个线程，但是因为只有一个CPU计算core，等于所有线程都是轮换占有同一个CPU core的计算时间。
当多线程任务全部是CPU密集计算时，这种情况下总的运行时间反而会变长，因为多线程来回切换需要消耗调度的时间。但是如果任务之间没有依赖，那么独立任务在客观时间跨度上就会比较平均，不会出现某个任务被严重推迟执行。
* 当一个线程负责CPU密集操作，一个线程负责IO等操作时，我们期望的是不需要CPU干预的那部分IO操作时间段和CPU密集操作重叠并发执行，从而节省总的时间，同时也让主线程能做更多的计算。
* 而在多核机器上，因为存在多个CPU计算core，多线程可以真正的分散在不同的core上运行，从而更能充分发挥CPU的能力，也是真正能实现并行计算。一般讲并行计算是指在客观时间上真正的同时计算执行，而并发不一定是并行计算。

{%asset_img swing-thread-concurrent-multi-cores.svg%}
如上图多核机器多线程示意图，无论是密集CPU操作，还是IO操作都可以在多个core上并行计算，从而获得更优的性能，当然前提是各操作间不能有太多的前后依赖。如果3个任务依次首尾都有依赖，那无论如何也只能顺序去执行。

### 线程安全

多线程能帮助我们充分利用CPU，但是也引入了线程安全的问题。线程安全问题的本质在于系统内存模型下对共享资源的调度和计算指令的重排序问题。

#### 指令重排序

在Java环境中： java文件（源文件）  →   class文件（jvm指令） →   jvm继续解析成cpu指令。而开发人员在java文件中编写的指令顺序，在编译，jvm运行，cpu执行过程中都有可能被重排序。当然这种重排序的目的是在保证代码语义一致性的情况下，提高运行效率。

代码语义一致性举例：有如下初始值和多线程角色

**Initial values**
```
String action = "next build";
booleanrelease = false;
```
**Thread T1(Boss)**
```
action = "a party";
release = true;
```
线程T1（Boss）的角度看{action = "a party";release = true;}和{release = true;action = "a party";}语义上是一致的，因为在T1中只是赋值没有读取action，release的值。这也是给各种优化提供了机会。

**Thread T2(Developer)**
```
if(release) {
    System.out.println("I want a celebration for the great release."); 
}
else{
    System.out.println("I have to prepare the next build.");
}
System.out.println("Let's have "+ action);
```
我们期望的结果是当Boss通知说 release成功，后续工作是开个party，那么Developer会欢呼并执行准备的庆祝方案。
{%asset_img swing-thread-concurrent-boss-developer-ok.svg%}
但是因为重排序的存在，Boss可能先执行release=true，然后Developer发现release == true，准备庆祝，但是 action可能依然是"next build"。 
{%asset_img swing-thread-concurrent-re-order.svg%}
因为release成功而没有party，Developer很生气，后果很严重。

#### 内存模型

现代计算机分CPU（运算器+cache），内存，存储器，IO设备等。内存模型简化成下图：
{%asset_img swing-thread-concurrent-memory-model.svg%}

如上图 Thread1和Thread2操作主内存中同一个数据，一般为了运算效率主内存中的数据在各自Thread对应的Cache中也有一份运行时备份。那么就有可能存在如下风险：
* T1修改了数据，只存在cache1中没有同步到主内存中， T2无法感知T1的操作结果。 内存可见性。
* T1 、T2都执行 a = a + 1； 起始状态a=0；经过2次计算我们期望值是a=2； 但是因为T1、T2运行时序因为编译优化、CPU调度等原因执行时序无法预计，很可能得到结果是a=1。  操作原子性。
所以按前面Boss-Developer的实例，即使Boss所有指令都先执行了，Developer很可能出现如下情况：
{%asset_img swing-thread-concurrent-menory-visible.svg%}

Developer看不到最新的状态，依然在苦逼的工作着。

#### happens-before规则
* 编码顺序规则：同一线程中的各操作顺序遵循源文件中的编码顺序（注意这里保证的是语义一致性，不是绝对的语句执行顺序。如上面举例的同一线程中{x=5;y=6;}和{y=6;x=5;}是一致的）。Program order rule. Each action in a thread happens before every action in that thread that comes later in the program order. 
* Monitor锁规则：对一个锁的解锁操作先于对同一个锁的加锁操作。An unlock on a monitor lock (exiting synchronized method/block) happens-before every subsequent acquiring on the same monitor lock. 
* Volatile变量规则：对同一个volatile变量的write操作先于read操作。Volatile variable rule. A write to a volatile field happens before every subsequent read of that same field.
* 线程start规则：A线程调用Thread.start()操作启动了B线程，那么A线程在Thread.start()之前的操作先于所有B线程中执行的操作。Thread start rule. A call to Thread.start on a thread happens before every action in the started thread.
* 线程join规则：A线程正常运行，B线程调用Thread.join等待A线程结束，那么A线程中的所有操作先于B线程调用Thread.join()后的操作。Thread termination rule. Any action in a thread happens before any other thread detects that thread has terminated, either by successfully return from Thread.join or by Thread.isAlive returning false.
* 传递性规则：A先于B，同时B先于C，则必然A先于C。Transitivity. If A happens before B, and B happens before C, then A happens before C.比如A-happens-before-B，那么A程序块的所有赋值对B程序块都可见，同时jvm等的重排序不会破坏A，B程序块的一致性语义。

在Java内存模型中，是通过提供一系列内存屏障指令来保证这种规则，便于理解列出简化的2种称之为barrier-store、barrier-load。Java compiler和处理器最终都会支持和遵循内存屏障指令。

java语言层面 | jvm层面 | 内存效果 | 重排序影响
---------|----------|---------|-----------
 write volatile/ exit synchronized block / unlock | barrier-store | 指令前的所有数据更新同步到主内存|禁止和前面的指令重排序
 read volatile/ enter synchronized block / lock | barrier-load | load前先把主内存内容同步到线程中|禁止和后面的指令重排序

那么再回到Boss-Developer的例子，如果希望多线程情况下能得到源文件一致的执行语义和解决内存可见性，我们只做一个小小的改动。

**Initial values**
```
String action = "next build";
volatilebooleanrelease = false;
```

现在共享变量release 被volatile修饰，Boss对release=true的操作之前的指令禁止重排序，Developer对if(release)之后的指令禁止重排序。那么一旦发生release=true，必然保证Developer可以看到所有最新的状态。完美+优雅！

但是这里忽略了一个问题，action和release是2个独立的变量，同时操作2个共享变量并不是原子的。而由于多线程调度的原因，有可能Boss线程进行一半，又切到Developer线程进行，然后又回到Boss线程。
{%asset_img swing-thread-concurrent-boss-dev-volatile.svg%}

好比Boss设置了action="a party"，然后走开去抽烟了。 Developer开始有点错乱，好像在开着party庆祝这次release失败。

所以最终为了保证不被错误的重排序、内存可见性、操作原子性，我们不得不给并发代码块加上一致的锁，因为锁保证原子性的同时也保证了内存可见性，所以不需要volatile了。

**Initial values**
```
String action = "next build";
booleanrelease = false;
ReentrantLock lock = newReentrantLock();
```
**Thread T1(Boss)**
```
lock.lock();
try{
    action = "a party";
    release = true;
}finally{
    lock.unlock();
}
```
**Thread T2(Developer)**
```
lock.lock();
try{
    if(release){
        System.out.println("I want a celebration for the great release."); 
    }else{
        System.out.println("I have to prepare the next build.");
    }
    System.out.println("Let's have "+ action);
}finally{
    lock.unlock();
}
```
如此历经波折，终于保证庆祝release成功的party正确而及时的召开。

那么实际开发设计中有哪些策略来实现线程安全呢？
不可变对象，对象在new完以后，所有状态都不在变化。 也就是说多个线程只是读取对象的属性而不做修改，这必然是线程安全的
将共享对象的访问约束在单一线程内， 就像Swing中的EDT， 对Component的写和读取(graphic paint)都是发生在EDT中，因此不会出现我们修改了值和界面不一致的情况。如果在后台线程对Component的值进行修改，就存在风险。等于是用特殊的单一线程隔离了共享对象，从而保证了系统整体的线程安全。
合理的使用锁，通过锁来保护对象中的共享变量和方法块。 
### java.util.concurrent包介绍

synchrinized和volatile关键字可以对多线程编程提供基础的支持，但是有其局限性和使用上的复杂性。JDK中的concurrent包对多线程、并发等提供了很多便捷的支持。

#### 原子操作和锁的支持

java.util.concurrent.atomic包，对数据更新、自增等操作的线程安全支持。

java.util.concurrent.lock包，各种锁的实现，基础是AbstractQueuedSynchronizer，简称AQS，在此基础上提供了ReentrantLock、ReentrantReadWriteLock等。另外也提供了几个有特殊场景的同步机制实现

CyclicBarrier: 同步栅栏，只有指定数量的线程全部到达栅栏，才能共同冲破栅栏继续下去。 可以设置冲破栅栏发生后的特殊回调函数，由最后一个到达栅栏的线程负责执行。
{%asset_img swing-thread-concurrent-cyclicBarrier.svg%}

Semaphore：信号灯，用于调度线程访问共享资源，初始化特定数量的权限。场景示例，幼儿园有一个可以放2个玩具的篮子，小朋友们都可以去拿玩具玩，当篮子内的2个玩具都被拿走，其他小朋友只能等待，直到拿走玩具的小朋友把玩具还回来，或者老师放入新的玩具。
{%asset_img swing-thread-concurrent-semaphore.svg%}

CountDownLatch：好比一个倒计时信号枪，初始化一个信号值，N个线程会等待倒计时命令， 其他线程会对信号值减数，当信号值到达0时，等待的N个线程会被通知放行。
{%asset_img swing-thread-concurrent-countdownlatch.svg%}

#### 线程安全容器类

ConcurrentMap: 线程安全的map，key-value的put操作 happen-before  其他线程中对同一个map的get、remove操作 。实现类ConcurrentHashMap

线程安全的non-blocking容器：CopyOnWriteArrayList、CopyOnWriteArraySet

线程安全的Blocking容器：可以作为生产者-消费者模型中的中间传递者，可以通过设定上限控制并发数量等。 ConcurrentLinkedQueue、ConcurrentLinkedDeque、ArrayBlockingQueue、LinkedBlockingQueue、LinkedBlockingDeque、PriorityBlockingQueue、SynchronousQueue

#### 多线程执行框架

ExecutorService: 可以在service中提交task， ExecutorService可以根据需求启动/停止service，可以管理控制并发线程。基础实现类：ThreadPoolExecutor，通过设置参数：核心线程数量，最大线程数量，无工作线程存活时间，线程新建策略，拒绝服务策略等来满足多种需求。

Executors ： ExecutorService的工厂方法类，通过一系列newXXX方法得到ExecutorService

CompletableFuture：和future的关系有点像Stream和Collection的关系。 CompletableFuture 本身也是一个异步调用的Future，当future状态变更时， CompletableFuture 链上声明的响应方法会被激活调用。

从开发角度看CompletableFuture 的结果无非就是正常返回和异常返回（包括任务执行过程中抛exception和任务被cancel）.
* 处理正常返回的方法：thenXXX 或者 thenXXXAsync,  假如CompletableFuture本身是在ThreadA中执行，那么正常结束时在ThreadA中继续执行thenXXX提供的回调函数。如果带Async那么这个回调函数会在另外一个ThreadB中被执行
* 处理异常返回的方法：exceptionally()
* 可以同时处理正常返回和异常返回的方法：hanlde(), handleAsync()
* 创建异步任务： supplyAsync() , runAsync()
* 创建一个已经正常结束的任务：completedFuture()
* 协调多个CompletableFuture：anyOf(), all()
一般开发中有并发多线程需求时，按如下优先级选用API是比较合理的： ExecutorService，Concurrent容器 >  concurrent包下Lock, AtomicXXX，CountDownLatch等 > synchronized ，volatile关键字。只有当对性能有极其严格要求和清楚自己在做什么时，才从最底层的支持开始构建自己的并发策略。

### 参考资料
1. 《JAVA并发编程实践》
2. [Happens-before rule](https://www.logicbig.com/tutorials/core-java-tutorial/java-multi-threading/happens-before.html)
3. [Java memory model]( http://www.cs.umd.edu/~pugh/java/memoryModel/) 
4. [Synchronization and the Java Memory Model](http://gee.cs.oswego.edu/dl/cpj/jmm.html)
5. https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html

