### 메시지와 인터페이스
객체지향 프로그래밍에 대한 가장 흔한 오해는 애플리케이션이 클래스의 집합으로 구성된다는 것이다.

클래스라는 구현 도구에 지나치게 집착하면 경직되고 유연하지 못한 설계에 이를 확률이 높아진다.

협력 안에서 객체가 수행하는 책임에 초점을 맞추고, 객체가 수신할 수 있는 메시지의 기반이 되어야 한다.

#### 협력과 메시지

##### 클라이언트-서버 모델
- 클라이언트(client) : 메시지를 전송하는 객체
- 서버(server) : 메시지를 수신하는 객체

협력은 클라이언트가 서버의 서비스를 요청하는 단방향 상호작용

##### 메시지와 메시지 전송
용어 정리
- 메시지 : 객체들이 협력하기 위해 사용하는 유일한 의사소통 수단
- 오퍼레이션 : 객체가 다른 객체에게 제공하는 추상적인 서비스, 메시지 전송자는 고려하지 않은 채 메시지 수신 객체의 인터페이스만 강조
- 메서드 : 메시지를 수신했을 때 실제로 실행되는 함수 또는 프로시저
- 퍼블릭 인터페이스 : 객체가 의사소통을 위해 외부에 공개하는 메시지의 집합
- 시그니처 : 오퍼레이션(또는 메서드)의 이름과 파라미터 목록을 합쳐 부른다.

#### 인터페이스와 설계 품질
좋은 인터페이스는 최소한의 인터페이스와 추상적인 인터페이스 라는 조건을 만족해야 한다.
- 최소한의 인터페이스 : 꼭 필요한 오퍼레이션만을 인터페이스에 포함
- 추상적인 인터페이스 : 무엇을 하는지를 표현

##### 디미터 법칙
협력하는 객체의 내부 구조에 대한 결합으로 인해 발생하는 설계 문제를 해결하기 위해 제안된 원칙

객체의 내부 구조에 강하게 결합되지 않도록 협력 경로를 제한

- 낯선 자에게 말하지 말라(don't talk to strangers)
- 오직 인접한 이웃하고만 말하라(only talk to your immediate neighbors)
- 오직 하나의 도트만 사용하라(use only one dot)

디미터 법칙을 따르기 위해서
- 메서드의 인자로 전달된 클래스
- 해당 메서드를 가진 클래스 자체
- 해당 메서드를 가진 클래스의 인스턴스 변수

즉 아래와 같은 경우라고 한정지을 수 있다.
- this 객체
- 메서드의 매개변수
- this의 속상
- this의 속성인 컬렉션의 요소
- 메서드 내에서 생성된 지역 객체

<table>
<tr>
<th>Before</th>
<th>After</th>
</tr>

<tr><td>

````java
public class ReservationAgency {
    private Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for (DiscountCondition condition : movie.getConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                        condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                        condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
            } else {
                discountable = condition.getSequence() == screening.getSequence();
            }

            if (discountable) {
                break;
            }
        }
        // ....
    }
}
````

</td>

<td>

````java
public class ReservationAgency {
    private Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Money fee = screening.calculateFee(audienceCount);
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
````

</td>
</tr>
</table>

- 부끄럼타는 코드(shy code)
  - 불필요한 어떤 것도 다른 객체에게 보여주지 않으며, 다른 객체의 구현에 의존하지 않는 코드

- 디미터 법칙과 캡슐화
  - 디미터 법칙은 캡슐화를 다른 관점에서 표현한 것
  - 클래스를 캡슐화하기 위해 따라야 하는 구체적인 지침 제공
  - 캡슐화 원칙이 클래스 내부 구현을 감춰야 한다는 사실을 강조한다면, 디미터 법칙은 협력하는 클래스의 캡슐화를 지키기 위해 접근해야 하는 요소 제한

기차 충동 (train wreck)
```java
// 전형적인 디미터 법칙 위반 코드
screening.getMovie().getDiscountConditions();
```
- 여러 대의 기차가 한 줄로 늘어서 충돌한 것처러 보이는 코드
- 클래스 내부 구현이 외부로 노출됐을 때 나타나는 전형적인 형태
- 메시지 전송자는 메시지 수신자의 내부 정보를 자세히 알게 된다
  - 수신자 캡슐화는 무너짐
  - 전송자가 수신자의 내부 구현에 강하게 결합

##### 묻지 말고 시켜라

