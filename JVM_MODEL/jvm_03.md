#锁的释放-获取建立的happens before关系

##释放锁 和 获取锁的内存语义
- 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
- 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使得被监视器保护的临界区代码必须要从主内存中去读取共享变量。

对比锁释放-获取的内存语义与volatile写-读的内存语义，可以看出：锁释放与volatile写有相同的内存语义；锁获取与volatile读有相同的内存语义。


##锁内存语义的底层实现

这里以可重入锁为例，在ReentrantLock中，调用lock()方法获取锁；调用unlock()方法释放锁。

ReentrantLock的实现依赖于java同步器框架AbstractQueuedSynchronizer（本文简称之为AQS）。**AQS使用一个整型的volatile变量（命名为state）来维护同步状态**，马上我们会看到，这个volatile变量是ReentrantLock内存语义实现的关键。 

[![Cjcqts.png](https://s1.ax1x.com/2018/06/15/Cjcqts.png)](https://imgchr.com/i/Cjcqts)

在非公平锁的加锁实现中，compareAndSet()方法以原子操作的方式更新state变量，可以把java的compareAndSet()方法调用简称为CAS。JDK文档对该方法的说明如下：如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。**此操作具有 volatile 读和写的内存语义**。
>CAS是原子操作 且 具有volatile 读和写的内存语义


**编译器不会对volatile读与volatile读后面的任意内存操作重排序；编译器不会对volatile写与volatile写前面的任意内存操作重排序。组合这两个条件，意味着为了同时实现volatile读和volatile写的内存语义，编译器不能对CAS与CAS前面和后面的任意内存操作重排序。**这意味着CAS的操作前后都是不可重排序的！！

>对于volatile变量的重排序要求如下：

	编译器不会对volatile写与volatile写前面的任意内存操作重排序。
	编译器不会对volatile读与volatile读后面的任意内存操作重排序

现在对公平锁和非公平锁的内存语义做个总结：

	   -公平锁和非公平锁释放时，最后都要写一个volatile变量state。
	
	   -公平锁获取时，首先会去读这个volatile变量。
	
	   -非公平锁获取时，首先会用CAS更新这个volatile变量,这个操作同时具有volatile读和volatile写的内存语义。

从对ReentrantLock的分析可以看出，锁释放-获取的内存语义的实现至少有**下面两种方式**：

	1.利用volatile变量的写-读所具有的内存语义。
	
	2.利用CAS所附带的volatile读和volatile写的内存语义。


##concurrent包的实现
由于java的CAS同时具有 volatile 读和volatile写的内存语义，因此Java线程之间的通信现在有了下面四种方式：（就是 2*2 ）

    A线程写volatile变量，随后B线程读这个volatile变量。
    A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
    A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
    A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。


如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式：

    首先，声明共享变量为volatile；
    然后，使用CAS的原子条件更新来实现线程之间的同步；
    同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

AQS，非阻塞数据结构和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。从整体来看，concurrent包的实现示意图如下：

>从此图可以看到 同步基本上还是依赖于 CAS 和 volatile的内存语义的

[![Cjcbkj.md.png](https://s1.ax1x.com/2018/06/15/Cjcbkj.md.png)](https://imgchr.com/i/Cjcbkj)