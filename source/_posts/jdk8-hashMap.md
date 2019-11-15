---
title: HashMap(jdk8)
date: 2018-02-01 13:19:49
tags: [java8, HashMap]
---

### HashMap静态常量

```java

	/**
     * 默认table数组大小，必须是2的倍数
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * table数组长度最大值，为2^30
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 负载因子（默认）
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 标识由链表转换为红黑树的阈值，只有当某一个链表大于等于此值的时候会把这个链表转换为红黑
     * 树。这个数必须要大于2，默认值是8，当然也推荐从8开始往上的值。（查看源码后，发现如果
     * table数组的长度小于MIN_TREEIFY_CAPACITY的值的时候，仅仅会进行扩容，并不会进行红黑
     * 树转换，见HashMap#treeifyBin()方法）
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 标识有红黑树转换为链表的阈值，只有当某个某个红黑树小于等于这个值的时候，会把红黑树转换
     * 为链表。具体方法为TreeNode#untreeify()
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 限定table长度的最小值，当大于等于这个值的时候，将这个链表转换为红黑树；否则，仅仅进行
     * 扩容。方法见HashMap#treeifyBin()
     */
    static final int MIN_TREEIFY_CAPACITY = 64;


```

<!--more-->

### Node<K,V>(HashMap静态内部类)

```java

	static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

```

Node注意点：

1. Node类是链表状态下每个节点的是实际类型，它实现了Map.Entity接口。每个Node节点有一个next成员变量，指向下一个Node节点的引用。
2. Node类为静态内部类，作用域为默认级别，即仅有此类本身以及相同包下的类能够访问。
3. hash与key两个类型均为final，意味着不能改变。
4. 每个Node节点的hashcode方法是由每个节点的key与value的hashcode方法的^，查看Object.hash()方法，发现当key或者value等于null的时候，hash为0；否则，就会调用这个对象的hashcode方法。同时，这意味着equals同时跟key与value有关系。
5. setValue方法会返回node中的原值。
6. equals方法中，并不排斥map中key或者value为null。

### HashMap静态方法

```java

static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

```

此方法当key为null时返回0，否则，就会用key的hash与hash的前16位异或。

```java

static Class<?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {
            Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
            if ((c = x.getClass()) == String.class) // bypass checks
                return c;
            if ((ts = c.getGenericInterfaces()) != null) {
                for (int i = 0; i < ts.length; ++i) {
                    if (((t = ts[i]) instanceof ParameterizedType) &&
                        ((p = (ParameterizedType)t).getRawType() ==
                         Comparable.class) &&
                        (as = p.getActualTypeArguments()) != null &&
                        as.length == 1 && as[0] == c) // type arg is c
                        return c;
                }
            }
        }
        return null;
    }

```
comparableClassFor方法用于判断x是不是可比较的（是否实现了Comparable接口），如果实现了此接口则返回Comparable接口中的泛型类型，否则返回null。

```java

static int compareComparables(Class<?> kc, Object k, Object x) {
        return (x == null || x.getClass() != kc ? 0 :
                ((Comparable)k).compareTo(x));
    }

```

返回k与x的比较结果。如果x为null或者x的class != kc，返回0。

```java

    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

```

这段就厉害了，此方法声明了一个算法，通过位运算，使得获得的数值是输入的两倍。先看n的位移操作部分，如果是0100，第一次位移后就变为了0110，第二次为0111，以此类推，最后的结果为0111，再加一所得结果为1000，扩容为了下一个2的次方。其原理在于在二进制数中，某个数的最高位上肯定是1，将这个1通过位运算以及或运算，依次向后延伸，所得的数肯定为2^n-1，最后+1，得到2^n次方。

再来看第一句话，第一句话保证了如果输入的本来就是2次方，所得结果是其本身。

当然，这个最后结果最大值为2^30。

### HashMap的实例字段

```java

	/**
     * 该字段为每个链表或红黑树的第一个Node组成的数组。此数组在第一次使用的时候进行初始化，
     * 当分配长度的时候，这个数组的长度总是2^n。
     */
    transient Node<K,V>[] table;

    /**
     * 缓存用于生成entrySet()的对象，此对象将在entrySet()方法第一次调用的时候进行创建。
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * HashMap的键值对数量
     */
    transient int size;

    /**
     * 此HashMap被修改解构的次数。例如删除、增加等。这个字段用于遍历视图的快速失败。
     */
    transient int modCount;

    /**
     * 进行resize阈值
     */
    int threshold;

    /**
     * 负载因子(final 不可变)
     */
    final float loadFactor;

```

### HashMap#put()

```java

	public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

```

HashMap流程图如下：

![HashMap.put()方法流程图](http://img.ylapl.cn/hashMap%20put%E6%96%B9%E6%B3%95%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

其中，有两个方法需要注意下：

1. TreeNode#putTreeVal()，此方法用来在红黑树中增加一个值，具体红黑树算法不在这里赘述。
2. HashMap#treeifyBin(),将链表转换为红黑树，代码如下：

```java

final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }

```

由此可见，当talbe为null或者长度小于MIN_TREEIFY_CAPACITY时，仅仅会进行扩容，并不会转换为红黑树。


### HashMap#resize()

```java

final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

```

![](http://img.ylapl.cn/jdk1.8%20hashMap%E6%89%A9%E5%AE%B9%E4%BE%8B%E5%9B%BE.png)

resize()中，对JDK1.7的resize方法进行了改进，JDK1.7中，resize方法会导致新生成的链表中的元素在原链表中的顺序变为相反的顺序（因为每次插入都是在头部进行插入，读取在头部开始读取），而在jdk1.8中则不会，会保留原有顺序（先生成链表，再把头放到talbe数组中）。由于上述原因，jdk1.7中可能会导致node的死循环，从而带来cpu使用率为100%的问题。该问题在1.8中并不存在（死循环问题不存在），但是仍然可能会存在丢失数据的问题，所以并不是线程安全的。


### 参考文章

[Java 8系列之重新认识HashMap](https://tech.meituan.com/java-hashmap.html)