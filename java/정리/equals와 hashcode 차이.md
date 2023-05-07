# Equals와 Hashcode 차이

equals는 동일한 객체인지 여부를 판단합니다. hashcode는 객체의 고유한 정수형값을 반환하며
동일한 객체라면 항상 같은 hashcode를 반환합니다. 객체의 동등성 비교에 사용합니다.
hashcode의 기본 구현은 객체의 메모리주소를 사용하지만, hashcode를 오버라이딩하여 객체의 동등성 비교에 사용하는 경우
hashcode 값을 객체가 가지고 있는 값들의 해시값으로 생성하여 사용합니다.
이때 equals 메서드 역시 오버라이딩하여 객체의 값들을 비교합니다.
그러므로 객체의 값들이 같으면 동일한 hashcode 값을 가지도록 구현해야 합니다.
객체 비교는 hashcode 값을 먼저 비교 후 hashcode가 같으면 equals 메서드를 통해 동등한 객체인지 비교를 합니다.
만약 equals만 재정의를 한다면 동등한 객체라도 hashcode 값이 다르게 나올수 있기에 hashcode와 equals 메서드를 같이 재정의 합니다.

#### hashcode 생성시 어떤 알고리즘이나 규칙을 지켜야 하나요?
Division Method: 저장소의 크기로 나누어 나온 나머지를 사용 Digit Folding: 문자열을 아스키코드로 바꾸고 그 값을 합해 사용
Multiplication Method: 숫자로 된 key값 k, 0~1 사이의 실수 A, 2의 제곱수 M 을 사용하여 계산 (kA mod 1)m
Univeral Hashing: 다수의 hash 함수를 만들어 특정한 장소에 넣어 무작위로 hash 함수를 선택해 사용

해시충돌이 일어난다면 대체방안 중 하나로 체이닝이 있습니다. 어떤 방식으로 처리하나요?
같은 해시끼리의 공간을 연결리스트로 저장합니다. 즉 LinkedList를 사용하여 데이터를 저장합니다.
하지만 시간복잡도가 해시 함수에 따라 최악의 경우 O(n)의 시간복잡도를 가질 수 있습니다.


