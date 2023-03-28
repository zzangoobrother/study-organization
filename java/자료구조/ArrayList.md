# ArraysList
ArrayList는 배열을 편하게 쓸수있도록 제공해주는 자료구조입니다. 일반 배열과 다르게 메모리가 가능한 추가할 수 있고, 
삭제도 해당 index를 비워두는게 아니고 재정렬해주는 기능을 제공합니다.

### 상속 확인
````java
public class ArrayList<E> extends AbstractListe<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
````
AbstractList를 extends 받고, List, RandomAccess, Cloneable, Serializable를 implements 받습니다.
RandomAccess는 index를 통해 직접 접근 할수 있는 자료구조라는 의미입니다.

````java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access
````
ArrayList에서 사용하는 배열입니다. 주석을 보면 처음 add 메소드를 사용할 때 배열 크기가 정해집니다.

### 생성자
````java
/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
````
생성자를 보면 elementData에 DEFAULTCAPACITY_EMPTY_ELEMENTDATA를 대입합니다.
아래 DEFAULTCAPACITY_EMPTY_ELEMENTDATA를 보면 빈 Array 배열 입니다.

````java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
````

다른 생성자입니다.
````java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
````
파라미터로 넘겨진 int값으로 배열을 초기화 하는 모습을 볼 수 있습니다.

### add
````java
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}

private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
````
제일 마지막 값을 하나 추가합니다. 중간에 값을 넣고 싶으면 add(index, Object)를 사용합니다.
배열의 길이와 지금까지 넣은 데이터의 길이를 비교하여 같으면 배열를 재산정 후 배열 마지막에 값을 추가합니다.
````java
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}

private Object[] grow() {
    return grow(size + 1);
}
````

newCapacity()를 보겠습니다.
````java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
````
로직을 보면 int newCapacity = oldCapacity + (oldCapacity >> 1) 로 배열의 크기를 계산합니다.
기존 크기에서 50%의 크기를 더해 새로운 크기를 산정합니다. 
그리고 만약 minCapacity보다 작다면 minCapacity 값으로 재산정합니다.

이렇게 add 메서드가 호출될 때 크기를 재산정, 실제 데이터가 들어가 있는 크기인 size index에 값을 넣은 후 1증가 시킵니다.
크기를 재산정 할때는 원래크기만큼 새로운 배열을 복하하므로 시간복잡도 O(n)을 가지며, 뒤에 추가만 할 때는 O(1)를 가집니다.

### remove
remove에는 index 기준 삭제와 value에 대한 삭제가 있습니다.
````java
/**
 * Removes the element at the specified position in this list.
 * Shifts any subsequent elements to the left (subtracts one from their
 * indices).
 *
 * @param index the index of the element to be removed
 * @return the element that was removed from the list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    Objects.checkIndex(index, size);
    final Object[] es = elementData;

    @SuppressWarnings("unchecked") E oldValue = (E) es[index];
    fastRemove(es, index);

    return oldValue;
}

/**
 * Removes the first occurrence of the specified element from this list,
 * if it is present.  If the list does not contain the element, it is
 * unchanged.  More formally, removes the element with the lowest index
 * {@code i} such that
 * {@code Objects.equals(o, get(i))}
 * (if such an element exists).  Returns {@code true} if this list
 * contained the specified element (or equivalently, if this list
 * changed as a result of the call).
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if this list contained the specified element
 */
public boolean remove(Object o) {
    final Object[] es = elementData;
    final int size = this.size;
    int i = 0;
    found: {
        if (o == null) {
            for (; i < size; i++)
                if (es[i] == null)
                    break found;
        } else {
            for (; i < size; i++)
                if (o.equals(es[i]))
                    break found;
        }
        return false;
    }
    fastRemove(es, i);
    return true;
}
````
index를 이용한 삭제는 복사한 배열과 해당 index를 fastRemove(es, index) 메소드에 전달하고, value는 해당 value를 찾고
없으면 false를 return 있다면 복사된 배열과 해당 value가 있는 index를 fastRemove(es, index) 메소드에 전달합니다.

````java
private void fastRemove(Object[] es, int i) {
    modCount++;
    final int newSize;
    if ((newSize = size - 1) > i)
        System.arraycopy(es, i + 1, es, i, newSize - i);
    es[size = newSize] = null;
}
````
arraycopy를 이용해 삭제되는 부분 + 1 ~ 마지막까지의 영역을 삭제되는 부분의 시작점을 기준으로 옮깁니다. 
그러면 삭제될 부분의 값은 다음 index의 값으로 겹쳐서 덮여 쓰여집니다.
그리고 size의 마지막 index는 size - 1의 값과 중복되기에 null 처리합니다. GC가 삭제할 수 있습니다.
그리고 삭제된 값을 리턴합니다.

삭제 할때 우리는 index + 1에서 부터 값을 index부터 시작하게끔 복사합니다. 그때 O(n)의 시간복잡도를 가지며 마지막에 null로 변경합니다.
이때는 O(1)의 시간복잡도를 가집니다.
삭제에 대해서는 항상 O(n)을 가진다고 볼 수 있습니다.
