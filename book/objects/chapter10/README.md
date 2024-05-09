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

##### 중복 코드 수정하기
- 새로운 요구사항
  - 통화 요금에 부과할 세금 계산 추가
  - Phone, NightlyDiscountPhone 모두 구현

```java
public class Phone {
    ...
    private double taxRate;

    public Phone(Money amount, Duration seconds, double taxRate) {
        this.amount = amount;
        this.seconds = seconds;
        this.taxRate = taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for (Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result.plus(result.times(taxRate));
    }
}

public class NightlyDiscountPhone {
  ...
  private double taxRate;
  
  public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
    this.nightlyAmount = nightlyAmount;
    this.regularAmount = regularAmount;
    this.seconds = seconds;
    this.taxRate = taxRate;
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

    return result.plus(result.times(taxRate));
  }
}
```

- 위 예제는 중복 코드가 가지는 단점을 보여준다. 
- Phone은 수정했지만 NightlyDiscountPhone은 수정하지 않은 채 배포했다면 장애가 발생한다.
- 더 큰 문제는 중복 코드를 서로 다르게 수정하기 쉽다.
- 중복 코드의 양이 많아질수록 버그의 수는 증가하며, 변경하는 속도는 점점 더 느려진다.

##### 타입 코드 사용하기
- 두 클래스 사이의 중복 코드를 제거하는 한 가지 방법은 하나로 합치기
- 타입 코드를 사용하여 로직을 분기시킨다
- 문제는 낮은 응집도와 높은 결합도

```java
public class Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    enum PhoneType {
        REGULAR,
        NIGHTLY;
    }

    private PhoneType type;
    private Money amount;
    private Money regularAmount;
    private Money nightlyAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this(PhoneType.REGULAR, amount, Money.ZERO, Money.ZERO, seconds);
    }

    public Phone(Money regularAmount, Money nightlyAmount, Duration seconds) {
        this(PhoneType.NIGHTLY, Money.ZERO, nightlyAmount, regularAmount, seconds);
    }

    public Phone(PhoneType type, Money amount, Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.type = type;
        this.amount = amount;
        this.regularAmount = regularAmount;
        this.nightlyAmount = nightlyAmount;
        this.seconds = seconds;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for (Call call : calls) {
            if (type == PhoneType.REGULAR) {
                result  = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
            } else {
                if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                    result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                } else {
                    result = result.plus(regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
                }
            }
        }
        
        return result;
    }
}
```

- 객체지향 프로그래밍 언어는 타입 코드를 사용하지 않고도 중복 코드를 관리할 수 있는 효과적인 방법을 제공, 바로 상속이다

#### 상속을 이용해서 중복 코드 제거하기
- NightlyDiscountPhone 클래스가 Phone 클래스 상속, 코드를 중복시키지 않고 대부분 재사용
```java
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

public class NightlyDiscountPhone extends Phone {
  private static final int LATE_NIGHT_HOUR = 22;

  private Money nightlyAmount;

  public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
    super(regularAmount, seconds);
    this.nightlyAmount = nightlyAmount;
  }

  public Money calculateFee() {
    Money result = super.calculateFee();

    Money nightlyFee = Money.ZERO;
    for (Call call : getCalls()) {
      if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
        nightlyFee = nightlyFee.plus(getAmount().minus(nightlyAmount).times(call.getDuration().getSeconds() / getSeconds().getSeconds()));
      }
    }

    return result.minus(nightlyFee);
  }
}
```

#### 강하게 결합된 Phone과 NightlyDiscountPhone
- 위 예제에서 세금 부과하는 요구사항이 추가된다면

```java
public class Phone {
    private Money amount;
    private Duration seconds;
    private double taxRate;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds, double taxRate) {
        this.amount = amount;
        this.seconds = seconds;
        this.taxRate = taxRate;
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

    public double getTaxRate() {
        return taxRate;
    }

    public Money calculateFee() {
        Money result = Money.ZERO;

        for (Call call : calls) {
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result.plus(result.times(taxRate));
    }
}

public class NightlyDiscountPhone extends Phone {
  private static final int LATE_NIGHT_HOUR = 22;

  private Money nightlyAmount;

  public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds, double taxRate) {
    super(regularAmount, seconds, taxRate);
    this.nightlyAmount = nightlyAmount;
  }

  public Money calculateFee() {
    Money result = super.calculateFee();

    Money nightlyFee = Money.ZERO;
    for (Call call : getCalls()) {
      if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
        nightlyFee = nightlyFee.plus(getAmount().minus(nightlyAmount).times(call.getDuration().getSeconds() / getSeconds().getSeconds()));
      }
    }

    return result.minus(nightlyFee.plus(nightlyAmount.times(getTaxRate())));
  }
}
```

- 세율을 인스턴스 변수로 포함하고, calculateFee 메서드에서 세금을 부과한다.
- 세금을 부과하는 로직을 추가하기 위해 Phone과 NightlyDiscountPhone에 로직을 추가한다.

> 자식 클래스의 메서드 안에서 super 참조를 이용해 부목 클래스의 메서드를 직접 호출할 경우
> 두 클래스는 강하게 결합된다. super 호출을 제거할 수 있는 방법을 찾아 결합도를 제거하라.

### 취약한 기반 클래스 문제
- 상속은 자식 클래스와 부모 클래스의 결합도를 높인다.
- 부모 클래스의 변경에 의해 자식 클래스가 영향을 받는 현상을 '취약한 기반 클래스 문제' 라고 한다.

#### 불필요한 인터페이스 상속 문제
1. Stack
- Stack은 Vector의 자식 클래스로 Vector의 퍼블릭 인터페이스를 사용할 수 있다.

```java
import java.util.Stack;

Stack<String> stack = new Stack<>();
stack.push("1st");
stack.push("2nd");
stack.push("3rd");

stack.add(0, "4th");

assertEquals("4th", stack.pop()); // 에러
```

2. Properties
- Properties는 HashTable의 자식 클래스이다.
- HashTable은 제네릭이 도입되기 전에 만들어졌기에 컴파일러가 키와 값의 타입이 String인지 체크할 수 없다.

```java
import java.util.Properties;

Properties properties = new Properties();
properties.setProperty("Bjarne Stroustrup", "C++");
properties.setProperty("James Gosling", "JAVA");

properties.put("Dennis Ritchie", 67);

assertEquals("C", properties.getProperty("Dennis Ritchie")); // 에러
```

- getProperty()는 반환값이 String이 아니면 null을 반환한다.

#### 메소드 오버라이딩의 오작용 문제
- 자식 클래스가 부모 클래스의 메서드를 오버라이딩할 경우 부모 클래스가 자신의 메서드를 사용하는 방법에 자식 클래스가 결합될 수 있다.
- 상속을 위해 클래스를 설계하고 문서화해야 하며, 그렇지 않은 경우 상속을 금지시켜야 한다고 주장

#### 부모 클래스와 자식 클래스의 동시 수정 문제

