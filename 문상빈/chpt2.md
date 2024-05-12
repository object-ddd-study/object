# 객체지향 프로그래밍

- 영화 예매 시스템 예제

### 협력, 객체, 클래스
- 대부분 객체지향 프로그램을 작성할 때 어떤 클래스가 필요할지부터 고민하지만 이는 잘못된 접근.
- 어떤 클래스가 필요한지 고민하기 전에 어떤 객체들이 필요한지 고민하자
	- Q. 잘 이해가 안된다. 필요한 객체를 고민하는 과정이 생략되어서 아쉽다.
- 객체는 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원
- 객체의 모양과 윤곽이 잡히면 공통된 특성과 상태를 가진 객체들을 타입으로 분류하고, 이 타입을 기반으로 클래스를 구현하자

### 도메인의 구조를 따르는 프로그램 구조
- 도메인이란?
	- 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야
- 객체지향 패러다임이 강력한 이유는 요구사항을 분석하는 초기 단계부터 구현하는 마지막 단계까지 '객체' 라는 동일한 추상화 기법을 사용할 수 있기 때문
- 41p 그림 2.3 참고


### 예시 코드

```java
public class Screening {  
  
   private Movie movie;  
   private int sequence;  
   private LocalDateTime whenScreened;  
  
   public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {  
      this.movie = movie;  
      this.sequence = sequence;  
      this.whenScreened = whenScreened;  
   }  
  
   public LocalDateTime getStartTime() {  
      return whenScreened;  
   }  
  
   public boolean isSequence(int sequence) {  
      return this.sequence == sequence;  
   }  
  
   public Money getMovieFee() {  
      return movie.getFee();  
   }  
  
   public Reservation reserve(Customer customer, int audienceCount) {  
      return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);  
   }  
  
   private Money calculateFee(int audienceCount) {  
      return movie.calculateMovieFee(this).times(audienceCount);  
   }  
}
```

- 접근자가 `public` / `private` 으로 구분되어 있음

#### 자율적인 객체
- 객체의 특징 두가지
	- 객체는 상태(state) 와 행동(behavior)을 함께 가진다
	- 객체는 스스로 판단하고 행동하는 자율적인 존재
- 객체 내부로에 접근을 통제하는 이유는 객체를 자율적인 존재로 만들기 위함. 
- 클라이언트는 객체에게 원하는 것을 요청하고, 객체가 스스로 최선의 방법을 결정할 수 있을 것이라고 믿고 기다려야 한다. 
- 일반적으로 객체의 상태는 숨기고 행동만 외부에 공개


### 협력하는 객체들의 공동체

```java
public class Screening {  
   public Reservation reserve(Customer customer, int audienceCount) {  
      return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);  
   }  
  
   private Money calculateFee(int audienceCount) {  
      return movie.calculateMovieFee(this).times(audienceCount);  
   }  
}

```

- 객체들 사이에 이뤄지는 상호작용 -> "협력"
	- 영화를 예매하기 위해 `Screening`, `Moive`, `Reservation` 의 인스턴스들이 서로의 메서드를 호출하며 상호작용

### 협력
- 메시지와 메서드 구분
	- 메시지
		- 객체끼리 주고 받는 요청 / 응답
	- 메서드
		- 수신된 메시지를 처리하기 위한 객체 자신만의 방법
	- `Screening` 이 `Movie` 의 `calculateMovieFee` 메서드를 호출 (X)
	- `Screening` 이 `Movie` 에게 `calculateMovieFee` 메시지를 전송 (O)
	- 메시지를 처리하는 것은 `Movie` 스스로의 문제. 객체는 메시지를 처리할 수 있는 방법을 자율적으로 결정할 수 있음


## 할인 요금 구하기

```java
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
```

```java
public abstract class DiscountPolicy {  
  
   private List<DiscountCondition> conditions = new ArrayList<>();  

  //생성자 파라미터 목록을 이용해 초기화에 필요한 정보를 전달하도록 강제
  // --> 올바른 상태를 가진 객체의 생성을 보장할 수 있음 
   public DiscountPolicy(DiscountCondition ... conditions) {  
      this.conditions = Arrays.asList(conditions);  
   }  
  
   public Money calculateDiscountAmount(Screening screening) {  
      for (DiscountCondition each : conditions) {  
         if (each.isSatisfiedBy(screening)) {  
            return getDiscountAmount(screening);  //템플릿 메서드 패턴
         }  
      }  
      return Money.ZERO;  
   }  
  
   abstract protected Money getDiscountAmount(Screening screening);  
   
}
```

```java
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
```

```java
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
```


## 상속과 다형성
- `Movie` 클래스는 어떻게 할인 정책을 선택할까?
	- 생성 시점에 `DiscountPolicy` 의 구체 클래스 인스턴스를 주입한다. 
- 코드의 의존성과 실행 시점의 의존성이 서로 다를 수 있다. 즉, 클래스 간 의존성과 객체 간 의존성이 동일하지 않을 수 있다.
- (주의) 설계가 유연해질수록 코드를 이해하고 디버깅하기는 점점 더 어려워진다.
	- 객체를 생성, 연결하는 부분을 찾아봐야 하기 때문
	- 유연성 <-> 가독성 trade-off

### 상속과 인터페이스
- 상속 이점
	- 부모 클래스가 제공하는 모든 인터페이스를 자식 클래스가 물려받을 수 있음
	- 여기서 인터페이스란 객체가 이해할 수 있는 메시지 목록을 의미

```java
public class Movie {

	public Money calculateMovieFee(Screening screening) {
		return fee.minus(discountPolicy.calculateDiscountAmount(screening));
	}

}
```

- `Movie` 는 자신이 협력하는 객체가 `calculateDiscountAmount` 메시지를 이해할 수만 있다면 객체가 어느 클래스의 인스턴스인지는 상관하지 않음

### 다형성
- 동일한 메시지를 전송하지만 실제로 어떤 메서드가 실행될 것인지는 메시지를 수신하는 객체의 클래스가 무엇이냐에 따라 달라진다.
- 다형적인 협력에 참여하는 객체들은 모두 같은 메시지를 이해할 수 있어야 함.
	- 즉, 인터페이스가 동일해야 함
- 메시지 vs 메서드
	- 메시지, compile time, early binding, static binding
	- 메서드, run time, lazy binding, dynamic binding
- Q. 클래스를 상속받는 것만이 다형성을 구현할 수 있는 유일한 방법이 아니다. 나중에 다양한 방법이 있다는 걸 알게 될 것


### 구현 상속 vs 인터페이스 상속
- 구현 상속
	- 코드를 재사용하기 위한 목적으로 상속을 사용
	- subclassing
	- 변경에 취약한 코드를 만들게 될 확률이 높음 
- 인터페이스 상속
	- 다형적인 협력을 위해 부모클래스와 자식 클래스가 인터페이스를 공유할 수 있도록 상속을 이용하는 것
	- subtyping
- 상속은 구현 상속이 아니라 **인터페이스 상속**을 위해 사용하는 것을 권장


### 인터페이스와 다형성
- interface vs abstract class
	- interface
		- 구현은 공유할 필요가 없고 순수하게 인터페이스만 공유하고 싶을 때 활용
	- abstract class
		- 구현을 공유할 필요가 있을 때 활용

## 추상화와 유연성
- 추상화 장점
	- 추상화 계층만 따로 떼어놓고 보면 요구사항의 정책을 높은 수준에서 서술할 수 있음
		- e.g. "영화 예매 요금은 최대 하나의 할인 정책과 다수의 할인 조건을 이용해 계산할 수 있다"
		- 65p 그림 2.13 참고
	- 설계가 더 유연해진다
		- e.g. `Movie`, `DiscountPolicy` 는 건들지 않고 `NoneDiscountPolicy` 라는 새로운 클래스를 추가하는 것만으로 어플리케이션 기능을 확장할 수 있음
- 따라서 유연성이 필요한 곳에 추상화를 사용하자
	- 단, 이 역시 가독성과 trade-off 가 생길 수 있으니 득실을 잘 따져보고 적용하자



### 코드 재사용
- **코드 재사용**을 위해서는 상속보다는 **합성(composition)** 을 활용하자
- 코드 재사용을 위한 상속의 단점
	- e.g. `AmountDiscountMovie`, `PercentDiscountMovie`
	- 캡슐화 위반
		- 자식 클래스가 부모 클래스에 강하게 결합. 부모 클래스를 변경할 때 자식 클래스도 함께 변경해야 할 확률이 높아짐
	- 설계가 유연하지 않음. 실행 시점에 객체 종류를 바꾸는 것이 불가능함

### 합성(composition)

```java
public class Movie {

	private DiscountPolicy discountPolicy;

  //실행 시점에 할인 정책을 간단하게 변경할 수 있음
	public void changeDiscountPolicy(DiscountPolicy discountPolicy){
	  this.discountPolicy = discountPolicy;
	}

}
```

- 인터페이스에 정의된 메시지를 통해서만 코드를 재활용
- 상속과 달리, `DiscountPolicy` 의 인터페이스를 통해 약하게 결합됨
	- 즉, `Movie` 는 `DiscountPolicy` 가 `calculateDiscountAmount` 를 외부에 제공한다는 것만 알고 내부 구현에 대해서는 알지 못함


---
- 마음에 드는 문장 또는 중요하다고 생각하는 내용
- 이야기 해보고 싶은 내용
- 자신과 생각이 다른 내용

- Q. 제어권한 을 누가 획득한다는 것인지?
	- 
- Q. object-C
	- 
- 직사각형 - 정사각형 문제
	- 인터페이스 상속 - 구현 상속
- 접근제어자가 오히려 객체지향을 해친다?
	- e.g. private 멤버 변수를 개발자가 볼 수 있다. 반면, c 언어는 헤더 파일이 따로 있어서 멤버 변수를 아예 감출 수 있다. 