# 객체, 설계

### 예제 코드

```java
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
```

### 문제점
- 변경 용이성, 가독성이 떨어짐
	- Audience, TicketSeller 를 변경할 경우 Theater 도 함께 변경해야 함

#### 변경에 취약한 코드
- e.g. Audience 가 bag 을 들고 있지 않도록 변경됨 -> theater 코드도 변경 필요
- 의존성은 변경에 대한 영향을 암시
	- 어떤 객체가 변경될 때 그 객체에 의존하는 다른 객체도 함께 변경될 수 있음
- 객체 지향 설계의 목표는 최소한의 의존성만 유지하고 불필요한 의존성을 제거하는 것
	- 즉, 객체 간 결합도를 낮춰 변경이 용이한 설계를 만드는 것

### 개선된 코드

```java
public class TicketSeller {  
   private TicketOffice ticketOffice;  
  
   public TicketSeller(TicketOffice ticketOffice) {  
      this.ticketOffice = ticketOffice;  
   }  
  
   public TicketOffice getTicketOffice() {  
      return ticketOffice;  
   }  
  
   public void sellTo(Audience audience) {  
      ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));  
   }  
}
```

```java
public class Theater {  
   private TicketSeller ticketSeller;  
  
   public Theater(TicketSeller ticketSeller) {  
      this.ticketSeller = ticketSeller;  
   }  
  
   public void enter(Audience audience) {  
      ticketSeller.sellTo(audience);  
   }  
}
```

```java
public class Audience {  
   private Bag bag;  
  
   public Audience(Bag bag) {  
      this.bag = bag;  
   }  
  
   public Bag getBag() {  
      return bag;  
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
```

- TicketSeller 가 ticketOffice 를 사용하는 코드를 TicketSeller 내부로 옮김
- Audience 가 Bag 을 사용하는 코드를 Audience 내부로 옮김 
- 객체의 응집도를 높이기 위해서는 객체 스스로 자신의 데이터를 책임져야 함 
- 데이터와 프로세스가 동일한 모듈 내부에 위치하도록 프로그래밍 하는 방식이 바로 "객체지향 프로그래밍" <-> "절차적 프로그래밍"
- (심화) 객체지향 설계의 핵심은 적절한 객체에 적절한 책임을 할당하는 것
	- 따라서 객체가 어떤 데이터를 가지느냐보다는 어떤 책임을 할당할 것이냐에 초점을 맞춰야 함


### 개선된 점
- 변경하기 쉬워짐
- Audience, TicketSeller 의 내부 구현이 달라지더라도 Theater 를 함께 변경할 필요가 없어짐
- 설계를 어렵게 만드는 건 의존성. 따라서 우리는 불필요한 의존성을 제거함으로써 객체 사이의 결합도를 낮춰야 한다.


### 추가 개선
```java
public class Bag {  
   private Long amount;  
   private Invitation invitation;  
   private Ticket ticket;  
  
   public Bag(Long amount) {  
      this(amount, null);  
   }  
  
   public Bag(Long amount, Invitation invitation) {  
      this.amount = amount;  
      this.invitation = invitation;  
   }  
  
   public long hold(Ticket ticket) {  
      if (hasInvitation()) {  
         setTicket(ticket);  
         return 0L;  
      } else {  
         setTicket(ticket);  
         minusAmount(ticket.getFee());  
         return ticket.getFee();  
      }  
   }  
  
   private boolean hasInvitation() {  
      return invitation != null;  
   }  
  
   private boolean hasTicket() {  
      return ticket != null;  
   }  
  
   private void setTicket(Ticket ticket) {  
      this.ticket = ticket;  
   }  
  
   private void minusAmount(Long amount) {  
      this.amount -= amount;  
   }  
  
   private void plusAmount(Long amount) {  
      this.amount += amount;  
   }  
}
```

```java
public class Audience {  
   private Bag bag;  
  
   public Audience(Bag bag) {  
      this.bag = bag;  
   }  
  
   public Bag getBag() {  
      return bag;  
   }  
  
   public Long buy(Ticket ticket) {  
      return bag.hold(ticket);  
   }  
}
```


### 의인화
- 비록 현실에서는 수동적인 존재라고 하더라도 객체지향 세계로 들어오면 모든 것이 능동적이고 자율적인 존재로 바뀐다!
	- e.g. `Bag`

### 요약
- 객체지향 세계에서 어플리케이션은 객체들로 구성되며 어플리케이션의 기능은 객체들 간의 상호작용을 통해 구현된다. 그리고 객체들 사이의 상호작용은 객체끼리 주고 받는 메시지로 표현된다.
- 훌륭한 객체지향 설계란 협력하는 객체 간 의존성을 적절하게 관리하는 설계이다.


