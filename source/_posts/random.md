---
title: random、ThreadLocalRandom、SecureRandom
date: 2018-02-27 16:06:38
tags: [JDK8]
---

### Random

Random可以算是我们常用的生成随机数的类，比如

```java

    Random random = new Random();
    int a = random.nextInt(5);

```

这样会生成一个0~4之间的数字。

<!-- more -->

#### Random构造函数：

```java

    //无参
    public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }

    private static long seedUniquifier() {
        // L'Ecuyer, "Tables of Linear Congruential Generators of
        // Different Sizes and Good Lattice Structure", 1999
        for (;;) {
            long current = seedUniquifier.get();
            long next = current * 181783497276652981L;
            if (seedUniquifier.compareAndSet(current, next))
                return next;
        }
    }

    private static final AtomicLong seedUniquifier
        = new AtomicLong(8682522807148012L);

    //有参
    public Random(long seed) {
        if (getClass() == Random.class)
            this.seed = new AtomicLong(initialScramble(seed));
        else {
            // subclass might have overriden setSeed
            this.seed = new AtomicLong();
            setSeed(seed);
        }
    }

    private static long initialScramble(long seed) {
        return (seed ^ multiplier) & mask;
    }

```

构造方法主要的作用是生成一个AtomicLong类型的seed，作为实例属性。

### nextInt()
```java

    public int nextInt() {
        return next(32);
    }

    public int nextInt(int bound) {
        if (bound <= 0)
            throw new IllegalArgumentException(BadBound);

        int r = next(31);
        int m = bound - 1;
        if ((bound & m) == 0)  // i.e., bound is a power of 2
            r = (int)((bound * (long)r) >> 31);
        else {
            for (int u = r;
                 u - (r = u % bound) + m < 0;
                 u = next(31))
                ;
        }
        return r;
    }

    protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask;
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }

```

我们主要看next()方法，nextseed的计算方式是固定的（(oldseed * multiplier + addend) & mask），所以next的返回结果是可计算的，计算得到nextseed后，使用compareAndSet方法将nextseed赋值到seed属性中（cas操作），如果成功，则返回(int)(nextseed >>> (48 - bits))，否则，重新获得oldseed并进行计算。所以，next方法返回值是可预测、有迹可循的，生成的结果是伪随机数。


```java

    Random random = new Random(1);
    int a = random.nextInt(5);
    random = new Random(1);
    int b = random.nextInt(5);
    random = new Random(1);
    int c = random.nextInt(5);

```

可以判断，以上代码中 a = b = c。

所以我们一定不能把这个种子写死，用当前时间毫秒数，还是比较好些。另外，重复new Random对象的意义也并不是那么大。

最后说一点，Random是线程安全的，去这里的官方文档可以看到，“Instances of java.util.Random are threadsafe.”。但是在多线程的表现中，他的性能很差。

### ThreadLocalRandom

```java

    public int nextInt() {
        return mix32(nextSeed());
    }

    final long nextSeed() {
        Thread t; long r; // read and update per-thread seed
        UNSAFE.putLong(t = Thread.currentThread(), SEED,
                       r = UNSAFE.getLong(t, SEED) + GAMMA);
        return r;
    }

    private static int mix32(long z) {
        z = (z ^ (z >>> 33)) * 0xff51afd7ed558ccdL;
        return (int)(((z ^ (z >>> 33)) * 0xc4ceb9fe1a85ec53L) >>> 32);
    }

    private static final long GAMMA = 0x9e3779b97f4a7c15L;

```

在查看ThreadLocalRandom类，nextInt方法主要是查找了当前线程中threadLocalRandomSeed属性的值，每次增加一个GAMMA，再将值重新放入线程中，然后调用mix32进行处理。由于ThreadLocalRandom并未大量使用cas，所以ThreadLocalRandom比Random更快。

### SecureRandom

在需要频繁生成随机数，或者安全要求较高的时候，不要使用Random，因为Random是可预测的。

This class provides a cryptographically strong random number generator (RNG).
SecureRandom 提供加密的强随机数生成器 (RNG)，要求种子必须是不可预知的，产生非确定性输出。
SecureRandom 也提供了与实现无关的算法，因此，调用方（应用程序代码）会请求特定的 RNG 算法并将它传回到该算法的 SecureRandom 对象中。

如果仅指定算法名称，如下所示：

```java

SecureRandom random = SecureRandom.getInstance("SHA1PRNG");

```

如果既指定了算法名称又指定了包提供程序，如下所示：

```java

SecureRandom random = SecureRandom.getInstance("SHA1PRNG", "SUN");

```

使用：

```java

SecureRandom random1 = SecureRandom.getInstance("SHA1PRNG");
SecureRandom random2 = SecureRandom.getInstance("SHA1PRNG");

for (int i = 0; i < 5; i++) {
    System.out.println(random1.nextInt() + " != " + random2.nextInt());
}

```
