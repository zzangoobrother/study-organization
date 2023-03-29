# LinkedList
![캡처1](https://user-images.githubusercontent.com/42162127/228527473-a3434fae-6783-49a9-9b44-8a05eebe09b3.PNG)<br>
ArrayList는 index를 가지고 값을 저장하고 있는 형태입니다. 이런 형태를 가지고 RandomAccess가 가능하고 군집해 있기에 
메모리적으로 이득인 부분이 있습니다.

![캡처2](https://user-images.githubusercontent.com/42162127/228528410-3f84f1d6-071c-4841-b886-b0b8e186300a.PNG)<br>
LinkedList는 node를 가지며 이전 node를 가리키는 prev, 다음 node를 가리키는 next를 가지고 값인 data를 가지고 있습니다.
ArrayList와 달리 앞, 뒤 node를 알고 있고 재정렬이 필요없기 때문에 삽입과 삭제가 빠릅니다.
ArrayList는 add에서 크기의 재산정이 필요하지만 LinkedList는 재산정이 필요없습니다.

### 내부변수
````java
/**
 * Pointer to first node.
 */
transient Node<E> first;

/**
 * Pointer to last node.
 */
transient Node<E> last;
````
내부변수는 firstNode와 lastNode에 값이 있습니다.
다음으로 Node Class를 보겠습니다.
````java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
````
item은 data 값, next는 다음 node, prev는 이전 node 입니다.

### 생성자
````java
/**
 * Constructs an empty list.
 */
public LinkedList() {
}
````
생성자에는 특별한일 없이 기본 생성자를 호출합니다.

### add
````java
/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #addLast}.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}
````
기본적인 add 메소드입니다. 내부 linkLast() 메소드는 마지막 node를 추가하는 메소드입니다.
````java
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
````
새로운 node를 생성, prev는 마지막 node, next는 null로 합니다.
기존 마지막 node의 next는 생성된 node를 바라봅니다.

마지막 node에 추가흐는 메소드가 있다면 제일 처음 node에 추가하는 메소드도 있습니다.
````java
/**
 * Inserts the specified element at the beginning of this list.
 *
 * @param e the element to add
 */
public void addFirst(E e) {
    linkFirst(e);
}

private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
````
node 제일 앞에 추가하는 로직이고, 내부는 linkLast와 동일합니다.

중간 node 삽입 입니다.
````java
/**
 * Inserts the specified element at the specified position in this list.
 * Shifts the element currently at that position (if any) and any
 * subsequent elements to the right (adds one to their indices).
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
````
중간 삽입을 위해 index 범위를 확인합니다.
size와 같다면 linkLast() 메서드로 마지막에 삽입하고 아니면 linkBefore() 메서드를 이용합니다.

````java
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}
````
index가 정상인지 확인합니다. 아니라면 IndexOutOfBoundsException()를 발생시킵니다.

linkBefore(element, node(index)) 메소드에서 node() 메소드를 먼저 보겠습니다.
````java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
````
이진탐색을 이용해 해당 node를 찾습니다.

````java
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
````
삽입할 node와 위 node() 메소드를 통해 찾은 node를 연결합니다. 새로운 node를 생성하며 찾은 node는 next가 되고
찾은 noded의 전 node는 prev가 됩니다. 그리고 size를 올리며 마무리 됩니다.

linkedList의 삽입은 제일 앞, 뒤는 O(1)이고, 중간 삽입은 별도로 index를 찾기 때문에 n/2 시간을 가지며 O(n)이라 할 수 있습니다.

### remove
````java
public E remove() {
    return removeFirst();
}
````
기본 remove() 메서드 입니다. 가장 앞 node를 삭제합니다.

````java
/**
 * Removes and returns the first element from this list.
 *
 * @return the first element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

/**
 * Removes and returns the last element from this list.
 *
 * @return the last element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
````
removeFirst()와 removeLast() 메서드에서는 삭제할 node를 선택, unlinkFirst(), unlinkLast() 메서드를 호출합니다.

```java
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
````
삭제할 node의 prev, next를 null로 합니다. 그래야 GC가 삭제를 합니다.

````java
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
````
삭제할 node의 prev, next를 null로 합니다. 그래야 GC가 삭제를 합니다.

````java
/**
 * Removes the element at the specified position in this list.  Shifts any
 * subsequent elements to the left (subtracts one from their indices).
 * Returns the element that was removed from the list.
 *
 * @param index the index of the element to be removed
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
````
index 기준으로 삭제를 합니다.
먼전 정상 범위의 index인지 판단합니다. 그 후 node() 메서드를 이용하여 index에 해당하는 node를 가져옵니다.
````java
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
````
본인 node를 기준으로 prev, next에 있는 node를 이어주고 본인은 null를 삽입합니다. 그리고 GC에 의해 삭제가 됩니다.

remove도 마찬가지로 가장 앞, 뒤 O(1) 입니다. 그리고 중간 index를 기준으로 삭제를 하면 node를 찾는데 O(n)의 시간이 걸립니다.
