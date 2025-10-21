# ArrayList源码

继承

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{ ... }
```



```
private static final long serialVersionUID = 8683452581122892189L;

private static final int DEFAULT_CAPACITY = 10;

//用于空实例的共享空数组实例。
private static final Object[] EMPTY_ELEMENTDATA = {};

//用于默认大小的空实例的共享空数组实例。我们将其与 EMPTY_ELEMENTDATA 区分开来，以了解添加第一个元素时要膨胀多少
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//存储 ArrayList 元素的数组，默认没有赋值，是一个null值
transient Object[] elementData; // non-private to simplify nested class access

//ArrayList 的大小（它包含的元素数量）
private int size;
```

构造器

主要是创建elementData数组

```
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

成员方法，分析几个常用的

contains方法

```
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
```

indexOf方法

```
 public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

add方法

```
  public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

ensureCapacityInternal方法

```
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

calculateCapacity方法

```
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    //添加元素之后的容量和当前默认的容量进行对比，返回这两个的最大值
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

ensureExplicitCapacity方法

```
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;  //目前仅作为计数，快速失败迭代器的时候会使用
    
    //如果添加元素之后的长度 要 大于当前数组的长度之后，需要扩容
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

grow扩容方法

```
private void grow(int minCapacity) {
    // overflow-conscious code
    //记录旧数组元素长度
    int oldCapacity = elementData.length;
    //新数组长度 约等于 老数组容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 避免扩容不足
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //数组超长
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //拷贝数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

hugeCapacity方法

```
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

Arrays.copyOf数组拷贝方法

```
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}
```

copyOf方法

```
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```
