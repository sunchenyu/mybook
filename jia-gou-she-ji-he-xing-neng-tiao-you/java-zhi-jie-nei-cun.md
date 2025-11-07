# Java直接内存

## 概述

整体可以分成 JVM 管理的内存 和 JVM 直接内存

JVM管理的内存包括

```
堆
方法区  运行时常量 和 元空间
虚拟机栈
本地方法栈
寄存器
```

直接内存包括

```
ByteBuffer.allocateDirect()分配的直接内存
Unsafe.allocateMemory()分配的直接内存
```

## JVM直接内存的两种方式

### 1. ByteBuffer.allocateDirect()

ByteBuffer.allocateDirect() 分配的直接内存，可以在JVM启动的时候配置大小，使用-XX:MaxDirectMemorySize参数

特点

* DirectByteBuffer 对象被 GC 回收时
* JVM 会调用 Cleaner 去 free native memory
* 这是一种 间接管理（不是实时的）

问题：

* GC 什么时候回收 DirectByteBuffer 不确定
* 直接内存释放时间不可控
* 容易内存泄漏（只要 DirectByteBuffer 对象有引用）

可以配置大小使用 -XX:MaxDirectMemorySize 参数超出限制之后 会报错

```
java.lang.OutOfMemoryError: Direct buffer memory
```

**DirectByteBuffer 的直接内存由谁释放？**

DirectByteBuffer 使用了一个 Cleaner（隐式清理器）：

```
Cleaner(cleanupAction)
```

当 DirectByteBuffer 对象被 GC 掉时，Cleaner 才会执行：

```
free(nativeAddress)
```

也就是：

```
Unsafe.freeMemory(address)
```

但注意：

* 不是立即执行
* &#x20;不是确定时机
* GC 不一定及时回收 DirectByteBuffer 对象
* 只要 DirectByteBuffer 有任何引用，直接内存就永远不会释放

源码如下

```java
 DirectByteBuffer(int cap) {                   // package-private
        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = UNSAFE.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        UNSAFE.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
        att = null;
    }
```

Deallocator的run方法中含有UNSAFE.freeMemory(address);

```
 private static class Deallocator
        implements Runnable
    {
        private long address;
        private long size;
        private int capacity;

        private Deallocator(long address, long size, int capacity) {
            assert (address != 0);
            this.address = address;
            this.size = size;
            this.capacity = capacity;
        }

        public void run() {
            if (address == 0) {
                // Paranoia
                return;
            }
            UNSAFE.freeMemory(address);
            address = 0;
            Bits.unreserveMemory(size, capacity);
        }
    }
```

### 2. Unsafe.allocateMemory()

Unsafe.allocateMemory() 是直接向是OS申请原始内存块，等价于C的 malloc方法

```
long addr = Unsafe.allocateMemory(size); 
Unsafe.freeMemory(addr);
```

特点：

* GC 完全不参与
* JVM 不知道你分配了多少直接内存
* 内存泄漏完全由你代码负责

这是最危险但性能最高的方式。

说明：JVM支持配置 DirectByteBuffer 分配的大小 ，但是它无法限制 Unsafe.allocateMemory() 也无法限制 OHC、Netty 自行分配的 native 内存。因此仍然存在“超出限制导致 native OOM”的风险。

## 框架

堆内缓存框架

Guava Cache



Caffeine



堆外缓存框架

OHC（Off-Heap Cache）



Ehcache3



## 内存泄漏产生的原因

DirectByteBuffer 泄漏方式

* DirectByteBuffer对象仍然被引用 → GC 不会清理 → Cleaner 不会触发 → 直接内存无法释放
* 大量短生命周期 DirectByteBuffer 使用频繁 → GC 无法跟上 → Direct Memory 爆掉

Unsafe.allocateMemory 泄漏方式

* 不调用 freeMemory → 永不释放



#### 直接内存回收

JVM 进程停止后，所有直接内存都会被操作系统完全回收。

因为只要 JVM 进程退出，进程的整个虚拟内存空间（包括 heap、off-heap、direct buffer、Unsafe 分配、mmap，都属于该进程）会被 OS 内核统一释放。用户态进程的虚拟地址空间，其生命周期与进程完全绑定。



