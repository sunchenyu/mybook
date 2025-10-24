# Java线程

## 什么是进程

进程是对运行时程序的封装，是系统进行资源调度和分配的基本单位。

## 什么是线程

线程是CPU调度和分派的基本单位。

## java创建线程的方式

从根源上来说，只有一种方式去创建线程，那就是实例化一个Thread，然后调用start方法

```
new Thread(...).start();
```

如果是java具体的方式，那就可以区分以下几种

1. 继承Thread类
2. Runnable构建Thread线程对象
3. FutureTask(Callable)构建Thread线程对象
4. 线程池提交Runnable任务
5. CompletableFuture构建异

## 线程的状态有哪几种

```
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

新建状态 NEW ：线程对象已创建，但还没启动（没调用 start()）

可运行状态 RUNNABLE ：线程正在运行或准备运行（取决于系统调度）

阻塞状态 BLOCKED ：线程在等待获取锁（synchronized）

无限等待状态 WAITING ：调用 wait() / join() / LockSupport.park() 等，等待被唤醒

限时等待状态 TIMED\_WAITING ：有时间限制的等待，例如 sleep(1000)、wait(timeout)

终止状态 TERMINATED： 线程执行完毕或抛异常结束

## Thread的常用方法

### 线程信息获取



### 线程控制



### 优先级和守护线程



### 线程结束和销毁







