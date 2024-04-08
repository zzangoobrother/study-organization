### 유연한 설계
#### 개방-폐쇄 원칙
- 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.
    - 확장에 대해 열려 있다 : 애플리케이션의 요구사항이 변경될 때 이 변경에 맞게 새로운 '동작'을 추가해서 애플리케이션의 기능을 확장할 수 있다.
    - 수정에 대해 닫혀 있다 : 기존의 '코드'를 수정하지 않고도 애플리케이션의 동작을 추가하거나 변경할 수 있다.

-> 기존 코드를 수정하지 않고도 애플리케이션의 동작을 확장할 수 있는 설계

##### 컴파일타임 의존성을 고정시키고 런타임 의존성을 변경하라
- 런타임 의존성 : 실행시에 협력에 참여하는 객체들 사이의 관계
- 컴파일타임 의존성 : 코드에서 드러나는 클래스들 사이의 관계

![P20240404_000458066_67FB5EEF-1677-4208-8869-984930441EB9.JPG](image_9_1.JPG)

위 그림에서
- 컴파일 의존성 관점 : Movie 클래스는 추상 클래스인 DiscountPolicy에 의존
- 런타임 의존성 관점 : Movie 인스턴스는 AmountDiscountPolicy와 PercentDiscountPolicy에 의존

![image_9_2.JPG](image_9_2.JPG)

위 그림은 중복 할인 정책을 구현하는 OverlappedDiscountPolicy 클래스 추가

개방-폐쇄 원칙을 따르는 설계란 컴파일타임 의존성은 유지하면서 런타임 의존성의 가능성을 확장하고 수정할 수 있는 구조

##### 추상화가 핵심이다
개방-폐쇄 원칙의 핵심은 추상화에 의존하는 것

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public abstract class DiscountPolicy {
  private List<DiscountCondition> conditions = new ArrayList<>();

  public DiscountPolicy(DiscountCondition... conditions) {
    this.conditions = Arrays.asList(conditions);
  }
  
  public Money calculateDiscountAmount(Screening screening) {
      for (DiscountCondition each : conditions) {
          if (each.isSatisfiedBy(screening)) {
              return getDiscountAmount(screening);
          }
      }
      
      return screening.getMovieFee();
  }
  
  abstract protected Money getDiscountAmount(Screening screening);
}
```

- 추상화 과정을 통해 생략된 부분은 할인 요금 계산
- 상속을 통해 생략된 부분을 구체화함으로써 할인 정책을 확장

```java
public class Movie {
  ...
  private DiscountPolicy discountPolicy;
  
  public Movie(String title, Duration ruuningTime, Money fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;
  }
  
  public Money calculateMovieFee(Screening screening) {
      return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

- 할인 정책을 추상화한 DiscountPolicy에만 의존
- DiscountPolicy의 자식 클래스를 추가하더라도 영향을 받지 않음

- 변하는 것과 변하지 않는 것이 무엇인지 이해하고, 이를 추상화의 목적으로 삼아야 한다.
- 변경되지 않을 부분을 신중하게 결정하고 올바른 추상화를 주의 깊게 선택했기 때문이라는 사실 기억

#### 생성 사용 분리
- 아래 코드는 추가하거나 변경하기 위해 기존 코드를 수정하도록 만들기 때문에 개방-폐쇄 원칙을 위반한다.
```java
public class Movie {
  ...
  private DiscountPolicy discountPolicy;
  
  public Movie(String title, Duration runningTime, Money fee) {
    ...
    this.discountPolicy = new AmountDiscountPolicy(...);
  }
  
  public Money calculateMovieFee(Screening screening) {
      return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```
- 결합도가 높아질수록 개방-폐쇄 원칙을 따르는 구조를 설계하기가 어려워진다.
- 문제는 객체 생성이 아니라, 부적절한 곳에서 객체를 생성한다는 것이 문제다.

문제점
- 메시지를 전송하지 않고 객체를 생성하기만 한다면 아무런 문제가 없다.
- 또는 객체를 생성하지 않고 메시지를 전송하기만 했다면 괜찮았을 것이다.
- 문제는 동일한 클래스 안에서 객체 생성과 사용이라는 두 가지 목적을 가진 코드의 공존이다.

객체에 대한 생성과 사용을 분리해야 한다.

보편적인 방법
- 객체 생성 책임을 클라이언트로 옮김

##### FACTORY 추가하기
FACTORY : 생성과 사용을 분리하기 위해 객체 생성에 특화된 객체

```java
import java.time.Duration;

public class Factory {
  public Movie createAvatarMovie() {
    return new Movie("아바타",
            Duration.ofMinutes(120),
            Money.wons(10000),
            new AmountDiscountPolicy(...));
  }
}

public class Client {
    private Factory factory;
    
    public Client(Factory factory) {
        this.factory = factory;
    }
    
    public Money getAvatarFee() {
        Movie avatar = factory.createAvatarMovie();
        return avatar.getFee();
    }
}
```
- Client는 오직 사용과 관련된 책임만 지고 생성과 관련되 어떤 지식도 가지지 않는다.

##### 순수한 가공물에게 책임 할당하기

