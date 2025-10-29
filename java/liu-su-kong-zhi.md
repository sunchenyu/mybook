# 流速控制

## 概述

在计算机网络和系统中，“流速控制”是保证系统稳定性、保护资源不被过载的重要手段。根据实现方式不同，常用算法主要包括：令牌桶算法、漏桶算法、固定窗口计数器、滑动窗口计数器等等。

## 令牌桶

### 核心思想

系统以固定速率产生“令牌”，放入桶中，每个请求必须消耗令牌才能通过，允许短时间内突发流量，但长期保持平均速率。

### 核心流程描述

1. 计算时间差
2. 根据令牌生成速率计算新增令牌数
3. 更新令牌桶数量，确保不超过桶上限
4. 更新最后生成令牌的时间
5. 尝试获取令牌（令牌减少）

### 实现

```java
package com.ultra.script.speed;

import java.util.concurrent.locks.ReentrantLock;

public class TokenRateLimit {
    private final int capacity;//令牌桶的容量
    private final int refillRate;//令牌桶的填充速度(每秒填充多少令牌)
    private int tokens;//当前令牌数量
    private long lastTime;//上次填充令牌的时间
    private final ReentrantLock reentrantLock = new ReentrantLock();//锁

    public TokenRateLimit(int capacity, int refillRate) {
        this.capacity = capacity;
        this.refillRate = refillRate;
        this.tokens = capacity;//初始化令牌数量为容量
        lastTime = System.nanoTime();
    }

    public boolean tryAcquire() {
        reentrantLock.lock();
        try {
            refill();
            if (tokens > 0) {
                tokens--;
                return true;
            }
            return false;
        } finally {
            reentrantLock.unlock();
        }
    }

    private void refill() {
        long now = System.nanoTime();
        double elapsedSeconds = (now - lastTime) / 1_000_000_000.0;
        if (elapsedSeconds > 0) {
            int add = (int) (elapsedSeconds * refillRate);
            if (add > 0) {
                tokens = Math.min(capacity, tokens + add);
                lastTime = now;
            }
        }
    }

    public static void main(String[] args) {
        TokenRateLimit limiter = new TokenRateLimit(5, 5); // 容量5，每秒5个令牌

        Runnable task = () -> {
            for (int i = 0; i < 20; i++) {
                if (limiter.tryAcquire()) {
                    System.out.println(Thread.currentThread().getName() + " 执行任务");
                } else {
                    System.out.println(Thread.currentThread().getName() + " 被限流");
                }
                try {
                    Thread.sleep(100);
                } catch (InterruptedException ignored) {
                }
            }
        };

        for (int i = 0; i < 3; i++) {
            new Thread(task, "线程-" + i).start();
        }
    }
}
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

## 漏桶

### 核心思想

请求暂存于固定容量的漏桶中，系统以恒定速率处理请求；当桶满时触发溢出拒绝策略，强制限制访问速度。

### 核心流程描述

1. 计算时间差
2. 根据漏水速率计算漏掉的请求数
3. 更新桶中剩余水量，确保水量不会为负数
4. 更新上次漏水时间
5. 处理请求（增加水量是否溢出）

### 实现

```java
package com.ultra.script.speed;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.ReentrantLock;

public class LeakyRateLimit {
    private final int capacity;// 桶的容量
    private double water;// 当前水量
    private long lastLeakTime;// 上次漏水时间
    private final double leakRate;// 漏水速率
    private final ReentrantLock lock = new ReentrantLock(); // 锁

    public LeakyRateLimit(int capacity, double leakRate) {
        this.capacity = capacity;
        this.leakRate = leakRate;
        this.water = 0;// 初始水量为0
        this.lastLeakTime = System.nanoTime();
    }

    public boolean tryAcquire() {
        lock.lock();
        try {
            leak();
            if (water < capacity) {
                water++;
                return true;
            } else {
                return false; // 桶满，拒绝请求
            }
        } finally {
            lock.unlock();
        }
    }

    private void leak() {
        long now = System.nanoTime();
        double elapsedSeconds = (now - lastLeakTime) / 1_000_000_000.0;
        double leaked = elapsedSeconds * leakRate;// 计算漏水量
        if (leaked > 0) {
            water = Math.max(0, water - leaked); // 更新水量
            lastLeakTime = now;
        }
    }

    public static void main(String[] args) {
        LeakyRateLimit limiter = new LeakyRateLimit(5, 5);// 容量为5，漏水速率为5/s

        AtomicInteger atomicInteger = new AtomicInteger(0);
        Runnable task = () -> {
            for (int i = 0; i < 10; i++) {
                int count = atomicInteger.incrementAndGet();
                if (limiter.tryAcquire()) {
                    System.out.println(Thread.currentThread().getName() + " 执行任务;总请求数：" + count);
                } else {
                    System.out.println(Thread.currentThread().getName() + " 被限流;总请求数" + count);
                }
                try {
                    Thread.sleep(100);
                } catch (InterruptedException ignored) {
                }
            }
        };

        for (int i = 0; i < 3; i++) {
            new Thread(task, "线程-" + i).start();
        }
    }
}
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

## 固定窗口计数器

### 核心思想

将时间划分为固定窗口，在窗口内统计请求数量，超过阈值则拒绝请求。

### 核心流程描述

1. 获取当前时间
2. 判断窗口是否到期
3. 判断请求数是否超限

### 实现

```java
package com.ultra.script.speed;

import java.util.concurrent.locks.ReentrantLock;

public class FixedWindowRateLimit {
    private final long windowNanos;//窗口大小
    private final int maxRequests;//最大请求数
    private volatile long windowStart;//窗口开始时间
    private int count;//当前窗口内请求数
    private ReentrantLock reentrantLock = new ReentrantLock();//锁

    public FixedWindowRateLimit(long windowNanos, int maxRequests) {
        this.windowNanos = windowNanos;
        this.maxRequests = maxRequests;
        this.windowStart = System.nanoTime();
        this.count = 0;
    }

    public boolean tryAcquire() {
        reentrantLock.lock();
        try {
            long now = System.nanoTime();
            if (now - windowStart >= windowNanos) {
                windowStart = now;
                count = 0;
            }
            if (count < maxRequests) {
                count++;
                return true;
            }
            return false;
        } finally {
            reentrantLock.unlock();
        }
    }

    public static void main(String[] args) {
        FixedWindowRateLimit limiter = new FixedWindowRateLimit(1_000_000_000, 5); // 窗口为1秒，每秒5个请求

        Runnable task = () -> {
            for (int i = 0; i < 10; i++) {
                if (limiter.tryAcquire()) {
                    System.out.println(Thread.currentThread().getName() + " 执行任务");
                } else {
                    System.out.println(Thread.currentThread().getName() + " 被限流");
                }
                try {
                    Thread.sleep(100);
                } catch (InterruptedException ignored) {
                }
            }
        };

        for (int i = 0; i < 3; i++) {
            new Thread(task, "线程-" + i).start();
        }
    }
}
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure></div>

## 滑动窗口计数器

### 核心思想

时间窗口连续滑动，而不是固定窗口，请求数按时间比例分配到相邻两个窗口中，平滑了固定窗口的突发问题。

### 核心流程描述

1. 获取当前时间
2. 确定当前桶
3. 刷新桶（清理过期数据）
4. 统计窗口内总请求数
5. 判断限流并更新计数

### 实现

```java
package com.ultra.script.speed;

import java.util.concurrent.atomic.AtomicIntegerArray;
import java.util.concurrent.locks.ReentrantLock;
public class SlidingWindowRateLimit {
    private final long windowTime;      // 总窗口时长
    private final int bucketCount;        // 桶数量
    private final int maxRequests;        // 窗口允许最大请求数
    private final long bucketMillis;      // 每个桶对应时间长度
    private final AtomicIntegerArray buckets; // 桶计数
    private final long[] bucketTimestamps;    // 桶时间戳
    private final ReentrantLock lock = new ReentrantLock(); // 线程安全锁

    public SlidingWindowRateLimit(long windowTime, int bucketCount, int maxRequests) {
        this.windowTime = windowTime;
        this.bucketCount = bucketCount;
        this.maxRequests = maxRequests;
        this.bucketMillis = windowTime / bucketCount;
        this.buckets = new AtomicIntegerArray(bucketCount);
        this.bucketTimestamps = new long[bucketCount];
    }

    private int currentIndex() {
        long now = System.nanoTime();
        return (int) ((now / bucketMillis) % bucketCount);
    }

    private void refreshBucketIfStale(int idx) {
        long now = System.nanoTime();
        long bucketStart = now - (now % bucketMillis);

        if (bucketTimestamps[idx] != bucketStart) {
            buckets.set(idx, 0);
            bucketTimestamps[idx] = bucketStart;
        }
    }

    public boolean tryAcquire() {
        lock.lock();
        try {
            int idx = currentIndex(); // 当前桶索引
            refreshBucketIfStale(idx); // 刷新当前桶时间戳

            // 统计整个窗口总请求数
            int total = 0;
            long windowStart = System.nanoTime() - windowTime;
            for (int i = 0; i < bucketCount; i++) {
                if (bucketTimestamps[i] >= windowStart) {
                    total += buckets.get(i); // 统计窗口内请求数
                }
            }

            if (total < maxRequests) { // 请求数小于最大限制
                buckets.incrementAndGet(idx); // 计数器加1
                return true;
            } else {
                return false;
            }
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        SlidingWindowRateLimit limiter = new SlidingWindowRateLimit(1_000_000_000, 5, 5); // 1秒窗口，5个桶，最大5请求

        Runnable task = () -> {
            for (int i = 0; i < 10; i++) {
                if (limiter.tryAcquire()) {
                    System.out.println(Thread.currentThread().getName() + " 执行任务");
                } else {
                    System.out.println(Thread.currentThread().getName() + " 被限流");
                }
                try {
                    Thread.sleep(200);
                } catch (InterruptedException ignored) {}
            }
        };

        for (int i = 0; i < 3; i++) {
            new Thread(task, "线程-" + i).start();
        }
    }
}
```

执行结果

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>
