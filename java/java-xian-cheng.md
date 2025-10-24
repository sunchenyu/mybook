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

```java
thread.getName();                       //获取线程名
thread.getId();                         //获取线程的唯一 ID
thread.getPriority();                   //获取线程优先级
thread.getState();                      //获取线程当前状态
thread.getThreadGroup();                //获取线程所属的线程组
thread.isAlive();                       //判断线程是否还在运行
thread.isDaemon();                      //判断或设置为守护线程
thread.isInterrupted();                 //检查线程是否被中断
```

### 线程控制

```java
Thread.sleep(1000);               //让当前线程休眠一段时间（不释放锁）
Thread.yield();                         //让出CPU执行权，回到就绪状态（不一定生效）
thread.join();                          //等待另一个线程执行完毕再继续
thread.join(1000);                //最多等待毫秒数
thread.join(1000, 100);     //等待的毫秒数 + 微秒数
thread.interrupt();                     //向线程发出中断信号（非强制中断）
Thread.interrupted();                   //检查并清除中断状态
```

### 优先级和守护线程

```java
thread.setName("MyThread");               //设置线程名字
thread.setPriority(Thread.MAX_PRIORITY);  //设置线程优先级（1~10，默认 5）
thread.setDaemon(true);                   //设置为守护线程（必须在 start() 前调用）
```

## 示例

### 查看线程状态

```java
Thread thread = new Thread(() -> {
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {}
});
System.out.println("未调用start方法：" + thread.getState()); // NEW
thread.start();
System.out.println("调用sleep方法前：" + thread.getState()); // RUNNABLE
Thread.sleep(10);
System.out.println("调用join方法前：" + System.currentTimeMillis() + " " +  thread.getState()); // TIMED_WAITING
thread.join();
System.out.println("线程处理完毕：" + System.currentTimeMillis() + " " +thread.getState()); // TERMINATED
```

打印结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure></div>



