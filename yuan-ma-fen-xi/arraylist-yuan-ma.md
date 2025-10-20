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



```
   private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

