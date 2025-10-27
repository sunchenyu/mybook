# Java异步编程

## 概述

允许程序在等待某个任务完成的同时，不阻塞主线程（或主流程），而是去执行其他任务。

## 学习样例

### 同步任务

```java
//假设这个任务需要2秒执行
try {
    TimeUnit.SECONDS.sleep(2);
} catch (InterruptedException e) {
    throw new RuntimeException(e);
}
String s = "Hello";
System.out.println(Thread.currentThread().getName() + " 执行任务");

System.out.println("main线程等待结果...");
System.out.println("结果 = " + s);
```

执行结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure></div>

我们的主线程只会一直等待任务处理完毕才能继续进行下一步，所以打印的main执行任务。

### 主线程等待异步结果

```java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println(Thread.currentThread().getName() + " 执行任务");
        return "Hello";
    });

System.out.println("main线程等待结果...");
String result = future.get();
System.out.println("结果 = " + result);
```

异步执行任务，然后通过get方法获取到异步任务的执行结果，get方法会阻塞当前线程，执行结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure></div>

我们的主线程先打印的main线程等待结果，这一步就可以去执行别的任务了。

### 指定异步任务执行的线程池

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    System.out.println(Thread.currentThread().getName() + " 执行任务");
    return "Data from task";
}, executor);

System.out.println("main 等待结果...");
System.out.println("结果 = " + future.get());

executor.shutdown();
```

执行结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure></div>

这个时候我们发现这个任务是通过我们构建的executor 线程池去跑的

### 异步任务配置和执行

```java
CompletableFuture<Void> future = CompletableFuture
            .supplyAsync(() -> {
//                try {
//                    TimeUnit.SECONDS.sleep(2);
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                }
                System.out.println(Thread.currentThread().getName() + " supplyAsync 执行");
                return "Hello";
            })
            .thenApply(s -> {
                System.out.println(Thread.currentThread().getName() + " thenApply 执行");
                return s + " World";
            })
            .thenAccept(s -> {
                System.out.println(Thread.currentThread().getName() + " thenAccept 执行");
                System.out.println("结果：" + s);
            })
            .thenRun(() -> {
                System.out.println(Thread.currentThread().getName() + " thenRun 执行");
            });

        future.get(); // 阻塞等待完成
```

执行结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure></div>

打开注释然后执行，获取到的执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure></div>

不同的点在于thenApply、thenAccept、thenRun对应的回调方法的执行位置。

#### 说明

如果前一个任务「已经完成」，那么注册回调的线程（可能是 main）立刻执行；避免额外线程切换的开销。

如果前一个任务「还没完成」，那回调就会被保存起来，等异步线程执行完后触发；

谁触发 complete()（也就是任务完成），谁就执行下一个阶段。

### 强制异步任务通过线程池执行

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName() + " supplyAsync 执行");
        return "Hello";
    })
    .thenApplyAsync(s -> {
        System.out.println(Thread.currentThread().getName() + " thenApplyAsync 执行");
        return s + " Async World";
    });

System.out.println("结果：" + future.get());
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure></div>

我们发现都是提交到线程池去执行的，不会存在上一个案例里面，有主线程执行的情况。

#### 说明

那为什么要有 thenApplyAsync？

因为有时候你希望一定在异步线程中执行，比如：

* 耗时操作（I/O、网络、文件读写、数据库查询）
* 不想阻塞 main 或业务线程

### 异常处理

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName() + " 执行任务");
        if (true) throw new RuntimeException("出错啦");
        return "OK";
    })
    .exceptionally(ex -> {
        System.out.println("捕获异常: " + ex.getMessage());
        return "默认值";
    });

System.out.println("结果：" + future.get());
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure></div>

### thenCompose方法

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName() + " 获取用户ID");
        return "User123";
    })
    .thenCompose(userId -> CompletableFuture.supplyAsync(() -> {
        System.out.println(Thread.currentThread().getName() + " 根据 " + userId + " 查询用户信息");
        return "用户信息：" + userId;
    }));

System.out.println("结果：" + future.get());
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure></div>

**为什么需要thenCompose方法？**

如果想在得到第一步的异步结果之后，还想要在构建一个CompletableFuture任务，并且想要返回回去，那这个情况下，用thenApply就只能返回类似于这样的返回值

```
CompletableFuture<CompletableFuture<String>>
```

thenCompose的返回值就是一个CompletableFuture\<String>，并且这个方法会将这个新的异步任务返回给上层进行编排。

### 手动构建

```java
// 手动创建一个 CompletableFuture（此时是空的，没有任务）
CompletableFuture<String> future = new CompletableFuture<>();

// 注册回调 —— 注意，这个阶段不会立即执行，因为还没有 complete()
future.thenApply(result -> {
    System.out.println(Thread.currentThread().getName() + " 执行 thenApply，收到结果：" + result);
    return result + " World";
}).thenAccept(result -> {
    System.out.println(Thread.currentThread().getName() + " 执行 thenAccept，最终结果：" + result);
});

// 模拟异步线程在后台计算结果
new Thread(() -> {
    try {
        System.out.println(Thread.currentThread().getName() + " 开始计算任务...");
        Thread.sleep(1000); // 模拟耗时
        // 任务完成后，手动触发 complete
        future.complete("Hello");
        System.out.println(Thread.currentThread().getName() + " 调用了 complete()");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}, "Worker-Thread").start();

// 阻塞等待所有完成
Thread.sleep(2000);
System.out.println(Thread.currentThread().getName() + " 主线程结束");
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure></div>

**为什么需要手动构建？**

如果我们在Netty中开发程序，我们很熟悉的一个地方就是

```
channel.writeAndFlush(msg); 
```

这个是不会获取到响应数据的，必须要在handle当中才能读取到数据，这个时候，我们就没办法使用supplyAsync等方法，这个时候就可以使用HashMap，将对应的消息id和CompletableFuture存储进去，以便在收到相应的时候可以手动匹配到，进行手动完成操作。

## 总结

thenAccept：只会在正常完成时执行。异常完成会触发 exceptionally / handle / whenComplete 的异常分支。

exceptionally：只在异常完成时调用。它不会改变原 future，但会返回一个新的、被“恢复”的 future。要想拿到恢复后的值，必须用返回的新 future。

handle：无论成功或异常，都会被调用。回调函数可返回任意类型的新结果。

whenComplete：成功或失败都会执行，但不会改变结果。

orTimeout：会让目标 future 在超时后异常完成（并通常是同一个 future）

