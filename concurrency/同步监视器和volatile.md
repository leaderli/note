**监视器**

![同步监视器](/Users/li/Documents/note/noteImgPool/同步监视器.png)

synchronized,wait,notify是任何对象都有的同步工具，他们是应用于同步问题的线程调度工具。纠其本质，java中每一个对象都有一个监视器，来检测并发代码的重入。在非多线程编码时，该监视器不发挥作用，反之如果在synchronized范围内，监视器发挥作用。

wait和notify必须存在在synchronized块中，这三个关键字是针对同一监视器(某对象的监视器)。这意味着，只有调用wait后，其它线程才可以进入同步块执行。

当某代码不再持有监视器的使用权是，如图中5的状态，即脱离同步代码块，如果调用wait或notify，会抛出java.lang.IllegalThreadStateException异常。如果在synchronized块调用另外一个对象的wait或notify，因为对象的监视器不同，也会抛出此异常。

**volatile**

![多线程内存占用](/Users/li/Documents/note/noteImgPool/多线程内存占用.png)

多线程的内存模型，main memory(主存)，working memory(线程栈)，在处理数据时，线程会把值从主存load到本地栈，完成操作后，再save回去。(volatile关键字的作用就是在一个CPU时间片内，针对变量的操作都触发一次完整的load和save)。针对多线程使用的变量如果不是volatile或final修饰的，可能发生不可预料的后果(某个线程修改了变量，但之后的线程load到的是变量修改前的数据)。其实本质上来说，同一实例的同一属性只会有一个副本，但多线程会缓存值，volatile的作用就是不去缓存值。在线程安全的情况下，用volatile会牺牲性能。

**底层实现**

volatile底层实现最终是加了内存屏障

1. 保证写volatile变量强制把CPU写缓存区的数据刷新到内存
2. 读volatile变量时，使缓存失效，强制从内存中读取数据。
3. 由于内存屏障的作用，volatile变量还能阻止重排序。

