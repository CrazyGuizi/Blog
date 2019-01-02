 `HashMap`是java中的一个容器类，实现了`Map`接口，可以存放键值对，键和值都可以为null（键仅可以有一个null，而值可以有多个null），在声明时传入的泛型必须是对象类型，诸如`int`、`double`等的基本类型是不行的，要转成包装类。它是非同步的，与之相反的是`concurrentHashMap`。
 
 `HashMap`的实现是基于散列表算法的，每个对象都有自己的哈希值，根据哈希值可以快速定位到元素的位置。`Hashmap`结合了数组和链表以及平衡树（红黑树）的方式对元素进行存储。它定义了一个`Node`类，用来存储键和值，同时`Node`有一个`Node`属性的`next`字段，所以可以生成链表。`HashMap`的存储过程是这样的，内部默认初始化一个容量大小为16的`Node`数组，称为哈希桶，插入元素时计算元素的哈希值在根据数组容量大小定位到数组的某个位置。
 
 从上的操作中可以看出有几个问题：
 
 - 为什么默认哈希桶默认初始值为16
 - 如何处理传入的键的哈希值并给出一个数组定位
 - 数组长度总是有限的，存储元素多了无可避免会发生冲突，`HashMap`是如何解决冲突的
 - 当存储元素多到一定时候，怎么扩展空间
 - 既然是非同步的，在多线程环境下可能发生什么情况
 
下图是`HashMap`的成员变量（没特别说明，源码都基于java 8）

![HashMap的属性字段](6B1398102FF54DD1BCD228BC4E68355B)

这是几个比较重要的字段：

- `DEFAULT_INITIAL_CAPACITY` 初始容量，大小为16
- `MAXIMUM_CAPACITY` 允许存储的最大容量
- `DEFAULT_LOAD_FACTOR` 装载因子，和空间重分配有关
- `TREEIFY_THRESHOLD` 链表转化为红黑树的阈值
- `table` 哈希桶，是Node类型的数组

通过`HashMap`的构造函数可以设定哈希桶的初始大小以及装载因子，但最终的大小并不一定是这个值，`HashMap`会根据传入的数进行合理运算，保证大小是2的幂次方。这涉及到根据hash值分配位置的原因，好的做法是根据哈希值做相应的操作得到均匀分散的位置，`HashMap`采用位运算求余，然后再经过扰动操作得出最终结果。

结合源码分析，通过`HashMap(int)`的构造函数可以看到和`tableSizeFor`方法有关。

~~~java
public HashMap(int initialCapacity, float loadFactor) {
        // 初始化大小是负数
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        // 超出最大容量则设为最大容量值                                       
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        // 装载因子是否合理
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        // 设置阈值
        this.threshold = tableSizeFor(initialCapacity);
    }
~~~

~~~java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
~~~

`tableSizeFor`方法执行完后会得到第一个不比传入整数小的符合2的幂次方大小的整数，比如传入3会得到4，传入7会得到8等。方法中一开始`n = cap - 1`是由于如果传入 数本身就是2的幂次方，得出的结果会不正确，所以要先减一。当进行第一次插入操作时，如果哈希桶是空的会进行`resize`操作，在`resize`方法中会设置容量大小为2的幂次方。

![putVal方法](3626CF935BB4482B851C37420EBEB4D1)

`HashMap`的键可能不是整数类型，定位数组坐标需要整数类型，在计算哈希值时利用`Object`的`hashCode`得到这个整数。`HashMap`在定位元素哈希桶位置时做了`(n - 1) & hash`的操作，其目的是求余，通过位运算可以提高效率，而n是2的幂次方可以让`hash`值分配得更均匀。比如A的`hash`值为10101010，B的哈希值为01011110，那么n-1的值是类似于11111全1的，它和`hash`值相与时能保证低位和原来的一样，若不是2的幂次方，则n-1可能会导致`hash`值低位会改变，本来差异挺大的`hash`值也可能碰撞，导致分配不均。此外，官方也利用了扰动函数对碰撞进行了优化。由上，`HashMap`在实现上要求哈希桶的容量大小为2的幂次方。同时，官方也建议在知道`HashMap`需要存储多少元素的情况下最后在构造方法中传入初始化大小。

插入元素根据位置插入到哈希桶，如果此时该位置上为`null`在放进去，如果有则判断是不是键一样，一样则覆盖原来的值，否则生成一个链表，在JAVA 8 中，如果链表长度超过了8，则会构造为红黑树。（在最差的情况下，数组查找为O(1),链表为O(n),而平衡树可达到O(log n)，将链表转化为红黑树可以提高速率）。

~~~java

 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 如果是空表或容量大小为0，则重新分配大小
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 根据哈希定位元素位置，当前位置为null则插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 插入元素的key和存在元素的一样，覆盖原值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果存在对象是红黑树
            // TreeNode继承自LinkedHashMapEntry继承自Node
            else if (p instanceof TreeNode)
            // 红黑树操作
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
            // 遍历链表
                for (int binCount = 0; ; ++binCount) {
                    // 已经是链表尾部，将新元素插入当前元素的后面
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 当链表大小超过红黑树阈值时将链表转为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 链表中找到一样的元素，覆盖之
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 链表的下一个元素
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // afterNodeAccess是空方法，此外还有afterNodeInsertion、afterNodeRemoval也一样
                // 分别意为节点访问后、节点插入后、节点移除，重写方法实现相应的功能
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 如果此时哈希桶大小超过阈值，进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
~~~

当哈希桶元素个数超过阈值时会进行扩容，`threshold`就是阈值，**阈值=装载因子 * 容量大小**，默认初始阈值为`DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY`，即值为6。`HashMap`并不是等哈希桶被装满后才进行扩容，所以设置了一个阈值，若低于这个阈值说明空间利用还充足，若超过则表明插入元素发生碰撞的几率很大了。扩容在`HashMap`的`resize`方法执行。

~~~java
final Node<K,V>[] resize() {
        // 记录原来的哈希桶、容量大小、阈值
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // 原容量大小 > 0
        if (oldCap > 0) {
            // 已经是最大容量了，阈值设为和容量大小一样
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 将容量和阈值扩为原来的1倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // 空表有阈值，将新容量设为原阈值大小
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
        // 空表空阈值，设置默认初始值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 如果新阈值为0，利用新容量大小*装载因子得到新阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 设置当前阈值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            // 声明一个扩容之后的新哈希桶，将原来元素装入
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 原表有值
        if (oldTab != null) {
            // 遍历哈希桶数组
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 数组j只有一个元素，计算新位置后直接放入
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果数组j是一个红黑树
                    else if (e instanceof TreeNode)
                    // 进行红黑树高度降低或直接解散红黑树 ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 将链表重分配，定义低位和高位。低位指原来的位置，高位为扩展后的位置（它为旧位置+原来容量大小）。
                        // 比如原来容量大小为4，存储着0->4->8，1->5,2,3这几个元素
                        // 未扩容时0、4和8根据哈希定位都在第0个（0 ^ 11 = 0，100 ^ 11 = 0,1000 ^ 11 = 0）
                        // 但扩容后4在新表的第5个，即hi = oldCap + j（j表示原来在第几个,100 ^ 111 = 4 > 0，4 + 0 = 4，在新表的tab[4]上，故为第5个）。
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 利用位运算得出如果等于零则给元素处于低位，否则为高位
                            // 表示扩容后处在低位，即例子说的0，8
                            if ((e.hash & oldCap) == 0) {
                                // 如果还是有碰撞，建立新的链
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 表示扩容后在高位
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        // 直至链表重新分完
                        } while ((e = next) != null);
                        // 这里的操作就是上面所说的，低位链表还是在低位，高位的则放置在newTab[j + oldCap]位置上
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
~~~

![HashMap扩容](A4DCCFEF42984F37851CE89F87361F74)
[图片来自](https://blog.csdn.net/zxt0601/article/details/77413921)

`HashMap`不是同步的，在高并发的情况下会导致这样的情况发生：在JDK1.7时，由于扩容后的新数组与原数组的指针引用关系会导致链表出现死循环，在JDK1.8则没有死锁问题，但是会在`put`操作时由于覆盖导致出现数据丢失的情况。

~~~java
static HashMap<Integer, Integer> sHashMap = new HashMap<>();

    public static void main(String[] args) throws InterruptedException {

        new Thread(() -> {
            for (int j = 0; j < 100000; j++) {
                int finalJ = j;
                new Thread(() -> {
                    sHashMap.put(finalJ, finalJ);
                }).start();
            }
        }).start();

        Thread.sleep(5000);
        for (int i = 0; i < 100000; i++) {
            if (sHashMap.get(i) == null)
                System.out.println("value is null");
        }
    }
    // output:
    // value is null
    // ……
~~~

`HashMap`与`Hashtable`、`ConcurrentHashMap`相比缺乏同步操作，`Hashtable`继承自`Dictionary`，在重要方法前加上了`synchronized`前缀，key和value不能为`null`，在定位数组位置时采用除模取余法，插入是先判断之前是否有记录，有则覆盖，否则数组`HashtableEntry`数量加一，此时如果超过阈值会进行扩容。从扩容方法`rehash`中`int newCapacity = (oldCapacity << 1) + 1`可知哈希桶的数量大小为奇数个，默认构造函数设置初始大小为11，`Hashtable`扩容时没有涉及红黑树，只是将链表拆分重新分配位置,采用的是头插法。
~~~java
// 从高位开始
for (int i = oldCapacity ; i-- > 0 ;) {
            for (HashtableEntry<K,V> old = (HashtableEntry<K,V>)oldMap[i] ; old != null ; ) {
                HashtableEntry<K,V> e = old;
                old = old.next;

                int index = (e.hash & 0x7FFFFFFF) % newCapacity;
                // 从这可知是头插法
                e.next = (HashtableEntry<K,V>)newMap[index];
                newMap[index] = e;
            }
        }
~~~

`ConcurrentHashMap`也是同步的，不过没有直接采用`synchronized`，而是使用了CAS，相比于`Hashtable`在性能上更快。多个线程对`Hashtable`哈希桶竞争写操作时会发生阻塞，而`ConcurrentHashMap`则将同步的粒度减小了，在JDK1.7通过对`segments`数组（每一个`segment`相当于一个`HashMap`）中的单个`segment`上锁可以让多个`segment`可以并发执行。