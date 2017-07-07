# javadoc
# 面试准备

## java设计模式
 - 策略模式 : 通过接口来实现,同一种行为，不同的实现方式(通过构造器的构造参数不同使得实现有所不同),和多态相似，多态是通过继承来实现
 - 职责链模式 :  设置每个职责的执行顺序,
来完成一系列业务逻辑的执行过程, 每个职责类通过上下文对象实现数据共享与串联
 - 观察者模式 : 耦合,根 发布/订阅模式的区别是,观察者模式是由具体主题(Subject)调度的,而发布/订阅模式是统一由调度中心调的。
 Subject -> ConcreteSubject 具体主题 (具体被观察者） add,remove,notify(notify负责调度)
 Observer -> ConcrereObserver 具体观察者

## hashmap
    当系统开始初始化 HashMap 时，系统会创建一个长度为 capacity 的 Entry 数组，这个数组里可以存储元素的位置被称为“桶（bucket）”，每个 bucket 都有其指定索引，系统可以根据其索引快速访问该 bucket 里存储的元素。
    如果 HashMap 的每个 bucket 里只有一个 Entry 时，HashMap 可以根据索引、快速地取出该 bucket 里的 Entry；在发生“Hash 冲突”的情况下，单个 bucket 里存储的不是一个 Entry，而是一个 Entry 链，系统只能必须按顺序遍历每个 Entry，直到找到想搜索的 Entry 为止——如果恰好要搜索的 Entry 位于该 Entry 链的最末端（该 Entry 是最早放入该 bucket 中），那系统必须循环到最后才能找到该元素。

### put函数大致的思路为：
    1. 对key的hashCode()做hash，然后再计算index;
    2. 如果没碰撞直接放到bucket里；
    3. 如果碰撞了，以链表的形式存在bucket；
    4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
    5. 如果节点已经存在就替换old value(保证key的唯一性)
    6. 如果bucket满了(超过load factor*current capacity)，就要resize。

### get操作
    1. bucket里的第一个节点，直接命中；
    2. 如果有冲突，则通过key.equals(k)去查找对应的entry
    3. 若为树，则在树中通过key.equals(k)查找，O(logn)；
    4. 若为链表，则在链表中通过key.equals(k)查找，O(n)。

## java多线程应用
 - javaWEB开发  程序员根本应用不到多线程编程, 框架底层已经做好了, bio,nio,aio
 - 自身业务 遇到IO操作需要返回值 为了提升性能 可以开多线程,可以用Future+CompletableFuture 组合


## Class.forName、ClassLoader、new
    - Class.forName除了将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块,运行时检查检查.class类型和路径
    - ClassLoader只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块
    - new关键字 隐式调用classLoader到JVM,然后是实例化(loadClass + newInstance),是在编译时就要检查.class类型和路径

## 双亲委派模型（Parent Delegation Model）
### 双亲委派模型的工作过程为：
    1. 当前 ClassLoader 首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。
    每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，
    等下次加载的时候就可以直接返回了。

    2. 当前 classLoader 的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到 bootstrap ClassLoader.
当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。


## Minor GC 和 Full GC
    新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。

    老年代GC（Major GC / Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上。


    新生代的GC：
    新生代通常存活时间较短，因此基于Copying算法来进行回收，所谓Copying算法就是扫描出存活的对象，并复制到一块新的完全未使用的空间中，对应于新生代，就是在Eden和FromSpace或ToSpace之间copy。新生代采用空闲指针的方式来控制GC触发，指针保持最后一个分配的对象在新生代区间的位置，当有新的对象要分配内存时，用于检查空间是否足够，不够就触发GC。当连续分配对象时，对象会逐渐从eden到survivor，最后到旧生代

    旧生代与新生代不同，对象存活的时间比较长，比较稳定，因此采用标记(Mark)算法来进行回收，所谓标记就是扫描出存活的对象，然后再进行回收未被标记的对象，回收后对用空出的空间要么进行合并，要么标记出来便于下次进行分配，总之就是要减少内存碎片带来的效率损耗。在执行机制上JVM提供了串行GC(SerialMSC)、并行GC(parallelMSC)和并发GC(CMS)，具体算法细节还有待进一步深入研究。

## 吞吐量
    吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间）。
    虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。
##  JVM垃圾回收机制
    JVM分别对新生代和旧生代采用不同的垃圾回收机制

    1. 垃圾回收器目前分为四种类型, 串行，并行，并发标记，G1。
    2. 小数据量和小型应用，使用串行垃圾回收器即可。
    3. 对于对响应时间无特殊要求的，可以使用并行垃圾回收器和并发标记垃圾回收器。（中大型应用）
    4. 对于heap可以分配很大的中大型应用，使用G1垃圾回收器比较好，进一步优化和减少了GC暂停时间。
    5. 没有银弹，针对不同的场景，选用不同的垃圾回收器。


    Serial	新生代	单线程	复制（新）、标记-整理（老）
    ParNew	新生代	多线程	复制（新）、标记-整理（老）
    Parallel Scavenge	新生代	多线程	吞吐量优先算法
    Serial Old	老生代	单线程	复制（新）、标记-整理（老）
    Parallel Old	老生代	多线程	复制（新）、标记-整理（老）
    CMS	老生代	多线程	标记-清除算法（初始标记、并发标记、重新标记、并发清除）
    G1	新生代&&老生代	多线程	标记-整理算法（初始标记、并发标记、最终标记、筛选回收）


## java四种引用方式
    1. 强引用 比如：Object object =new Object();  String str ="hello";
    2. 软引用（SoftReference）
    3. 弱引用（WeakReference）
    4. 虚引用（PhantomReference）

## JVM Runtime Area
 - 程序员写的所有程序都被加载到运行时数据区域中，不同类别存放在heap, java stack, native method stack, PC register, method area.

 - 1、PC程序计数器(PC register)：一块较小的内存空间，可以看做是当前线程所执行的字节码的行号指示器, NAMELY存储每个线程下一步将执行的JVM指令，如该方法为native的，则PC寄存器中不存储任何信息。Java 的多线程机制离不开程序计数器，每个线程都有一个自己的PC，以便完成不同线程上下文环境的切换。

 - 2、java虚拟机栈(java stack)：与 PC 一样，java 虚拟机栈也是线程私有的。每一个 JVM 线程都有自己的 java 虚拟机栈，这个栈与线程同时创建，它的生命周期与线程相同。虚拟机栈描述的是Java 方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

 - 3、本地方法栈(native method stack)：与虚拟机栈的作用相似，虚拟机栈为虚拟机执行执行java方法服务，而本地方法栈则为虚拟机使用到的本地方法服务。

 - 4、Java堆(heap)：被所有线程共享的一块存储区域，在虚拟机启动时创建，它是JVM用来存储对象实例以及数组值的区域，可以认为Java中所有通过new创建的对象的内存都在此分配。
 Java堆在JVM启动的时候就被创建，堆中储存了各种对象，这些对象被自动管理内存系统（Automatic Storage Management System，也即是常说的 “Garbage Collector（垃圾回收器）”）所管理。这些对象无需、也无法显示地被销毁。


 - 5、方法区(method area)
方法区和堆区域一样，是各个线程共享的内存区域，它用于存储每一个类的结构信息，例如运行时常量池，成员变量和方法数据，构造函数和普通函数的字节码内容，还包括一些在类、实例、接口初始化时用到的特殊方法。当开发人员在程序中通过Class对象中的getName、isInstance等方法获取信息时，这些数据都来自方法区。

方法区也是全局共享的，在虚拟机启动时候创建。在一定条件下它也会被GC。这块区域对应Permanent Generation 持久代。 XX：PermSize指定大小。

 - 6、运行时常量池
其空间从方法区中分配，存放的为类中固定的常量信息、方法和域的引用信息。


## java heap
 - JVM将Heap分为两块：新生代New Generation和旧生代Old Generation
 - 堆在JVM是所有线程共享的，因此在其上进行对象内存的分配均需要进行加锁，这也是new开销比较大的原因。
 - 鉴于上面的原因，Sun Hotspot JVM为了提升对象内存分配的效率，对于所创建的线程都会分配一块独立的空间，这块空间又称为TLAB,
 - TLAB仅作用于新生代的Eden Space，因此在编写Java程序时，通常多个小的对象比大的对象分配起来更加高效
 - 新生代 Young Generation : 1.Eden Space -> 2.S0 Suvivor Space -> S1 Survivor Space

    1. Eden Space 任何新进入运行时数据区域的实例都会存放在此
    2. S0 Suvivor Space 存在时间较长，经过垃圾回收没有被清除的实例，就从Eden 搬到了S0
    3. S1 Survivor Space 同理，存在时间更长的实例，就从S0 搬到了S1

 - 旧生代 Old Generation/tenured : 同理，存在时间更长的实例，对象多次回收没被清除，就从S1 搬到了tenured

 - Perm 存放运行时数据区的方法区(java8换成了元空间metaspace)

 - 不同的世代使用不同的 GC 算法。

### 为什么移除PermGen space(永久带)  
 - 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
 - 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
 - 字符串存在永久代中，容易出现性能问题和内存溢出。


何为垃圾？
Java中那些不可达的对象就会变成垃圾。那么什么叫做不可达？其实就是没有办法再引用到该对象了。主要有以下情况使对象变为垃圾：
1.对非线程的对象来说，所有的活动线程都不能访问该对象，那么该对象就会变为垃圾。
2.对线程对象来说，满足上面的条件，且线程未启动或者已停止。


## jvm 对象分配规则
    - 对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。
    - 大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）

## Minor GC、Major GC、Full GC
### Minor GC 从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC

### Full GC 是清理整个堆空间—包括新生代、老生代、元空间
#### FULL GC 触发条件
    1. 老年代空间不足
    2. 元空间不足


## MQ
 - 消息丢失： 把消息持久化,没有消费就把消息堆积在那里
 - 消息重复： 网络不可达。只要通过网络交换数据，就无法避免这个问题,消费端处理消息的业务逻辑保持幂等性
 - 消息顺序： 默认是有序的,不能完全保证消息有序,不用太care消息顺序的问题(要理论上保证100%有序,就要牺牲消息可能会丢失的情况)


### mq事务消息
    本地事务，本地落地，补偿发送。
    本质上是对消息中间件的一种特殊利用，它是将本地事务和发消息放在了一个分布式事务里，保证要么本地操作成功 成功并且对外发消息成功，要么两者都失败，开源的RocketMQ就支持这一特性

    基于消息中间件的两阶段提交往往用在高并发场景下，将一个分布式事务拆成一个消息事务（A系统的本地操作+发消息）+B系统的本地操作，其中B系统的操作由消息驱动，只要消息事务成功，那么A操作一定成功，消息也一定发出来了，这时候B会收到消息去执行本地操作，如果本地操作失败，消息会重投，直到B操作成功，这样就变相地实现了A与B的分布式事务。

    虽然上面的方案能够完成A和B的操作，但是A和B并不是严格一致的，而是最终一致的，我们在这里牺牲了一致性，换来了性能的大幅度提升。当然，这种玩法也是有风险的，如果B一直执行不成功，那么一致性会被破坏，具体要不要玩，还是得看业务能够承担多少风险。

    1. 发送预备消息
    2. 发送成功执行本地事务
    3. commit or rollback
    4. 实现check 回调, mq broker 会定时检查你的本地事务状态,来决定是commit还是rollback

## dubbo

### dubbo的容错策略
    1. Failover 失效切换（缺省）,失败自动切换，当出现失败，重试其它服务器。
    2. Failfast 快速失败, 快速失败，只发起一次调用，失败立即报错。
    3. failsaft cluster 失败安全，出现异常时，直接忽略，通常用于写入审计日志等操作
    4. failback cluster 失败自动恢复，后台记录失败请求，定时重发，通常用于消息通知操作
    5. forking cluster 并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多的服务器资源。可通过forks=“2”来设置最大并行数。




## oauth2.0
### 授权吗模式 (一般是服务端 对接 服务端 )

- 客户端通过app_key + scope + state(防止CSRF攻击,随机码) + redirect_uri + response_type 向服务器换取授权码
 资源服务器通过 客户端的 redirect_uri 把code返回给客户端, code有过期时间（10分钟),只能用一次
- 客户端 通过 app_key + app_secret + code + redirect_uri + grant_type 获取access token
- 资源服务器 会返回 accessToken,refreshToken,expires_in
- accessToken 和 refreshToken 都有过期时间, accessToken 的过期时间相对比较短，accessToken过期了 可用app_key + refreshToken 重新获取,
refreshToken 过期了 需要重新 授权

### 客户端模式 (服务端对客户端)

- app端 通过公钥 + sign(秘钥,uri,ticket,timestamp) 获取 accessToken
- 资源服务器返回 accessToken,refreshToken,expires_in
- accessToken 和 refreshToken 都有过期时间, accessToken 的过期时间相对比较短，accessToken过期了 可用app_key + refreshToken + sign 重新获取,
  refreshToken 过期了 需要重新 授权


## Collection
不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator
方式,如果并发操作,需要对 Iterator 对象加锁。
```java
final void checkForComodification() {
    if (modCount != expectedModCount)
    throw new ConcurrentModificationException();
}
```

ArrayList.add()方法，每执行一次都会modCount++(当然删除就是modCount--)，但不改变expectedModCount的值。expectedModCount的值是在构建迭代的时候初始为expectedModCount=modCount的。

这就是楼主说的在构建迭代器之后，再使用ArrayList.add()方法就造成了modCount != expectedModCount

所以构建迭代器后，用迭代器来add和remove就没有问题。因为它会在改变modCount的值之后，又把值赋给了expectedModCount，从而保证modCount=expectedModCount
expectedModCount是迭代器里的变量, 获得迭代器的时候就会赋值, 而modCount是List中的变量, 会随着集合修改而变的


## Timer
多线程并行处理定时任务时,Timer 运行多个 TimeTask 时,只要其中之一没有捕获 抛出的异常,其它任务便会自动终止运行,使用 ScheduledExecutorService 则没有这个问题

## Random (实例包括 java.util.Random 的实例或者 Math.random()的方式)
避免 Random 实例被多线程使用,虽然共享该实例是线程安全的,但会因竞争同一 seed 导致的性能下降。
在 JDK7 之后,可以直接使用 API ThreadLocalRandom,而在 JDK7 之前,需要编码保 证每个线程持有一个实例。
