# HashMap
HashMap은 Key, Value를 저장하는 구현체이다. HashMap는 Key를 Hashing하여 저장하며 빠르게 처리한다.
입력과 삭제에 대해 시간복잡도가 O(1)인 자료구조이다.

### 초기화
HashMap를 사용한다면 가장 먼저 초기화 합니다.
아래와 같이 초기화 할 것입니다.

![hashmap2](https://user-images.githubusercontent.com/42162127/227927621-6c8cb62e-0665-4c21-8087-8df1bdef88c1.PNG)

HashMap 생성자를 보겠습니다.

![hashmap1](https://user-images.githubusercontent.com/42162127/227927852-c320b2a1-e051-4a49-827c-0ae78bc57e3d.PNG)

내용을 살펴보겠습니다. loadFactor에 DEFAULT_LOAD_FACTOR(0.75f)를 주입합니다. 다음으로 put 메소드를 보겠습니다.

### put

![hashmap3](https://user-images.githubusercontent.com/42162127/227928526-75e9e8d1-7adc-4eda-af0f-e2775c1bdc77.PNG)

put 메소드에서는 putVal 이라는 메소드를 호출합니다. 우선 hash 메소드를 보겠습니다.

![hashmap4](https://user-images.githubusercontent.com/42162127/227928877-c6f2f26c-fc5a-4c60-b647-0e83dbb7d646.PNG)


