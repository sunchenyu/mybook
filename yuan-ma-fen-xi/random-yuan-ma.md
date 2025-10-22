# Random源码

## 成员变量

```
static final long serialVersionUID = 3905348978240129619L;

//随机数种子
private final AtomicLong seed;

private static final long multiplier = 0x5DEECE66DL;
private static final long addend = 0xBL;
private static final long mask = (1L << 48) - 1;

private static final double DOUBLE_UNIT = 0x1.0p-53; // 1.0 / (1L << 53)

static final String BadBound = "bound must be positive";
static final String BadRange = "bound must be greater than origin";
static final String BadSize  = "size must be non-negative";

private static final AtomicLong seedUniquifier
    = new AtomicLong(8682522807148012L);
```

## 基本使用方法

```
Random random = new Random();
System.out.println(random.nextInt());
```

其他包括

<div align="left"><figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

## 构造器

```
public Random() {
    this(seedUniquifier() ^ System.nanoTime());
}
```

参数为 seedUniquifier() ^ System.nanoTime()\
由seedUniquifier()生成的long值 异或 上System.nanoTime()

nanoTime方法\
在虚拟机启动时，会随机的选择一个固定的long类型的值作为时间的起点，每次调用该方法，会返回从虚拟机设定的时间起点到调用时经过的时间，单位为纳秒

seedUniquifier方法\
该函数得到的是一个固定的值，但是为什么要这么计算，是为了防止短期内同一时刻调用System.nanoTime()生成同样的数字导致随机数相同\
所以采用自旋操作

```
private static long seedUniquifier() {
    // L'Ecuyer, "Tables of Linear Congruential Generators of
    // Different Sizes and Good Lattice Structure", 1999
    for (;;) {
        //这个current是一个常量，是一个Random类中的静态常量8682522807148012L
        long current = seedUniquifier.get();
        //next是相乘得到的一个固定值
        long next = current * 181783497276652981L;
        if (seedUniquifier.compareAndSet(current, next))
            return next;
    }
}
```

由此我们发现随机数的随机是由System.nanoTime()决定的

会走到如下地方

```
public Random(long seed) {
    if (getClass() == Random.class)
        this.seed = new AtomicLong(initialScramble(seed));
    else {
        // 子类可能重写setSeed方法
        this.seed = new AtomicLong();
        setSeed(seed);
    }
}
```

setSeed方法

```
synchronized public void setSeed(long seed) {
    this.seed.set(initialScramble(seed));
    haveNextNextGaussian = false;
}
```

initialScramble方法

```
private static long initialScramble(long seed) {
    return (seed ^ multiplier) & mask;
}
```

## next方法

不管是nextLong还是nextInt都会走到next方法

```
protected int next(int bits) {
        long oldseed, nextseed;
        //拿到seed
        AtomicLong seed = this.seed;
        do {
            //获取seed数值
            oldseed = seed.get();
            // 这里是一个线性同余方程
            nextseed = (oldseed * multiplier + addend) & mask;
            //每次设置种子为 利用上个种子生成             
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }
```

这块存在一个数学概念，线性同余方程

## 线性同余方程

线性同余方法是目前应用广泛的伪随机数生成算法，其基本思想是通过对前一个数进行线性运算并取模从而得到下一个数，递归公式为：

```
xn+1=(axn+c)mod(m)
xn+1=(axn+c)mod(m)
yn+1=xn+1/m
yn+1=xn+1/m
```

其中a称为乘数，c称为增量，m称为模数，当a=0时为和同余法，当c=0时为乘同余法，c≠0时为混合同余法。 乘数、增量和模数的选取可以多种多样，只要保证产生的随机数有较好的均匀性和随机性即可，一般采用m=2km=2k混合同余法。

线性同余法的最大周期是m，但一般情况下会小于m。要使周期达到最大，应该满足以下条件：

1. c和m互质；
2. m的所有质因子的积能整除a-1；
3. 若m是4的倍数，则a-1也是；
4. a，c，x0x0（初值，一般即种子）都比m小；
5. a，c是正整数。

线性同余方法速度快，如果对乘数和模数进行适当的选择，可以满足用于评价一个随机数产生器的3 种准则：

1.   这个函数应该是一个完整周期的产生函数。也就是说，这个函数应该在重复之前产生出0 到m之间的所有数；
2. 产生的序列应该看起来是随机的；
3. 这个函数应该用32bit 算术高效实现。

