### 객체지향 프로그래밍

#### 영화 예매 시스템
##### 요구사항

- 할인 조건 (discount confition) : 다수의 할인 조건 할당 가능, 순서/기간 조건 섞는거 가능
  - 순서 조건 (sequence condition) : 상영 순번을 이용해 할인 여부 결정
  - 기간 조건 (period condition) : 영화 상영 시작 시간을 이용해 할인 여부 결정
- 할인 정책 (discount policy) : 할인을 지정하지 않거나, 하나의 영화만 할당 가능
  - 금액 할인 정책 (amount discount policy) : 예매 요금에서 일정 금액을 할인
  - 비율 할인 정책 (percent discount policy) : 정가에서 일정 비율의 요금을 할인

#### 객체지향 프로그래밍을 향해
##### 협력, 객체, 클래스

클래스 기반의 객체지향 언어에 익숙한 사람이라면 어떤 클래스가 필요한지 고민하고, 어떤 속성과 메서드가 필요한지 고민한다.
이는 객체지향의 본질과는 멀다.
클래스가 아닌 객체에 초점을 맞춰야 한다.
- 어떤 클래스가 필요한지 고민하기 전에 어떤 객체를 필요한지 고민하라.
- 객체를 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원으로 봐라.

##### 도메인의 구조를 따르는 프로그램 구조

- 도메인 : 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야
- 객체지향 패러다임 특징 : 요구사항을 분석하는 초기 단계부터 프로그램을 구혀하는 마지막 단계까지 객체라는 동일한 추상화 기법을 사용, 
요구사항과 프로그램을 객체라는 동일한 관점에서 바라볼 수 있기에 도메인을 구성하는 개념들이 프로그램의 객체와 클래스로 매끄럽게 연결

###### 자율적인 객체
- 객체 특징
  - 상태와 행동을 함께 가지는 복합적인 존재
  - 객체가 스스로 판단하고 행동하는 자율적인 존재
- 객체지향은 객체라는 단위 안에 데이터와 기능을 한 덩어리로 묶음, 이를 캡슐화라고 한다.
- 한 걸음 더 나아가 외부에서의 접근을 통제할 수 있는 접근 제어도 제공
- 객체지향의 핵심은 스스로 상태를 관리하고, 판단하고, 행동하는 자율적인 객체들의 공동체 구성
- 외부에서 접근 가능한 부분을 퍼블릭 인터페이스
- 외부에서 접근 불가능하고, 내부에서만 접근 가능한 부분을 구현,
- 인터페이스와 구현의 분리

###### 프로그래머의 자유
- 클라이언트 프로그래머
  - 필요한 클래스들을 엮어 애플리케이션을 빠르고 안정적으로 구축
- 클래스 작성자
  - 구현 은닉, 클라이언트 프로그래머가 내부에게 필요한 부분만 공개

설계가 필요한 이유는 변경을 관리하기 위한 것, 객체의 변경을 관리할 수 있는 기법 중 가장 대표적인 것이 바로 접근 제어

##### 협력하는 객체들의 공동체
시스템의 기능을 구현하기 위해 객체들 사이에 이뤄지는 상호작용을 협력이라고 부름.

객체지향을 작성할 때는 먼전
- 어던 객체가 필요하지 결정
- 객체들의 공통 상태와 행위를 구현하기 위해 클래스 작성

##### 협력에 관한 짧은 이야기
- 객체의 내부는 외부에서 접근하지 못하도록 감춰야 한다.
- 외부에 공개하는 퍼블릭 인터페이스를 통해 접근 가능
- 요청 받은 객체는 자율적인 방법에 따라 요청을 처리 후 응답

#### 할인 요금 구하기
- DiscountPolicy 는 중복 코드를 제거하기 위해 공통 코드를 보관할 장소 필요,
- 실제 애플리케이션에서는 DiscountPolicy의 인스턴스를 생성할 필요가 없기에 추상 클래스로 구현

#### 버전1

<details>
<summary>버전1 코드</summary>

- 손님 : Customer

````java
public class Customer {
}
````

- 예매 : Reservation

````java
public class Reservation {
  private Customer customer;
  private Screening screening;
  private Money fee;
  private int audienceCount;

  public Reservation(Customer customer, Screening screening, Money fee, int audienceCount) {
    this.customer = customer;
    this.screening = screening;
    this.fee = fee;
    this.audienceCount = audienceCount;
  }
}
````

- 상영 : Screening

````java
import java.time.LocalDateTime;

public class Screening {
  private Movie movie;
  private int sequence;
  private LocalDateTime whenScreened;

  public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
    this.movie = movie;
    this.sequence = sequence;
    this.whenScreened = whenScreened;
  }

  public Reservation reserve(Customer customer, int audienceCount) {
    return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
  }

  private Money calculateFee(int audienceCount) {
    return movie.calculateMovieFee(this).times(audienceCount);
  }

  public LocalDateTime getStartTime() {
    return whenScreened;
  }

  public boolean isSequence(int sequence) {
    return this.sequence == sequence;
  }

  public Money getMovieFee(){
    return movie.getFee();
  }
}
````

- 돈 : Money

````java
import java.math.BigDecimal;

public class Money {
    public static final Money ZERO = Money.wons(0);

    private final BigDecimal amount;

    private Money(BigDecimal amount) {
        this.amount = amount;
    }

    public static Money wons(long amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public static Money wons(double amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public Money plus(Money amount) {
        return new Money(this.amount.add(amount.amount));
    }

    public Money minus(Money amount) {
        return new Money(this.amount.subtract(amount.amount));
    }

    public Money times(double percent) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(percent)));
    }

    public boolean isLessThan(Money other) {
        return amount.compareTo(other.amount) < 0;
    }

    public boolean isGreaterThanOrEqual(Money other) {
        return amount.compareTo(other.amount) >= 0;
    }
}
````

- 영화 : Movie

````java
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
````

- 할인 정책 : DiscountPolicy

````java
import java.util.Arrays;
import java.util.List;

public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions;

    public DiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition condition : conditions) {
            if (condition.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }

        return Money.ZERO;
    }

    abstract protected Money getDiscountAmount(Screening screening);
}
````

- 금액 할인 정책 : AmountDiscountPolicy

````java
public class AmountDiscountPolicy extends DiscountPolicy {
    private Money discountAmount;

    public AmountDiscountPolicy(Money discountAmount, DiscountCondition... conditions) {
        super(conditions);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return discountAmount;
    }
}
````

- 기간 할인 정책 : PercentDiscountPolicy

````java
public class PercentDiscountPolicy extends DiscountPolicy {
    private double percent;

    public PercentDiscountPolicy(double percent, DiscountCondition... conditions) {
        super(conditions);
        this.percent = percent;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return screening.getMovieFee().times(percent);
    }
}
````

- 할인 조건 : DiscountCondition

````java
public interface DiscountCondition {
  boolean isSatisfiedBy(Screening screening);
}
````

- 순번 할인 조건 : SequenceCondition

````java
public class SequenceCondition implements DiscountCondition {
  private int sequence;

  public SequenceCondition(int sequence) {
    this.sequence = sequence;
  }

  @Override
  public boolean isSatisfiedBy(Screening screening) {
    return screening.isSequence(sequence);
  }
}
````

- 기간 할인 조건 : PeriodCondition

````java
import java.time.DayOfWeek;
import java.time.LocalTime;

public class PeriodCondition implements DiscountCondition {
  private DayOfWeek dayOfWeek;
  private LocalTime startTime;
  private LocalTime endTime;

  public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
    this.dayOfWeek = dayOfWeek;
    this.startTime = startTime;
    this.endTime = endTime;
  }

  @Override
  public boolean isSatisfiedBy(Screening screening) {
    return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
            startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
            endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
  }
}
````

##### Templat Method Pattern : 부모 클래스에 기본적인 알고리즘의 흐름을 구현하고, 중간에 필요한 처리를 자식 클래스에 위임

![class-diagram-chapter2-v1-1.png](v1%2Fclass-diagram-chapter2-v1-1.png)
</details>

#### 상속과 다형성

##### 컴파일 시간 의존성과 실행 시간 의존성
코드의 의존성과 실행 시점의 의존성이 다를 수 있다. 즉, 클래스 사이의 의존성과 객체 사이의 의존성이 동일하지 않을 수 있다.

유연하고, 쉽게 재사용할 수 있으며, 확장 가능한 객체지향 설계가 가지는 특징, 하지만 코드의 의존성과 실행 시점의 의존성이 다르면 코드를 이해하기 어려워 진다.

훌륭한 객체지향을 하기 위해 항상 유연성과 가독성 사이에서 고민해야 한다.
무조건 유연한 설계, 무조건 읽기 쉬운 코드가 정답이 아니다.

##### 차이에 의한 프로그래밍 : 부모 클래스와 다른 부분만 추가해서 새로운 클래스를 쉽고 빠르게 만드는 방법

##### 다형성
동일한 메시지를 전송하지만 실제로 어떤 메서드가 실행될 것인지는 메시지를 수신하는 객체의 클래스가 무엇이냐에 따라 달라진다. 이를 다형성이라 부른다.

다형성을 구현하는 방법은 다양하지만 실행될 메서드를 컴파일 시점이 아닌 실행 시점에 결정한다는 공통점이 있다.
이를 지연 바인딩(lazy binding) 또는 동적 바인딩(dynamic binding) 이라 한다. 컴파일 시점에 실행될 함수나 프로시저를 경정하는 것을 
초기 바인딩(early binding) 또는 정적 바인딩(static binding) 이라 한다.

##### 추상화와 유연성
###### 추상화의 힘
- 추상화의 장점
  - 추상화의 계층만 따로 떼어 놓고 살펴보면 요구사항의 정책을 높은 수준에서 서술 가능
  - 추상화를 이용하면 설계가 좀 더 유연

- 상위 정책 기술 -> 기본적인 애플리케이션의 협력 흐름 기술
- 상위 정책을 표현하면 기존 구조를 수정하지 않고도 새로운 기능 추가, 확장 가능

###### 유연한 설계
할인 정책 없는 영화라면 어떻게 할까?
````java
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        if (discountPolicy == null) {
            return fee;
        }
        
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
````
문제점 : 할인 정책이 없는 경우 예외 케이스 취급, 일관성이 무너짐
- 책임의 위치를 결정하기 위해 조건문을 사용하는 것은 협력의 설계 측면에서 대부분 좋지 않은 선택

````java
public class NoneDiscountPolicy extends DiscountPolicy {
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.ZERO;
    }
}
````
일관된 설계를 위해 NoneDiscountPolicy을 추가

Movie와 DiscountPolicy를 수정하지 않고 NoneDiscountPolicy, 새로운 클래스를 추가하여 애플리케이션의 기능 확장

###### 추상 클래스와 인터페이스 트레이드 오프

<details>
<summary>버전2 코드</summary>

- DiscountPolicy

````java
public interface DiscountPolicy {
  Money calculateDiscountAmount(Screening screening);
}
````

- DefaultDiscountPolicy

````java
import java.util.Arrays;
import java.util.List;

public abstract class DefaultDiscountPolicy implements DiscountPolicy {
  private List<DiscountCondition> conditions;

  public DefaultDiscountPolicy(DiscountCondition... conditions) {
    this.conditions = Arrays.asList(conditions);
  }

  @Override
  public Money calculateDiscountAmount(Screening screening) {
    for (DiscountCondition condition : conditions) {
      if (condition.isSatisfiedBy(screening)) {
        return getDiscountAmount(screening);
      }
    }

    return Money.ZERO;
  }

  abstract protected Money getDiscountAmount(Screening screening);
}
````

- 상영 : AmountDiscountPolicy

````java
public class AmountDiscountPolicy extends DefaultDiscountPolicy {
  private Money discountAmount;

  public AmountDiscountPolicy(Money discountAmount, DiscountCondition... conditions) {
    super(conditions);
    this.discountAmount = discountAmount;
  }

  @Override
  protected Money getDiscountAmount(Screening screening) {
    return discountAmount;
  }
}
````

- PercentDiscountPolicy

````java
public class PercentDiscountPolicy extends DefaultDiscountPolicy {
  private double percent;

  public PercentDiscountPolicy(double percent, DiscountCondition... conditions) {
    super(conditions);
    this.percent = percent;
  }

  @Override
  protected Money getDiscountAmount(Screening screening) {
    return screening.getMovieFee().times(percent);
  }
}
````

- NoneDiscountPolicy

````java
public class NoneDiscountPolicy implements DiscountPolicy {
  @Override
  public Money calculateDiscountAmount(Screening screening) {
    return Money.ZERO;
  }
}
````

</details>

- 위 코드는 이상적으로 인터페이스를 사용하도록 변경한 설계이다.
- 현실적으로는 인터페이스 추가가 과하다고 생각할 수 있다.

-> 구현과 관련된 모든 것들이 트레이드오프의 대상이 될 수 있다.

###### 상속 VS 합성
- 상속 단점
  - 상속이 캡슐화 위반
    - 부모 클래스의 내부 구조를 잘 안다.
    - 자식 클래스가 부모 클래스에 강하게 결합, 부모 클래스 변경시 자식 클래스도 변경될 확률 높음
  - 설계를 유연하지 못하게 만듬
    - 부모와 자식 관계를 컴파일 시점에 결정, 따라서 실행 시점에 객체의 종류 변경 불가능

- 합성은 상속이 가지는 두 가지 문제점 모두 해결
- 메시지를 통해서만 재사용이 가능하기에 캡슐화 가능
- 의존하는 인스턴스를 교체하는 것이 비교적 쉽기에 슈연한 설계 가능

그렇다고 상속을 절대 사용하지 말라는 것은 아님, 대부분 상속과 합성 함께 사용
