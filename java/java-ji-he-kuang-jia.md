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

### List

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

### Set

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

### Queue

#### Deque

ArrayDeque

底层：循环数组特点：

* 非阻塞队列
* 比 LinkedList 更快
* 用作栈、双端队列的最佳选择

ConcurrentLinkedDeque



LinkedBlockingDeque



#### BlockingQueue

ArrayBlockingQueue



LinkedBlockingQueue



PriorityBlockingQueue



DelayQueue



SynchronousQueue



LinkedTransferQueue



#### ConcurrentLinkedQueue



#### PriorityQueue



## Map

### HashMap



### LinkedHashMap



### TreeMap



### WeakHashMap



### ConcurrentMap

#### ConcurrentHashMap



#### ConcurrentSkipListMap



#### Hashtable

