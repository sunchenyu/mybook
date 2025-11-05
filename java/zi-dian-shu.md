# 字典树

## 场景描述

系统内记录了很多接入号，比如10010、10016、10655等，但是用户发起的接入号可能是100101234，我需要根据用户接入号找到系统内匹配到的最长接入号数据

## 特点

* 系统里有一组固定接入号，例如 "10010", "10016", "10655", “1001011”
* 用户可能输入更长的接入号，例如 "100101234"。
* 我们需要找到 系统中与用户输入匹配的最长接入号。

是一个标准的最长前缀匹配算法，很适合字典树的结构。

## 字典树概念

字典树又称前缀树或Trie树，是一种哈希树的变种树形数据结构。其核心特征是通过共享字符串的公共前缀减少存储空间和缩短查询时间，查询效率优于传统哈希树。每个节点存储单一字符，子节点字符互不相同，路径构成对应字符串。

## 实现方案

```
package com.ultra.script.tree;

/**
 * 前缀数字字典树（Trie）实现，支持只包含字符 '0' - '9' 的前缀插入、删除、最长匹配查询等操作。
 * <p>
 * 约定与特性：
 *  - 该实现只支持数字字符 '0' 到 '9'（所以每个节点的 children 长度为 10）。
 *  - 存储泛型值 T（当某个前缀是结束节点时，节点上保存对应的 value）。
 *  - 非线程安全：若在多线程环境中使用，请在外部加锁或改造为线程安全实现。
 *  - 插入与查询时间复杂度为 O(L)，L 为字符串长度；删除最坏情况为 O(L)（并可能回溯做节点清理）。
 * <p>
 * 使用场景示例：
 *  - 电话号段前缀匹配（如电信路由、号码归属地查找等）。
 *  - 数字序列前缀路由/分发。
 * <p>
 * 设计决策：
 *  - 每个节点用固定长度数组 children[10] 存子节点，便于快速索引字符到子节点（无需 HashMap 的开销）。
 *  - size 字段记录当前树中标记为结束的前缀数量（相当于已存条目数）。
 */
public class PrefixTrieNMatcher<T> {

    /**
     * 内部节点定义。
     * <p>
     * children:
     *  - 使用数组而不是 Map，索引 0 对应字符 '0'，索引 9 对应字符 '9'。
     *  - 这种方式在字符集小且稠密时更高效（查询 O(1) 数组访问）。
     * <p>
     * isEnd:
     *  - 标记该节点是否为某个已插入前缀的结束位置。
     * <p>
     * value:
     *  - 当 isEnd 为 true 时，保存与该前缀关联的 value（泛型 T）。
     */
    private static class TrieNode<T> {
        @SuppressWarnings("unchecked")
        TrieNode<T>[] children = (TrieNode<T>[]) new TrieNode[10];
        boolean isEnd;
        T value;
    }

    /** 根节点（不存字符自身，仅为入口） */
    private final TrieNode<T> root = new TrieNode<>();
    /** 当前已插入且仍存在的前缀数量（即 isEnd 为 true 的节点数） */
    private int size;

    /**
     * 插入一个数字前缀并关联 value。
     * <p>
     * 规则与行为：
     *  - 前缀不能为空或 null，否则抛出 IllegalArgumentException。
     *  - 仅支持字符 '0' 到 '9'，遇到非法字符抛出 IllegalArgumentException。
     *  - 如果该前缀已存在（即末节点 isEnd 已为 true），仅覆盖 value，但不增加 size。
     *  - 如果是第一次插入该前缀，则把对应末节点标记 isEnd 并 size++。
     * <p>
     * 时间复杂度：O(L)，L 为 prefix 长度。
     *
     * @param prefix 只包含 '0' - '9' 的字符串前缀
     * @param value 与该前缀关联的值，可以为 null（表示只标记前缀而不保存额外数据）
     * @throws IllegalArgumentException 如果 prefix 为 null/空，或包含非法字符
     */
    public void insert(String prefix, T value) {
        if (prefix == null || prefix.isEmpty()) {
            throw new IllegalArgumentException("前缀不能为空");
        }

        TrieNode<T> node = root;
        // 逐字符向下遍历或创建子节点
        for (int i = 0; i < prefix.length(); i++) {
            char c = prefix.charAt(i);
            // 验证字符范围，仅支持数字
            if (c < '0' || c > '9') {
                throw new IllegalArgumentException("非法字符: " + c + "（只支持0-9）");
            }
            int idx = c - '0';
            TrieNode<T> next = node.children[idx];
            if (next == null) {
                // 如果子节点不存在则创建
                next = new TrieNode<>();
                node.children[idx] = next;
            }
            node = next;
        }

        // 只有第一次把 isEnd 从 false -> true 时，才增加 size
        if (!node.isEnd) {
            node.isEnd = true;
            size++;
        }
        // 更新/设置与前缀关联的值（覆盖旧值）
        node.value = value;
    }

    /**
     * 删除一个已存在的前缀。
     * <p>
     * 行为说明：
     *  - 如果 prefix 为 null 或 空字符串，直接返回 false（不允许删除空前缀）。
     *  - 返回值表示该前缀是否存在并被删除成功（true）还是不存在（false）。
     *  - 删除时，会尝试回溯并清理不再被任何前缀使用的中间节点，节省内存。
     * <p>
     * 时间复杂度：O(L)（其中 L 为 prefix 长度），并在回溯时检查子节点是否为空。
     *
     * @param prefix 要删除的前缀
     * @return 若前缀存在并删除成功返回 true，否则 false
     */
    public boolean delete(String prefix) {
        if (prefix == null || prefix.isEmpty()) return false;
        return deleteRecursive(root, prefix, 0);
    }

    /**
     * 递归删除辅助方法。
     * <p>
     * 逻辑：
     *  - 如果到达 prefix 的末尾（index == prefix.length()）：
     *      - 若当前节点不是结束节点（isEnd == false），则该前缀不存在，返回 false。
     *      - 否则将 isEnd 设为 false，清除 value，size--，并判断当前节点是否可以删除（无子节点则可删除）。
     *  - 否则继续向下递归到下一个字符对应的子节点。
     *  - 递归返回值 shouldDeleteChild 表示子节点是否被删除（即该子节点可以安全置空）。
     *  - 若 shouldDeleteChild 为 true，则当前节点对应的 children[index] 置 null。
     *  - 最后返回当前节点是否可以删除（当前节点非结束且无子节点）。
     * <p>
     * 返回值含义（对父调用者）：
     *  - true  表示父节点应把对应该子节点的引用置 null（子节点已安全删除）
     *  - false 表示父节点不能删除子节点引用（子节点仍被使用）
     *
     * @param node  当前递归正在处理的节点
     * @param prefix 前缀字符串
     * @param index 当前字符索引
     * @return 当前节点是否可以删除（供上层调用决定是否清理引用）
     */
    private boolean deleteRecursive(TrieNode<T> node, String prefix, int index) {
        if (index == prefix.length()) {
            if (!node.isEnd) {
                // 到达末尾但没有标记为结束，表示前缀不存在
                return false;
            }
            // 取消结束标记和关联值
            node.isEnd = false;
            node.value = null;
            size--;
            // 如果该节点无任何子节点，则告知上层可以删除该节点
            return isEmpty(node);
        }

        int idx = prefix.charAt(index) - '0';
        TrieNode<T> child = node.children[idx];
        if (child == null) return false; // 子节点不存在，前缀不存在

        // 递归删除子节点，查看子节点是否可以被移除
        boolean shouldDeleteChild = deleteRecursive(child, prefix, index + 1);
        if (shouldDeleteChild) {
            // 回溯时把引用切断，便于 GC
            node.children[idx] = null;
        }

        // 当前节点可删除当且仅当：自己不是结束节点，并且没有任何子节点
        return !node.isEnd && isEmpty(node);
    }

    /**
     * 判断指定节点是否有子节点。
     *
     * @param node 要检查的节点
     * @return 若节点没有任何子节点返回 true，否则 false
     */
    private boolean isEmpty(TrieNode<T> node) {
        for (TrieNode<T> child : node.children) {
            if (child != null) return false;
        }
        return true;
    }

    /**
     * 在给定的文本中查找与之最长匹配的已插入前缀（从文本开头开始匹配）。
     * <p>
     * 说明：
     *  - 该方法从 text 的第 0 个字符开始逐字符向下匹配 Trie。
     *  - 每当遇到一个节点的 isEnd 为 true，即记录当前位置的 value 为 lastMatch（但继续向下查找以期得到更长匹配）。
     *  - 当遇到非法字符（非 '0'-'9'）或子节点为 null 时停止查找并返回当前记录的 lastMatch。
     *  - 如果没有任何匹配返回 null。
     * <p>
     * 时间复杂度：O(min(L_text, L_longestPrefix))，主要是按字符遍历直到不匹配为止。
     *
     * @param text 要匹配的文本（从开头开始匹配）
     * @return 与文本最长匹配的 value（若没有匹配返回 null）
     */
    public T findLongestMatch(String text) {
        if (text == null || text.isEmpty()) return null;

        TrieNode<T> node = root;
        T lastMatch = null;
        for (int i = 0; i < text.length(); i++) {
            char c = text.charAt(i);
            // 只支持数字字符，遇到其他字符直接停止匹配
            if (c < '0' || c > '9') break;
            int idx = c - '0';
            node = node.children[idx];
            if (node == null) break; // 无对应子节点，停止
            if (node.isEnd) lastMatch = node.value; // 记录当前匹配（可能继续找更长的）
        }
        return lastMatch;
    }

    /**
     * 清空整棵字典树（释放直接引用，帮助 GC 回收子节点）。
     * <p>
     * 说明：
     *  - 仅将根节点的 children 引用全部置空并重置根节点的标志与 size。
     *  - 该操作不会递归清理所有子节点对象，但断开了根到这些子节点的唯一强引用链，GC 可回收它们。
     */
    public void clear() {
        for (int i = 0; i < 10; i++) {
            root.children[i] = null;
        }
        root.isEnd = false;
        root.value = null;
        size = 0;
    }

    /**
     * 返回当前已插入的前缀数量（即被标记为结束节点的节点数）。
     *
     * @return 前缀数量
     */
    public int size() {
        return size;
    }
}
```

## 测试代码

```
package com.ultra.script.tree;

import java.util.*;
import java.util.concurrent.TimeUnit;
public class PrefixTrieNMatcherTest {

    private static final int PREFIX_COUNT = 1_000_000;  // 插入前缀数量
    private static final int QUERY_COUNT = 1_000_000;   // 查询次数
    private static final Random RANDOM = new Random();

    public static void main(String[] args) throws Exception {
        PrefixTrieNMatcher<EcDO> trie = new PrefixTrieNMatcher<>();

        // === 1. 生成随机前缀 ===
        List<String> prefixes = generatePrefixes(PREFIX_COUNT);
        System.out.println("Generated " + prefixes.size() + " prefixes.");

        // === 2. 记录内存初始状态 ===
        printMemory("初始状态");

        // === 3. 插入性能测试 ===
        long insertStart = System.nanoTime();
        for (String prefix : prefixes) {
            trie.insert(prefix, new EcDO(prefix, "account", "pass123456"));
        }
        long insertEnd = System.nanoTime();

        printMemory("插入完成后");
        System.out.println("Insert done. Time: " +
                TimeUnit.NANOSECONDS.toMillis(insertEnd - insertStart) + " ms");

        // === 4. 生成随机查询号码 ===
        List<String> numbers = generateNumbers(QUERY_COUNT);

        // === 5. 查询性能测试 ===
        long queryStart = System.nanoTime();
        int hitCount = 0;
        for (String num : numbers) {
            EcDO ecDO = trie.findLongestMatch(num);
            if (ecDO != null) {
                if (hitCount == 0) {
                    System.out.println(ecDO);
                }
                hitCount++;
            }
        }
        long queryEnd = System.nanoTime();

        // === 6. 打印结果 ===
        long totalNanos = queryEnd - queryStart;
        double avgNs = (double) totalNanos / QUERY_COUNT;

        System.out.println("\n===== PrefixTrieMatcher 性能测试结果 =====");
        System.out.println("前缀数量       : " + PREFIX_COUNT);
        System.out.println("查询次数       : " + QUERY_COUNT);
        System.out.println("命中次数       : " + hitCount);
        System.out.println("总查询耗时(ms) : " + TimeUnit.NANOSECONDS.toMillis(totalNanos));
        System.out.println("平均单次耗时(ns): " + String.format("%.2f", avgNs));
        System.out.println("平均单次耗时(µs): " + String.format("%.4f", avgNs / 1000.0));

        printMemory("查询完成后");
        System.out.println("当前数量" + trie.size());
    }

    /**
     * 打印当前 JVM 内存占用
     */
    private static void printMemory(String phase) {
        Runtime runtime = Runtime.getRuntime();
        runtime.gc();
        long used = runtime.totalMemory() - runtime.freeMemory();
        System.out.printf("[内存统计] %-12s => 使用: %.2f MB, 总: %.2f MB, 最大: %.2f MB, 可用: %.2f MB%n",
                phase,
                used / 1024.0 / 1024.0,
                runtime.totalMemory() / 1024.0 / 1024.0,
                runtime.maxMemory() / 1024.0 / 1024.0,
                runtime.freeMemory() / 1024.0 / 1024.0);
    }


    /**
     * 生成随机前缀（长度4~8）
     */
    private static List<String> generatePrefixes(int count) {
        List<String> list = new ArrayList<>(count);
        for (int i = 0; i < count; i++) {
            list.add(randomNumber(4 + RANDOM.nextInt(17)));
        }
        return list;
    }

    /**
     * 生成随机号码（长度10~12）
     */
    private static List<String> generateNumbers(int count) {
        List<String> list = new ArrayList<>(count);
        for (int i = 0; i < count; i++) {
            list.add(randomNumber(10 + RANDOM.nextInt(3)));
        }
        return list;
    }

    /**
     * 生成随机数字字符串
     */
    private static String randomNumber(int len) {
        StringBuilder sb = new StringBuilder(len);
        for (int i = 0; i < len; i++) {
            sb.append(RANDOM.nextInt(10));
        }
        return sb.toString();
    }
}

```

打印结果如下

<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>
