### 객체, 설계

#### 이론 vs 설계
- 이론을 적립할 수 없는 초기에는 실무가 먼저 발전
- 짧은 역사를 가진 소프트웨어, 이론보다 실무가 더 앞서 있기에 실무가 중요

##### 소프트웨어 설계와 솥프트웨어 유지보수
- 대부분 설계 원치과 개념은 실무에서 반복적으로 적용된 기법들이 이론화 됨
- 다양한 규모의 소프트웨어를 유지보수하지만 유지보수와 관련된 효과적인 이론이 발표된 적은 거의 없음

### 티켓 판매 어플리케이션 구현하기
#### 요구사항

- 추첨을 통해 일부 관란객에게 무료 관람권 증정
- 이벤트 당첨 관람객 : 무료 입장
- 이벤트 미당첨 관람객 : 티켓 구매 필수

#### 버전1

<details>
<summary>버전1 코드</summary>

- 이벤트 당첨자 초대장 : Invitation

````java
import java.time.LocalDateTime;

public class Invitation {
    private LocalDateTime when;
}
````

- 티켓 : Ticket -> 이벤트 당첨자는 초대장을 티켓으로 교환

````java
public class Ticket {

    private Long fee;

    public Long getFee() {
        return fee;
    }
}
````

- 티켓 : Ticket -> 이벤트 당첨자는 초대장을 티켓으로 교환

````java
public class Ticket {

    private Long fee;

    public Long getFee() {
        return fee;
    }
}
````

- 관람객의 가방 : Bag

````java
public class Bag {

    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

    public Bag(long amount) {
        this(null, amount);
    }

    public Bag(Invitation invitation, long amount) {
        this.invitation = invitation;
        this.amount = amount;
    }

    public boolean hasInvitation() {
        return invitation != null;
    }

    public boolean hasTicket() {
        return ticket != null;
    }

    public void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
````

- 관람객 : Audience

````java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Bag getBag() {
        return bag;
    }
}
````

- 매표소 : TicketOffice

````java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public TicketOffice(Long amount, Ticket... tickets) {
        this.amount = amount;
        this.tickets.addAll(Arrays.asList(tickets));
    }

    public Ticket getTicket() {
        return tickets.remove(0);
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
````

- 판매원 : TicketSeller

````java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public TicketOffice getTicketOffice() {
        return ticketOffice;
    }
}
````

- 소극장 : Theater

````java
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
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

![class-diagram-v1.png](v1%2Fclass-diagram-v1.png)
</details>

### 무엇이 문제인가
#### 소프트웨어 모듈의 세가지 기능

- 실행 중 제대로 동작
- 변경을 위해 존재: 간단한 작업만으로 변경 가능해야 함
- 코드를 읽는 사람과 의사소통

버전1의 코드는 제대로 동작하지만 변경 용이성과 읽는 사람과의 의사소통을 만족하지 못한다.

#### 예상을 빗나가는 코드

Theater 코드를 보면 관람객과 판매원이 소극장의 통제를 받는 수동적인 존재이다.
우리가 예상하는 방식으로 동작하는 코드가 되어야 한다.

- 관람객이 초대장이나 돈을 지불하여 티켓 구입
- Theater의 enter에서 관람객의 가방을 확인, 우리의 예상을 벗어남
- 제대로 의사소통을 하지 못 함

#### 변경에 취약한 코드

- 고객이 현금말고 카드를 사용한다면, 판매원이 판매소 밖에서 티켓을 판매한다면 수정해야 할 범위가 넓어진다. 

1. 의존성
- 객체 사이의 의존성 문제 발생
- 어떤 객체가 변경될 때 그 객체에게 의존하는 다른 객체도 변경될 수 있다.
- 그렇다고 의존성을 완전히 없애는 것이 정답은 아님
- 우리의 목표는 최소한의 의존성만 유지

2. 결합도
- 객체 사이의 의존성이 강한 경우: 결합도가 높다
- 합리적인 수준으로 의존할 경우: 결합도가 낮다
- 결합도가 높을수록 변경될 확률도 높아진다.

### 설계 개선하기

관람객과 판매원을 자율적인 존재로 만들자

Theater의 enter에서 TicketOffice에 접근하는 코드를 TicketSeller 내부로 숨긴다.

다음으로 TicketSeller가 Bag에 접근하는 코드를 Audience 내부로 숨기자

<details>
<summary>버전2 변경된 코드</summary>

- 소극장 : Theater

````java
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        ticketSeller.sellTo(audience);
    }
}
````

- 판매원 : TicketSeller

````java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public void sellTo(Audience audience) {
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
}
````

- 관람객 : Audience

````java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Long buy(Ticket ticket) {
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

</details>

#### 무엇이 개선되었는가?

- 자신 스스로 관리한다.
- TicketSeller, Audience 내부가 변경되더라도 Theater에 영향을 주지 않는다.

#### 캡슐화와 응집도

- 객체 내부 상태를 캡슐화, 오직 메시지를 통해 상호작용
- 밀접하게 연관된 작업만 수행, 연관성 없는 작업은 다른 객체에게 위임 -> 응집도 높음

객체지향 설계 핵심
- 캡슐화를 이용해 의존성을 적절히 관리, 결합도를 낮춘다.
- 객체가 자신이 맡은 일을 스스로 하도록 책임을 개별 객체로 이동

#### 추가 개선

Bag을 자율적인 존재로 변경
TicketOffice를 자율적인 존재로 변경

<details>
<summary>버전3 변경된 코드</summary>

- 가방 : Bag

````java
public class Bag {

    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        }

        setTicket(ticket);
        minusAmount(ticket.getFee());
        return ticket.getFee();
    }

    private boolean hasInvitation() {
        return invitation != null;
    }

    public void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
````

- 관람객 : Audience

````java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    public Long buy(Ticket ticket) {
        return bag.hold(ticket);
    }
}
````

- 매표소 : TicketOffice

````java
public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    public void sellTicketTo(Audience audience) {
        plusAmount(audience.buy(getTicket()));
    }

    private Ticket getTicket() {
        return tickets.remove(0);
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    private void plusAmount(Long amount) {
        this.amount += amount;
    }
}
````

- 판매원 : TicketSeller

````java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    public void sellTo(Audience audience) {
        ticketOffice.sellTicketTo(audience);
    }
}
````

</details>

하지만, TicketOffice와 Audience 의 의존성 발생

- 어떤 기능을 설계하는 방법은 한가지 이상이다.
- 동일한 기능을 한 가지 이상의 방법으로 설계 할 수 있기에 트레이드오프 발생

객체지향 세계에 들어오면 모든 것이 능동적이고 자율적인 존재
객체를 설계하는 원칙을 가리켜 '의인화'라고 부른다.

### 좋은 설계란?

- 기능을 온전히 수행하는 코드
- 내일의 변경을 매끄럽게 수용할 수 있는 설계
