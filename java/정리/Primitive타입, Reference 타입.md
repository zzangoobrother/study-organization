Primitive 타입과 Reference 타입

Primitive 타입은 데이터의 실제 값 자체를 저장하고, Reference 타입은 데이터의 메모리 주소를 저장합니다
즉, Primitive 타입은 값 자체를 복사하고, Reference 타입은 메모리 주소를 복사하는 것입니다.

primitive 타입은 int, long, float, double, boolean, byte, char, short 이 있습니다.

float는 4byte, double는 8byte 입니다. 바이트 수가 다르기 때문에 다루는 데이터 범위가 다릅니다. 

double은 float보다 많은 바이트 수를 다루기 때문에 double의 정밀도가 높습니다.

float는 1비트가 부호비트, 8비트는 지수 피드, 23비트는 가수 필드 형식을 가진 단일 정밀도 형식이고, 
double은 1비트가 부호비트, 11비트는 지수 비트, 52비트는 가수 비트 형식을 가진 복수 정밀도 형식 입니다..

부동소수점
부동소수점은 유리수를 이진수의 지수와 가수로 변환하여 근사한 값을 표현하는 방식으로, 
이진수로 변환해도 모든 소수를 정확하게 표현하기 어렵기 때문에 오차가 발생합니다.
그러므로 계산의 정확도를 극대화하기 위해서는 적절한 반올림을 통한 오차를 줄이는 기법 등을 사용합니다.

언더플로우는 메모리가 표현할 수 있는 최솟값(음수의 경우)보다 더 작은 값을 저장하는 것을 의미합니다
오버플로우는 메모리가 표현할 수 있는 최댓값보다 더 큰 값을 저장하는 것을 의미합니다.
