### 설계 품질과 트레이드오프
- 객체지향 설계의 핵심은 역할, 책임, 협력
- 책임 주도 설계, 책임이 적절하게 할당되지 못한 상황에서 원활한 협력도 없다. 역할은 책임의 집합이기에 책임이 적절하지 못하면 협력과 조화를 이루지 못한다. 
- 결국 책임이 객체지향 애플리케이션 전체의 품질을 결정한다.
- 객체지향 설계는 올바른 책임을 할당하면서 낮은 결합도, 높은 응집도를 가진 구조를 창조하는 활동
- 두가지 관점, 첫 번째 관점, 객체지향 설계의 핵심이 책임이다. 두 번째 관점은 책임을 할당하는 작업이 응집도와 결합도 같은 설계 품질과 깊이 연관돼 있다.
- 객체의 상태가 아니라 객체의 행동에 초점을 맞추자.

#### 데이터 중심의 영화 예매 시스템
- 객체지향 설계에서 두 가지 방법
  1. 상태를 분할의 중심축으로 삼는 방법
     - 객체는 자신이 포함하고 있는 데이터를 조작하는 데 필요한 오퍼레이션을 정의
     - 구현에 관한 세부사항이 객체의 인터페이스에 스며들게 되어 캡슐화 원칙 무너짐.
  2. 책임을 분할의 중심축으로 삼는 방법
     - 객체는 다른 객체가 요청할 수 있는 오퍼레이션을 위해 필요한 상태를 보관
     - 인터페이스 뒤로 책임을 수행하는 데 필요한 상태를 캡슐화함으로 구현 변경에 대한 파장이 외부로 퍼져나가는 것을 방지

##### 데이터를 준비하자
책임 중심의 설계가 '책임이 무엇인가'를 묻는 것으로 시작한다면 데이터 중심의 설계는 객체가 내부에 저장해야 하는 '데이터가 무엇인가'를 묻는 것으로 시작

<table>
<tr>
<th>책임 중심</th>
<th>데이터 중심</th>
</tr>

<tr><td>

````java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;
}
````

</td>

<td>

````java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    
    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;
}
````

</td>
</tr>
</table>

책임 중심과 데이터 중심 클래스의 차이점은 할인 조건 목록(discountConditions), 할인 금액(discountAmount), 할인 비율(discountPercent)을 Movie 안에서 직접 정의한다.

금액 할인 정책인지, 비율 할인 정책인지, 미적용인지 알기 위해 MovieType을 작성한다.
````java
public enum MovieType {
    AMOUNT_DISCOUNT, // 금액 할인 정책
    PERCENT_DISCOUNT, // 비율 할인 정책
    NONE_DISCOUNT // 미적용
}
````

Movie와 같은 방식으로 DiscountCondition도 작성하자

<table>
<tr>
<th>책임 중심</th>
<th>데이터 중심</th>
</tr>

<tr><td>

````java
public interface DiscountCondition {
    boolean isSatisfiedBy(Screening screening);
}
public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
}
public class SequenceCondition implements DiscountCondition {
    private int sequence;
}
````

</td>

<td>

````java
public class DiscountCondition {
    private DiscountConditionType type;

    private int sequence;

    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
}
public enum DiscountConditionType {
    SEQUENCE, // 순번 조건
    PERIOD // 기간 조건
}
````

</td>
</tr>
</table>

데이터 중심의 영화 예매
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

        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            switch (movie.getMovieType()) {
                case AMOUNT_DISCOUNT -> discountAmount = movie.getDiscountAmount();
                case PERCENT_DISCOUNT -> discountAmount = movie.getFee().times(movie.getDiscountPercent());
                case NONE_DISCOUNT -> discountAmount = Money.ZERO;
            }

            fee = movie.getFee().minus(discountAmount);
        } else {
            fee = movie.getFee();
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
````

크게 두 부분으로 분할
- DiscountCondition 순회하며 할인 가능 여부 확인
- 할인 가능 여부 체크 후 적절한 예매 요금 계산

#### 설계 트레이드오프

