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

先看一下两个最顶级的接口

Collection

```java
package java.util;

import java.util.function.Predicate;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;

public interface Collection<E> extends Iterable<E> {
    int size();                    //获取元素长度
    boolean isEmpty();             //容器是否为空
    boolean contains(Object o);    //是否包含某个元素
    Iterator<E> iterator();        //获取迭代器
    Object[] toArray();            //返回一个包含集合所有元素的数组
    <T> T[] toArray(T[] a);        //将当前集合元素拷贝到一个指定类型的数组中，并返回该数组
    boolean add(E e);              //给集合增加元素
    boolean remove(Object o);        //移除集合元素
    boolean containsAll(Collection<?> c);    //当前集合是否包含某集合元素
    boolean addAll(Collection<? extends E> c);  //某集合加入到当前集合
    boolean removeAll(Collection<?> c);    //移除集合元素
    default boolean removeIf(Predicate<? super E> filter) {  //根据给定的条件（Predicate）删除集合中所有满足条件的元素。
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    boolean retainAll(Collection<?> c);    //保留集合中与另一个集合共有的元素
    void clear();                          //清空集合或映射中的所有元素
   
    @Override
    default Spliterator<E> spliterator() {    //可分割迭代器，为了支持并行流
        return Spliterators.spliterator(this, 0);
    }
    default Stream<E> stream() {                //生成流    
        return StreamSupport.stream(spliterator(), false);
    }
    default Stream<E> parallelStream() {        //生成一个并行流
        return StreamSupport.stream(spliterator(), true);
    }
}
```

Map接口

```java
package java.util;

import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Function;
import java.io.Serializable;

public interface Map<K,V> {
    int size();                            //返回Map中键值对的数量
    boolean isEmpty();                     //map是否为空
    boolean containsKey(Object key);       //map是否包含某个key
    boolean containsValue(Object value);   //map是否包含某个value
    V get(Object key);                     //根据key获取value
    V put(K key, V value);                 //放入key和value
    V remove(Object key);                  //根据key移除数据
    void putAll(Map<? extends K, ? extends V> m);
    void clear();                          //清理map
    Set<K> keySet();                       //获取key的集合
    Collection<V> values();                //获取value的集合
    Set<Map.Entry<K, V>> entrySet();       //获取键值对集合
    interface Entry<K,V> {                 //表示Map中的一个键值对
        K getKey();
        V getValue();
        V setValue(V value);
        boolean equals(Object o);
        int hashCode();
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() { 
         //按键自然顺序排序
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }
        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
        //按值自然顺序排序
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
        //按键自定义Comparator
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
        }
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) { 
        //按值自定义Comparator
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
        }
    }

    boolean equals(Object o);
    int hashCode();
    default V getOrDefault(Object key, V defaultValue) {    //获取不到返回默认值
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }
    default void forEach(BiConsumer<? super K, ? super V> action) {    //遍历Map中所有键值对，执行BiConsumer操作
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }

    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) { //批量替换Map中的所有值
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }

    default V putIfAbsent(K key, V value) {            //只有当key不存在或对应值为null时才插入
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    default boolean remove(Object key, Object value) {  //只有当key对应的值等于给定value时才删除
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }

    default boolean replace(K key, V oldValue, V newValue) {  //仅当key对应值为oldValue时替换为newValue
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }

    default V replace(K key, V value) {                //替换指定key的value
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }

    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) { //如果key不存在，则使用mappingFunction计算value并插入
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }

    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) { //如果key存在，则重新计算value
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }

    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {  //不管key是否存在，都重新计算value
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }

    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) { //将新value和旧value合并
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
}
```
