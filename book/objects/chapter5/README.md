### 책임 할당하기
- GRASP Pattern : General Responsiblity Assignment Software Pattern, 책임 할당을 위한 소프르퉤어 패턴, 객체에게 책임을 할당할 때 지침으로 삼을 수 있는 원칙들의 집합을 패턴 형식으로 정리한 것
  - INFORMATION EXPERT 패턴 : 책임을 수행하는 데 필요한 정보를 가지고 있는 객체에게 할당하라
  - LOW COUPLING 패턴 : 설계의 전체적인 결합도가 낮게 유지되도록 책임을 할당하라
  - HIGH COHESION 패턴 : 높은 응집도를 유지할 수 있게 책임을 할당하라
  - CREATOR 패턴 : 생성되는 객체와 연결되거나 관련될 필요가 있는 객체에 해당 객체를 생성할 책임을 맡기는 것이다
  - POLYMORPHISM 패턴 : 타입을 명시적으로 정의하고 각 타입에 다형적으로 행동하는 책임을 할당하라
  - PROTECTED VARIATIONS 패턴 : 변화가 예상되는 불안정한 지점들을 식별하고 그 주위에 안정된 인터페이스를 형성하도록 책임을 할당하라

#### 책임 주도 설계를 향해
데이터 중심 설계 => 책임 중심 설계 전환을 위한 두 가지 원칙
- 데이터보다 행동을 먼저 결정하라
- 협력이라는 문맥 안에서 책임을 결정하라

##### 데이터보다 행동을 먼저 결정하라

- 데이터 중심 설계 결정 순서
  - '이 객체가 포함해야 하는 데이터가 무엇인가'
  - '데이터를 처리하는 데 필요한 오퍼레이션은 무엇인가'

- 책임 중심 설계
  - '이 객체가 수행해야 하는 책임은 무엇인가'
  - '이 책임을 수행하는 데 필요한 데이터는 무엇인가'

##### 협력이라는 문맥 안에서 책임을 결정하라
객체를 결정한 후에 메시지를 선택하는 것이 아니라 메시지를 결정한 후에 객체를 선택해야 한다.

올바른 객체지향 설계는 클라이언트가 전송할 메시지를 결정한 후에야 비로소 객체의 상태를 저장하는 데 필요한 내부 데이터에 관해 고민하기 시작한다.

#### 책임 할당을 위한 GRASP 패턴

##### 도메인 개념에서 출발하기
설계를 시작하는 단계에서는 개념들의 의미와 관계가 정확하거나 완벽할 필요가 없다.

책임을 할당받을 객체들의 종류와 관계에 대한 유용한 정보를 제공할 수 있다면 충분하다.

<pre>
<b>올바른 도메인 모델이란 존재하지 않는다.</b> 

많은 사람들이 도메인 모델은 구현과는 무관하다고 생각한다.
도메인 모델이 구현을 염두에 두고 구조화되는 것이 바람직하다.
반대로 코드의 구조가 도메인을 바라보는 관점을 바꾸기도 한다.

필요한 것은 도메인을 그대로 투영한 모델이 아니라 구현에 도움이 되는 모델이다. 
실용적이면서도 유용한 모델이 답이다.
</pre>

##### 정보 전문가에게 책임을 할당하라
- 책임을 수행하는 데 필요한 메시지를 결정해야 한다.
  - 메시지는 메시지를 수신할 객체가 아니라 메시지를 전송할 객체의 의도를 반영해서 결정해야 한다.
  - 메시지를 전송할 객체는 무엇을 원하는가? -> 메시지를 수신할 적합한 객체는 누구인가?

<pre>
<b>INFORMATION EXPERT 패턴</b> 
책임을 수행하는 데 필요한 정보를 가지고 있는 객체에게 할당하라

객체가 자율적인 존재여야 한다.
정보와 행동을 최대한 가까운 곳에 위치시키기 때문에 캡슐화를 유지
필요한 정보를 가진 객체들로 책임이 분산되기 때문에 높은 응집도 가능
결과적으로 결합도가 낮아지고, 간결하고, 유지보수하기 쉬운 시스템을 구축
</pre>

- 영화 예매를 위한 정보 전문가 -> '예매하라'
- 예매에 필요한 정보를 가장 많이 알고 있는 객체에게 메시지를 처리할 책임 할당

##### Screening
- 메시지를 철히하기 위한 절차와 구현 고민
- 스스로 처리할 수 없다면 외부에 도움 요청 -> 새로운 메시지 -> 이 메시지가 새로운 객체의 책임 할당
  - 예매 가격 계산 필요 -> 영화 한 편 가격 x 인원수
  - 가격을 계산하는 데 필요한 정보를 모름 -> 외부 객체에게 도움 요청
  - 새로운 메시지 '가격을 계산하라'

- Movie 객체가 책임을 가짐
  - '할인 여부를 판단하라' 라는 새로운 메시지 도움 요청
- DiscountCondition 객체가 책임을 가짐

##### 높은 응집도와 낮은 결합도
설계는 트레이드오프 활동, 높은 응집도와 낮은 결합도를 항상 고려하여 선택해야 한다.

GRASP에서는 LOW COUPLING 패턴, HIGH COHESION 패턴 이라 부른다.
- LOW COUPLING 패턴
  - 설계의 전체적인 결합도가 낮게 유지되도록 책임을 할당

- HIGH COHESION 패턴
  - 높은 응집도를 유지할 수 있게 책임을 할당

Screening의 가장 중요한 책임은 예매를 생성하는 것이다. DiscountCondition과 협력한다면 영화 요금 계산과 관련된 책임 일부를 떠안아야 한다.
예매 요금 계산 방식이 변경된다면 Screening도 함께 변경될 확률이 크다.

##### 창조자에게 객체 생성 책임을 할당하라
<pre>
<br>CREATOR 패턴</br>
: 객체 A를 생성해야 할 때, 아래 조건을 최대한 많이 만족하는 B에게 객체 생성 책임을 할당하라
- B가 A 객체를 포함하거나 참조한다.
- B가 A 객체를 기록한다.
- B가 A 객체를 긴밀하게 사용한다.
- B가 A 객체를 초기화하는 데 필요한 데이터를 가지고 있다.(이 경우 B는 A에 대한 정보 전문가다)

생성되는 객체와 연결되거나 관련될 필요가 있는 객체에 해당 객체를 생성할 책임을 맡기는 것이다.
-> 생성될 객체에 대해 잘 알고 있거나, 그 객체를 사용해야 하는 객체는 어떤 방식으로든 생성될 객체와 연결될 것이다.
</pre>

- Reservation 객체를 생성해야 한다.
- 초기화에 필요한 데이터를 가지고 있는 객체는 무엇인가?
- Screening 이다. CREATOR로 선택하는 것이 적절하다.

#### 구현을 통한 검증
[영화예매시스템 v1](https://github.com/zzangoobrother/study-project/tree/master/book-objects/src/main/java/chapter5/v1)

##### DiscountCondition 개선하기
DiscountCondition은 하나 이상의 변경 이유를 가지기 때문에 응집도가 낮다.

- 새로운 할인 조건 추가
  - isSatisfiedBy 메서드 안 if ~ else 구문을 수정해야 한다.
  - 새로운 할인 조건이 새로운 데이터를 요구하면 DiscountCondition 속성 추가
- 순번 조건을 판단하는 로직 변경
  - isSatisfiedBySequence 메서드 내부 구현 수정
  - 순번 조건을 판단하는 데 필요한 데이터가 변경된다면 DiscountCondition의 sequence 속성 변경
- 기간 조건을 판단하는 로직이 변경되는 경우
  - isSatisfiedByPeriod 메서드 내부 구현 수정
  - 기간 조건을 판단하는 데 필요한 데이터가 변경된다면 DiscountCondition의 속성 변경

###### 변경의 이유에 따라 클래스를 분리
- 코드를 통해 변경의 이유 파악할 수 있는 방법
  - 인스턴스 변수가 초기화되는 시점 살펴보는 것
    - 응집도가 낮은 클래스는 객체의 속성 중 일부만 초기화하고 일부는 초기화되지 않은 상태로 남겨진다.
    - 함께 초기화되는 속성을 기준으로 코드를 분리해야 한다.
  - 메서드들이 인스턴스 변수를 사용하는 방식 살펴보는 것
    - 모든 메서드가 모든 속성을 사용한다면 클래스의 응집도 높음
    - 메서드들이 사용하는 속성에 따라 그룹이 나뉜다면 클래스의 응집도가 낮음
    - 속성 그룹과 해당 그룹에 접근하는 메서드 그룹을 기준으로 코드를 분리해야 한다.

##### 타입 분리하기
- DiscountCondition -> PeriodCondition, SequenceCondition으로 분리
- Movie에 List<PeriodCondition>, List<SequenceCondition> 로 Movie의 결합도가 높아짐

````java
public class PeriodCondition {
  private DayOfWeek dayOfWeek;
  private LocalTime startTime;
  private LocalTime endTime;

  public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
    this.dayOfWeek = dayOfWeek;
    this.startTime = startTime;
    this.endTime = endTime;
  }

  public boolean isSatisfiedBy(Screening screening) {
    return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
            startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
            endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
  }
}
````

````java
public class SequenceCondition {
  private int sequence;

  public SequenceCondition(int sequence) {
    this.sequence = sequence;
  }

  public boolean isSatisfiedBy(Screening screening) {
    return sequence == screening.getSequence();
  }
}
````

````java
public class Movie {
    
  private List<PeriodCondition> periodConditions;
  private List<SequenceCondition> sequenceConditions;

  private boolean isDiscountable(Screening screening) {
    return checkPeriodConditions(screening) || checkSequenceConditions(screening);
  }

  private boolean checkPeriodConditions(Screening screening) {
    return periodConditions.stream()
            .anyMatch(condition -> condition.isSatisfiedBy(screening));
  }

  private boolean checkSequenceConditions(Screening screening) {
    return sequenceConditions.stream()
            .anyMatch(condition -> condition.isSatisfiedBy(screening));
  }
}
````

##### 다형성을 통해 분리하기
Movie는 할인 가능 여부만 반환 받으면 되기에 PeriodCondition, SequenceCondition 인지 상관없다. 오직 역할에 대해서만 결합되도록 의존성을 제한할 수 있다.

````java
public interface DiscountCondition {
  boolean isSatisfiedBy(Screening screening);
}
````

````java
public class PeriodCondition implements DiscountCondition {
    
}
````

````java
public class SequenceCondition implements DiscountCondition {
    
}
````

````java
public class Movie {
  private List<DiscountCondition> discountConditions;

  public Money calculateMovieFee(Screening screening) {
    if (isDiscountable(screening)) {
      return fee.minus(calculateDiscountAmount());
    }

    return fee;
  }

  private boolean isDiscountable(Screening screening) {
    return discountConditions.stream()
            .anyMatch(condition -> condition.isSatisfiedBy(screening));
  }
}
````

POLYMORPHISM(다형성) 패턴 : 변하는 행동이 있다면 타입을 분리하고 변화하는 행동을 각 타입의 책임으로 할당하라

<pre>
<br>POLYMORPHISM 패턴</br>
: 타입을 명시적으로 정의하고 각 타입에 다형적으로 행동하는 책임을 할당
- 객체의 타입에 따라 대안들을 수행하는 조건적인 논리를 사용하지 말라고 경고
- 다형성을 이용해 새로운 변화를 다루기 쉽게 확장하라고 권고
</pre>

##### 변경으로부터 보호하기
Movie로부터 PeriodCondition, SequenceCondition의 존재를 감춤

GRASP에서는 PROTECTED VARIATIONS(변경 보호) 패턴이라 부름
<pre>
<br>PROTECTED VARIATIONS 패턴</br>
: 변화가 얘상되는 불안정한 지점들을 식별하고 그 주위에 안정된 인터페이스를 형성하도록 책임을 할당
- 변경이 될 가능성이 높은가? 그렇다면 캡슐화하라
</pre>

#### Movie 클래스 개선하기
영화 할인 정책, 두 가지를 하나의 클래스 안에 구현하고 있기에 Movie는 하나 이상의 이유로 변경된다. 응집도가 낮다.
- 금액 할인 정책 : AmountDiscountMovie
- 비율 할인 정책 : PercentDiscountMovie
- 할인 정책을 적용하지 않음 : NoneDiscountMovie

````java
public abstract class Movie {
  private String title;
  private Duration runningTime;
  private Money fee;

  private List<DiscountCondition> discountConditions;

  public Movie(String title, Duration runningTime, Money fee, DiscountCondition... discountConditions) {
    this.title = title;
    this.runningTime = runningTime;
    this.fee = fee;
    this.discountConditions = Arrays.asList(discountConditions);
  }

  public Money calculateMovieFee(Screening screening) {
    if (isDiscountable(screening)) {
      return fee.minus(calculateDiscountAmount());
    }

    return fee;
  }

  private boolean isDiscountable(Screening screening) {
    return discountConditions.stream()
            .anyMatch(condition -> condition.isSatisfiedBy(screening));
  }

  abstract protected Money calculateDiscountAmount();

  protected Money getFee() {
    return fee;
  }
}
````

````java
public class AmountDiscountMovie extends Movie {
  private Money discountAmount;

  public AmountDiscountMovie(String title, Duration runningTime, Money fee, Money discountAmount, DiscountCondition... discountConditions) {
    super(title, runningTime, fee, discountConditions);
    this.discountAmount = discountAmount;
  }

  @Override
  protected Money calculateDiscountAmount() {
    return discountAmount;
  }
}
````

````java
public class PercentDiscountMovie extends Movie {
  private double percent;

  public PercentDiscountMovie(String title, Duration runningTime, Money fee, double percent, DiscountCondition... discountConditions) {
    super(title, runningTime, fee, discountConditions);
    this.percent = percent;
  }

  @Override
  protected Money calculateDiscountAmount() {
    return getFee().times(percent);
  }
}
````

````java
public class NoneDiscountMovie extends Movie {

  public NoneDiscountMovie(String title, Duration runningTime, Money fee) {
    super(title, runningTime, fee);
  }

  @Override
  protected Money calculateDiscountAmount() {
    return Money.ZERO;
  }
}
````

##### 변경과 유연성
문제점
- 새로운 할인 정책이 추가될 때마다 인스턴스를 생성, 상태를 복사, 식별자를 관리하는 코드를 추가하여 오류가 발생하기 쉽다.

합성을 사용하여 해결하자
- Movie 상속 계층에 구현된 할인 정책을 독립적인 DiscountPolicy로 분리 후 합성시키자.
=> 2장 영화 예매 시스템 전체 구조 동일

#### 책임 주도 설계의 대안
##### 메서드 응집도
영화 예매 시스템의 모든 절차가 집중되어 있는 ReservationAgency

````java
public class ReservationAgency {
  private Reservation reserve(Screening screening, Customer customer, int audienceCount) {
    Movie movie = screening.getMovie();

    boolean discountable = false;
    for (DiscountCondition condition : movie.getDiscountConditions()) {
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
      chapter4.v1.Money discountAmount = chapter4.v1.Money.ZERO;
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

주석을 추가하는 대신 메서드를 작게 분해해서 각 메서드의 응집도를 높여라


