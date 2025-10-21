# Java集合

Collection存储单个元素

```
Collection（接口）
 ├── List（接口）
 │    ├── ArrayList
 │    ├── LinkedList
 │    └── Vector
 │         └── Stack
 │
 ├── Set（接口）
 │    ├── HashSet
 │    │     └── LinkedHashSet
 │    └── TreeSet
 │
 └── Queue（接口）
      ├── LinkedList
      ├── PriorityQueue
      └── Deque（接口）
           ├── ArrayDeque
           └── LinkedList

```

Map存储键值对

```
Map（接口）
 ├── HashMap
 │     └── LinkedHashMap
 ├── TreeMap
 ├── Hashtable
 └── ConcurrentHashMap

```

接下来需要一个一个去分析每个类的特点，方便在开发当中选择合适的容器进行使用
