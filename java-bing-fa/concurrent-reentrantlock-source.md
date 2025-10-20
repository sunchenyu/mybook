# ReentrantLock源码分析

一、基本使用方法

```
ReentrantLock reentrantLock = new ReentrantLock();

reentrantLock.lock();

reentrantLock.unlock();
```

## 二、查看方法实现

### （1）实例化

```
public ReentrantLock() {
    sync = new NonfairSync();
}
```

### （2）lock方法

```
public void lock() {
    sync.lock();
}
```

### （3）unlock方法

```
public void unlock() {
    sync.release(1);
}
```

都是调用了NonfairSync对象中的方法，那我们查看一下这个NonfairSync是什么

## 三、NonfairSync

### 继承关系

NonfairSync => Sync => AbstractQueuedSynchronizer => AbstractOwnableSynchronizer

### （1）AbstractOwnableSynchronizer

关键是持有一个Thread 变量，表示一个锁必须要记录当前拥有的线程是什么。

```
private transient Thread exclusiveOwnerThread;
```

### （2）AbstractQueuedSynchronizer

关键是持有head，tail，state。主要作用是维护一个同步的线程等待队列和锁记录标识。线程等待队列的实现依靠head、tail节点和CAS方法。锁依靠state字段和CAS方法。

```
    private transient volatile Node head;

    private transient volatile Node tail;

    private volatile int state;
```

### （3）Sync和NonfairSync

利用AbstractQueuedSynchronizer中封装的方法，实现锁的逻辑。

## 四、锁的使用流程分析

### （1）lock方法

作用：尝试加锁，加锁成功则标记当前线程、加锁失败则将当前线程挂起，线程入队，等待唤醒。

```
final void lock() {
    //获取锁成功，则记录锁持有的线程为当前线程
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        //获取锁失败，则需要将当前线程挂起，并放入到队列中
        acquire(1);
}
```

compareAndSetState方法，利用unsafe中的CAS修改state状态，即锁的状态。锁获取成功返回true，失败返回false。

```
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

setExclusiveOwnerThread方法，记录当前持有锁的线程。

```
protected final void setExclusiveOwnerThread(Thread thread) {
    exclusiveOwnerThread = thread;
}
```

关键点在于acquire方法，最终目的是挂起当前线程，放入队列，并在被唤醒之后重新抢锁。

```
public final void acquire(int arg) {
    //再次抢锁
    //如果抢锁成功，则不进行acquireQueued方法（也就是不需要挂起和入队）
    //如果抢锁失败，则进行acquireQueued方法，（目的是挂起当前线程，放入队列）
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

tryAcquire方法 最终会走到的方法是nonfairTryAcquire，抢锁的具体实现，成功返回true，失败返回false。

```
final boolean nonfairTryAcquire(int acquires) {
    //拿到当前线程
    final Thread current = Thread.currentThread();
    //获取state字段的状态
    int c = getState();
    //c为0时，才能去抢锁
    if (c == 0) {
        //抢锁成功 返回true
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //如果是持有锁的线程是当前线程
    else if (current == getExclusiveOwnerThread()) {
        //计数+1之后，设置state，返回true
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    //抢锁失败
    return false;
}
```

addWaiter方法，将当前线程包装成Node节点，并且通过死循环和CAS完成同步的入队操作。acquireQueued主要是在入队操作之后，阻塞当前的线程。

```
    private Node addWaiter(Node mode) {
        //先将当前的线程构建成一个node对象
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;

        //如果tail节点不为null，就尝试把当前线程构造的node节点加入到tail后面
        if (pred != null) {
            node.prev = pred;
            //尝试将新建的node加入到队尾，如果失败的话，可能有多种结果，所以后面的enq会死循环的去处理
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }

        //如果tail节点为null，或者上一步尝试加入尾节点失败，那就调用enq方法
        enq(node);
        return node;
    }
```

enq方法。作用：死循环的将当前线程入队，同步入队的关键方法。

```
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        //如果tail为null，表示还没有初始化head和tail，设置head节点和头节点
        if (t == null) { // Must initialize（初始化操作）
            //创建一个新的node，赋值给head
            if (compareAndSetHead(new Node()))
                //赋值完给head之后，让tail和head指向同一个新的node
                tail = head;
        } else {
            node.prev = t;
            //将tail设置为当前的node
            if (compareAndSetTail(t, node)) {
                //将上次的tail的next修改为当前的node
                t.next = node;

                //这个地方才会结束循环，返回的是当前线程node
                return t;
            }
        }
    }
}
```

acquireQueued方法 作用：阻塞当前线程，并在被唤醒之后重新抢锁。

```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //获取node的前置节点
            final Node p = node.predecessor();
            //如果是队头，则进行抢锁操作
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //不是队列的线程，或者抢锁失败的线程（这种线程需要挂起等待唤醒）
            if (shouldParkAfterFailedAcquire(p, node) &&
                //挂起当前线程
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### （2）unlock方法

作用：解锁，唤醒下一个线程。

```
  public void unlock() {
        sync.release(1);
    }
```

release方法。具体解锁的实现。

```
public final boolean release(int arg) {
    //如果解锁成功
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //释放成功的话，需要去唤醒线程
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

tryRelease方法。作用：尝试释放独占锁。

```
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    //如果当前线程不是持有锁的线程，表示存在问题。
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    //0的情况下才会完全释放锁
    if (c == 0) {
        free = true;
        //设置锁的持有线程为null
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

unparkSuccessor方法。作用：唤醒队列中下一个线程。

```
private void unparkSuccessor(Node node) {

    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //目的就是为了调用unpak方法唤醒线程
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
