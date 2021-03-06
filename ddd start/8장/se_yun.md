# 8장. 애그리거트 트랜잭션 관리

## 애그리거트와 트랜잭션

![image](https://user-images.githubusercontent.com/53366407/127736664-94b157c2-63d5-4897-b9bc-68405f056a6e.png)

- 위의 사진을 보면 운영자가 배송 상태로 변경을 하고 고객이 배송지를 변경해버렸다. 메모리 캐시를 사용하지 않은 경우 두 스레드는 같은 주문 애그리거트를 나타내는 다른 객체를 구하게 될텐데, 운영자 스레드가 변경한 것이 고객 스레드에 영향을 주지 않아 변경을 하게 될것이다.
- 그렇다면 배송 상태로 변경했는데 고객은 배송지를 변경하고 실제 이전 배송지로 배송이 될것이다.
- 그럼 이 문제를 어떻게 해결할 수 있을까?
    - 운영자가 변경하는 동안 고객이 변경하지 못하도록 한다.
    - 운영자가 배송지 정보 조회 이후 고객이 정보를 변경하면 운영자가 애그리거트를 다시 조회한 뒤 수정하도록 한다.
- 문제 해결 방법 둘다 트랜잭션과 관련이 있다.
- 바로 선점(Pessimistic) 잠금과 비선점(Optimistic) 잠금이다.

## 선점 잠금

![image](https://user-images.githubusercontent.com/53366407/127736668-bbc93995-8ade-406d-881c-daae33c2d9f5.png)

- 먼저 애그리거트를 구한 스레드가 애그리거트 사용이 끝날때 까지 다른 스레드가 해당 애그리거트를 수정하는 것을 막는 방식이다.
- 즉 잠금을 하는데, 잠금을 해제 되기 전까지 블로킹을 할 것이다.
- 그 이후 트랜잭션을 커밋하면 잠금이 해제되고 애그리거트 접근이 가능하다.
- 이렇게 하면 데이터 충돌을 막을수가 있을 것이다.
- 선점 잠금은 DBMS의 행 단위 잠금을 사용해서 구현한다.

### 선점 잠금과 교착상태

- 여기서 주의해야 할게 있는데, 교착 상태(deadlock)가 발생하지 않도록 주의해야 한다는 것이다.
- 예를들어 아래와 같은 순서로 진행된다고 해보자.
    - 스레드 1이 A 애그리거트에 대한 선점 잠금을 구함
    - 스레드 2가 B 애그리거트에 대한 선점 잠금을 구함
    - 스레드 1이 B 애그리거트에 대한 선점 잠금을 시도한다.
    - 스레드 2가 A 애그리거트에 대한 선점 잠금을 시도한다.
- 위와 같이 순서로 진행되면 서로 선점 잠금을 구할 수 없어 교착 상태에 빠지게 될 것이다.
- 따라서 잠금을 구할 때 까지 최대 대기 시간을 지정하는 것이 중요하다.
- cf) DBMS마다 교착 상태에 빠진 커넥션을 처리하는 방법이 다르다.
    - 쿼리별로 대기 시간을 지정할 수 있기도 하고 커넥션 단위로만 대기 시간을 지정할수도 있다.

## 비선점 잠금

![image](https://user-images.githubusercontent.com/53366407/127736669-79ad748a-51fa-4071-92b6-91ce61b5c893.png)

- 선점 잠금만으로 처리할 수 없을때까지 있다. 바로 위와 같은 상황인데 배송 상태를 변경하는 사이에 고객이 배송지를 변경할수도 있다.
- 잠금을 해서 동시에 접근하는 것을 막는 대신 변경한 데이터를 실제 DBMS에 반영하는 시점에 변경 가능 여부를 확인하는 방식이다.

![image](https://user-images.githubusercontent.com/53366407/127736672-6a670d5a-3cfb-4ab1-b5a5-1014480b0fde.png)

- 즉, 버전을 가지고 같은 버전을 갖는 애그리거트를 읽어와 수정하고 있는데 버전이 업데이트가 되면 실패하도록 만드는 것이다.

![image](https://user-images.githubusercontent.com/53366407/127736675-d83418a1-1f62-4c7a-b4cd-d0239ffc2125.png)

### 강제 버전 증가

- 애그리거트에 애그리거트 루트 외에 다른 엔티티가 존재하는데 기능 실행 도중 루트가 아닌 다른 엔티티의 값만 변경된다고 하면 루트 엔티티의 값을 증가하지 않을 것이다.
- 따라서 강제로 업데이트를 해야 한다.

## 오프라인 선점 잠금

- 엄격하게 데이터 충돌을 막고 싶다면 수정 화면을 보고 있을 때 수정 화면 자체를 실행하지 못하도록 하는 것이 있다.
- 여러 트랜잭션에 걸쳐 동시 변경을 막는 것이 바로 오프라인 선점 잠금이다.
- 그러나 수정 요청을 수행하지 않고 프로그램을 종료해버린다면 영원히 잠금을 구할 수 없는 상황이 되버릴 것이다. 이런 사태를 방지하기 위해 잠금 유효 시간을 가져야 한다.
- 따라서 잠금 선점 시도, 잠금 확인, 잠금 해제, 락 유효 시간 연장 네가지 기능을 제공해야 한다.
