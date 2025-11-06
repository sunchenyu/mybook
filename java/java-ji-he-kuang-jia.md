# Java集合框架

## 结构

```
Collection（接口）
 ├── List（接口）
 │     ├── ArrayList
 │     ├── LinkedList
 │     ├── Vector
 │     │      └── Stack
 │     ├── CopyOnWriteArrayList（并发）
 │     └── SynchronizedList（Collections.synchronizedList 包装）
 │
 ├── Set（接口）
 │     ├── HashSet
 │     │      └── LinkedHashSet
 │     ├── TreeSet
 │     ├── CopyOnWriteArraySet（并发）
 │     └── ConcurrentSkipListSet（并发、有序）
 │
 └── Queue（接口）
       ├── Deque（接口）
       │      ├── LinkedList
       │      ├── ArrayDeque
       │      ├── ConcurrentLinkedDeque（并发）
       │      └── LinkedBlockingDeque（阻塞）
       │
       ├── BlockingQueue（接口）
       │      ├── ArrayBlockingQueue
       │      ├── LinkedBlockingQueue
       │      ├── PriorityBlockingQueue
       │      ├── DelayQueue
       │      ├── SynchronousQueue
       │      └── LinkedTransferQueue
       │
       ├── ConcurrentLinkedQueue（并发）
       └── PriorityQueue

────────────────────────────────────────

Map（接口）
 ├── HashMap
 │      └── LinkedHashMap
 ├── TreeMap
 ├── IdentityHashMap
 ├── WeakHashMap（弱引用）
 │
 ├── ConcurrentMap（接口）
 │      ├── ConcurrentHashMap
 │      ├── ConcurrentSkipListMap（有序）
 │      └── Hashtable（早期同步，性能差）
 │
 └── 相关（扩展/第三方）
        ├── ConcurrentReferenceHashMap（Spring）
        └── Properties（继承 Hashtable）

```

## 集合

### List接口

有序、元素可重复、可以通过索引访问

#### **ArrayList**

底层结构：动态数组

* 优点：
* 随机访问 O(1)
* 内存连续，CPU 缓存友好
* 适合读多、遍历多

缺点：

* 扩容需要复制数组
* 头部插入/删除成本高 O(n)

#### **LinkedList**

底层结构：双向链表

优点：

* 插入删除快（O(1)）
* 可以作为 Queue / Deque 使用

缺点：

* 随机访问慢（O(n)）
* 额外指针开销

#### **Vector**

* 早期线程安全动态数组
* 全方法 synchronized，性能差
* 已过时，不推荐使用

#### **Stack**

* 基于 Vector 的 LIFO 栈
* 推荐使用 Deque 替代

#### **CopyOnWriteArrayList**

底层：写时复制 + 不可变读快照适合：

* 读远多于写
* 配置、白名单、缓存稳定列表

特点：

* 写操作时复制整个数组，非常安全但昂贵
* 读完全无锁，性能极高

#### **SynchronizedList**

通过 Collections.synchronizedList(list) 获得的线程安全包装类。特点：

* 粗粒度锁，不如并发集合效率高
* 但比 Vector 更合理

### Set接口

不允许重复元素、无索引

#### HashSet

底层：HashMap（key=元素，value=常量）特点：

* 无序
* 插入/查询 O(1)
* 适合判重

#### LinkedHashSet

底层：LinkedHashMap特点：

* 保留插入顺序
* 访问顺序可设置 LRU（用于缓存）

#### TreeSet

底层：红黑树（有序结构）特点：

* 元素按“排序规则”自动排序
* 查询、添加 O(log n)
* 适合范围查询 / 有序数据

#### CopyOnWriteArraySet

底层：CopyOnWriteArrayList特点：

* 读无锁，写复制
* 读多写少场景

#### ConcurrentSkipListSet

底层：跳表（SkipList）特点：

* 并发有序集合
* 性能接近红黑树 + 并发友好

典型适用：

* 有序排行榜
* 并发区间查询

### Queue接口

#### Deque接口

**ArrayDeque**

底层：循环数组特点：

* 非阻塞队列
* 比 LinkedList 更快
* 用作栈、双端队列的最佳选择

**ConcurrentLinkedDeque**

特点： 无锁双端队列（CAS），高并发下性能优秀。

* 适合多生产/多消费
* 非阻塞
* 空间占用比 ArrayDeque 大

**LinkedBlockingDeque**

特点： 带容量的阻塞双端队列，支持 take/put。

* 可用于生产者-消费者
* 可限制容量，防止 OOM
* 内部两把锁（head/tail），并发能力好

#### BlockingQueue

**ArrayBlockingQueue**

特点： 固定长度的数组阻塞队列。

* 有界队列（强制容量）
* FIFO
* 性能稳定，适合线程池

**LinkedBlockingQueue**

特点： 链表结构的阻塞队列（默认无界）。

* 默认容量为 Integer.MAX\_VALUE（容易 OOM）
* 常用在线程池（Executors.newFixedThreadPool）
* 吞吐比 ArrayBlockingQueue 略高

**PriorityBlockingQueue**

特点： 基于堆的优先级队列，可自定义排序。

* 非 FIFO
* 无界队列（危险）
* 延迟任务、调度任务常用

**DelayQueue**

特点： 基于 PriorityQueue 的延迟队列，元素延迟到期才能取出。

* 必须实现 `Delayed` 接口
* 常用于定时任务、订单过期处理
* 无界（也可能 OOM）

**SynchronousQueue**

特点： 不存储元素，put 必须等待 take。

* 零容量队列
* 常用于 Executors.newCachedThreadPool
* 适合“直接交付任务”的模型

**LinkedTransferQueue**

特点： 高并发无界队列，支持 transfer() 直接将任务交给消费者。

* 性能比 LinkedBlockingQueue 更高
* 可直接“点对点”传输数据
* 适用于高并发生产/消费模型

#### ConcurrentLinkedQueue

特点： 无锁（CAS）队列，高并发且非阻塞。

* 性能极佳
* 非阻塞，无背压
* 无界（风险：堆积导致内存压力）

#### PriorityQueue

特点： 基于小顶堆的优先级队列。

* 非阻塞
* 非线程安全
* 常用于调度系统、任务排序

## Map接口

### HashMap

特点： 基于哈希表的无序 Map，查询/插入平均 O(1)，使用最广。

* 允许 null key / null value
* 速度快，但线程不安全
* 扩容会触发 rehash

### LinkedHashMap

特点： 可保持插入顺序或访问顺序的 HashMap。

* 适合实现 LRU 缓存
* 查询效率与 HashMap 几乎一致
* 额外链表结构，内存略高

### TreeMap

特点： 基于红黑树的有序 Map，按 key 排序。

* 查询/插入 O(log n)
* 支持范围查询（subMap 等）
* key 必须可比较

### WeakHashMap

特点： key 为弱引用，GC 时自动删除 entry。

* 常用于缓存
* 行为依赖 GC，不确定性较高
* 线程不安全

### ConcurrentMap接口

#### ConcurrentHashMap

特点： 高并发 Map，读基本无锁，写低锁竞争。

* 不允许 null key/value
* 原子操作（compute/putIfAbsent）强大
* 适用于绝大多数并发业务

#### ConcurrentSkipListMap

特点： 并发 + 有序 Map，基于跳表结构。

* 有序且线程安全
* 范围查询性能好
* 比 CHM 慢一些、占用更多内存

#### Hashtable

特点： 老旧的线程安全 Map，整表加锁。

* 不允许 null
* 性能最差
* 仅用于兼容遗留系统
