# java线程池

## 一、基本使用方法

### （1）使用示例

```
package com.suncy.article.article9;

import org.apache.commons.lang3.concurrent.BasicThreadFactory;

import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class BasicUse {
    private static ThreadFactory basicThreadFactory = new BasicThreadFactory.Builder()
            .namingPattern("basicThreadFactory-").build();

    public static void main(String[] args) {
        //参数分别为 核心线程数 最大线程数 线程保活时间 线程保活时间单位 任务队列 线程工厂 拒绝策略
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                1,
                20,
                60L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(2),
                basicThreadFactory,
                new ThreadPoolExecutor.AbortPolicy()
        );

        //提交Runnable任务
        threadPoolExecutor.submit(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread() + "执行任务");
            }
        });
        //执行完关闭线程池
        threadPoolExecutor.shutdown();
    }
}
```

### （2）说明

**线程池执行的任务是什么：**

Runnable实例

**线程池主要目的：**

复用Thread

**ThreadPoolExecutor的参数：**

1、int corePoolSize 核心线程数&#x20;

2、int maximumPoolSize 最大线程数&#x20;

3、long keepAliveTime 线程保活时间&#x20;

4、TimeUnit unit 线程保活时间单位

&#x20;5、BlockingQueue workQueue 线程任务队列&#x20;

6、ThreadFactory threadFactory 线程构建工厂&#x20;

7、RejectedExecutionHandler handler 拒绝策略

**线程池使用流程**

**1、构建Runnable任务实例** Runnable tast = new Runnable(){};&#x20;

**2、提交Runnable任务实例task**&#x20;

（1）如果没有达到corePoolSize核心线程数，则使用threadFactory线程工厂创建核心线程（Thread），并使用这个线程执行Runnable任务。&#x20;

（2）如果达到了corePoolSize核心线程数，而workQueue没有满，则将这个Runnable任务实例直接扔到workQueue队列中，线程需要去从队列抢任务执行。&#x20;

（3）如果达到了corePoolSize核心线程数，而workQueue满了，但是没有达到maximumPoolSize最大线程数，那么就使用threadFactory线程工厂创建线程（Thread），使用这个新的线程执行Runnable实例任务。 （4）如果workQueue满了，也达到了maximumPoolSize最大线程数，那么根据handler拒绝策略，进行不同的拒绝操作。

&#x20;**3、核心线程和最大线程**

&#x20;如果任务少了，最大线程有20，核心线程有10，那么就会释放其中10个线程的资源，减少cpu的消耗，这个释放线程资源的条件为keepAliveTime和unit参数（代表线程空闲时间）

## 二、Executors提供的四种线程池

### （1）Executors构建的线程池

```
Executors.newSingleThreadExecutor();  
Executors.newCachedThreadPool();
Executors.newFixedThreadPool(4);
Executors.newScheduledThreadPool(4);
```

### （2）说明

**上面四种线程池分别是：**&#x5355;线程化的线程池、可缓存线程池、定长线程池、可定时线程池。&#x20;

**上面四种方法最终的构造方法：**&#x54;hreadPoolExecutor，只是根据不同的策略，设置了不同的参数。&#x20;

**newSingleThreadExecutor参数：**&#x63;orePoolSize为1、maximumPoolSize为1、保活时间为0、workQueue为LinkedBlockingQueue。&#x20;

**newCachedThreadPool参数：**&#x63;orePoolSize为0、maximumPoolSize为Integer.MAX\_VALUE、保活时间60秒、workQueue为SynchronousQueue。

&#x20;**newFixedThreadPool参数：**&#x63;orePoolSize为参数4（上面传的参数是4）、maximumPoolSize为4、保活时间0、workQueue为LinkedBlockingQueue。&#x20;

**newScheduledThreadPool参数：**&#x63;orePoolSize为0、maximumPoolSize为Integer.MAX\_VALUE、保活时间0、workQueue为DelayedWorkQueue&#x20;

**总结：**&#x6240;以你想要什么线程池，完全可以使用ThreadPoolExecutor这个类，直接自定义参数即可。

## 三、拒绝策略

### （1）四种拒绝策略

**ThreadPoolExecutor.AbortPolicy：**&#x4E22;弃任务并抛出RejectedExecutionException异常。 **ThreadPoolExecutor.DiscardPolicy：**&#x4E22;弃新任务，但是不抛出异常。 **ThreadPoolExecutor.DiscardOldestPolicy：**&#x4E22;弃队列最前面的任务，提交新的任务到队列中 **ThreadPoolExecutor.CallerRunsPolicy：** 由调用线程（提交任务的线程）处理该任务

### （2）拒绝策略测试demo

**AbortPolicy**

```
package com.suncy.article.article9;

import lombok.SneakyThrows;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;

import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class AbortDemo {
    private static ThreadFactory basicThreadFactory = new BasicThreadFactory.Builder()
            .namingPattern("basicThreadFactory-").build();

    public static void main(String[] args) {
        //构建线程池 核心方法
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                1,
                1,
                60L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(2),
                basicThreadFactory,
                new ThreadPoolExecutor.AbortPolicy()

        );

        for (int i = 0; i < 5; i++) {
            int finalI = i;
            threadPoolExecutor.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println("进入线程执行,第" + finalI + "个");
                    try {
                        Thread.sleep(finalI * 100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("执行完毕,第" + finalI + "个");
                }
            });
        }

        threadPoolExecutor.shutdownNow();
    }
}
```

**结果：**&#x629B;出异常，后面的任务都不会提交成功。&#x20;

![AbortPolicy测试结果](<../.gitbook/assets/image (19).png>)

**DiscardPolicy**

```
package com.suncy.article.article9;

import lombok.SneakyThrows;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;

import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class RejectionStrategyDemo {
    private static ThreadFactory basicThreadFactory = new BasicThreadFactory.Builder()
            .namingPattern("basicThreadFactory-").build();

    public static void main(String[] args) throws InterruptedException {
        //构建线程池 核心方法
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                1,
                1,
                60L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(2),
                basicThreadFactory,
                new ThreadPoolExecutor.DiscardPolicy()
        );

        for (int i = 0; i < 5; i++) {
            int finalI = i;

            threadPoolExecutor.submit(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    System.out.println("进入线程执行,第" + finalI + "个");
                    Thread.sleep(finalI * 100);
                    System.out.println("执行完毕,第" + finalI + "个");
                }
            });
        }

        Thread.sleep(5000);
        threadPoolExecutor.shutdownNow();
    }
}
```

**结果：**&#x629B;弃了新的任务，没有任何提示&#x20;

![ DiscardPolicy测试结果](<../.gitbook/assets/image (6) (1).png>)

**DiscardOldestPolicy**

```
package com.suncy.article.article9;

import lombok.SneakyThrows;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;

import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class RejectionStrategyDemo {
    private static ThreadFactory basicThreadFactory = new BasicThreadFactory.Builder()
            .namingPattern("basicThreadFactory-").build();

    public static void main(String[] args) throws InterruptedException {
        //构建线程池 核心方法
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                1,
                1,
                60L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(2),
                basicThreadFactory,
                new ThreadPoolExecutor.DiscardOldestPolicy()
        );

        for (int i = 0; i < 5; i++) {
            int finalI = i;

            threadPoolExecutor.submit(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    System.out.println("进入线程执行,第" + finalI + "个");
                    Thread.sleep(finalI * 100);
                    System.out.println("执行完毕,第" + finalI + "个");
                }
            });
        }

        Thread.sleep(5000);
        threadPoolExecutor.shutdownNow();
    }
}
```

**结果：**&#x629B;弃了队列中前两个任务，没有任何提示&#x20;

![ DiscardOldestPolicy测试结果](<../.gitbook/assets/image (22).png>)

**CallerRunsPolicy**

```
package com.suncy.article.article9;

import lombok.SneakyThrows;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;

import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class RejectionStrategyDemo {
    private static ThreadFactory basicThreadFactory = new BasicThreadFactory.Builder()
            .namingPattern("basicThreadFactory-").build();

    public static void main(String[] args) throws InterruptedException {
        //构建线程池 核心方法
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                1,
                1,
                60L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<Runnable>(2),
                basicThreadFactory,
                new ThreadPoolExecutor.CallerRunsPolicy()
        );

        for (int i = 0; i < 5; i++) {
            int finalI = i;

            threadPoolExecutor.submit(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    System.out.println(Thread.currentThread() + "进入线程执行,第" + finalI + "个");
                    Thread.sleep(finalI * 100);
                    System.out.println(Thread.currentThread() + "执行完毕,第" + finalI + "个");
                }
            });
        }
        Thread.sleep(5000);
        threadPoolExecutor.shutdownNow();
    }
}
```

**结果：**&#x5B58;在main线程执行的任务&#x20;

![ CallerRunsPolicy测试结果](<../.gitbook/assets/image (35).png>)

### （3）拒绝策略总结

为了防止任务被抛弃，个人经常使用的策略是CallerRunsPolicy策略。 为了更容易预测线程池的行为，所以推荐直接使用ThreadPoolExecutor。
