由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工
作内存(有些地方称为栈空间)，用于存储线程私有的数据，而Java内存模型中规定所有变量都存储在
主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工
作内存中进行，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成
后再将变量写回主内存，不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷
贝，前面说过，工作内存是每个线程的私有数据区域，因此不同的线程间无法访问对方的工作内存，
线程间的通信(传值)必须通过主内存来完成。

主内存

主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该实例对象是

成员变量还是方法中的本地变量(也称局部变量)，当然也包括了共享的类信息、常量、静态变

量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

工作内存

主要存储当前方法的所有本地变量信息(工作内存中存储着主内存中的变量副本拷贝)，每个线程

只能访问自己的工作内存，即线程中的本地变量对其它线程是不可见的，就算是两个线程执行的

是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了

字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间

无法相互访问工作内存，因此存储在工作内存的数据不存在线程安全问题。

至于static变量以及类本身相关信息将会存储在主内存中。需要注意的是，在主内存中的实例对象可以被多线程共享，倘若两个线程同时调用了同一个对象

的同一个方法，那么两条线程会将要操作的数据拷贝一份到自己的工作内存中，执行完成操作后才刷新到主内存。

JVM定义了一组规则，通过这组规则来决定一个线程对共享变量的写入何

时对另一个线程可见，这组规则也称为Java内存模型（即JMM），JMM是围绕着程序执行的原子

性、有序性、可见性。

原子性

原子性指的是一个操作是不可中断的，即使是在多线程环境下，一个操作一旦开始就不会被其他线程影响。

有序性

指令重排：
计算机在执行程序时，为了提高性能，编译器和处理器的常常会对指令做重排

可见性

可见性指的是当一个线程修改了某个共享变量的值，其他线程是否能够马上得知这个修改的值

JMM提供的解决方案

原子性问题，除了JVM自身提供的对基本数据类型读
写操作的原子性外，对于方法级别或者代码块级别的原子性操作，可以使用synchronized关键字或者

重入锁(ReentrantLock)保证程序执行的原子性。

可见性问题,工作内存与主内存同步延迟现象导致的，可以使用synchronized关键字或者volatile关键字解决，它们都可以使一个线程修改后的变量立即对

其他线程可见。

对于指令重排导致的可见性问题和有序性问题，则可以利用volatile关键字解决，因为volatile的另外一个作用就是禁止重排序优化。

除了靠sychronized和volatile关键字来保证原子性、可见性以及有序性外，JMM内部还定义一套happens­before 原则来保证多线程环境下两个操作间的原子性、可见性以及有序性。

happens-before

1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
2. 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法。

下面是happens-before原则规则：

程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作；注意是执行结果，因为虚拟机、处理器会对指令进行重排序（重排序后面会详细介绍）。虽然重排序了，但是并不会影响程序的执行结果，所以程序最终执行的结果与顺序执行的结果是一致的。故而这个规则只对单线程有效，在多线程环境下无法保证正确性。

锁定规则：一个unLock操作先行发生于后面对同一个锁额lock操作；

volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作；

传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C；

线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作；

线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；

线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行；

对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始；
