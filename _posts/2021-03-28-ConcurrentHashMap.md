---
layout: post
title: ConcurrentHashMap
categories: [Java]
tags: [Java, multi-thread]
description: ConcurrentHashMap 내부 코드 구경하기
fullview: false
comments: true
---

자바에서는 Hash와 관련된 다양한 자료구조를 제공한다. `HashMap`, `HashTable`, `ConcurrentHashMap`등이 있는데, 이번에는 `ConcurrentHashMap`을 자세히 살펴볼 예정이다.   

`ConcurrentHashMap`은 Thread-safe하고 `HashTable`에 비해 성능이 좋은 것은 알고 있다. 하지만 어떤 방식으로 구현을 하였고, 왜 성능이 좋은지에 대해 더 알아보기 위해 내부 코드를 들여다보았다. 

## 생성자

```java
/* ---------------- Public operations -------------- */

    /**
     * Creates a new, empty map with the default initial table size (16).
     */
    public ConcurrentHashMap() {
    }

    /**
     * Creates a new, empty map with an initial table size
     * accommodating the specified number of elements without the need
     * to dynamically resize.
     *
     * @param initialCapacity The implementation performs internal
     * sizing to accommodate this many elements.
     * @throws IllegalArgumentException if the initial capacity of
     * elements is negative
     */
    public ConcurrentHashMap(int initialCapacity) {
        this(initialCapacity, LOAD_FACTOR, 1);
    }

    /**
     * Creates a new map with the same mappings as the given map.
     *
     * @param m the map
     */
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }

    /**
     * Creates a new, empty map with an initial table size based on
     * the given number of elements ({@code initialCapacity}) and
     * initial table density ({@code loadFactor}).
     *
     * @param initialCapacity the initial capacity. The implementation
     * performs internal sizing to accommodate this many elements,
     * given the specified load factor.
     * @param loadFactor the load factor (table density) for
     * establishing the initial table size
     * @throws IllegalArgumentException if the initial capacity of
     * elements is negative or the load factor is nonpositive
     *
     * @since 1.6
     */
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }

    /**
     * Creates a new, empty map with an initial table size based on
     * the given number of elements ({@code initialCapacity}), initial
     * table density ({@code loadFactor}), and number of concurrently
     * updating threads ({@code concurrencyLevel}).
     *
     * @param initialCapacity the initial capacity. The implementation
     * performs internal sizing to accommodate this many elements,
     * given the specified load factor.
     * @param loadFactor the load factor (table density) for
     * establishing the initial table size
     * @param concurrencyLevel the estimated number of concurrently
     * updating threads. The implementation may use this value as
     * a sizing hint.
     * @throws IllegalArgumentException if the initial capacity is
     * negative or the load factor or concurrencyLevel are
     * nonpositive
     */
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // concurrencyLevel이 초기 capacity보다 클 경우 초기 capacity 값을 concurrencyLevel로 셋팅한다.
            initialCapacity = concurrencyLevel; 
        long size = (long)(1.0 + (long)initialCapacity / loadFactor); // 여기서 테이블의 실제 크기를 loadFactor와 연관지어 설정, HashTable의 크기는 2의 제곱으로 계산된다.
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }

```

다양한 형태의 생성자를 제공한다. 이 생성자에서는 초기 해시테이블(associate array) 사이즈를 결정하는데, 지정해주지 않으면 `DEFAULT_CAPACITY`는 `16`이다. 생성자에서 직접 해시테이블을 생성하지 않고, 첫 노드가 삽입될 때 생성된다(lazily initialized).    
생성자에서 제공하는 파라미터는 아래와 같다. 

* `int initialCapacity` : 초기 해시테이블 크기를 결정한다
* `float loadFactor` : `HashMap`에서 사용되는 부하계수와 동일하다. 여기서는 초기 테이블의 크기를 설정하기 위한 용도로만 사용된다.`HashMap`에서는 이 값에 따라 해시테이블의 resize되는 시점이 결정되나, `ConcurrentHashMap`은 이 인자 값과 상관없이 0.75f로 동작한다.
* `int concurrencyLevel` : 예상되는 스레드수로, 초기화 할 때 테이블 크기를 지정하는데에만 사용된다.


<br/>
<br/>

### 동기화 처리 방식
`ConcurrentHashMap`내부 코드를 보면 알 수 있겠지만, 메소드에 `synchronized`를 선언하여 동기화 처리를 하지 않았다. 그리고 `Hashtable`에서는 단순 조회성 메소드에도 `synchronized`를 메소드에 걸었는데, `ConcurrentHashMap`은 수정과 관련된 메소드에만 메소드 내부 특정 블록에 `synchronized`를 걸어 thread-safe를 보장하였다. 여기서는 가장 주요 메소드인 `put()`에 대해 살펴본다.  

```java
    /**
     * Maps the specified key to the specified value in this table.
     * Neither the key nor the value can be null.
     *
     * <p>The value can be retrieved by calling the {@code get} method
     * with a key that is equal to the original key.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with {@code key}, or
     *         {@code null} if there was no mapping for {@code key}
     * @throws NullPointerException if the specified key or value is null
     */
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh; K fk; V fv;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else if (onlyIfAbsent // check first node without acquiring lock
                     && fh == hash
                     && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                     && (fv = f.val) != null)
                return fv;
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value);
                                    break;
                                }
                            }
                        }
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
                        else if (f instanceof ReservationNode)
                            throw new IllegalStateException("Recursive update");
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

`put()`메소드는 실질적으로 `putVal()`을 호출하는데, 여기서는  
1. 빈 해시 버킷에 노드를 삽입하는 경우  
2. 노드가 존재하는 경우  
를 다르게 처리하여 성능을 개선하였다. 

#### 빈 해시 버킷에 노드를 삽입하는 경우
여기서는 `lock`을 걸지 않고 **Compare and Swap**을 이용하여 새로운 노드에 해시 버킷을 삽입하였다.

#### 노드가 존재하는 경우
`synchronized(노드가 존재하는 해시 버킷 객체, Node<K,V>타입)`를 이용해 하나의 스레드만 접근할 수 있도록 제어한다. 서로 다른 스레드가 같은 해시 버킷에 접근할 때만 해당 블록이 잠기게 된다.


### 가변 배열 리사이징
`HashMap`에서 리사이징은 단순히 `resize()` 함수를 통해 새로운 배열을 만들어 복사하였다.  
`ConcurrentHashMap`에서는 기존 배열(table)에서 새로운 배열(nextTable)로 버킷을 하나씩 전송(transfer)하는 방식이다. 이 과정에서 다른 스레드가 버킷 전송에 참여 할 수 있다. 전송이 모두 끝나면 크기가 기존의 2배인 nextTable이 새로운 배열이 된다.
`SIZECTL`과 `resizeStamp()`메소드를 통해 resize() 과정이 중복으로 일어나지 않도록 방지한다.



```java
if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                if (sc < 0) {
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSetInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
```


### 버킷의 Seperate Chaining <-> Tree 변환 구간
`HashMap`과 동일하게, 버킷에 8개의 키-값 쌍이 모이면 링크드리스트를 트리로 변경하고, 데이터를 삭제하여 개수가 6개에 이르면 다시 링크드리스트로 변경한다.  

```java

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2, and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;
```


### 트리의 대소판단
`HashMap`과 `ConcurrentHashMap`에서 사용되는 트리는 `Red-Black Tree`이다. 여기서 사용하는 대소 판단 기준은 해시 함수 값이다. 해시 값을 대소 판단 기준으로 하면 Total Ordering에 문제가 생길 수 있어 `tieBreakOrder()`메소드로 충돌문제를 해결하였다. 

```java

        /**
         * Tie-breaking utility for ordering insertions when equal
         * hashCodes and non-comparable. We don't require a total
         * order, just a consistent insertion rule to maintain
         * equivalence across rebalancings. Tie-breaking further than
         * necessary simplifies testing a bit.
         */
        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }
```

위 코드에서 보면 알 수 있듯이 `System.identifyHashCode()`를 이용하였다.

### 추가 참고자료
java 8 이전의 `ConcurrentHashMap`은 `ReentrantLock`을 상속받는 `Segment`를 이용하여 동기화를 구현했다. 이 때 영역을 구분하여(기본 16개) 영역 별로 Lock을 하는 방식을 차용했다. 각 영역에는 리사이징이 가능한 해시 테이블을 가졌다.  
하지만 java 8에서는 각 테이블 버킷을 독립적으로 잠근다. 빈 버킷의 노드 삽입은 lock을 사용하지 않고 단순히 CAS만을 이용해 삽입한다. 그 외 업데이트 작업은 lock(synchronized)를 이용하지만, 각 버킷의 첫 번째 노드를 기준으로 lock을 획득하여 쓰레드 경합을 최소화 하고 안전한 업데이트를 제공한다. 

***
참고 자료 : 
1. [Java HashMap은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311)  
2. [[java]ConcurrentHashMap 동기화 방식](https://pplenty.tistory.com/17)

