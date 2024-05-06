## 상속과 코드 재사용
### 상속과 중복 코드
#### DRY 원칙
- 프로그램의 본질은 비즈니스와 관련된 지식을 코드로 변환하는 것, 이 지식은 항상 변한다.
- DRY(Don't Repeat Yourself) 원칙 : '반복하지 마라', 동일한 지식을 중복하지 말라는 것
  - 한 번, 단 한번(Once and Only Once) 원칙
  - 단일 지점 제어(Single Point Control) 원칙

#### 중복과 변경
##### 중복 코드 살펴보기

시간당 요금을 계산하는 간단한 전화 요금 애플리케이션

- 기본 구현

```java
public class Money {
    public static final Money ZERO = Money.wons(0);
    private final BigDecimal amount;

    public Money(BigDecimal amount) {
        this.amount = amount;
    }

    public static Money wons(long amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public Money times(long percent) {
        return new Money(amount.multiply(BigDecimal.valueOf(percent)));
    }

    public Money plus(Money amount) {
        return new Money(this.amount.add(amount.getAmount()));
    }

    public BigDecimal getAmount() {
        return amount;
    }
}

public class Call {
  private LocalDateTime from;
  private LocalDateTime to;

  public Call(LocalDateTime from, LocalDateTime to) {
    this.from = from;
    this.to = to;
  }

  public Duration getDuration() {
    return Duration.between(from, to);
  }

  public LocalDateTime getFrom() {
    return from;
  }
}

public class Phone {
  private Money amount;
  private Duration seconds;
  private List<Call> calls = new ArrayList<>();

  public Phone(Money amount, Duration seconds) {
    this.amount = amount;
    this.seconds = seconds;
  }

  public void call(Call call) {
    calls.add(call);
  }

  public List<Call> getCalls() {
    return calls;
  }

  public Money getAmount() {
    return amount;
  }

  public Duration getSeconds() {
    return seconds;
  }

  public Money calculateFee() {
    Money result = Money.ZERO;

    for (Call call : calls) {
      result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
    }

    return result;
  }
}

public class Client {

  public static void main(String[] args) {
    Phone phone = new Phone(Money.wons(5), Duration.ofSeconds(10));
    phone.call(new Call(LocalDateTime.of(2024, 5, 6, 12, 10, 0), LocalDateTime.of(2024, 5, 6, 12, 11, 0)));
    phone.call(new Call(LocalDateTime.of(2024, 5, 6, 12, 10, 0), LocalDateTime.of(2024, 5, 6, 12, 11, 0)));

    Money money = phone.calculateFee();

    System.out.println(money.getAmount());
  }
}
```

- 요구사항 변경
  - '심야 할인 요금제' 추가, 밤 10시 이후의 통화에 대한 요금 할인
  - Phone의 코드를 복사해서 NightlyDiscountPhone 새로운 클래스 생성

```java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    public void call(Call call) {
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Duration getSeconds() {
        return seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for (Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            }
        }

        return result;
    }
}
```

- NightlyDiscountPhone은 밤 10시 기준으로 요금만 다를 뿐 Phone과 거의 유사하다.
- 요구사항을 아주 잛은 시간 안에 구현할 수 있다.

