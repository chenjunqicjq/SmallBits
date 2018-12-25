---
title: HashMap分析
date: 2018-04-06 18:56:55
tags:
---
//继承了AbstractMap类，实现了Map,Cloneable,Serializable类
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    private static final long serialVersionUID = 362498820763181265L;
    //初始化容量默认16，且必须为2的倍数
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    //最大容量为2的30 次方，如果传入的数大于这个数，还是被这个数代替
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //加载因子，默认0.75，可以更改
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //链表转红黑树的临界值
    static final int TREEIFY_THRESHOLD = 8;
	//红黑树转为链表的临界值
    static final int UNTREEIFY_THRESHOLD = 6;
    //桶可能转为树形结构的最小容量的临界值
    static final int MIN_TREEIFY_CAPACITY = 64;

   
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

    /* ---------------- Static utilities -------------- */

    
    static final int hash(Object key) {
        //h=key.hashCode取哈希值
		//h^(h>>>16) 高位运算
		int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

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

    /**
     * Returns k.compareTo(x) if x matches kc (k's screened comparable
     * class), else 0.
     */
    @SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
    static int compareComparables(Class<?> kc, Object k, Object x) {
        return (x == null || x.getClass() != kc ? 0 :
                ((Comparable)k).compareTo(x));
    }
	 //判断哈希数组的大小，只能是2 的幂，如传入13，则返回16，
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

    /* ---------------- Fields -------------- */
    //哈希表的链表数组
    transient Node<K,V>[] table;
    transient Set<Map.Entry<K,V>> entrySet;

    transient int size;
     //记录HashMap改变的次数
    transient int modCount;
     //阈值，判断是否需要调整HashMap的大小，=长度*加载因子
    int threshold;
     //实际加载因子
    final float loadFactor;
    //指定“容量大小”和“负载因子”的构造器
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //最大容量只能是2的30次方
		if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
    //指定“容量大小”的构造器
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
    //默认构造器
    public HashMap() {
	//设置加载因子
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
   //带“子map”的构造器
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }

    //先哈希表中添加整个集合
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
		  //如果数组为空，则初始化参数
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
			//不为空，且超过了阈值，扩容
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
				//先计算哈希值
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

    
    public int size() {
        return size;
    }

   
    public boolean isEmpty() {
        return size == 0;
    }

   //获取key对应的value值
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    //HashMap将“key为null”的元素都放在table的位置0处，即table[0]中；“key不为null”的放在table的其余位置！
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; 
		Node<K,V> first, e;
		int n; 
		K k;
		//如果table不为空，长度不为零，该hash值对应的数组的下标的元素不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
			//判断第一个结点是否是目标结点，实则返回
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
			//判断结点是否有下一个
            if ((e = first.next) != null) {
			    //判断是否是红黑树，是则用getTreeNode 方法搜索
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
				//如果是链表结果，遍历链表，判断结点是否目标结点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
		//不符合条件结点，返回null;
        return null;
    }

   //hashmap是否包含key
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
    //添加
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab;
		Node<K,V> p; 
		int n, i;
		//判断是否为空，空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
		//判断该值的hash值对应的数组的下标位置是否为空，为空，则把值直接放入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
		//说明该位置已经有值
        else {
            Node<K,V> e; K k;
			//判断头节点是否与传进来的值相等，若相等，e指向链表
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
			//不相等判断是否为树，是则进去
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //则是链表
			else {
                for (int binCount = 0; ; ++binCount) {
                    //当最后一个结点为空，创新节点
					if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                       //判断新加结点后是否转变为树结构
					   if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
					//如果遍历中有结点与传的值相同，跳出遍历
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
			//说明有相同的值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
				//onlyIfAbsent默认为false,新值把旧值覆盖
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
		//结构改变，数值加1
        ++modCount;
		判断现在的大小是否超过阈值，超过则扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
    
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
           //如果大于最大值，阈值改为最大值
		   if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
			//扩容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
		//老的长度小于0且老的阈值大于0
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
		//都小，初始化，
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
		//计算新的resize上限
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
		//把每个桶都移动到新的里面
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
					//直接在末尾加一个结点
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
					//判断是否为树结构，
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    //链表优化
					else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
							//情况1：若结果为0，则挂在lo链上
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
							//情况2：挂在hi链上
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
						//把lo链挂到原来的桶里
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
						//hi链挂在j+老的桶长度
						//扩容后计算hash值的结果也会跟着改变
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
     
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
		//链表都改为数
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
				//头节点不为空时
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
			//将桶中的元素与树的头节点连接起来
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }

    
    public void putAll(Map<? extends K, ? extends V> m) {
        putMapEntries(m, true);
    }
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab;
		Node<K,V> p;
		int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
           //node存储找到要删除的hash值
		   Node<K,V> node = null, e; K k; V v;
            //如果头节点等于要删除的值
			if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                //判断是否为树结构，
				if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
				//链表则循环找，找到则跳出循环
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                //删除头节点
				else if (node == p)
                    tab[index] = node.next;
                //删除中间结点
				else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
    //清除桶
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
  //通过值V来查找集合中是否有这个值
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
常用的几个方法完了，后面的有些复杂，等以后在来补回来	
	
	
	
	
	
	
	
	
	