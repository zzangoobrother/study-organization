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

[영화예매시스템 리팩토링 v1](https://github.com/zzangoobrother/study-project/tree/master/book-objects/src/main/java/chapter4/v1)
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
##### 캡슐화
- 변경될 가능성이 높은 부분을 구현이라 부르고, 상대적으로 안정적인 부분을 인터페이스라 부른다.
- 캡슐화의 중요성 : 불안정한 부분과 안정적인 부분을 분리해서 변경의 영향을 통제할 수 있기 때문

##### 응집도
- 모듈에 포함된 내부 요소들이 연관돼 있는 정도
- 변경이 발생할 때 모듈 내부에서 발생하는 변경의 정도

##### 결합도
- 의존성의 정도
- 한 모듈이 변경되기 위해서 다른 모듈의 변경을 요구하는 정도

좋은 설계란 오늘의 기능을 수행하면서 내일의 변경을 수용할 수 있는 설계

#### 데이터 중신의 영화 예매 시스템의 문제점
- 캡슐화 위반
  - Movie의 fee는 접근자 제한자로 제한을 하지만 getFee(), setFee()로 변수의 존재 명시

- 높은 결합도
  - 데이터 중심 설계는 객체의 캡슐화를 약화시키기 때문에 클라이언트가 객체의 구현에 강하게 결합
  - 하나의 제어 객체가 다수의 데이터 객체에 강하게 결합

- 낮은 응집도
  - 변경과 아무 상관 없는 코드들이 영향을 받게 된다.
  - 하나의 요구사항 변경을 반영하기 위해 동시에 여러 모듈을 수정해야 한다.

#### 자율적인 객체를 향해
##### 캡슐화를 지켜라

```java
class Rectangle {
    private int left;
    private int top;
    private int right;
    private int bottom;

    public Rectangle(int left, int top, int right, int bottom) {
        this.left = left;
        this.top = top;
        this.right = right;
        this.bottom = bottom;
    }

    public int getLeft() {
        return left;
    }

    public void setLeft(int left) {
        this.left = left;
    }

    public int getTop() {
        return top;
    }

    public void setTop(int top) {
        this.top = top;
    }

    public int getRight() {
        return right;
    }

    public void setRight(int right) {
        this.right = right;
    }

    public int getBottom() {
        return bottom;
    }

    public void setBottom(int bottom) {
        this.bottom = bottom;
    }
}

class AnyClass {
    void anyMethod(Rectangle rectangle, int multiple) {
        rectangle.setRight(rectangle.getRight() * multiple);
        rectangle.setBottom(rectangle.getBottom() * multiple);
    }
}
```

- 코드 중복 발생 : 사각형의 너비와 높이를 증가시키는 코드가 필요하다면, right와 bottom을 가져온 후 수정자 메서드를 이용
- 변경에 취약 : right, bottom -> length, height 변경 한다면, 기존 접근자 메서드를 사용하던 모든 코드에 영향을 미친다.
```java
class Rectangle {
    public void enlarge(int multiple) {
        right *= multiple;
        bottom *= multiple;
    }
}
```

- Rectangle 변경 주체를 외부의 객체에서 Rectangle로 이동
- 책임을 객체 내부로 이동

##### 스스로 자신의 데이터를 책임지는 객체
- 이 객체가 어떤 데이터를 포함해야 하는가?
  - 이 객체가 어떤 데이터 를 포함해야 하는가?
  - 이 객체가 데이터에 대해 수행해야 하는 오퍼레이션은 무엇인가?

[영화예매시스템 리팩토링 v2](https://github.com/zzangoobrother/study-project/tree/master/book-objects/src/main/java/chapter4/v2)

##### 하지만 여전히 부족하다
- 캡슐화 위반
  - isDiscountable(DayOfWeek dayOfWeek, LocalTime time), isDiscountable(int sequence)의 파라미터로 DayOfWeek, time, sequence 의 파라미터가 포함된다는 것을 보여준다.
    내부 구현이 노출되면서 캡슐화 부족 '파급 효과'가 발생한다.
  - Movie도 calculateAmountDiscountFee, calculatePercentDiscountFee, calculateNoneDiscountFee 메소드로 어떤 할인 정책을 외부에 노출시키고 있다.

- 높은 결합도
  - 두 객체 사이에 결합도가 높을 경우 한 객체의 구현을 변경할 때 다른 객체에게 변경의 영향이 전파될 확률이 높아진다.
  - DiscountCondition이 변경된다면 Movie에게 영향을 미칠수 있다.
    - 할인 조건 추가/삭제/변경
  - 유연한 설계를 창조하기 위해 캡슐화를 설계의 첫 번째 목표로 삼아야 한다.

- 낮은 응집도
  - DiscountCondition이 변경된다면 Movie 뿐만 아니라 Movie의 isDiscountable 메서드를 사용하는 Screening도 수정해야 한다.

##### 데이터 중심 설계의 문제점
- 데이터 중심 설계가 변경에 취약한 이유
  - 본질적으로 너무 이른 시기에 데이터에 관해 결정하도록 강요한다.
  - 협력이라는 문맥을 고려하지 않고 객체를 고립시킨 채 오퍼레이션을 결정한다.

- 데이터 중심 설계는 객체의 행동보다는 상태에 초점을 맞춘다
  - 데이터 중심 설계에 익숙한 개발자들은 일반적으로 데이터와 기능을 분리하는 절차적 프로그래밍 방식을 따른다. 상태와 행동을 하나의 단위로 캡슐화하는 객체지향 패러다임에 반하는 것
  - 데이터를 먼저 결정하고 데이터를 처리하는 데 필요한 오퍼레이션을 나중에 결정하는 방식은 데이터에 관한 지식이 객체의 인터페이스에 노출
  결론적으로 데이터 중심 설계는 너무 이른 시기에 데이터에 대해 고민하기 때문에 캡슐화에 실패하게 된다.

- 데이터 중심 설계는 객체를 고립시킨 채 오퍼레이션을 정의하도록 만든다
  - 올바른 객체지향은 무게 중심을 항상 객체 내부가 아니라 외부에 맞춰져 있어야 한다.
  - 중요한 것은 객체가 다른 객체와 협력하는 방법이다.
  - 객체의 구현이 이미 결정된 상태에서 다른 객체와의 협력 방법을 고민하기 때문에 이미 구현된 객체의 인터페이스를 억지로 끼워맞출 수밖에 없다.
  - 객체의 인터페이스에 구현이 노출 -> 협력이 구현 세부사항에 종속 -> 객체 내부 구현이 변경됐을 때 협력하는 객체 모두 영향을 받을 수밖에 없다.
