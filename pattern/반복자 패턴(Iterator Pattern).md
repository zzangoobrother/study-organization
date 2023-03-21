# 반복자 패턴 (Iterator Pattern)
접근기능과 자료구조를 분리시켜서 객체화
서로 다른 구조를 가지고 있는 저장 객체에 대해서 접근하기 위해서 interface를 통일시키고 싶을 때 사용하는 패턴
Iterator는 이러한 자료구조에 상관없이 객체 접근 방식을 통일시키고자 할 떄 사용됩니다.

````java
public class Item {
    private String name;
    private int cost;

    public Item(String name, int cost) {
        this.name = name;
        this.cost = cost;
    }

    @Override
    public String toString() {
        return "(" + name + ", " + cost + ")";
    }
}
````

````java
public interface Aggregator {
    Iterator iterator();
}
````

````java
public class Array implements Aggregator {
    private Item[] items;

    public Array(Item[] items) {
        this.items = items;
    }

    public Item getItem(int index) {
        return items[index];
    }

    public int getCount() {
        return items.length;
    }

    @Override
    public Iterator iterator() {
        return new ArrayIterator(this);
    }
}
````

````java
public interface Iterator {
    boolean next();
    Object current();
}
````

````java
public class ArrayIterator implements Iterator {
    private Array array;
    private int index;

    public ArrayIterator(Array array) {
        this.array = array;
        this.index = -1;
    }

    @Override
    public boolean next() {
        index++;
        return index < array.getCount();
    }

    @Override
    public Object current() {
        return array.getItem(index);
    }
}
````

````java
public class Main {
    public static void main(String[] args) {
        Item[] items = {
                new Item("CPU", 1000),
                new Item("Keyboard", 1000),
                new Item("Mouse", 1000),
                new Item("MDD", 1000)
        };

        Array array = new Array(items);
        Iterator iterator = array.iterator();

        while (iterator.next()) {
            Item item = (Item)iterator.current();
            System.out.println(item);
        }
    }
}
````
