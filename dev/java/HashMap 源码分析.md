### HashMap中容量扩充的倍数是2的幂次方
- ####  通过`tableSizeFor`方法，使容量扩充的倍数是2的幂次方
```java
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
    此算法的目的是找到大于当前容量的2的幂次的最小值。`>>>`表示无符号的右移。
    
    思路：一个数的二进制，其最高位(除去符号位)必然是1（假设其余位数为n），那么最接近的2次幂数在2^(n+1)和2^n之间。如十进制数101的二进制表示：`110 0101`，与这个数想接近的2次幂数必然在`1000 0000(128)`和`100 0000(64)`之间产生。`tableSizeFor`方法就是在通过按位或等（|=）和右移（>>>），使得n的二进制位等于1111……11，那么 n+1则为一个2的幂数。

- ####  哈希运算时可以用位运算代替模运算，提高运算效率
```java
if ((first = tab[i = (n - 1) & hash]) != null)
```

计算机在对除法和模运算的处理是比较慢的，因此我们可以将模运算转换成位运算：`a % b == (b - 1) & a`，当b是2的指数时等式成立。

例如：
```
a = 1395272， b = 16
1111 & 101010100101001001000 = 1000(8)
a = 14523, b = 16
1111 & 11100010111011 = 1011(11)
```

### hash()方法
```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
这个函数是扰动函数，将key hash值的高位和低位进行异或，加大了低位的随机性。
[具体参考知乎胖君的回答](https://www.zhihu.com/question/20733617)

### HashMap 扩容
```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {   // 原来有容量
            if (oldCap >= MAXIMUM_CAPACITY) {   // 扩容前容量大于等于最大容量，阈值赋值为Integer的最大值，返回原来容量
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)    // 新容量小于最大容量并且旧容量大于等于初始容量(16)，更新阈值为原来2倍
                newThr = oldThr << 1;
        }
        else if (oldThr > 0) // 对应初始构造 HashMap(int initialCapacity) 和 HashMap(int initialCapacity, float loadFactor)情况，将新容量赋值为初始阈值
            newCap = oldThr;
        else {               // 对应无参构造情况。为容量和阈值附初始值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {  // 对应上面 oldThr > 0的情况，设置新阈值
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {   // 将oldTab数据放到newTab中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null) // oldTab元素中没有数据
                        newTab[e.hash & (newCap - 1)] = e;  // 将newTab中对应位置元素置空
                    else if (e instanceof TreeNode) // 如果oldTab每个元素存放着红黑树（存放一条长度大于8的链表将会自动转变为红黑树），则按照红黑树的方法进行赋值
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // oldTab中每个元素存放的链表长度小于8
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {   // e.hash & oldCap 不太理解
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
`e.hash & oldCap` 不太理解

### HashMap 链表转换成红黑树
#### TreeNode结构
```java
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
    }
```
除此之外，还有继承自`LinkedHashMap.Entry`的
```java
Entry<K,V> before, after;
```
继承自`HashMap.Node`的
```java
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
```

#### 关于红黑树的重要参数
```java
static final int TREEIFY_THRESHOLD = 8;
```
当链表长度超过8时会将链表节点转换成红黑树节点

```java
static final int UNTREEIFY_THRESHOLD = 6;
```
当小于6时会将红黑树节点转换成链表节点

```java
static final int MIN_TREEIFY_CAPACITY = 64;
```
容器可以树化的最小容量。当hash表中的容量大于这个值，会将表中的桶进行树形化；否则桶内的元素过多时会扩容，而不是树形化

#### 树形化
```java
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY) // 如果hashTable为空或表长度小于最小树形化容量(64)，则新建/扩容
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {   // 找到需要树形化的桶
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null); // 将链表节点转换成红黑树节点
                if (tl == null)
                    hd = p; // 头结点
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)  // 让桶的第一个元素指向新建的红黑树头结点
                hd.treeify(tab);    // 构造红黑树
        }
    }

    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        return new TreeNode<>(p.hash, p.key, p.value, next);
    }
```

构造红黑树在 `TreeNode`类中
```java
    final void treeify(Node<K,V>[] tab) {
        TreeNode<K,V> root = null;
        for (TreeNode<K,V> x = this, next; x != null; x = next) {
            next = (TreeNode<K,V>)x.next;
            x.left = x.right = null;
            if (root == null) {
                x.parent = null;
                x.red = false;
                root = x;
            }
            else {
                K k = x.key;
                int h = x.hash;
                Class<?> kc = null;
                for (TreeNode<K,V> p = root;;) {
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

                    TreeNode<K,V> xp = p;
                    if ((p = (dir <= 0) ? p.left : p.right) == null) {
                        x.parent = xp;
                        if (dir <= 0)
                            xp.left = x;
                        else
                            xp.right = x;
                        root = balanceInsertion(root, x);
                        break;
                    }
                }
            }
        }
        moveRootToFront(tab, root);
    }

    static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
        int n;
        if (root != null && tab != null && (n = tab.length) > 0) {
            int index = (n - 1) & root.hash;
            TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
            if (root != first) {
                Node<K,V> rn;
                tab[index] = root;
                TreeNode<K,V> rp = root.prev;
                if ((rn = root.next) != null)
                    ((TreeNode<K,V>)rn).prev = rp;
                if (rp != null)
                    rp.next = rn;
                if (first != null)
                    first.prev = root;
                root.next = first;
                root.prev = null;
            }
            assert checkInvariants(root);
        }
    }
```
