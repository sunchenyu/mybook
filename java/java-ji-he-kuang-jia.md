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

## 特点整理

### 集合

#### List

有序、元素可重复、可以通过索引访问

ArrayList

底层结构：动态数组

* 优点：
* 随机访问 O(1)
* 内存连续，CPU 缓存友好
* 适合读多、遍历多

缺点：

* 扩容需要复制数组
* 头部插入/删除成本高 O(n)



#### Set





#### Queue



### Map





