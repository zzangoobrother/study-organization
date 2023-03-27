# HashMap
HashMap은 Key, Value를 저장하는 구현체이다. HashMap는 Key를 Hashing하여 저장하며 빠르게 처리한다.
입력과 삭제에 대해 시간복잡도가 O(1)인 자료구조이다.

### 초기화
HashMap를 사용한다면 가장 먼저 초기화 합니다.
아래와 같이 초기화 할 것입니다.

````java
Map<String, String> map = new HashMap<>();
````

HashMap 생성자를 보겠습니다.

````java
    /**
     * Constructs an empty {@code HashMap} with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
````

내용을 살펴보겠습니다. loadFactor에 DEFAULT_LOAD_FACTOR(0.75f)를 주입합니다. 다음으로 put 메소드를 보겠습니다.

### put

````java
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with {@code key}, or
     *         {@code null} if there was no mapping for {@code key}.
     *         (A {@code null} return can also indicate that the map
     *         previously associated {@code null} with {@code key}.)
     */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */

````

put 메소드에서는 putVal 이라는 메소드를 호출합니다. 우선 hash 메소드를 보겠습니다.

````java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
````

key 값이 null이 아니면 key의 Object 클래스에 있는 hashcode()를 실행하고 리턴값인 int를 얻어 16번 오른쪽으로 unsigned right shift한
값을 XOR을 하여 hash값을 얻습니다.
이렇게 얻은 값을 putVal() 메소드 파라미터로 전달해 실행합니다.

````java
    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
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
````

putVal의 로직을 하나씩 따라가보자
````java
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
````

멤버변수 Node<K,V>[] table에 값이 있는지 확인합니다. table은 한 번이라도 put을 했다면 null이 아닙니다.
resize()를 보겠습니다.
````java
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
````
위에서 확인한 table 변수가 이미 초기화 되어있는지 아닌지 확인 후 
````java
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
````
위와 같은 배열을 생성합니다. 그리고 기존 배열이 있다면 index의 위치를 재조정 합니다.
소스를 전체적으로 보면 tail, next 같은 단어들이 많이 나옵니다. 그리고 전체적인 로직을들 보면 보일겁니다.
사실 HashMap는 LinkedList배열 입니다.

resize() 후 (n - 1) & hash에 index가 없다면 아래와 같이 새로운 노드를 할당 합니다.
````java
if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
````
아래와 같이 새로운 Node를 만드어 할당 합니다.
````java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
        return new Node<>(hash, key, value, next);
}
````
Node의 구성 입니다.
````java
static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  V value;
  Node<K,V> next;
  ...
}
````
만약 위에서 구한 index가 null이 아니라면
````java
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
````
LinkedList의 마지막에 추가합니다.
소스를 같이 보겠습니다.
hash값이 실제 값과 같다면 기존 값을 교체합니다.
````java
if (p.hash == hash &&
  ((k = p.key) == key || (key != null && key.equals(k))))
  e = p;
````
아니라면 다음 로직을 실행합니다. TreeNode 인지 판단하고 그렇다면 TreeNode에 값을 넣고 아니라면 마지막 else를 실행합니다.
node에 새로운 node를 넣습니다.
````java
p.next = newNode(hash, key, value, null);
````
HashMap의 put의 시간복잡도가 1을 가지는 이유는 hashing을 배열의 index값으로 하기 때문입니다.

### remove
key값에 대해 탐색하여 해당 key의 value가 존재한다면 삭제하며, 존재하지 않는다면 null값을 반환한다.
````java
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
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
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
            else if (node == p)
                tab[index] = node.next;
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
````

removeNode()를 보겠습니다. 해당 table의 index에 값이 있는지 없는지 확인하여 index가 있다면 로직을 수행합니다.
````java
if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null)
````
첫번째 index의 node가 true인지 확인합니다. 아니라면 내부를 순회하며 true인 경우가 있을 때까지 순회합니다.
````java
if (p.hash == hash &&
    ((k = p.key) == key || (key != null && key.equals(k))))
    node = p;
else if ((e = p.next) != null) {
    if (p instanceof TreeNode)
        node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
    else {
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
````

node에 값이 있으면 지울 node가 존재하고, 없다면 null을 바로 return 합니다.
````java
if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
    if (node instanceof TreeNode)
        ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
    else if (node == p)
        tab[index] = node.next;
    else
        p.next = node.next;
    ++modCount;
    --size;
    afterNodeRemoval(node);
    return node;
}
````
### 정리
- HashMap는 Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap] 로 되어있다.
- put을 하면 Node<K,V>[hashing 값 & 배열 크기 - 1]에 값이 들어가고, key가 같으면 LinkedList 자료구조로 이어 붙인다.
- remove를 하면 Node<K,V>[hashing 값 & 배열 크기 - 1]에 값이 있는지 확인 후 key 값을 비교후 가져오거나 null을 반환한다.
- 많은 put로 인해 loadFactor를 넘으면 배열의 크기를 재산정한다.(현재 크기 2배)
- 하나의 index에 들어가는 값이 많아지면 tree의 형태로 변환합니다.(java 8 이상)
- 시간복잡도가 O(1)인 이유는 key값을 (hashing & n - 1)으로 직접 index를 가져오기 때문이다.
![1_w1mRVHC1hNc2ywDoYibkiA](https://user-images.githubusercontent.com/42162127/227996822-cd2b499d-0b3e-4aa8-af00-4e0e87ceee54.png)
