# EnumMap
Enum을 key로 하는 자료구조
HashMap을 보면 key의 충돌이 발생하기에 Node<K, V>[] 의 Hash Table을 가지고 있습니다.
하지만 EnumMap의 key는 Enum의 순서인데 key의 충돌이 일어나지 않기에 이런 구조를 가집니다.
![캡처4](https://user-images.githubusercontent.com/42162127/230070384-3b931110-a60e-460c-ae5d-88b5cb7ddc46.PNG)

### 상속
````java
public class EnumMap<K extends Enum<K>, V> extends AbstractMap<K, V>
    implements java.io.Serializable, Cloneable
````
HashMap과 AbstractMap을 상속 받는거는 같습니다. 하지만 Map 인터페이스는 받고 있지 않습니다.

다음으로 필드 변수를 보겠습니다.
````java
private final Class<K> keyType;

// key와 value
private transient K[] keyUniverse;
private transient Object[] vals;
````
key와 value 모두 단순한 배열로 되어 있습니다.

### 생성자
````java
public EnumMap(Class<K> keyType) {
    this.keyType = keyType;
    keyUniverse = getKeyUniverse(keyType);
    vals = new Object[keyUniverse.length];
}
````
Class Type을 받아 keyType, kye, value을 세팅 합니다. key와 value 모두 현재 enum의 갯수 만큼 배열의 크기를 만듭니다.

````java
public EnumMap(Map<K, ? extends V> m) {
    if (m instanceof EnumMap) {
        EnumMap<K, ? extends V> em = (EnumMap<K, ? extends V>) m;
        keyType = em.keyType;
        keyUniverse = em.keyUniverse;
        vals = em.vals.clone();
        size = em.size;
    } else {
        if (m.isEmpty())
            throw new IllegalArgumentException("Specified map is empty");
        keyType = m.keySet().iterator().next().getDeclaringClass();
        keyUniverse = getKeyUniverse(keyType);
        vals = new Object[keyUniverse.length];
        putAll(m);
    }
}
````
Map 인터페이스를 사용하는 HashMap 같은 Class로 초기화 할 수 있습니다.
파라미터로 받은 map을 key, value를 이용하여 초기화 합니다.

### put
key와 value는 이미 size가 고정되어 있는 배열이기에 enum의 index만 ordinal() 메서드를 통해 가져와 
index의 value 배열에 값을 변경합니다.
그렇기에 O(1)의 시간복잡도를 가집니다.
````java
public V put(K key, V value) {
    typeCheck(key); // 동일한 key type인지 확인

    int index = key.ordinal();  // enum 순번확인
    Object oldValue = vals[index];  // enum 순번을 key index로 기존 값 확인
    vals[index] = maskNull(value);  // 새로운 값 넣기
    if (oldValue == null) // key에 value가 비어있다면 size 증가
        size++;
    return unmaskNull(oldValue);
}
````

### get
찾고자하는 value는 enum의 ordinal()를 이용한 index에 있기에 항상 O(1)로 찾을 수 있습니다.
````java
public V get(Object key) {
    return (isValidKey(key) ? // key가 정상인지 판단
            unmaskNull(vals[((Enum<?>)key).ordinal()]) : null);
}

private boolean isValidKey(Object key) {
    if (key == null)
        return false;

    // Cheaper than instanceof Enum followed by getDeclaringClass
    Class<?> keyClass = key.getClass();
    return keyClass == keyType || keyClass.getSuperclass() == keyType;
}

private V unmaskNull(Object value) {
    return (V)(value == NULL ? null : value);
}
````

