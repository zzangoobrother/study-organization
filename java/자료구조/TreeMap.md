# TreeMap
TreeMap는 key 값을 기준으로 정렬하여 가지고 있는 자료구조 입니다.
정렬된 순서를 알 수 없는 HashMap과 차이가 있습니다.
HashMap은 key 값에 대해 HashCode함수를 이용하여 값을 bucket 크기로 나눈 나머지를 index로 이용하고,
index가 겹치면 LinkedList를 통해 이어나갑니다.
그렇기 때문에 key의 순서대로 순환을 하지 못합니다.

그러면 TreeMap은 어떻게 순서를 유지할 수 있을까요?

TreeMap은 내부에서 레드-블랙 트리의 자료구조를 이용합니다.
레드-블랙 트리는 이진탐색트리의 문제점을 보완합니다.
이진탐색트리의 문제는 한쪽으로 치우쳐 값을 채우게 되면 기존 O(logN)의 탐색속도가 최악의 경우 O(N)으로 됩니다.
이를 보완하기 위해 레드-블랙 트리가 나옵니다.

레드-블랙 트리은 항상 넣을때, 뺄때 그리고 순회에 대해 모두 O(logN)을 만족합니다.
데이터를 넣을때, 뺄때 Restructuring과 Recoloring으로 추가 프로세스로 진행하여 균형을 맞춥니다.
레드-블랙 트리의 조건은 4가지가 있습니다.
1. Root : 루트노드의 색은 검정이다.
2. External : 모든 external 노드들은 검정이다.
3. Internal : 빨강 노드의 작식은 검정이다.
4. Depth : 모든 리프노드에서 검정 Depth는 같다.

루트부터 리프노드까지의 검정 노드의 갯수는 같다 라는 기조입니다.
레드-블랙 트리의 조건을 맞추면 루트로부터 리프까지의 최소길이와 최대길이의 차이가 2배 이하로 됩니다.

### 상속
````java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
````
특이한 부분만 보면 NavigableMap<K,V>을 implements 받고 있습니다. NavigableMap은 정렬된 Map에서 여러가지 탐색을 제공합니다.

### 내부변수
````java
private final Comparator<? super K> comparator;

private transient Entry<K,V> root;

/**
 * The number of entries in the tree
 */
private transient int size = 0;

/**
 * The number of structural modifications to the tree.
 */
private transient int modCount = 0;
````
정렬을 위해 comparator 변수가 있고, root 이름을 가진 Entry 타입의 변수가 있습니다.

### 생성자
````java
public TreeMap() {
    comparator = null;
}
    
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
````
파라미터 없이 생성자를 생성할 수 있고, 정렬하고 싶은 key를 파라미터로 생성할 수 있습니다.

### put
````java
private V put(K key, V value, boolean replaceOld) {
    Entry<K,V> t = root;
    if (t == null) {
        addEntryToEmptyMap(key, value);
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else {
                V oldValue = t.value;
                if (replaceOld || oldValue == null) {
                    t.value = value;
                }
                return oldValue;
            }
        } while (t != null);
    } else {
        Objects.requireNonNull(key);
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else {
                V oldValue = t.value;
                if (replaceOld || oldValue == null) {
                    t.value = value;
                }
                return oldValue;
            }
        } while (t != null);
    }
    addEntry(key, value, parent, cmp < 0);
    return null;
}

private void addEntryToEmptyMap(K key, V value) {
    compare(key, key); // type (and possibly null) check
    root = new Entry<>(key, value, null);
    size = 1;
    modCount++;
}

private void addEntry(K key, V value, Entry<K, V> parent, boolean addToLeft) {
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (addToLeft)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
}
````
put 메서드는 key 값이 충돌이면 이전 값을 리턴하고 새로운 값을 넣습니다.
처음 put을 한다면 root에 새로운 Entty를 만들어 넣습니다. 
그 다음 key를 넣으면 루트부터 차례대로 비교를 합니다. 부모보다 작다면 왼쪽에 부모보다 크면 오른쪽으로 갑니다.
같을 경우 값을 교체합니다.

그리고 한쪽으로 치우치지 않도록 균형을 잡아줍니다. 그 메서드는 fixAfterInsertion(e) 입니다.

### 삭제
```` java
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // 양쪽 노드 다 있다면 직후 노드와 위치 변경
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    if (replacement != null) {
        // Link replacement to parent
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // Fix replacement
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
        if (p.color == BLACK)
            fixAfterDeletion(p);

        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
````
삭제시 트리의 재조정이 필요합니다.
삭제하려는 노드의 자식이 양쪽에 모두 있다면 삭제 직후 원소와 자리를 교체한 후 삭제합니다.
색상을 건드리지 않고 키만 바뀌는 것으로 트리의 특성에 영향을 미치지 않습니다.
삭제 이후 레드-블랙 트리의 규칙에 위반되지 않도록 재배치합니다.

### 순환
````java
@Override
public void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    int expectedModCount = modCount;
    for (Entry<K, V> e = getFirstEntry(); e != null; e = successor(e)) {
        action.accept(e.key, e.value);

        if (expectedModCount != modCount) {
            throw new ConcurrentModificationException();
        }
    }
}

static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
````
오름차순으로 순환하기 위해 중위순환으로 트리를 탐색합니다.
가장 왼쪽 노드를 먼저 반환, 부모노드, 오른쪽 노드의 순서로 반환합니다.
