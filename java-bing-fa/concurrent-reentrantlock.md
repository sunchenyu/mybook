# 手写ReentrantLock锁

## 一、前置准备条件

我们需要知道如何进行比较并交换的原子操作、如何进行线程的挂起和唤醒。&#x20;

（1）[Unsafe中的CAS ](https://1010329679.gitbook.io/suncy/java/unsafe-cas)

（2）[Unsafe中的park和unpark](https://1010329679.gitbook.io/suncy/java/unsafe-park)

## 二、一个错误的锁

```
package com.suncy.article.article6;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.LinkedBlockingQueue;

public class ErrorLockCode {
    private int state = 0;
    private LinkedBlockingQueue<Thread> threadLinkedBlockingQueue = new LinkedBlockingQueue<Thread>();

    //unsafe包 和 state字段的偏移量
    private static final Unsafe unsafe;
    private static final long stateOffset;

    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
            stateOffset = unsafe.objectFieldOffset
                    (ErrorLockCode.class.getDeclaredField("state"));
        } catch (Exception ex) {
            throw new Error(ex);
        }
    }

    public void lock() {
        if (!unsafe.compareAndSwapInt(this, stateOffset, 0, 1)) {
            //修改state失败，获取锁失败，线程加入队列，等待被唤醒
            threadLinkedBlockingQueue.add(Thread.currentThread());
            unsafe.park(false, 0);
            threadLinkedBlockingQueue.poll();
        }
    }

    public void unlock() {
        if (unsafe.compareAndSwapInt(this, stateOffset, 1, 0)) {
            //修改state成功，表示解锁成功，唤醒线程
            Thread thread = threadLinkedBlockingQueue.peek();
            unsafe.unpark(thread);
        }
    }

    public static void main(String[] args) {
        ErrorLockCode errorLockCode = new ErrorLockCode();
        for (int i = 0; i < 100; i++) {
            int finalI = i;
            new Thread(() -> {
                errorLockCode.lock();
                System.out.println("==========" + finalI);
                System.out.println("加锁成功" + finalI);
                System.out.println("执行业务" + finalI);
                System.out.println("解锁成功" + finalI);
                System.out.println("==========" + finalI);
                errorLockCode.unlock();
            }).start();
        }
    }
}
```

**测试结果：**

![测试结果](<../.gitbook/assets/image (46).png>)

**流程描述：**&#x20;

1、在lock方法中，尝试修改state状态0，修改成功，则表示锁成功。修改失败，则将线程放入到队列，并使用park挂起当前线程。

&#x20;2、在unlock方法中，尝试修改state状态1，修改成功，则表示解锁，并唤醒线程中队头的数据。修改失败，不进行任何处理。

**错误原因分析：**

&#x20;我们第一个线程肯定是成功的，但当第一个线程进行了unlock方法的时候，这个unlock会唤醒一个线程，假如这个时候又重新启了一个线程，那么会造成两个线程同时执行到lock和unlock当中的代码，导致锁失效

**解决方式：** 唤醒的线程需要重新获取锁

## 三、一个不可重入锁

```
package com.suncy.article.article6;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.LinkedBlockingQueue;

public class MyNotReentrantLock {
    private volatile int state = 0;
    private LinkedBlockingQueue<Thread> linkedBlockingQueue = new LinkedBlockingQueue<Thread>(3333);

    //unsafe包 和 state字段的偏移量
    private static final Unsafe unsafe;
    private static final long stateOffset;

    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
            stateOffset = unsafe.objectFieldOffset
                    (MyNotReentrantLock.class.getDeclaredField("state"));

        } catch (Exception ex) {
            throw new Error(ex);
        }
    }

    public void lock() {
        while (!unsafe.compareAndSwapInt(this, stateOffset, 0, 1)) {
            //失败，线程加入队列，等待精准唤醒
            //System.out.println("线程入队");
            linkedBlockingQueue.add(Thread.currentThread());
            unsafe.park(false, 0);
            linkedBlockingQueue.remove(Thread.currentThread());
        }
    }

    public void unlock() {
        //成功，表示解锁
        if (unsafe.compareAndSwapInt(this, stateOffset, 1, 0)) {
            //唤醒队列中的线程 不需要删除 只需要查看就行 因为被唤醒之后，会自动移除队列数据
            Thread th = linkedBlockingQueue.peek();
            if (th != null) {
                unsafe.unpark(th);
            }
        }
    }

    public static void main(String[] args) {
        MyNotReentrantLock myNotReentrantLock = new MyNotReentrantLock();
        for (int i = 0; i < 100; i++) {
            int finalI = i;
            new Thread(() -> {
                myNotReentrantLock.lock();
                System.out.println("==========" + finalI);
                System.out.println("加锁成功" + finalI);
                System.out.println("执行业务" + finalI);
                System.out.println("解锁成功" + finalI);
                System.out.println("==========" + finalI);
                myNotReentrantLock.unlock();
            }).start();
        }
    }
}
```

**执行结果：**&#x20;

![执行结果](<../.gitbook/assets/image (10).png>)

**结果分析：**&#x20;

1、在lock方法中，唤醒的线程重新去抢锁，这个时候保证了在unlock之后唤醒的线程和重新启动的线程重新抢锁，满足了要求。

&#x20;2、但是这个锁是不可重入的，假如我们同时执行两遍lock方法，就会导致线程死锁，修改方式为判断是否是当前线程，并记录数量。

## 四、一个可重入的锁

```
package com.suncy.article.article6;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.atomic.AtomicReference;

public class MyReentrantLock {

    private volatile int state = 0;
    AtomicReference<Thread> owner = new AtomicReference<>();
    private LinkedBlockingQueue<Thread> linkedBlockingQueue = new LinkedBlockingQueue<>();

    //unsafe包 和 state字段的偏移量
    private static final Unsafe unsafe;
    private static final long stateOffset;

    static {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(null);
            stateOffset = unsafe.objectFieldOffset
                    (MyNotReentrantLock.class.getDeclaredField("state"));
        } catch (Exception ex) {
            throw new Error(ex);
        }
    }

    public boolean tryLock() {
        int c = state;

        //如果count不等于0，说明存在锁
        if (c != 0) {
            //判断是不是当前线程持有锁
            if (owner.get() == Thread.currentThread()) {
                state = c + 1;
                //加锁成功
                return true;
            } else {
                //加锁失败
                return false;
            }
        } else {
            //count等于0，说明不存在锁
            if (unsafe.compareAndSwapInt(this, stateOffset, c, c + 1)) {
                //抢锁成功，设置owner为当前线程的引用
                owner.set(Thread.currentThread());
                return true;
            } else {
                //加锁失败
                return false;
            }
        }
    }

    public void lock() {
        //尝试抢锁
        while (!tryLock()) {
            //如果失败，放入到队列中
            linkedBlockingQueue.offer(Thread.currentThread());
            //并挂起
            unsafe.park(false, 0);
            //唤醒之后，则从队列中移除，并重新抢锁
            linkedBlockingQueue.remove(Thread.currentThread());
        }
    }

    public void unlock() {
        //解锁成功
        if (tryUnlock()) {
            //唤醒队列中的线程
            Thread th = linkedBlockingQueue.peek();
            if (th != null) {
                unsafe.unpark(th);
            }
        }
    }

    public boolean tryUnlock() {
        if (owner.get() != Thread.currentThread()) {
            throw new IllegalMonitorStateException();
        } else {
            //如果当前线程占有锁，则将计数减1
            state--;
            //判断count值是否为0,0的话，设置owner为null
            if (state == 0) {
                owner.compareAndSet(Thread.currentThread(), null);
                return true;
            } else {
                return false;
            }
        }
    }

    public static void main(String[] args) {
        MyReentrantLock myReentrantLock = new MyReentrantLock();
        for (int i = 0; i < 100; i++) {
            int finalI = i;
            new Thread(() -> {
                myReentrantLock.lock();
                myReentrantLock.lock();
                System.out.println("==========" + finalI);
                System.out.println("加锁成功" + finalI);
                System.out.println("执行业务" + finalI);
                System.out.println("解锁成功" + finalI);
                System.out.println("==========" + finalI);
                myReentrantLock.unlock();
                myReentrantLock.unlock();
            }).start();
        }
    }
}
```

**执行结果：**&#x20;

![执行结果](<../.gitbook/assets/image (9).png>)

**结果分析：** 结果正常，并且执行两遍lock方法也能够运行。
