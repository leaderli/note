**概述**

 AbstractQueuedSynchronizer(AQS),抽象的队列式同步框架，许多同步类实现都依赖于它。

**说明**

(01) 关于waitStatus请参考下表(中扩号内为waitStatus的值)，更多关于waitStatus的内容，可以参考前面的Node类的介绍。

```shell
CANCELLED[1]  -- 当前线程已被取消

SIGNAL[-1]    -- “当前线程的后继线程需要被unpark(唤醒)”。一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。

CONDITION[-2] -- 当前线程(处在Condition休眠状态)在等待Condition唤醒

PROPAGATE[-3] -- (共享锁)其它线程获取到“共享锁”

[0]           -- 当前线程不属于上面的任何一种状态。
```

(02) shouldParkAfterFailedAcquire()通过以下规则，判断“当前线程”是否需要被阻塞。

```shell
规则1：如果前继节点状态为SIGNAL，表明当前节点需要被unpark(唤醒)，此时则返回true。

规则2：如果前继节点状态为CANCELLED(ws>0)，说明前继节点已经被取消，则通过先前回溯找到一个有效(非CANCELLED状态)的节点，并返回false。

规则3：如果前继节点状态为非SIGNAL、非CANCELLED，则设置前继的状态为SIGNAL，并返回false。
```





**框架**

![AQS等待线程](/Users/li/Documents/note/noteImgPool/AQS等待线程.png)

它维护了一个volatile int state(代表共享资源)和一个FIFO线程等待队列(多线程争用被阻塞时进入此队列)，state的访问方式有三种：

- getState()

- setState()

- compareAndSetState()

自定义同步器在实现时只需要实现共享资源的state的获取与释放即可，至于具体线程等待队列的维护，AQS在顶层实现好了。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程调用lock()方法时，会调用tryAcquire()独占锁并将state+1。此后，其它线程再tryAcquire()就会失败，直到线程A调用unlock()到state＝0(即释放锁)。当然，锁在释放之前，A线程自己可以重复获得此锁(state会累加)，这就是可重入的概念。但是要注意，获取多少次就要释放多少次，这样才能保证state回到零状态。

**源码详解**

##acquire(int)

此方法是独占模式下获取共享资源的顶层入口。如果获取到资源，线程直接返回，否则进入等待队列，直到线程获得资源为止，且整个过程忽略中断的影响。这也正是lock的语义，当然不仅仅只限于lock()。获取到资源后，线程就可以执行其临界区代码了。

- tryAcquire()尝试直接去获取资源，如果成功则直接返回。


```java
protected boolean tryAcquire(int arg) {
     throw new UnsupportedOperationException();
 }
```

什么？直接throw异常？说好的功能呢？好吧，**还记得概述里讲的AQS只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现吗？**就是这里了！！！AQS这里只定义了一个接口，具体资源的获取交由自定义同步器去实现了（通过state的get/set/CAS）！！！至于能不能重入，能不能加塞，那就看具体的自定义同步器怎么去设计了！！！当然，自定义同步器在进行资源访问时要考虑线程安全的影响。

　　这里之所以没有定义成abstract，是因为独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。说到底，Doug Lea还是站在咱们开发者的角度，尽量减少不必要的工作量。

- addWaiter()将线程加入等待队列的尾部，并标记为独占模式。

  ​	此方法用于将当前线程加入到等待队列的队尾，并返回当前线程所在的结点。

  ```java
  private Node addWaiter(Node mode) {
      //以给定模式构造结点。mode有两种：EXCLUSIVE（独占）和SHARED（共享）
      Node node = new Node(Thread.currentThread(), mode);
      
      //尝试快速方式直接放到队尾。
      Node pred = tail;
      if (pred != null) {
          node.prev = pred;
          if (compareAndSetTail(pred, node)) {
              pred.next = node;
              return node;
          }
      }
      
      //上一步失败则通过enq入队。
      enq(node);
      return node;
  }
  ```

  ```java
  private Node enq(final Node node) {
      //CAS"自旋"，直到成功加入队尾
      for (;;) {
          Node t = tail;
          if (t == null) { // 队列为空，创建一个空的标志结点作为head结点，并将tail也指向它。
              if (compareAndSetHead(new Node()))
                  tail = head;
          } else {//正常流程，放入队尾
              node.prev = t;
              if (compareAndSetTail(t, node)) {
                  t.next = node;
                  return t;
              }
          }
      }
  }
  ```

- acquireQueued()使线程在等待队列中获取资源，一直等待直到获取到资源才返回。如果在真个等待过程中被中断过，则返回true，否则返回false

  ​	OK，通过tryAcquire()和addWaiter()，该线程获取资源失败，已经被放入等待队列尾部了。聪明的你立刻应该能想到该线程下一部该干什么了吧：**进入等待状态休息，直到其他线程彻底释放资源后唤醒自己，自己再拿到资源，然后就可以去干自己想干的事了**。没错，就是这样！是不是跟医院排队拿号有点相似~~acquireQueued()就是干这件事：**在等待队列中排队拿号（中间没其它事干可以休息），直到拿到号后再返回**。这个函数非常关键，还是上源码吧

  ```java
  final boolean acquireQueued(final Node node, int arg) {
      boolean failed = true;//标记是否成功拿到资源
      try {
          boolean interrupted = false;//标记等待过程中是否被中断过
          
          //又是一个“自旋”！
          for (;;) {
              final Node p = node.predecessor();//拿到前驱
              //如果前驱是head，即该结点已成老二，那么便有资格去尝试获取资源（可能是老大释放完资源唤醒自己的，当然也可能被interrupt了）。
              if (p == head && tryAcquire(arg)) {
                  setHead(node);//拿到资源后，将head指向该结点。所以head所指的标杆结点，就是当前获取到资源的那个结点或null。
                  p.next = null; // setHead中node.prev已置为null，此处再将head.next置为null，就是为了方便GC回收以前的head结点。也就意味着之前拿完资源的结点出队了！
                  failed = false;
                  return interrupted;//返回等待过程中是否被中断过
              }
              
              //如果自己可以休息了，就进入waiting状态，直到被unpark()
              if (shouldParkAfterFailedAcquire(p, node) &&
                  parkAndCheckInterrupt())
                  interrupted = true;//如果等待过程中被中断过，哪怕只有那么一次，就将interrupted标记为true
          }
      } finally {
          if (failed)
              cancelAcquire(node);
      }
  }
  ```

  此方法主要用于检查状态，看看自己是否真的可以去休息了（进入waiting状态，如果线程状态转换不熟，万一队列前边的线程都放弃了只是瞎站着，那也说不定，对吧！

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;//拿到前驱的状态
    if (ws == Node.SIGNAL)
        //如果已经告诉前驱拿完号后通知自己一下，那就可以安心休息了
        return true;
    if (ws > 0) {
        /*
         * 如果前驱放弃了，那就一直往前找，直到找到最近一个正常等待的状态，并排在它的后边。
         * 注意：那些放弃的结点，由于被自己“加塞”到它们前边，它们相当于形成一个无引用链，稍后就会被保安大叔赶走了(GC回收)！
         */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
         //如果前驱正常，那就把前驱的状态设置成SIGNAL，告诉它拿完号后通知自己一下。有可能失败，人家说不定刚刚释放完呢！
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

整个流程中，如果前驱结点的状态不是SIGNAL，那么自己就不能安心去休息，需要去找个安心的休息点，同时可以再尝试下看有没有机会轮到自己拿号。

如果线程找好安全休息点后，那就可以安心去休息了。此方法就是让线程去休息，真正进入等待状态。

```java
private final boolean parkAndCheckInterrupt() {
  	LockSupport.park(this);//调用park()使线程进入waiting状态
    return Thread.interrupted();//如果被唤醒，查看自己是不是被中断的。
}
```





![acquire](/Users/li/Documents/note/noteImgPool/acquire.png)

##release(int)



上一小节已经把acquire()说完了，这一小节就来讲讲它的反操作release()吧。此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。这也正是unlock()的语义，当然不仅仅只限于unlock()。下面是release()的源码：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

　逻辑并不复杂。它调用tryRelease()来释放资源。有一点需要注意的是，**它是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！所以自定义同步器在设计tryRelease()的时候要明确这一点！！**

```java
protected boolean tryRelease(int arg) {
   throw new UnsupportedOperationException();
}
```

跟tryAcquire()一样，这个方法是需要独占模式的自定义同步器去实现的。正常来说，tryRelease()都会成功的，因为这是独占模式，该线程来释放资源，那么它肯定已经拿到独占资源了，直接减掉相应量的资源即可(state-=arg)，也不需要考虑线程安全的问题。但要注意它的返回值，上面已经提到了，release()是根据tryRelease()的返回值来判断该线程是否已经完成释放掉资源了！所以自义定同步器在实现时，如果已经彻底释放资源(state=0)，要返回true，否则返回false。

```java
private void unparkSuccessor(Node node) {
    //这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

这个函数并不复杂。一句话概括：**用unpark()唤醒等待队列中最前边的那个未放弃线程**，这里我们也用s来表示吧。此时，再和acquireQueued()联系起来，s被唤醒后，进入if (p == head && tryAcquire(arg))的判断（即使p!=head也没关系，它会再进入shouldParkAfterFailedAcquire()寻找一个安全点。这里既然s已经是等待队列中最前边的那个未放弃线程了，那么通过shouldParkAfterFailedAcquire()的调整，s也必然会跑到head的next结点，下一次自旋p==head就成立啦），然后s把自己设置成head标杆结点，表示自己已经获取到资源了，acquire()也返回了！！And then, DO what you WANT!



