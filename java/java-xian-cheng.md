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
Thread.sleep(1000);                     //让当前线程休眠一段时间（不释放锁）
Thread.yield();                         //让出CPU执行权，回到就绪状态（不一定生效）
thread.join();                          //等待另一个线程执行完毕再继续
thread.join(1000);                      //最多等待毫秒数
thread.join(1000, 100);                 //等待的毫秒数 + 微秒数
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

### 让出CPU执行权

```
Runnable task = () -> {
    for (int i = 0; i < 5; i++) {
        System.out.println(Thread.currentThread().getName() + " - " + i);
        Thread.yield(); // 提示CPU可以切换线程（不一定生效）
    }
};

Thread t1 = new Thread(task, "A");
Thread t2 = new Thread(task, "B");

t1.start();
t2.start();
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

### 主线程等待子线程执行完毕

```
Thread t = new Thread(() -> {
    try {
        System.out.println("子线程开始");
        Thread.sleep(2000);
        System.out.println("子线程结束");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

t.start();
System.out.println("主线程等待子线程执行完...");
t.join(); // 一直等到子线程结束
System.out.println("主线程继续执行");
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

### 向线程发出中断信号

```
Thread t = new Thread(() -> {
    while (true) {
        if (Thread.currentThread().isInterrupted()) {
            System.out.println("检测到中断信号，退出线程");
            break;
        }
        System.out.println("线程运行中...");
        try {
            Thread.sleep(500); // 如果在sleep中被中断，会抛出异常
        } catch (InterruptedException e) {
            System.out.println("sleep中被中断！");
            Thread.currentThread().interrupt(); // 重新设置中断状态
        }
    }
});

t.start();
Thread.sleep(1500); // 主线程等待1.5秒
t.interrupt(); // 发送中断信号
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure></div>

Thread.currentThread().interrupt() 只是设置一个中断标志，并不会强制停止线程；

在线程处于阻塞状态（如 sleep、wait）时，会抛出 InterruptedException

### 检查并清除中断状态

```
Thread.currentThread().interrupt(); // 设置中断标志为true
System.out.println("第一次检查：" + Thread.interrupted()); // true（并清除标志）
System.out.println("第二次检查：" + Thread.interrupted()); // false（因为已清除）
```

打印结果

<div align="left"><figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

### thread.isInterrupted() 和 Thread.interrupted()的区别

thread.isInterrupted()：询问某个线程是否被中断，只检查，不清除中断状态

Thread.interrupted()：静态方法，询问当前线程是否被中断，并重置中断状态

### 守护线程

调用线程的setDaemon方法，设置为ture代表守护线程。

守护线程一直在后台运行。当主线程结束，JVM 退出，守护线程被自动终止。

```
// 创建守护线程
Thread daemonThread = new Thread(() -> {
    while (true) {
        System.out.println("守护线程运行中...");
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            break;
        }
    }
});

// 设置为守护线程（必须在 start() 之前）
daemonThread.setDaemon(true);
daemonThread.start();

// 主线程（用户线程）
for (int i = 0; i < 3; i++) {
    System.out.println("主线程工作：" + i);
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

System.out.println("主线程结束，JVM退出，守护线程被终止。");
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure></div>
