# PriorityQueue
PriorityQueue는 Heap 자료구조 기반으로 되어 있습니다.

### Heap 자료구조
최소값과 최대값을 빠르게 찾기위한 완전 이진 트리를 사용합니다.
최소힙은 root 노드값이 제일 작으며, 부모노드는 자식 노드보다 값이 작음을 항상 만족합니다.
최대힙은 root 노드값이 제일 크며, 부모노드는 자식 노드보다 값이 큼을 항상 만족합니다.

### Heap 삽입
1. 가장 밑단의 노드에 삽입합니다.
![20230414](https://user-images.githubusercontent.com/42162127/232048269-7d158d84-4af8-47b4-aaf8-cf079499744a.PNG)

2. 부모노드와 비교해서 자식노드가 크다면 교환, 작다면 해당 부분에서 정지합니다.
![20230414](https://user-images.githubusercontent.com/42162127/232048434-1daa9218-dfc0-4880-b55e-66b308358862.PNG)

3. 다시 부모노드와 비교, 크다면 교환, 작다면 해당 부분에서 정지합니다.
![202304141](https://user-images.githubusercontent.com/42162127/232048739-6eb4f970-f70e-4ee6-b513-93c94a363d5b.PNG)

4. 다시 큰 값을 가진 자식과 위치를 교환 합니다. 최대힙 구조가 유지되면서 완성입니다.
![202304142](https://user-images.githubusercontent.com/42162127/232049183-d67c379e-a879-4841-aecf-f50b87bfad47.PNG)

### Heap 삭제
최대힙에서 최대값은 root 노드이므로 root 노드가 삭제된다. 그리고 삭제된 root 노드에는 Heap의 마지막 노드를 가져옵니다.
![202304143](https://user-images.githubusercontent.com/42162127/232089049-f4f73401-acf9-419b-b15d-371704b0c5ce.PNG)

![202304144](https://user-images.githubusercontent.com/42162127/232089150-e083b1c9-5365-442f-9b92-a54cf546efac.PNG)
그리고 교환한 노드에서 큰 값을 가진 자식과 위치를 마지막까지 교환 합니다.

![202304145](https://user-images.githubusercontent.com/42162127/232089512-53540e9c-9cc8-4433-a318-10f9daf0e82a.PNG)

![202304146](https://user-images.githubusercontent.com/42162127/232089525-c9de8404-68e7-4d25-9f80-77a32bbfce5f.PNG)

````java
// 내림차순
Queue<Integer> queue = new PriorityQueue<>();

// 오름차순
Queue<Integer> queue = new PriorityQueue<>(Collections.reverseOrder());
````

### 클래스
PriorityQueue의 상속 구조입니다. AbstractQueue를 상속받고 있고, AbstractQueue는 인터페이스 Queue와 추상 클래스
AbstractCollection을 상속 받습니다.
````java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {
````

#### 변수
````java
균형잡힌 이진 힙으로 표현되는 우선 순위 큐: 
큐[n]의 두 자식은 큐[2*n+1] 및 큐[2*(n+1)입니다. 
우선순위 대기열은 비교기 또는 요소의 자연 순서에 따라 정렬됩니다
transient Object[] queue; // non-private to simplify nested class access

int size;

@SuppressWarnings("serial") // Conditionally serializable
private final Comparator<? super E> comparator;
````
내부 구현은 Object배열을 통해 이루어집니다. queue변수 상단 문장을 보면 알 수 있습니다.

### 생성자
PriorityQueue 생성자는 파라미터가 없는 생성자, initialCapacity만 있는 생성자, comparator를 호출하는 생성자가 있습니다.
하지만 결국 3가지 생성자 모두 하나의 생성자를 호출하게 됩니다.
````java
Params:
initialCapacity – 우선 순위 대기열의 초기 용량 
comparator – 우선 순위 대기열을 정렬하는 데 사용할 비교기입니다. null인 경우 요소의 자연스러운 순서가 사용됩니다.
Throws:
IllegalArgumentException – if initialCapacity is less than 1
public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}
````
queue의 Object 배열을 초기화 하고 비교의 키를 설정합니다.

### 삽입
PriorityQueue는 AbstractCollection를 상속받는데 AbstractCollection에 있는 add 메서드를 사용할 수 있습니다.
하지만 add는 offer 메서드에 위임하는 역할만 하기에 offer 메서드만 보면 됩니다.
````java
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1); // queue의 크기를 넘어가면 재산정
    siftUp(i, e);
    size = i + 1;
    return true;
}

private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    // queue의 크기가 64보다 작으면 2증가 크면 2배 증가
    int newCapacity = ArraysSupport.newLength(oldCapacity,
            minCapacity - oldCapacity, /* minimum growth */
            oldCapacity < 64 ? oldCapacity + 2 : oldCapacity >> 1
                                       /* preferred growth */);
    queue = Arrays.copyOf(queue, newCapacity);
}

private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x, queue, comparator);
    else
        siftUpComparable(k, x, queue);
}

private static <T> void siftUpUsingComparator(
    int k, T x, Object[] es, Comparator<? super T> cmp) {
    while (k > 0) { // root 까지 반복
        int parent = (k - 1) >>> 1; // 부모 노드는 queue[(k - 1)/2]
        Object e = es[parent];
        if (cmp.compare(x, (T) e) >= 0)
            break;
        es[k] = e;
        k = parent;
    }
    es[k] = x;
}

private static <T> void siftUpComparable(int k, T x, Object[] es) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = es[parent];
        if (key.compareTo((T) e) >= 0)
            break;
        es[k] = e;
        k = parent;
    }
    es[k] = key;
}
````
배열의 마지막 index에 값을 넣은 후 Heapify를 진행

### 출력
보통 queue는 poll은 가장 먼저 들어온 값을 반환합니다. 하지만 PriorityQueue는
우선순위가 가장 높은 값을 반환합니다.
````java
public E poll() {
    final Object[] es;
    final E result;

    if ((result = (E) ((es = queue)[0])) != null) {
        modCount++;
        final int n;
        final E x = (E) es[(n = --size)];
        es[n] = null;
        if (n > 0) {
            final Comparator<? super E> cmp;
            if ((cmp = comparator) == null)
                siftDownComparable(0, x, es, n);
            else
                siftDownUsingComparator(0, x, es, n, cmp);
        }
    }
    return result;
}

private static <T> void siftDownComparable(int k, T x, Object[] es, int n) {
    // assert n > 0;
    Comparable<? super T> key = (Comparable<? super T>)x;
    int half = n >>> 1;           // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = es[child];
        int right = child + 1;
        if (right < n &&
            ((Comparable<? super T>) c).compareTo((T) es[right]) > 0)
            c = es[child = right];
        if (key.compareTo((T) c) <= 0)
            break;
        es[k] = c;
        k = child;
    }
    es[k] = key;
}

private static <T> void siftDownUsingComparator(
    int k, T x, Object[] es, int n, Comparator<? super T> cmp) {
    // assert n > 0;
    int half = n >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = es[child];
        int right = child + 1;
        if (right < n && cmp.compare((T) c, (T) es[right]) > 0)
            c = es[child = right];
        if (cmp.compare(x, (T) c) <= 0)
            break;
        es[k] = c;
        k = child;
    }
    es[k] = x;
}
````
root 값이 반환되어 사라지며 마지막 노드 값을 root로 올려 아래의 값들과 비교하며 heapify를 재구성한다.

### 순환
PriorityQueue의 순환은 간단합니다. 배열로 되어있어 index 0번 부터 마지막까지 순환합니다.
````java
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    final Object[] es = queue;
    for (int i = 0, n = size; i < n; i++)
        action.accept((E) es[i]);
    if (expectedModCount != modCount)
        throw new ConcurrentModificationException();
}
````
순환했을 때 우선순위데 따라 노출한다고 보기 어렵습니다.
Heap 자료구조는 같은 형제 노드라고 할 때 왼쪽의 값이 오른쪽보다 무조건 더 크지 않습니다. 따라서 우선순위 출력이 아닙니다.
