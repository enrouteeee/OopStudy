이 글은 '오브젝트' 책의 내용을 읽고 정리한 것 입니다.

객체지향 설계의 핵심은 역할, 책임, 협력이다.

협력은 애플리케이션의 기능을 구현하기 위해 메시지를 주고받는 객체들 사이의 상호작용이다.

책임은 객체가 다른 객체와 협력하기 위해 수행하는 행동이다.

역할은 대체 가능한 책임의 집합이다.

이 중 가장 중요한 것은 책임이다. 객체들이 수행할 책임이 적절하게 할당되지 못한 상황에서는 원활한 협력도 기대할 수 없을 것이다. 역할은 책임의 집합이기 때문에 책임이 적절하지 못하면 역할 역시 협력과 조화를 이루지 못한다. 결국 책임이 객체지향 애플리케이션 전체의 품질을 결정하는 것이다.

객체지향 설계란 올바른 객체에게 올바른 책임을 할당하면서 낮은 결합도와 높은 응집도를 가진 구조를 창조하는 활동이다.

결합도와 응집도를 합리적인 수준으로 유지할 수 있는 중요한 원칙이 있다. 객체의 상태가 아니라 객체의 행동에 초점을 맞추는 것이다.

## 1\. 데이터 중심의 영화 예매 시스템

객체지향 설계에서는 두 가지 방법을 이용해 시스템을 객체로 분할할 수 있다. 상태를 분할의 중심으로 삼는 방법과 책임을 분할의 중심축으로 삼는 방법이다.

상태 중심의 관점에서는 객체는 자신이 포함하고 있는 데이터를 조작하는 데 필요한 오퍼레이션을 정의한다.

책임 중심의 관점에서는 객체는 다른 객체가 요청할 수 있는 오퍼레이션을 위해 필요한 상태를 보관한다.

훌륭한 객체지향 설계는 데이터(상태)가 아니라 책임에 초점을 맞춰야한다. 이유는 변경과 관련이 있다.

객체의 상태는 구현에 속한다. 구현은 불안정하기 때문에 변하기 쉽다. 상태를 객체 분할의 중심축으로 삼으면 구현에 관한 세부사항이 객체의 인터페이스에 스며들게 되어 캡슐화의 원칙이 무너진다. 결과적으로 상태 변경은 인터페이스의 변경을 초래하며 이 인터페이스에 의존하는 모든 객체에게 변경의 영향이 퍼지게 된다.

그에 비해 객체의 책임은 인터페이스에 속한다. 객체는 책임을 드러내는 안정적인 인터페이스 뒤로 책임을 수행하는 데 필요한 상태를 캡슐화함으로써 구현 변경에 대한 파장이 외부로 퍼져나가는 것을 방지한다. 다라서 책임에 초점을 맞추면 상대적으로 변경에 안정적인 설계를 얻을 수 있게 된다.

#### 데이터를 준비하자

데이터를 기준으로 분할한 영화 예매 시스템의 설계를 살펴보겠다. 먼저 Movie에 저장될 데이터를 결정하는 것으로 설계를 시작하자.

```
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    
  	private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;
}
```

영화 제목, 상영시간, 기본 요금을 인스턴스 변수로 포함하는 것은 기존의 설계와 동일한 부분이다.

차이점은 할인 조건의 목록이 인스턴스 변수로 직접 포함되어 있따는 것이다. 또한 할인 정책을 DiscountPolicy라는 별도의 클래스로 분리했던 이전 예제와 달린 금액 할인 정책에 사용되는 할인 금액과 비율 할인 정책에 사용되는 할인 비율을 Movie 안에서 직접 정의하고 있다.

영화에 사용된 할인 정책의 종류는 movieType 열거형 타입을 통해 알 수 있다.

```
public enum MovieType {
	AMOUNT_DISCOUNT,
    PERCENT_DISCOUNT,
    NONE_DISCOUNT
}
```

데이터 중심의 설계에서는 객체가 포함해야 하는 데이터에 집중한다. 이 객체가 포함해야 하는 데이터는 무엇인가? 특히 Movie 클래스의 경우처럼 객체의 종류를 저장하는 인스턴스 변수(moiveType)와 인스턴스의 종류에 따라 배타적으로 사용될 인스턴스 변수(discountAmount, discountPercent)를 하나의 클래스 안에 함께 포함시키는 방식은 데이터 중심의 설계 안에서 흔히 볼 수 있는 패턴이다.

데이터 중심의 설계 방법을 따르기 때문에 할인 조건을 설계하기 위해 해야 하는 질문은 다음과 같다. 할인 조건을 구현하는 데 필요한 데이터는 무엇인가? 먼저 현재의 할인 조건의 종류를 저장할 데이터가 필요하다. 할인 조건의 타입을 저장할 DiscountConditionType을 정의하자.

```
public enum DiscountConditionType {
    SEQUENCE,
    PERIOD
}
```

```
public class DiscountCondition {
    private DiscountConditionType type;

    private int sequence;

    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    public DiscountConditionType getType() {
        return type;
    }

    public void setType(DiscountConditionType type) {
        this.type = type;
    }

    public int getSequence() {
        return sequence;
    }

    public void setSequence(int sequence) {
        this.sequence = sequence;
    }

    public DayOfWeek getDayOfWeek() {
        return dayOfWeek;
    }

    public void setDayOfWeek(DayOfWeek dayOfWeek) {
        this.dayOfWeek = dayOfWeek;
    }

    public LocalTime getStartTime() {
        return startTime;
    }

    public void setStartTime(LocalTime startTime) {
        this.startTime = startTime;
    }

    public LocalTime getEndTime() {
        return endTime;
    }

    public void setEndTime(LocalTime endTime) {
        this.endTime = endTime;
    }
}
```

마찬가지로 Screening, Reservation, Customer를 생성하자.

**영화를 예매하자**

```
public class ReservationAgency {
    public Reservation reservation(Screening screening, Customer customer, int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;
        for(DiscountCondition condition : movie.getDiscountConditions()) {
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
            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }

            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
            fee = movie.getFee().times(audienceCount);
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

## 2\. 설계의 트레이드오프

데이터 중심 설계와 책임 중심 설계의 장단점을 비교하기 위해 캡슐화, 응집도, 결합도를 사용하겠다.

**캡슐화**

상태와 행동을 하나의 객체 안에 모으는 이유는 객체의 내부 구현을 외부로부터 감추기 위해서이다. 여기서 구현이란 나중에 변경될 가능성이 높은 어떤 것을 가리킨다. 객체지향이 강력한 이유는 한 곳에서 일어난 변경이 전체 시스템에 영향을 끼치지 않도록 파급효과를 적절하게 조절할 수 있는 장치를 제공하기 때문이다.

변경될 가능성이 높은 부분을 구현이라고 부르고 상대적으로 안정적인 부분을 인터페이스라고 부른다. 객체를 설계하는 가장 기본적인 아이디어는 변경의 정도에 따라 구현과 인터페이스를 분리하고 외부에서는 인터페이스에만 의존하도록 관계를 조절하는 것이다.

캡슐화는 외부에서 알 필요가 없는 부분을 감춤으로써 대상을 단순화하는 추상화의 한 종류이다. 가장 중요한 원리는 불안정한 구현 세부사항을 안정적인 인터페이스 뒤로 캡슐화하는 것이다.

**응집도와 결합도**

응집도는 모듈에 포함된 내부 요소들이 연관돼 있는 정도를 나타낸다. 모듈 내의 요소들이 하나의 목적을 위해 긴밀하게 협력한다면 그 모듈은 높은 응집도를 가진다. 모듈 내의 요소들이 서로 다른 목적을 추구한다면 그 모듈은 낮은 응집도를 가진다. 객체지향의 관점에서 응집도는 객체 또는 클래스에 얼마나 관련 높은 책임들을 할당했는지를 나타낸다.

결합도는 의존성의 정도를 나타내며 다른 모듈에 대해 얼마나 많은 지식을 갖고 있는지를 나타내는 척도다. 어떤 모듈이 다른 모듈에 대해 너무 자세한 부분까지 알고 있다면 두 모듈은 높은 결합도를 가진다. 객체지향의 관점에서 결합도는 객체 또는 클래스가 협력에 필요한 적절한 수준의 관계만을 유지하고 있는지를 나타낸다.

일반적으로 좋은 설계란 높은 응집도와 낮은 결합도를 가진 모듈로 구성된 설계를 의미한다. 좋은 설계란 오늘의 기능을 수행하면서 내일의 변경을 수용할 수 있는 설계다.

높은 응집도와 낮은 결합도를 가진 설계를 추구해야 하는 이유는 단 한 가지다. 그것이 설계를 변경하기 쉽게 만들기 때문이다. 변경의 관점에서 응집도란 변경이 발생할 때 모듈 내부에서 발생하는 변경의 정도로 측정할 수 있다.

응집도가 높을수록 변경의 대상과 범위가 명확해지기 때문에 코드를 변경하기 쉬워진다. 변경으로 인해 수정되는 부분을 파악하기 위해 코드 구석구석을 해매고 다니거나 여러 모듈을 동시에 수정할 필요가 없으며 변경을 반영하기 위해 오직 하나의 모듈만 수정하면 된다.

결합도 역시 변경의 관점에서 설명할 수 있다. 결합도는 한 모듈이 변경되기 위해서 다른 모듈의 변경을 요구하는 정도로 측정할 수 있다. 하나의 모듈을 수정할 때 얼마나 많은 모듈을 함께 수정해야 하는지를 나타낸다. 결합도가 높으면 높을수록 함께 변경해야 하는 모듈의 수가 늘어나기 때문에 변경하기가 어려워진다.

마지막으로 캡슐화의 정도가 응집도와 결합도에 영향을 미친다는 사실이다. 캡슐화를 지키면 모듈 안의 응집도는 높아지고 모듈 사이의 결합도는 낮아진다. 캡슐화를 향상시키기 위해 노력하자.

## 3\. 데이터 중심의 영화 예매 시스템의 문제점

근본적인 차이점은 캡슐화를 다루는 방식이다. 데이터 중심의 설계는 캡슐화를 위반하고 객체의 내부 구현을 인터페이스의 일부로 만든다. 반면 책임 중심의 설계는 객체의 내부 구현을 안정적인 인터페이스 뒤로 캡슐화한다.

**캡슐화 위반**

접근자와 수정자 메서드는 객체 내부에의 상태에 대한 어떤 정보도 캡슐화하지 못한다. 설계할 때 협력에 관해 고민하지 않으면 캡슐화를 위반하는 과도한 접근자와 수정자를 가지게 되는 경향이 있다. 객체가 사용될 문맥을 추측할 수밖에 없는 경우 최대한 많은 접근자 메서드를 추가하게 되는 것이다. 이러한 설계 방식을 추측에 의한 설계 전략이라고 부른다. 객체가 사용될 협력을 고려하지 않고 다양한 상황에서 사용될 수 있을 것이라는 막연한 추측을 기반으로 설계를 진행한다. 결과적으로 대부분의 내부 구현이 퍼블릭 인터페이스에 그대로 노출될 수밖에 없는 것이다. 그 결과 캡슐화의 원칙을 위반하는 변경에 취약한 설계를 얻게 된다.

**높은 결합도**

객체 내부의 구현이 객체의 인터페이스에 드러난다는 것은 클라이언트가 구현에 강하게 결갑된다는 것을 의미한다. 또한 단지 객체의 내부 구현을 변경했음에도 이 인터페이스에 의존하는 모든 클라이언트들도 함께 변경해야 한다는 것이다.

데이터 중심의 설계는 전체 시스템을 하나의 거대한 의존성 덩어리로 만들어 버리기 때문에 어떤 변경이라도 일단 발생하고 나면 시스템 전체가 요동칠 수밖에 없다.

**낮은 응집도**

새로운 할인 정책을 추가하거나 새로운 할인 조건을 추가하기 위해 하나 이상의 클래스를 동시에 수정해야 한다. 어떤 요구사항 변경을 수용하기 위해 하나 이상의 클래스를 수정해야 하는 것은 설계의 응집도가 낮다는 증거다.

## 4\. 자율적인 객체를 향해

**캡슐화를 지켜라**

캡슐화는 설계의 제1원리다. 객체는 자신이 어떤 데이터를 가지고 있는지를 내부에 캡슐화하고 외부에 공개해서는 안된다. 객체는 스스로의 상태를 책임져야 하며 외부에서는 인터페이스에 정의된 메서드를 통해서만 상태에 접근할 수 있어야 한다.

접근자나 수정자를 의미하는 것이 아니다. 객체에게 의미 있는 메서드는 객체가 책임져야 하는 무언가를 수행하는 메서드다. 속성의 가시성을 private으로 설정했다고 해도 접근자와 수정자를 통해 속성을 외부로 제공하고 있다면 캡슐화를 위반하는 것이다.

**스스로 자신의 데이터를 책임지는 객체**

상태와 행동을 객체라는 하나의 단위로 묶는 이유는 객체 스스로 자신의 상태를 처리할 수 있게 하기 위해서이다. 객체는 단순한 데이터 제공자가 아니다. 객체 내부에 저장되는 데이터보다는 객체가 협력에 참여하면서 수행할 책임을 정의하는 오퍼에이션이 더 중요하다.

## 5\. 하지만 여전히 부족하다

**캡슐화의 진정한 의미**

캡슐화가 단순히 객체 내부의 데이터를 외부로부터 감추는 것 이상의 의미를 가진다는 것을 잘 보여준다. 사실 캡슐화는 변경될 수 있는 어떤 것이라도 감추는 것을 의미한다. 내부 속성을 외부로부터 감추는 것은 '데이터 캡슐화'라고 불리는 캡슐화의 한 종류일 뿐이다.

캡슐화란 변할 수 있는 어떤 것이라도 감추는 것이다. 그것이 속성의 타입이건, 할인 정책의 종류건, 상관 없이 내부 구현의 변경으로 인해 외부의 객체가 영향을 받는다면 캡슐화를 위반하는 것이다. 설계에서 변하는 것이 무엇인지 고려하고 변하는 개념을 캡슐화해야 한다. 이것이 캡슐화라는 용어를 통해 말하고자 하는 진정한 의미다.

## 6\. 데이터 중심 설계의 문제점

데이터 중심의 설계가 변경에 취약한 이유는 두 가지이다.

\- 데이터 중심의 설계는 본질적으로 너무 이른 시기에 데이터에 관해 결정하도록 강요한다.

\- 데이터 중심의 설계에서는 협력이라는 문맥을 고려하지 않고 객체를 고립시킨 채 오퍼레이션을 결정한다.