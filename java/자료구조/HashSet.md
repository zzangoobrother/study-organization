# HashSet
Set은 List와 다르게 중복을 허용하지 않은 자료구조이다.

### 상속, 멤버변수
````java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
````
내부 모습입니다. HashMap의 모습이 보입니다. HashMap의 구조는 [HashMap](https://github.com/zzangoobrother/study-organization/blob/main/java/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/HashMap.md) 에서
참고바랍니다. HashMap은 Object 클래스의 equals와 hashcode 메소드를 이용하여 중복값을 제거합니다.

PRESENT는 HashSet 내부에 있는 HashMap의 value에 들어가는 더미값 입니다. HashMap의 key는 사용하지만 value는 사용하지 않기 때문에
더미값을 채웁니다.

### 생성자
````java
/**
 * Constructs a new, empty set; the backing {@code HashMap} instance has
 * default initial capacity (16) and load factor (0.75).
 */
public HashSet() {
    map = new HashMap<>();
}
````
HashSet의 인스턴스는 HashMap이 새로 생성되는 것을 볼 수 있습니다.

### add
````java
/**
 * Adds the specified element to this set if it is not already present.
 * More formally, adds the specified element {@code e} to this set if
 * this set contains no element {@code e2} such that
 * {@code Objects.equals(e, e2)}.
 * If this set already contains the element, the call leaves the set
 * unchanged and returns {@code false}.
 *
 * @param e element to be added to this set
 * @return {@code true} if this set did not already contain the specified
 * element
 */
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
````
add 메소드는 단순합니다. HashMap에 put를 하고 끝입니다. HashMap에 key를 넣게 되면 내부 알고리즘으로 equals, hashcode를 
비교하여 값을 넣습니다. 그리고 key 중복이 존재하면 예전값을 반환되고 없으면 반환하지 않습니다.
따라서 중복값이 존재하면 false, 존재하지 않으면 true가 반환됩니다.

### remove
````java
/**
 * Removes the specified element from this set if it is present.
 * More formally, removes an element {@code e} such that
 * {@code Objects.equals(o, e)},
 * if this set contains such an element.  Returns {@code true} if
 * this set contained the element (or equivalently, if this set
 * changed as a result of the call).  (This set will not contain the
 * element once the call returns.)
 *
 * @param o object to be removed from this set, if present
 * @return {@code true} if the set contained the specified element
 */
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
````
remove 역시 단순하게 HashMap remove를 호출한 후 마무리 됩니다. add 메소드와 동일하게 equals, hashcode와 비교하여 내부 값을 제거합니다.
값이 존재하면 true, 없다면 false를 반환합니다.

HashSet은 HashMap을 내부에 구현하여 중복이 일어나지 않도록 합니다. 다른 Collection과 다르게 HashSet은 HashMap을 이용하기 때문에
코드가 길지 않습니다.
