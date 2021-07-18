# 5장 리포지터리의 조회 기능

# 검색을 위한 스펙

- 검색 조건의 조합이 다양해지면 모든 조합별로 find 메서드를 정의할 수 없다.
- 스펙: 애그리거트가 특정 조건을 충족하는지 여부를 검사한다.

```java
public interface Specification<T> {
	public boolean isSatisfiedBy(T aggregate);
}
```

- 리포지터리는 스펙을 전달받아 애그리거트를 걸러내는 용도로 사용한다.

```java
public class MemoryOrderRepository implements OrderRepository {
	public List<Order> findAll(Specification specification) {
		return findAll().stream()
				.filter(order -> specification.isSatisfiedBy(order)
				.collect(toList())
	}
}
```

- 특정 조건을 충족하는 애그리거트를 찾으려면 원하는 스펙을 생성하여 리포지터리에 전달해 주면 된다.

## 스펙 조합

- 두 스펙을 AND 연산자나 OR 연산자로 조합해서 새로운 스펙을 만들 수 있다.
- 조합한 스펙을 다시 조합해서 더 복잡한 스펙을 만들 수 있다.

```java
public class AndSpecification<T> implements Specification<T> {
	private List<Specification<T>> specifications;

	public AndSpecification(Specification<T> ...specifications) {
		this.specifications = Arrays.asList(specifications);
	}

	public boolean isSatisfiedBy(T aggregate) {
		return Arrays.stream(specifications)
				.allMatch(specification -> specification.isSatisfiedBy(aggregate);
	}
}
```

```java
Specification<Order> ordererSpecification = new OrdererSpecification("madvirus");
Specification<Order> orderDateSpecification = new OrderDateSpecification(fromDate, toDate);
AndSpecification<Order> specification = new AndSpecification(ordererSpecification, orderDateSpecification);
List<Order> orders = orderRepository.findAll(specification);
```

# JPA를 위한 스펙 구현

- 실제 구현에서는 모든 데이터를 가져와 필터링하지 않고 쿼리 WHERE절을 통해 처리한다.
- JPA `CriteriaBuilder`와 `Predicate`를 이용하여 검색 조건을 구현할 수 있다.

## JPA 스펙 구현

- 147p 참조
- criteria를 사용할 때 정적 메타 모델을 이용하거나 문자열을 사용할 수 있는데, 컴파일 타입에 오타를 방지하기 위해서는 type safety를 제공하는 정적 메타 모델을 이용하는 것이 좋다.
- 조회에 필요한 specfication을 하나의 클래스에 모아 작성할 수 있다.

## 스펙을 사용하는 JPA 리포지터리 구현

- 153p 참조

### 리포지터리 구현 기술 의존

- JPA Specification 인터페이스는 JPA Root, CriteriaBuilder에 의존하고 있으므로 JPA 의존적이게 된다.
- 리포지터리 구현 기술에 의존하지 않는 Specification을 만들기 위해서는 많은 부분을 추상화 해야하므로 편의상 사용한다.

# 정렬 구현

- 정렬 순서를 응용 서비스에 결정하는 경우에는 정렬 순서를 리포지터리에 전달할 수 있어야 한다.

### 정렬 순서를 타입으로 구현하기

```java
SortOrder sortOrder = new Ascending("name");
List<Member> members = memberRepository.findAll(specifications, sortOrder);
```

## 페이징과 개수 구하기 구현

# 조회 전용 기능 구현

- 조회 조건이 복잡하면 JPQL이나 Criteria로 처리하기 어렵다.
- 리파지토리를 조회 목적으로 사용하는 것은 적합하지 않다.

## 동적 인스턴스 생성

- 163p 참조
- 조회 전용 모델은 표현 영역을 통해 사용자에게 데이터를 보여주기 위해 만든다.
- 동적 인스턴스를 이용하면 JPQL을 그대로 사용하여 객체 기준으로 쿼리를 작성하면서 동시에 지연/즉시로딩과 같은 고민 없이 원하는 모습으로 데이터를 조회할 수 있다.

## 하이버네이트 @SubSelect 사용

- 쿼리결과를 @Entity로 매핑할 수 있다.
- 매핑 필드를 수정하면 업데이트 쿼리를 수행하려고 시도하고, 매핑한 테이블이 없어 실패한다.
- @Immutable을 사용해야 매핑 테이블의 값을 변경하려는 시도를 하지 않아 DB에 결과가 반영되지 않는다.
