# LinkedHashMap
LinkedHashMap을 이해하기 위해 우선 HashMap에 대한 이해가 필요합니다. HashMap에 대하 쓴 글이 있으니 확인해주세요.

### HashMap은 어떤 순서로 읽을까?
HashMap은 node를 꺼낼 때 순서를 보장하지 않습니다. TreeMap과 같은 Map은 순서를 보장하지만 HashMap은 보장하지 않습니다.
````java
public final void forEach(Consumer<? super K> action) {
    Node<K,V>[] tab;
    if (action == null)
        throw new NullPointerException();
    if (size > 0 && (tab = table) != null) {
        int mc = modCount;
        for (Node<K,V> e : tab) {
            for (; e != null; e = e.next)
                action.accept(e.key);
        }
        if (modCount != mc)
            throw new ConcurrentModificationException();
    }
}
````
HashMap의 keySet의 forEach입니다. 단순하게 0부터 순회를 합니다. 이렇게 순회하면 hash값이 낮은 순서로 순회합니다.
하지만 데이터가 늘어나면 loadfactor를 넘는 순간 hashcode의 재분배가 일어나면서 순서는 뒤바뀝니다. 순서를 보장할 수 없습니다.

이제 LinkedHashMap을 보겠습니다. LinkedHashMap는 기존 HashMap에서 LinkedList를 섞어놓은 자료구조입니다.
이렇게 2개를 합친 자료구조는 for, foreach를 통해 탐색을 할때 Map에 넣은 순서를 유지할 수 있습니다.

### 상속
LinkedHashMap은 HashMap을 상속받고 있습니다. 그리고 head와 tail, before, after 변수가 추가되어있습니다.
````java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
  static class Entry<K,V> extends HashMap.Node<K,V> {
      Entry<K,V> before, after;
      Entry(int hash, K key, V value, Node<K,V> next) {
          super(hash, key, value, next);
      }
  }
  
  transient LinkedHashMap.Entry<K,V> head;
  
  transient LinkedHashMap.Entry<K,V> tail;
}
````
![캡처3](https://user-images.githubusercontent.com/42162127/229112739-4f5ec713-f919-4bf3-83c7-92b86eb11d48.PNG)

### put
LinkedHashMap는 HashMap의 put 메서드를 사용합니다. put 내부의 putVal 메서드 사용을 볼 수 있습니다.
````java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) // 처음 값 입력
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) // key 중복 없을 때 새로운 노드 생성
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
중요 포인트를 하나씩 보겠습니다.
우선 newNode() 입니다.
````java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
````
newNode() 메서드를 보면 새로운 node를 생성합니다. 그리고 linkNodeLast() 메서드를 통해 기존 마지막 node에 추가합니다.

다음으로 만약 key의 중복이 존재한다면 afterNodeAccess() 메서드를 실행합니다. 기존 node를 향하던 link를 새로운 node로 향하도록 합니다.
````java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
````

### 삭제
삭제 또한 HashMap의 remove를 이용합니다. 대부분의 코드는 HashMap에 남아있지만 afterNodeRemoval() 메서드만 LinkedHashMap 클래스에
있습니다.
````java
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
````
afterNodeRemoval() 메서드는 삭제할 node의 before, after의 link를 끈습니다.

### 순환
LinkedHashMap이 어떻게 순회할 때 순서를 보장하는지 보겠습니다. 아래가 forEach 코드입니다.
보시면 head에서 시작해서 after 까지 null일 때 까지 반복합니다. HashMap이 link로 연결되어 있기에 HashMap처럼 재산정은 되지 않습니다.

````java
public final void forEach(Consumer<? super K> action) {
    if (action == null)
        throw new NullPointerException();
    int mc = modCount;
    for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
        action.accept(e.key);
    if (modCount != mc)
        throw new ConcurrentModificationException();
}
````

