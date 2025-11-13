# NavigableMap

继承了SortedMap，SortedMap继承了Map接口\
SortedMap接口如下

```java
package java.util;

public interface SortedMap<K,V> extends Map<K,V> {
    Comparator<? super K> comparator();    //返回用于排序的比较器。如果返回 null，表示使用键的自然顺序，即 Comparable
    SortedMap<K,V> subMap(K fromKey, K toKey);    //返回 [fromKey, toKey) 区间的子视图（包含起始，不包含结尾）
    SortedMap<K,V> headMap(K toKey);              //返回小于toKey的所有键的子视图(排序号左侧)。
    SortedMap<K,V> tailMap(K fromKey);            //返回大于等于fromKey的所有键的子视图（排序后右侧）。
    K firstKey();                         //返回排序后第一个key
    K lastKey();                          //返回排序后最后一个key
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();
}
```

NavigableMap在有序Map的基础上，实现了导航功能：支持向前/向后查找、范围检索、最近匹配\
接口如下

```java
package java.util;

public interface NavigableMap<K,V> extends SortedMap<K,V> {
    Map.Entry<K,V> lowerEntry(K key);  //返回小于给定key的最大键值对
    K lowerKey(K key);                 //返回小于给定key的最大的key
    Map.Entry<K,V> floorEntry(K key);  //返回小于或等于key的最大键值对
    K floorKey(K key);                 //返回小于或等于key的最大的key
    Map.Entry<K,V> ceilingEntry(K key);//返回大于或等于key的最小键值对
    K ceilingKey(K key);                //返回大于或等于key的最小的key
    Map.Entry<K,V> higherEntry(K key);  //返回大于key的最小键值对
    K higherKey(K key);                 //返回大于key的最小的key

    Map.Entry<K,V> firstEntry();        //返回最小key的键值对
    Map.Entry<K,V> lastEntry();         //返回最大key的键值对
    Map.Entry<K,V> pollFirstEntry();    //删除并返回最小key的entry
    Map.Entry<K,V> pollLastEntry();     //删除并返回最大key的entry

    NavigableMap<K,V> descendingMap();   //生成当前Map的反向视图
    NavigableSet<K> navigableKeySet();   //返回一个支持导航的 key 集（升序）
    NavigableSet<K> descendingKeySet();  //返回降序的 key 集

    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive, K toKey,   boolean toInclusive);     //可自定义边界是否包含。修改会影响原 Map
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);    //返回小于toKey的所有键值对,支持是否包含边界。修改会影响原 Map
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);  //返回大于fromKey的所有键值对,支持是否包含边界。修改会影响原 Map
    SortedMap<K,V> subMap(K fromKey, K toKey);           //返回[fromKey, toKey)范围的视图，from包含，to不包含。修改会影响原 Map
    SortedMap<K,V> headMap(K toKey);                     //返回小于toKey的所有键值对。修改会影响原 Map
    SortedMap<K,V> tailMap(K fromKey);                    //返回大于等于fromKey的所有键值对。修改会影响原 Map
}
```

TreeMap实现了上面的功能

```
package com.ultra.script.tree;

import java.util.*;

public class NavigableMapDemo {
    public static void main(String[] args) {
        // TreeMap 实现了 NavigableMap 接口（按 key 排序）
        NavigableMap<String, String> phoneMap = new TreeMap<>();

        // 插入一些手机号前缀与运营商名称的映射
        phoneMap.put("130", "联通");
        phoneMap.put("131", "联通");
        phoneMap.put("132", "联通");
        phoneMap.put("133", "电信");
        phoneMap.put("134", "移动");
        phoneMap.put("135", "移动");
        phoneMap.put("136", "移动");

        System.out.println("== 所有数据（按升序） ==");
        for (Map.Entry<String, String> e : phoneMap.entrySet()) {
            System.out.println(e.getKey() + " -> " + e.getValue());
        }

        //floorEntry()：返回小于等于给定 key 的最大 entry
        System.out.println("\n== floorEntry 示例 ==");
        System.out.println("floorEntry(134): " + phoneMap.floorEntry("134"));

        //ceilingEntry()：返回大于等于给定 key 的最小 entry
        System.out.println("ceilingEntry(134): " + phoneMap.ceilingEntry("134"));

        //higherEntry() / lowerEntry()：返回大于/小于指定 key 的 entry
        System.out.println("higherEntry(134): " + phoneMap.higherEntry("134"));
        System.out.println("lowerEntry(134): " + phoneMap.lowerEntry("134"));

        //subMap(fromKey, toKey)：获取范围视图（不包含toKey）
        NavigableMap<String, String> range = phoneMap.subMap("132", true, "136", false);
        System.out.println("\n== subMap(132, 136) ==");
        range.forEach((k, v) -> System.out.println(k + " -> " + v));

        System.out.println("\n== 通过视图删除134数据");
        range.remove("134");

        NavigableMap<String, String> range1 = phoneMap.subMap("132", true, "136", false);
        System.out.println("\n== subMap(132, 136) == 这个是通过subMap视图删除数据在通过原数据查询");
        range1.forEach((k, v) -> System.out.println(k + " -> " + v));

        //descendingMap()：反向视图（降序）
        System.out.println("\n== 降序遍历 ==");
        phoneMap.descendingMap().forEach((k, v) -> System.out.println(k + " -> " + v));
    }
}
```

执行结果如下

<div align="left"><figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure></div>

总结

SortedMap：保证键的有序性，支持范围视图。

NavigableMap：在SortedMap基础上提供了上下界查找、反向遍历、精确范围控制等高级功能。

TreeMap：是最常用的NavigableMap实现。基于红黑树。

ConcurrentSkipListMap：NavigableMap实现的并发版本，基于跳表。

