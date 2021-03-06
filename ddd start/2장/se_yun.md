# 2장. 아키텍처 개요

## 네 개의 영역

### 표현 영역

![image](https://user-images.githubusercontent.com/53366407/121773489-43178280-cbb7-11eb-8dbd-fd5a46ae47f6.png)

- 표현 영역(UI 영역)은 사용자 요청을 받아 응용 영역에 전달하고 응용 영역의 처리 결과를 다시 사용자에게 보여주는 역할을 한다.
- 스프링에서는 MVC 프레임워크가 표현영역을 위한 기술에 해당한다.
- 여기사 사용자는 웹 브라우저가 될수도 있지만, REST API를 호출하는 외부 시스템일수도 있다.

### 응용 영역

![image](https://user-images.githubusercontent.com/53366407/121773492-490d6380-cbb7-11eb-8284-3ae4b7e99ddf.png)

- 사용자의 요청을 전달받아 사용자에게 제공해야 할 기능을 구현해 응답을 표현영역에 전달한다.
- 그러나 로직을 직접 수행하는 것이 아니라 도메일 모델에 로직 수행을 위임한다.

### 도메인 영역

- 핵심 로직을 구현한 영역

### 인프라스트럭쳐 영역

![image](https://user-images.githubusercontent.com/53366407/121773495-4d398100-cbb7-11eb-9d6e-3be753082bf8.png)

- 구현 기술에 대한 것을 다루는 곳으로 예를들면 RDBMS 연동이나 메시징 큐에 메시지를 전송하거나 수신하거나 몽고DB, HBase를 사용해 데이터베이스 연동을 처리하는 곳이다.
- 또한 SMTP를 이용한 메일 발송 기능이나 HTTP 클라이언트를 이용해 REST API를 호출하는 것도 처리한다.
- 논리적인 개념보다는 실제 구현을 다루는 곳이다.

## 계층 구조 아키텍처

![image](https://user-images.githubusercontent.com/53366407/121773496-51fe3500-cbb7-11eb-9725-9c3431300d14.png)

- 도메인의 복잡도에 따라 응용과 도메인을 분리하기도 하고 한 계층으로 합치기도 하지만 전체적인 아키텍처는 위의 그림을 따라간다.
- 계층 구조는 특성상 상위 계층에서 하위 계층으로의 의존만 존재하고 하위 계층은 상위 계층에 의존하지 않는다.
- 그러나 구현의 편리함을 위해 계층 구조를 유연하게 가져가는 경우도 있다. 예를 들면 응용 계층은 도메인 계층에 의존하지만 외부 시스템과 연동을 위해 더 아래 계층인 인프라스트럭처 계층에 의존하기도 한다.

![image](https://user-images.githubusercontent.com/53366407/121773501-5591bc00-cbb7-11eb-9dd7-b1ce4da97151.png)

- 표현, 응용, 도메인 계층이 상세한 규현 기술을 다루는 인프라스트럭처 계층에 종속된다.
- cf) [룰 엔진](https://javacan.tistory.com/entry/118)

- 그러나 인프라스트럭처에 의존하면 두가지 문제점이 있다.
    1. 테스트 어려움
    2. 기능 확장의 어려움

## DIP(Dependency Inversion Principle, 의존역전원칙)

![image](https://user-images.githubusercontent.com/53366407/121773504-588cac80-cbb7-11eb-9e07-6a2466ba23ff.png)

- 위의 2개의 문제를 해결하는 방법으로 저수준 모듈이 고수준 모듈에 의존하도록 변경하는 것이다.
- 고수준 모듈이 제대로 동작하려면 저수준 모듈을 사용해야 하는데, 고수준 모듈이 저수준 모듈을 사용하면 앞서 계층 구조 아키텍처에서 언급했던 두 가지 문제가 발생한다. 따라서 저수준 모듈이 고수준 모듈에 의존하도록 변경해야 하는 것이다.
- 그렇다면 저수준 모듈이 고수준 모듈에 의존하도록 하려면 어떻게 해야 할까? 바로 추상화한 인터페이스에 비밀이 숨어 있다.
- 즉, 아래와 같이 인터페이스를 정의해 DIP를 구현하면 되는 것이다.

![image](https://user-images.githubusercontent.com/53366407/121773508-5cb8ca00-cbb7-11eb-9021-90055eefe2ea.png)

- 고수준 모듈을 저수준 모듈을 사용하려면 고수준 모듈이 저수준 모듈에 의존해야 하는데, 반대로 저수준 모듈이 고수준 모듈에 의존한다고 해서 이를 DIP라고 한다.

### 테스트 어려움 해결 방법

- 대용 객체(mock)을 사용해서 테스트를 진행하면 되기 때문에 테스트를 하기 편리하다.

### 기능 확장의 어려움 해결 방법

- 추상화한 인터페이스를 통해 저수준 구현 객체는 의존 주입을 받게 되고 구현 기술을 변경하면 저수준 구현 객체를 생성하는 부분만 변경하면 된다.

### DIP의 주의사항

- DIP를 잘못 이해하면 단순히 인터페이스와 구현 클래스를 분리하는 정도라고 생각할 수 있는데, 핵심은 고수준 모듈이 저수준 모듈에 의존하지 않도록 하기 위함이라는걸 잊지 말아야 한다.
- 도메인 영역이 구현 기술을 다루는 인프라스트럭처 영역에 의존하고 있다면 여전히 고수준 모듈이 저수준 모듈에 의존하고 있고 인터페이스는 고수준 모듈인 도메인 관점이 아니라 룰 엔진이라는 저수준 모듈 관점에서 도출한것이다.
- 따라서 하위 기능을 추상화한 인터페이스는 고수준 모듈 관점에서 도출해야 한다. 추상화한 인터페이스는 저수준 모듈이 아닌 고수준 모듈에 위치해야한다.

### DIP와 아키텍처

- 인프라스트럭처 영역은 구현 기술을 다루는 저수준 모듈이고 응용 영역과 도메인 영역은 고수준 모듈이다. 인프라스트럭처에 위치한 클래스나 도메인이나 응용 영역에 정의한 인터페이스를 상속받아 구현하는 구조가 도므로 도메인과 응용 영역에 대한 영향을 주지 않거나 최소화하면서 구현 기술을 변경하는 것이 가능하다.

![image](https://user-images.githubusercontent.com/53366407/121773513-617d7e00-cbb7-11eb-9458-1a04652b9f47.png)

## 도메인 영역의 주요 구성 요소

### 엔티티(Entity)

- 고유의 식별자를 갖는 객체로 자신의 라이프사이클을 갖는다. 도메일 모델의 데이터를 포함하며 해당 데이터와 관련된 기능을 함께 제공한다.
- 주의 해야 할 점은 실제 도메인 모델의 엔티티와 DB 관계형 모델의 엔티티는 같은 것이 아님을 알게 되었다. 이 두 모델의 가장 큰 차이점은 도메인 모델의 엔티티는 데이터와 함께 도메인 기능을 함께 제공한다는 점이다.
- 두번째 차이점은 모데인 모델의 엔티티는 두 개 이상의 데이터가 개념적으로 하나인 경우 밸류 타입을 이용해서 표현할 수 있다는 것이다.

### 밸류(Value)

- 고유의 식별자를 갖지 않는 객체로 주로 개념적으로 하나의 도메인 객체의 속성을 표현하는데 사용된다. 엔티티의 속성으로 사용될 뿐만 아니라 다른 밸류 타입의 속성으로 사용될 수 있다.
- 밸류는 불변으로 구현하는 것을 권장하는데, 데이터를 변경할 때 객체 자체를 완전히 교체한다는 것을 의미한다.

### 애그리거트(Aggregate)

- 관련된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것이다.
- 도메인 모델이 복잡해지면서 전체 구조가 아닌 한 개 엔티티오 밸류에만 집중하게 되는데, 상위 수준에서 모델을 관리하기보다 개별 요소에만 초점을 맞추다 보면 큰 수준에서 모델을 이해하지 못해 큰 틀에서 모델을 관리할 수 없는 상황에 빠질 수 있다.
- 애그리거트는 위와 같은 문제를 해결할 수 있고 쉽게 한마디로 정의한다면 관련 객체를 하나로 묶은 군집이다.
- 개별 객체가 아닌 관련 객체를 묶어서 객체 군집 단위로 모델을 바라볼 수 있게 되어 개별 객체 간의 관계가 아닌 애그리거트 간의 관계로 도메인 모델을 이해하고 구현할 수 있게 되며, 이를 통해 큰 틀에서 도메인 모델을 관리할 수 있게 된다.
- 군집에 속한 객체들을 관리하는 루트 엔티티를 갖게 되고 루트 엔티티는 애그리거트에 속해 있는 엔티티와 밸류 객체를 이용해서 애그리거트가 구현해야 할 기능을 제공한다.
- 애그리거트의 기능을 사용하려면 애그리거트 루트가 제공하는 기능을 실행하고 간접적으로 애그리거트 내의 다른 엔티티나 밸류 객체에 접근하게 되면서 내부 구현을 숨겨서 애그리거트 단위로 구현을 캡슐화 할 수 있다.
- 애그리거트를 구현할 때는 고려할 것이 많다. 어떻게 구성하냐에 따라 복잡도가 결정되고 트랜잭션 범위가 달라지고 구현 기술에 따라 구현에 제약이 생기기도 한다.

### 리포지터리(Repository)

- 도메인 모델의 영속성을 처리한다.

### 도메인 서비스(Domain Service)

- 특정 엔티티에 속하지 않은 도메인 로직을 제공한다. 도메인 로직이 여러 엔티티와 밸류를 필요로 할 경우 도메인 서비스에서 로직을 구현한다.

### 응용 서비스와 도메인 서비스의 연관

1. 응용 서비스는 필요한 도메인 객체를 구하거나 저장할 때 리포지터리를 사용한다.
2. 응용 서비스는 트랜잭션을 관리하는데, 트랜잭션 처리는 리포지터리 구현 기술에 영향을 받는다.
- 따라서 아래와같은 메서드를 제공한다.
1. 애그리거트를 저장하는 메서드
2. 애그리거트 루트 식별자로 애그리거트를 조회하는 메서드

## 요청 처리 흐름

![image](https://user-images.githubusercontent.com/53366407/121773516-65a99b80-cbb7-11eb-9cf2-7d37f50fc41c.png)

## 인프라스트럭처 개요

- 위에서 인프라스트럭처에 대한 의존을 없애라고 했지만, 무조건 적으로 없애는게 좋은 건 아니다.
- 예를 들면 스프링에서 트랜잭션 처리를 위해 사용하는 `@Transactional` 을 사용하는 것이 편리한 것 처럼, 또한 `@Entity`, `@Table` 과 같은 애노테이션을 사용하는 것이 편리한것 처럼 말이다.
- 구현의 편리함은 DIP가 주는 다른 장점만큼 중요하기 때문에 DIP의 장점을 해치지 않는 범위에서 응용 영역과 도메인 영역에서 구현 기술에 대한 의존을 가져가는 것이 현명하다.

## 모듈 구성

- 하위 도메인으로 나누고 각 하위 도메인마다 별도의 패키지를 구성한다. 또한 domain 모듈은 도메인에 속한 애그리거트 기준으로 다시 패키지를 구성한다.
![image](https://user-images.githubusercontent.com/53366407/121773518-693d2280-cbb7-11eb-9c22-bd58d37b2947.png)
![image](https://user-images.githubusercontent.com/53366407/121773521-6cd0a980-cbb7-11eb-9e4e-8ac601183efb.png)
![image](https://user-images.githubusercontent.com/53366407/121773524-70fcc700-cbb7-11eb-9cb4-8d8348a4b244.png)

- 위와 같이 모듈 구조를 세분화 했지만, 정답은 따로 존재하지 않다. 단지, 한 패키지에 너무 많은 타입이 몰려서 코드를 찾을 때 불편한 정도만 아니면 된다.
