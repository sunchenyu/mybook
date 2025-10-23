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





