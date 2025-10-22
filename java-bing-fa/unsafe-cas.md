# Unsafe中的CAS

## 一、Unsafe是什么

Unsafe，一个JAVA类，位于sun.misc包中，是jvm提供的手动管理内存的实现，里面的方法都是native标记的，说明这些方法的实现不在java中，而且其他语言编写的，这个Unsafe类，就是包装的底层的c++代码，提供给我们一种直接操作内存的方式

## 二、Unsafe中的几个方法

### （1）获取非静态属性在对象中的偏移量方法

```
public native long objectFieldOffset(Field var1);
```

objectFieldOffset方法用于获取非静态属性Field在对象实例中的偏移量，读写对象的非静态属性时会用到这个偏移量

### （2）比较并交换的原子操作

**作用：修改对象中的字段**

```
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

**参数说明：** Object var1 被修改的对象

long var2 字段在内存中的偏移量（可以使用Unsafe中的获取偏移量相关的方法）

var4 期望值 可以是int，long，Object，用这个值与 var1 + var2的值对比，相同则修改，并返回true，不同则不修改，返回false

var5 预期结果值 可以是int，long，Object，如果与上一步对比相等，则将这个值赋值到 var1 + var2位置

### （3）测试Unsafe中的CAS

字段顺序如果是 int long Object：

```
package com.suncy.article.article4;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

public class CasTest {
    //字段分别为int long  Object
    private int a;
    private long b;
    private Object c;

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        CasTest casTest = new CasTest();

        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);

        //拿到a字段在对象中的偏移量
        long aOffset = unsafe.objectFieldOffset
                (CasTest.class.getDeclaredField("a"));
        System.out.println("Test测试结果：");
        System.out.println("a offset:" + aOffset);
        System.out.println("a = " + casTest.a);
        System.out.println(unsafe.compareAndSwapInt(casTest, aOffset, 0, 1));
        System.out.println("a = " + casTest.a);
        System.out.println(unsafe.compareAndSwapInt(casTest, aOffset, 0, 1));
        System.out.println("a = " + casTest.a);
        System.out.println("===========================");

        //拿到b字段在对象中的偏移量
        long bOffset = unsafe.objectFieldOffset
                (CasTest.class.getDeclaredField("b"));
        System.out.println("b offset:" + bOffset);
        System.out.println("b = " + casTest.b);
        System.out.println(unsafe.compareAndSwapLong(casTest, bOffset, 0, 1));
        System.out.println("b = " + casTest.b);
        System.out.println(unsafe.compareAndSwapLong(casTest, bOffset, 0, 1));
        System.out.println("b = " + casTest.b);
        System.out.println("===========================");

        //拿到c字段在对象中的偏移量
        long cOffset = unsafe.objectFieldOffset
                (CasTest.class.getDeclaredField("c"));
        System.out.println("c offset:" + cOffset);
        System.out.println("c = " + casTest.c);
        System.out.println(unsafe.compareAndSwapObject(casTest, cOffset, casTest.c, new Object()));
        System.out.println("c = " + casTest.c);
        System.out.println(unsafe.compareAndSwapObject(casTest, cOffset, new Object(), new Object()));
        System.out.println("c = " + casTest.c);
    }
}
```

输出结果：&#x20;

![测试结果](<../.gitbook/assets/image (34).png>)

字段顺序如果是long int Object：

```
package com.suncy.article.article4;

import sun.misc.Unsafe;

import java.lang.reflect.Field;

public class CasTest2 {
    //字段分别为long int Object
    private long a;
    private int b;
    private Object c;

    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        CasTest2 test = new CasTest2();

        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);

        //拿到a字段在对象中的偏移量
        long aOffset = unsafe.objectFieldOffset
                (CasTest2.class.getDeclaredField("a"));
        System.out.println("Test2测试结果：");
        System.out.println("a offset:" + aOffset);
        System.out.println("a = " + test.a);
        System.out.println(unsafe.compareAndSwapLong(test, aOffset, 0, 1));
        System.out.println("a = " + test.a);
        System.out.println(unsafe.compareAndSwapLong(test, aOffset, 0, 1));
        System.out.println("a = " + test.a);
        System.out.println("===========================");

        //拿到b字段在对象中的偏移量
        long bOffset = unsafe.objectFieldOffset
                (CasTest2.class.getDeclaredField("b"));
        System.out.println("b offset:" + bOffset);
        System.out.println("b = " + test.b);
        System.out.println(unsafe.compareAndSwapInt(test, bOffset, 0, 1));
        System.out.println("b = " + test.b);
        System.out.println(unsafe.compareAndSwapInt(test, bOffset, 0, 1));
        System.out.println("b = " + test.b);
        System.out.println("===========================");

        //拿到c字段在对象中的偏移量
        long cOffset = unsafe.objectFieldOffset
                (CasTest2.class.getDeclaredField("c"));
        System.out.println("c offset:" + cOffset);
        System.out.println("c = " + test.c);
        System.out.println(unsafe.compareAndSwapObject(test, cOffset, test.c, new Object()));
        System.out.println("c = " + test.c);
        System.out.println(unsafe.compareAndSwapObject(test, cOffset, new Object(), new Object()));
        System.out.println("c = " + test.c);
    }
}
```

输出结果：

![测试结果](<../.gitbook/assets/image (8) (1).png>)

## 三、测试CAS结果

### （1）结果表现

**CasTest：**

CasTes中的字段为int a，long b，Object c， a的offset为12 b的offset为16 c的offset为24， a的第一次比较并交换结果为成功，值变为1， a的第二次比较并交换结果为失败，因为预期值不一致，值没变，还是1。

**CasTest2：** CasTest2中的字段为long a，int b，Object c， a的offset为16，b的offset为12，c的offset为24， a的第一次比较并交换结果为成功，值变为1， a的第二次比较并交换结果为失败，因为预期值不一致，值没变，还是1。

**其他字段的cas操作同这个结果一致**

### （2）但为什么两次结果的第一个字段的offset都是12开始呢？

这证明了一个基础知识点：存在对象头（但这个对象头可能是8字节，也可能是12字节，和是否开启指针压缩有关）

### （3）为什么没有直接用Unsafe.getUnsafe()方法直接获取Unsafe

因为会抛出异常，测试代码如下：

```
package com.suncy.article.article4;

import sun.misc.Unsafe;

public class Test {
    public static void main(String[] args) {
        Unsafe unsafe = Unsafe.getUnsafe();
        System.out.println(unsafe);
    }
}
```

执行结果：

![执行结果](<../.gitbook/assets/image (42).png>)

原因：这个类只能被JDK信任的类实例化。
