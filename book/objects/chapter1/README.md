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


