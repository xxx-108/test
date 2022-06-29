# AQS 

AQS全称（AbstractQueueSynchronizer）抽象队列同步器

这个类在java.util.concurrent.locks包下面

作用：构建锁和同步器的一个框架 

使用AQS能简单且高效的构建处应用广泛的大量的同步器

比如:

ReentrantLock :可重入锁就是他构建的 

Semaphore ：信号量   控制访问特定资源的线程数量

ReentrantReadWriteLock  可重入读写锁

SynchronousQueue   同步队列

FutureTask   异步获取执行结果或者取消执行任务

当然我们可以利用它来构建满足我们需求的同步器

# AQS原理分析

AQS核心思想：

如果请求资源空闲，则将当前请求资源的线程设置为有效线程，并且将共享西苑设置为锁定状态。

如果请求资源被占用，那么就需要一套线程阻塞，等待和唤醒的时机和如何分配锁机制，这个机制就是AQS的CLH队列锁来实现的，即将获取不到锁的线程加入到队列中

CLH:

虚拟双向队列  不存在实例，仅仅存在节点之间的关系

原理：将请求访问资源的线程封装成队列节点实现锁的分配。

AQS使用一个int成员变量表示同步状态，通过内置FIFO队列完成获取资源的排队工作

FIFO：先入先出队列

AQS使用CAS对该同步状态进行原子操作实现对值的修改

CAS:Compare and Swap   比较交换     无锁算法        乐观锁。

状态信息通过：getState   setSttate    compareAndsetState来进行操作



如果当前同步状态的值等于期望值：

原子的CAS操作  将同步状态值设置为给定的update

怎么理解这句话：  三个值  同步状态值  期望值   update值

这样关联起来：当同步状态值和期望值相等     同步状态值等于update值

AQS对资源共享的方式：

AQS定义两种资源共享的方式：

Exclusive:独占：只有一个线程能执行，入ReentrantLock。

独占这种情况又可以分公平锁和非公平锁

也就是说在一个线程独占的时候，然后这个线程事情做完了，接下来谁获取这个锁

公平锁：按照线程在队列中的排队顺序，先到者先拿到锁

非公平锁：无视排队顺序，谁抢到就是谁的。

share：共享方式

多个线程可以同时执行

如Semaphore  信号锁

CountDownLatch  倒计时锁存器  Latch  锁

Semaphor  信号锁

CountDownLatch   倒计时锁存器

CycliBarrier  循环障碍锁

ReadWriteLock  读写锁

我们都会在后面讲到。



ReentrantReadWriteLock我们可以看做组合式锁，看名字就知道

它允许多个线程对统一资源进行读。

不同的自定义同步器挣用共享资源的方式也不同，自定义同步器在实现是只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等）AQS已经实现好了。

AQS底层使用了模板方法模式

同步器的设计是基于模板方法设计的，如果需要自定义同步器一般方式是这样的。

1.使用者继承AbstractQueuedSynchronizer并重写指定的方法

这些重写方法很简单，无非是对于共享资源状态state的获取和释放

2.将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写方法。

这和我们以往通过接口实现的方式有很大区别，这是模板方法很经典的一个应用。

模板方法模式：定义一个算法的总体流程步骤，然后允许子类为其中一个或者多个步骤提供特定的实现方式。



AQS提供的模板方法

isHeldExclusively()该线程是不是正在独占资源  只有用到condition才需要去实现它。

tryAcquire()独占的方式,尝试获取资源，成功则返回true,失败就返回false

tryRelease独占方式  尝试释放资源，成功返回资源，，失败返回false

tryAcquireShared（）共享方式  尝试获取资源，负数表示失败，0表示成功 ，但是没有剩余资源，正数表示成功，且有剩余资源。

tryReleaseShared共享方式  。尝试释放资源  成功就返回true  失败就返回false

子类按需要重写这些方法，就可以实现自定义同步。

默认情况下，每个方法都抛出UnsupportedOperationException，这些方法的实现必须是内部安全的，并且通常应该简短而不是阻塞，AQS类中的方法都是final的无法被其他类使用，只有这几个方法可以被其他类使用。

以ReentrantLock为例，state的初始化为0，表示未锁定的状态，A线程Lock()时，会调用tryAcquie方法独占锁并将state+1，此后其他线程再tryAcquire时就会失败，直到A线程unlock()到state=0为止，其他线程才有机会获取锁，当然释放锁之前，A线程可以重复获得该锁，就是状态要加1，这就是可重入的概念，但是要注意获取多少次就要释放多少次。这样才能保证state回到0的状态。

再以CountDownLatch为例，任务分为N个子线程去执行，state也初始化为N  N要与线程的个数一致，这N个子线程是并行执行的，每个子线程执行完后countDown次，state会CAS减一，等到所有子线程都执行完后，state=0，会unpark主调用线程，主调用线程就会从await函数返回，继续执行后续动作。

一般来说，自定义同步器要么独占方法，要么共享方式，他们也只需要实现tryAcquire  try-Release

tryAcquireShared tryReleasedShared中的一个就可以了，但AQS也支持自定义同步器实现独占和共享的方式，如ReentrantReadWriterLock。



AQS组件总结：

semaphore（信号量）允许多个线程同时访问，synchronized和ReentrantLock都是一次只允许一个线程访问某个资源。但是semaphore可以指定多个线程同时访问某个资源

CountDownLatch倒计时器：CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，这个工具通常用来控制线程等待，他可以计某一个线程等待直到倒计时结束，再开始执行。

cycliBarrier(循环栅栏) CycliBarrier和CountDownLatch非常类似，它也可以实现线程间的技术等待，但是他的功能要比CountDownLattch更加复杂，强大。主要应用场景和CountDownLatch类似，他的意思是可循环使用的屏障，他所做的事情是让一组线程达到一组屏障。同步点时阻塞，直到最后一个线程到达屏障，屏障菜户开门，构造参数传入的就是被拦截的线程数量，每个线程调用await方法告诉CyclicBarrier,我已经到达屏障，然后当前线程被阻塞。



讲了什么，讲了线程获取共享资源      和现线程同步  







































