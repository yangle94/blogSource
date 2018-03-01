---
title: ConcurrentHashMap(JDK8)
date: 2018-02-26 16:28:54
tags: [java8,concurrent]
---

### 静态常量

```java

    /**
     * table数组的最大长度
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认table数组长度
     */
    private static final int DEFAULT_CAPACITY = 16;

    /**
     * 最大数组长度，toArray以及相关的方法会用到
     */
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 此表的默认并发级别。JDK8中未使用，但是为了兼容原来的版本，加入了该属性
     */
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

    /**
     * 负载因子。修改这个值仅仅会影响初始容量，实际上不被使用，而是使用了n-n>>2这样的表达式
     */
    private static final float LOAD_FACTOR = 0.75f;

    /**
     * 链表转换为红黑树的阈值，当链表大于等于该值时，会将链表转换为红黑树。该值必须大于2，
     * 并且应该至少有8个
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 当小于这个值的时候，会将红黑树转换为链表
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 这个值代表了当进行链表转红黑树的时候，如果table数组的长度小于此值，会进行扩容操作，
     * 而不是进行红黑树转换工作。通过扩容table数组，减少某个桶内的链表长度，来减少时间。
     * 值应该至少为4 * TREEIFY_THRESHOLD，
     * 以避免在调整大小和设置treeification阈值之间发生冲突。
     */
    static final int MIN_TREEIFY_CAPACITY = 64;

    /**
     * 扩容时单核获得的最低桶数大小
     */
    private static final int MIN_TRANSFER_STRIDE = 16;

    /**
     * 在sizeCtl中用于生成戳记的比特数。32位数组必须至少有6个
     */
    private static int RESIZE_STAMP_BITS = 16;

    /**
     */
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

    /**
     */
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

```

<!-- more -->

### Node 

```java

static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }

```

可以看到，Node类的val跟next属性全部都由关键字volatile修饰，代表获得的值总会是最新的值。其中，setValue()会直接抛出UnsupportedOperationException异常，所以不能通过setValue()修改value的值。find()用于遍历Node数组，找出符合的Node对象。

### cas操作方法

```java

    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        //获取obj对象中offset偏移地址对应的object型field的值
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }

    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        //在obj的offset位置比较object field和期望的值，如果相同则更新。这个方法的操作应该是原子的，因此提供了一种不可中断的方式更新object field。
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }

    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        //设置obj对象中offset偏移地址对应的object型field的值为指定值。
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }

```

### TreeNode

```java

static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, Node<K,V> next,
                 TreeNode<K,V> parent) {
            super(hash, key, val, next);
            this.parent = parent;
        }

        
        Node<K,V> find(int h, Object k) {
            return findTreeNode(h, k, null);
        }

        //查找hash为h，key为k的节点
        final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
            if (k != null) {
                TreeNode<K,V> p = this;
                do  {
                    int ph, dir; K pk; TreeNode<K,V> q;
                    TreeNode<K,V> pl = p.left, pr = p.right;
                    if ((ph = p.hash) > h)
                        p = pl;
                    else if (ph < h)
                        p = pr;
                    else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                        return p;
                    else if (pl == null)
                        p = pr;
                    else if (pr == null)
                        p = pl;
                    else if ((kc != null ||
                            (kc = comparableClassFor(k)) != null) &&
                            (dir = compareComparables(kc, k, pk)) != 0)
                        p = (dir < 0) ? pl : pr;
                    else if ((q = pr.findTreeNode(h, k, kc)) != null)
                        return q;
                    else
                        p = pl;
                } while (p != null);
            }
            return null;
        }
    }

```

当链表超过指定长度的时候，会将其转换为红黑树，所使用的节点包装类就是TreeBin类，但是，需要注意的是，其并不是直接放在table数组中，而是放在TreeBin对象中，再将TreeBin对象放入table数组。

### TreeBin

```java

static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K, V> root;
        volatile TreeNode<K, V> first;
        volatile Thread waiter;
        volatile int lockState;
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock
        
        TreeBin(TreeNode<K, V> b) {
            super(TREEBIN, null, null, null);
            this.first = b;
            TreeNode<K, V> r = null;
            for (TreeNode<K, V> x = b, next; x != null; x = next) {
                next = (TreeNode<K, V>) x.next;
                x.left = x.right = null;
                if (r == null) {
                    x.parent = null;
                    x.red = false;
                    r = x;
                } else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K, V> p = r; ; ) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                (kc = comparableClassFor(k)) == null) ||
                                (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);
                        TreeNode<K, V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            r = balanceInsertion(r, x);
                            break;
                        }
                    }
                }
            }
            this.root = r;
            assert checkInvariants(root);
        }
        
            ……省略其他方法
            
    }

```

可以看到，TreeBin的构造方法其实就是创建了一个红黑树，其中的节点为TreeNode。

### ForwardingNode

```java

static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }

        Node<K,V> find(int h, Object k) {
            // loop to avoid arbitrarily deep recursion on forwarding nodes
            outer: for (Node<K,V>[] tab = nextTable;;) {
                Node<K,V> e; int n;
                if (k == null || tab == null || (n = tab.length) == 0 ||
                        (e = tabAt(tab, (n - 1) & h)) == null)
                    return null;
                for (;;) {
                    int eh; K ek;
                    if ((eh = e.hash) == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                    if (eh < 0) {
                        if (e instanceof ForwardingNode) {
                            tab = ((ForwardingNode<K,V>)e).nextTable;
                            continue outer;
                        }
                        else
                            return e.find(h, k);
                    }
                    if ((e = e.next) == null)
                        return null;
                }
            }
        }
    }

```

此类为一个辅助类，仅仅作用于ConcurrentHashMap扩容操作时。只是一个标志节点，并且指向nextTable，它提供find方法。该类也是继承Node节点，其hash为-1，key、value、next均为null。

### 实例属性

```java

    /**
     * 桶数组，长度总为2的n次方，进行第一次操作时进行初始化。
     */
    transient volatile Node<K,V>[] table;

    /**
     * 使用的下一个table数组，仅仅在调整大小的时候使用。
     */
    private transient volatile Node<K,V>[] nextTable;

    /**
     * ConcurrentHashMap中元素个数,但返回的不一定是当前Map的真实元素个数。基于CAS无锁更
     * 新
     */
    private transient volatile long baseCount;

    /**
     * volatile修饰的int类型，是其他线程可见的。
     * 表初始化和调整大小控制。
     * 当负值时，table数组被初始化或调整大小:-1用于初始化，其它的 -(1 +主动调整大小的线程数)。
     * 否则，当table数组为空时，保存初始表大小以在创建时使用，或默认为0。初始化后，保存下一个元素count值，以调整表的大小。
     */
    private transient volatile int sizeCtl;

    /**
     * 记录下一个切分tables数组的索引
     */
    private transient volatile int transferIndex;

    /**
     * 自旋锁(通过CAS锁定)在调整和/或创建反单元时使用
     */
    private transient volatile int cellsBusy;

    /**
     * 计数器数组。当非空值时，大小是2的幂
     */
    private transient volatile CounterCell[] counterCells;

    // 显示用
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;

```

### put()、putIfAbsent()

```java

	public V put(K key, V value) {
        return putVal(key, value, false);
    }
    
    public V putIfAbsent(K key, V value) {
        return putVal(key, value, true);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
    	
    	//不允许key或者value为null
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        
        //循环进行，如果赋值成功会break退出
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            //如果tab未进行初始化，则进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
                
            //使用tabAt()，查找出改key的hash在table数组
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //使用cas的方式向table数组进行赋值，成功则退出，失败则进入下次循环
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            
            //如果头结点的hash为-1，说明正在扩容,则进入帮助进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                //这里代表当前要插入的值发生撞库，需要对链表或者红黑树进行操作
                V oldVal = null;
                //对f加锁
                synchronized (f) {
                    //先判断当前头节点是不是还是此f节点，如果不是，说明其它线程已经更改了结构，重新进行for循环尝试。
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                //如果key相同并且允许覆盖，则进行覆盖
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                //e = e.next，如果最后一个都不是，则在尾部增加一个
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //如果f是treebin类型
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //如果链表长度不为0并且大于等于TREEIFY_THRESHOLD，则将链表转换为红黑树
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        //size增加1
        addCount(1L, binCount);
        return null;
    }
    
```

按照上面的源码，我们可以确定put整个流程如下：

- 判空；ConcurrentHashMap的key、value都不允许为null
- 计算hash。利用spread()计算hash值。

```java

    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
    
```

- 遍历table，进行节点插入操作，过程如下：

    1. 如果table为空，则表示ConcurrentHashMap未初始化，进行初始化操作：initTable()
    2. 根据hash值获取节点的位置i，若该位置为空，则直接插入（不需要加锁）。计算f位置：i=(n – 1) & hash
    3. 如果检测到fh = f.hash == -1，则f是ForwardingNode节点，表示有其他线程正在进行扩容操作，则帮助线程一起进行扩容操作
    4. 如果f.hash >= 0 表示是链表结构，则遍历链表，如果存在当前key节点则替换value，否则插入到链表尾部。如果f是TreeBin类型节点，则按照红黑树的方法更新或者增加节点
    5. 若链表长度 > TREEIFY_THRESHOLD(默认是8)，则将链表转换为红黑树结构
- 调用addCount方法，ConcurrentHashMap的size + 1

### 初始化table数组

```java
    /**
     * Initializes table, using the size recorded in sizeCtl.
     */
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        //循环判断table数组是否初始化
        while ((tab = table) == null || tab.length == 0) {
        
        	//如果sizeCtl<0(上部分说过，sizeCtl<0代表已经有线程在进行扩容操作)
            if ((sc = sizeCtl) < 0)
            	//让出cpu时间
                Thread.yield(); // lost initialization race; just spin
                
            //通过cas操作，使一个线程进行进入
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    //重新判断tab是否已经初始化，防止极端情况。
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        //sc为n的0.75
                        sc = n - (n >>> 2);
                    }
                } finally {
                		//给sizeCtl设置下一次的阈值（这里用的是直接复制，而没用cas）
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
    
```

### helpTransfer协助转移元素

```java

    /**
     * Helps transfer if a resize is in progress.
     */
    final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }

```

### transfer() 转移方法

```java

    /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        //计算每个线程一次性处理的table数组的长度
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range

        //判断nextTab是否有值，有值则说明已经有线程完成了nextTable的初始化，当然，这里不能排除多个线程同时进入的并发情况（单就方法内部来说），我查看了transfer的所有调用位置，全部都放在一个cas操作下进行，保证了这个nextTab == null只可能有一个线程进行。
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        //扩容后table的长度
        int nextn = nextTab.length;
        //创建ForwardingNode
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        //是否推进
        boolean advance = true;
        //提交nextTable之前确保正确
        boolean finishing = false; // to ensure sweep before committing nextTab

        //i代表当前线程所迁移桶的索引（从大到小逆序）；bound表示已经分配到当前线程的桶的最小的索引
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;

            //更新待迁移的hash桶索引
            while (advance) {
                int nextIndex, nextBound;
                //--i用来更新桶的索引，
                if (--i >= bound || finishing)
                    advance = false;
                //如果未被分配的table数组长度小于等于0
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                //使用cas，更新transferIndex的值（表示待分配任务的table数组长度）
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    //更新成功，将减去后的table数组长度赋值到bound
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }

            //选择最后一次分配
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    //最后一个迁移的线程，recheck后，做收尾工作，然后退出
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    /**
                     第一个扩容的线程，执行transfer方法之前，会设置 sizeCtl = (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)
                     后续帮其扩容的线程，执行transfer方法之前，会设置 sizeCtl = sizeCtl+1
                     每一个退出transfer的方法的线程，退出之前，会设置 sizeCtl = sizeCtl-1
                     那么最后一个线程退出时：
                     必然有sc == (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2)，即 (sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT
                    */
                    
                    //不相等，说明不到最后一个线程，直接退出transfer方法
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    //最后退出的线程要重新check下是否全部迁移完毕
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                //对桶f进行加锁
                synchronized (f) {
                    //判断f是否是当前的桶
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;

                        //fh >= 0代表链表
                        if (fh >= 0) {
                            //通过fh & n先遍历一次node，获得最后一段hash & n相同的链表，这样分离的时候可以直接使用，不需要再次去new。
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }

                            //此段将node列表分为两个链表，算法跟hashmap的分离方法一样。
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        //红黑树迁移
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }

```

### 参考文章

[ConcurrentHashMap源码分析（JDK8） 扩容实现机制](https://www.jianshu.com/p/487d00afe6ca)

[深入分析ConcurrentHashMap1.8的扩容实现](https://www.jianshu.com/p/f6730d5784ad)