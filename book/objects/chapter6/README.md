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

디미터 법칙은 훌륭한 메시지는 객체의 상태에 관해 묻지 말고 원하는 것을 시켜야 한다는 사실을 강조
 - 절차적인 코드는 정보를 얻은 후에 결정한다. 객체지향 코드는 객체에게 그것을 하도록 시킨다.

객체지향의 기본은 함께 변경될 확률이 높은 정보와 행동을 하나의 단위로 통합하는 것이다.

객체의 정보를 이용하는 행동을 객체의 외부가 아닌 내부에 위치시키기 때문에 자연스럽게 정보와 행동을 동일한 클래스 안에 두게 된다.

내부의 상태를 묻는 오퍼레이션을 인터페이스에 포함시키고 있다면 더 나은 방법은 없는지 고민해 보라

##### 의도를 드러내는 인터페이스
컨트 벡은 <Smalltalk Best Practice Patterns>에서 메서드를 명명하는 두 가지 방법을 설명했다.

<table>
<tr>
<th>메서드의 이름은 내부의 구현 방법을 드러낸다</th>
<th>'어떻게'가 아니라 '무엇'을 하는지를 드러낸다.</th>
</tr>

<tr><td>

````java
public class PeriodCondition {
    public boolean isSatisfiedByPeriod(Screening screening) {...}
}

public class SequenceCondition {
  public boolean isSatisfiedBySequence(Screening screening) {...}
}
````
위 코드가 좋지 않은 이유 두 가지
- 동일한 작업을 수행한다. 하지만 메서드의 이름이 다르기 때문에 두 메서드가 동일한 작업을 수행한다는 사실을 알기 어렵다.
- 메서드 수준에서 캡슐화를 위반한다. 메서드 이름을 변경한다면 메시지를 전송하는 클라이언트의 코드도 함께 변경해야 한다.
</td>

<td>

- 객체가 협력 안에서 수행해야 하는 책임에 관해 고민 해야함
  - 외부의 객체가 메시지를 전송하는 목적을 먼저 생각하도록 만듦
  - 결과적으로 협력하는 클라이언트의 의도에 부합하도록 메서드 이름을 짓게 된다.

````java
public class PeriodCondition {
  public boolean isSatisfiedBy(Screening screening) {...}
}

public class SequenceCondition {
  public boolean isSatisfiedBy(Screening screening) {...}
}
````

- 클라이언트 입장에서 두 메서드는 동일한 메시지르 서로 다른 방법으로 처리하기 때문에 서로 대체 가능하다.

````java
public interface DiscountCondition { 
    boolean isSatisfiedBy(Screening screening);
}
````
</td>
</tr>
</table>

메서드에 의도를 드러낼 수 있는 이름을 붙이기 위한 조언
- 매두 다른 두 번째 구현을 상상하라, 해당 메서드에 동일한 이름을 붙인다고 상상해보라, 그렇게 하면 아마도 가장 추상적인 이름을 메서드에 붙일 것이다.

결과와 목적만을 포함하도록 클래스와 어퍼레이션의 이름을 부여하라.

방정식을 푸는 방법을 제시하지 말고 이를 공식으로 표현하라. 문제를 내라. 하지만 문제를 푸는 방법을 표현해서는 안 된다.

#### 함께 모으기
- 디미터 법칙을 위반하는 티켓 판매 도메인
````java
public class Theater { 
    private TicketSeller ticketSeller;
    
    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }
    
    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            // ticketSeller가 TicketOffice 뿐만 아니라 getTicketOffice 포함된 Ticket도 알게 된다.
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
          Ticket ticket = ticketSeller.getTicketOffice().getTicket();
          // 기차 충돌 스타일
          // 내부 구조에 대해 결합
          audience.getBag().minusAmount(ticket.getFee());
          ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
          audience.getBag().setTicket(ticket);
        }
    }
}
````
- 디미터 법칙을 위반한 코드를 수정하는 일반적인 방법 : Audience와 TicketSeller의 내부 구조를 묻는 대신 직접 자신의 책임을 수행하도록 시키는 것

##### 묻지 말고 시켜라
- Theater가 TicketSeller에게 시키고 싶은 일을 가지도록 만들도록 수정
  - TicketSeller에게 setTicket() 추가
````java
public class Theater {
    public void enter(Audience audience) {
        ticketSeller.setTicket(audience);
    }
}
````

````java
public class TicketSeller {
    public void setTicket(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketOffice.getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketOffice.getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketOffice.plusAmount(ticket.getFee());
        }
    }
}
````

- Audience가 스스로 티켓을 가지도록 만들자
````java
public class Audience {
    public void setTicket(Ticket ticket) {
      if (bag.hasInvitation()) {
        bag.setTicket(ticket);
        return 0L;
      } else {
        bag.setTicket(ticket);
        bag.minusAmount(ticket.getFee());
        return ticket.getFee();
      }
    }
}
````

````java
public class TicketSeller {
    public void setTicket(Audience audience) {
        ticketOffice.plusAmount(audience.setTicket(ticketOffice.getTicket()));
    }
}
````

- Audience가 Bag에게 원하는 일을 시키기 전에 hasInvitation 메서드를 이용해 초대권을 가지고 있는지 묻고 있음, 디미터 법칙 위반
````java
public class Bag {
    public Long setTicket(Ticket ticket) {
        if (hasInvitation()) {
            this.ticket = ticket;
            return 0L;
        } else {
            this.ticket = ticket;
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
    
    private boolean hasInvitation() {
        return invitation != null;
    }
    
    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
````

````java
public class Audience {
    public void setTicket(Ticket ticket) {
      return bag.setTicket(ticket);
    }
}
````

##### 인터페이스에 의도를 드러내자
현재의 인터페이스는 클라이언트의 의도를 명확하게 드러내지 못한다.

- Theater -> TicketSeller에게 setTicket => Audience에게 티켓을 판매
  - setTicket -> sellTo 가 의도를 더 명확하게 표현
- TicketSeller -> Audienct에게 setTicket => Audience가 티켓을 사도록 만드는 목적
  - setTicket -> buy 변경
- Audienct -> Bag에게 setTicket => 티켓을 보관하도록 만드는 목적
  - setTicket -> hold 변경

### 원칙의 함정
#### 디미터 법틱은 하나의 도트(.)를 강제하는 규칙이 아니다

````java
IntStream.of(1, 15, 20, 3, 9).filter(x -> x > 10).distinct().count();
````
디미터 법칙을 위반하지 않았다.
-> IntStream을 다른 IntStream으로 변환할 뿐이다.

객체 내부 구현에 대한 어떤 정보도 외부로 노출하지 않았다.

#### 결합도와 응집도의 충돌
<table>
<tr>
<th>Before</th>
<th>After</th>
</tr>

<tr><td>

````java
public class Theater { 
    public void enter(Audience audience) {
        // Bag에 질문
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            // Bag 상태 변경
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
````
</td>

<td>

````java
public class Audience { 
    public void setTicket(Ticket ticket) {
        if (bag.hasInvitation()) {
            bag.setTicket(ticket);
            return 0L;
        } else {
            bag.setTicket(ticket);
            bag.minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
````
</td>
</tr>
</table>

안타깝게도 묻지 말고 시켜라와 디미터 법칙이 항상 긍정적인 결과로만 귀결되는 것은 아니다.
- 맹목적으로 위임 메서드를 추가하면 어울리지 않는 오퍼레이션들 공존
- 결과적으로 응집도가 낮아짐

<table>
<tr>
<th>Before</th>
<th>After</th>
</tr>

<tr><td>

````java
public class PeriodCondition implements DiscountCondition { 
    public boolean isSatisfiedBy(Screening screening) {
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
                startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
                endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
    }
}
````
Screening 의 내부 상태를 가져와서 사용하기 때문에 캡슐화를 위반한 것으로 보임
</td>

<td>

````java
import java.time.DayOfWeek;
import java.time.LocalTime;

public class Screening {
  public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
      return whenScreened.getDayOfWeek().equals(dayOfWeek) && 
              startTime.compareTo(whenScreened.toLocalTime()) <= 0 &&
              endTime.compareTo(whenScreened.toLocalTime()) >= 0;
  }
}

public class PeriodCondition implements DiscountCondition {
    public boolean isSatisfiedBy(Screening screening) {
        return screening.isDiscountable(dayOfWeek, startTime, endTime);
    }
}
````

</td>
</tr>
</table>

- Screening이 담당해야 하는 본질적인 책임과 멀다
- Screening의 본질적인 책임은 영화 예매이다.
- PeriodCondition의 입장은 할인 조건을 판단하는 책임이 본질적이다.

가끔씩은 묻는 것 외에 다른 방법이 존재하지 않는 경우도 존재
````java
for(Movie each : movies) {
    total += each.getFee();
}
````

로버트 마틴 <클린코드>에서 디미터 법칙의 대상이 객체인지 자료구조인지 달렸다고 한다.

객체 내부 구조를 숨겨야 하므로 디미터 법칙을 따르는게 좋고, 자료 구조라면 내부를 노출해야 한다.

- 소프트웨어 설계에 법칙이란 존재하지 않는다.
- 원칙을 맹신하지 마라
- 적절한 상황과 부적절한 상황을 판단하는 안목을 길러라
- 설계는 트레이드오프의 산물이다.
- 소프트웨어 설계에 존재하는 몇 안 되는 법칙 중 하나는 "경우에 따라 다르다"라는 사실이다.

### 명령-쿼리 분리 원칙
<table>
<tr>
<th>프로시저</th>
<th>함수</th>
</tr>

<tr>
<td>

- 정해진 절차에 따라 내부의 상태를 변경
- 부수효과를 발생시킬 수 있지만 값을 반환할 수 없다.

</td>

<td>

- 어떤 절차에 따라 필요한 값을 계산해서 반화는 루틴
- 값을 반환할 수 있지만 부수효과를 발생시킬 수 없다.

</td>
</tr>
</table>

<table>
<tr>
<th>명령</th>
<th>쿼리</th>
</tr>

<tr>
<td>

- 객체의 상태를 수정하는 오퍼레이션
- 객체의 상태를 변경하는 명령은 반환값을 가질 수 없다.

</td>

<td>

- 객체와 관련된 정보를 반환하는 오퍼레이션
- 객체의 정보를 반환하는 쿼리는 상태를 변경할 수 없다.

</td>
</tr>
</table>

##### 반복 일정의 명령과 쿼리 분리하기

````java
public class Event {
  public boolean isSatisfied(RecurringSchedule schedule) {
    if (from.getDayOfWeek() != schedule.getDayOfWeek() ||
            !from.toLocalTime().equals(schedule.getFrom())) {
      reschedule(schedule);
      return false;
    }

    return true;
  }

  private void reschedule(RecurringSchedule schedule) {
    from = LocalDateTime.of(from.toLocalDate().plusDays(daysDistance(schedule)), schedule.getFrom());
    duration = schedule.getDuration();
  }

  private long daysDistance(RecurringSchedule schedule) {
    return schedule.getDayOfWeek().getValue() - from.getDayOfWeek().getValue();
  }
}
````
- isSatisfied 메서드에는 RecurringSchedule의 조건에 부합하는지 판단
- 명령과 쿼리를 뒤섞으면 예측하기 어렵다.
- 내부적으로 부수효과를 가지는 메서드는 이해하기 어렵고, 잘못 하용하기 쉬우며, 버그를 양상하는 경향이 있다.
- 명령과 쿼리를 명확하게 분리 해보자

````java
public class Event {
  public boolean isSatisfied(RecurringSchedule schedule) {
    if (from.getDayOfWeek() != schedule.getDayOfWeek() ||
            !from.toLocalTime().equals(schedule.getFrom()) ||
            !duration.eqquals(schedule.getFrom())) {
      return false;
    }

    return true;
  }

  public void reschedule(RecurringSchedule schedule) {
    from = LocalDateTime.of(from.toLocalDate().plusDays(daysDistance(schedule)), schedule.getFrom());
    duration = schedule.getDuration();
  }

  private long daysDistance(RecurringSchedule schedule) {
    return schedule.getDayOfWeek().getValue() - from.getDayOfWeek().getValue();
  }
}
````

##### 명령-쿼리 분리와 참조 투명성
명령과 쿼리를 분리함으로 명령형 언어의 틀 안에서 참조 투명성의 장점을 제한적으로 누릴 수 있다.
- 참조 투명성 : 어떤 표현식 e가 있을 때 e의 값으로 e가 나타나는 모든 위치를 교체하더라도 결과가 달라지지 않는 특성
- 불변성 : 어떤 값이 변하지 않는 성질 => 부수효과가 발생하지 않는다

##### 책임에 초점을 맞춰라
- 디미터 법칙 : 메시지가 객체를 선택하게 함으로써 의도적으로 디미터 법칙을 위반할 위험 최소화
- 묻지 말고 시켜라 : 메시지를 먼저 선택하면 묻지 말고 시켜라
- 의도를 드러내는 인터페이스 : 클라이언트의 관점에서 메시지의 이름을 정한다, 클라이언트가 무엇을 원하는지, 그 의도가 분명하게 드러난다.
- 명령-쿼리 분리 원칙 : 협력 속에서 객체의 상태를 예측하고 이해하기 쉽게 만들기 위한 방법에 관해 고민하게 됨
