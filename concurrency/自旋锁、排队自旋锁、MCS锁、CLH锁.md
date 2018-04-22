### 自旋锁（Spin lock）

自旋锁是指当一个线程尝试获取某个锁时，如果该锁已被其他线程占用，就一直循环检测锁是否被释放，而不是进入线程挂起或睡眠状态。

自旋锁适用于锁保护的临界区很小的情况，临界区很小的话，锁占用的时间就很短。

```java
private AtomicReference<Thread> owner = new AtomicReference<>();

public void lock() {
      Thread current = Thread.currentThread();
      boolean flag = false;
      while (!owner.compareAndSet(null, current)){
         
       }
}

public void unLock() {
      Thread current = Thread.currentThread();
      owner.compareAndSet(current, null);
}
```

SimpleSpinLock里有一个owner属性持有锁当前拥有者的线程的引用，如果该引用为null，则表示锁未被占用，不为null则被占用。

这里用AtomicReference是为了使用它的原子性的compareAndSet方法（CAS操作），解决了多线程并发操作导致数据不一致的问题，确保其他线程可以看到锁的真实状态。

#### 缺点

1. CAS操作需要硬件的配合；
2. 保证各个CPU的缓存（L1、L2、L3、跨CPU Socket、主存）的数据一致性，通讯开销很大，在多处理器系统上更严重；
3. 没法保证公平性，不保证等待进程/线程按照FIFO顺序获得锁。

### Ticket Lock

Ticket Lock 是为了解决上面的公平性问题，类似于现实中银行柜台的排队叫号：锁拥有一个服务号，表示正在服务的线程，还有一个排队号；每个线程尝试获取锁之前先拿一个排队号，然后不断轮询锁的当前服务号是否是自己的排队号，如果是，则表示自己拥有了锁，不是则继续轮询。

当线程释放锁时，将服务号加1，这样下一个线程看到这个变化，就退出自旋。

```java
private AtomicInteger ticket = new AtomicInteger();
private AtomicInteger service = new AtomicInteger();
private int status;

public void lock() {
  int now = ticket.getAndIncrement();
  while (service.get() != now) {
  }
  status = now;
}

public void unLock() {
  int next = status +1;
  service.compareAndSet(status, next);
  System.out.println(Thread.currentThread().getName() + " unLock");
}
```

#### 缺点

Ticket Lock 虽然解决了公平性的问题，但是多处理器系统上，每个进程/线程占用的处理器都在读写同一个变量serviceNum ，每次读写操作都必须在多个处理器缓存之间进行缓存同步，这会导致繁重的系统总线和内存的流量，大大降低系统整体的性能。

### CLH锁

CLH锁也是一种基于链表的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，它不断轮询前驱的状态，如果发现前驱释放了锁就结束自旋。

```java
private class Node {
  private volatile boolean isLocked = true;
}

private volatile AtomicReference<Node> tail = new AtomicReference<>();
private volatile ThreadLocal<Node> local = new ThreadLocal<>();

private void lock() {
  Node current = new Node();
  local.set(current);
  Node prev = tail.getAndSet(current);
  if (prev != null) {
    while (prev.isLocked) {}
  }
}

private void unLock() {
  Node current = local.get();
  if (!tail.compareAndSet(current, null)) {
    current.isLocked = false;
  }
}
```



### MCS锁

MCS Spinlock 是一种基于链表的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，直接前驱负责通知其结束自旋，从而极大地减少了不必要的处理器缓存同步的次数，降低了总线和内存的开销。

```java
private class Node {
  private volatile boolean isLocked = true;
  private AtomicReference<Node> next = new AtomicReference<>();
}
private AtomicReference<Node> tail = new AtomicReference<>();

public void lock() {
  Node node = new Node();
  if (!tail.compareAndSet(null, node)) {
    Node next = tail.get();
    while (!next.next.compareAndSet(null, node)) {
      next = next.next.get();
    }
    while (node.block) {}
  }
}

public void unlock() {
  Node node = tail.get();
  if (node.next.get() != null) {
    node.next.get().block = false;
    tail.set(node.next.get());
  } else {
    tail.set(null);
  }
}
```



1. 从代码实现来看，CLH比MCS要简单得多。
2. 从自旋的条件来看，CLH是在前驱节点的属性上自旋，而MCS是在本地属性变量上自旋
3. 从链表队列来看，CLH的队列是隐式的，CLHNode并不实际持有下一个节点；MCS的队列是物理存在的。
4. CLH锁释放时只需要改变自己的属性，MCS锁释放则需要改变后继节点的属性。

注意：这里实现的锁都是独占的，且不能重入的。