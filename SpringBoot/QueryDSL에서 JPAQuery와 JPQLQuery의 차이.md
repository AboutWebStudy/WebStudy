### 👉 궁금했던 점

---

- QueryDSL 코드를 짜면서 찾다 보니 데이터 타입을 JPAQuery와 JPQLQuery로 섞어 사용하길래 둘의 차이를 알아보게 됨
<br>
<br>

### 👉 JPQL(Java Persistence Query Language)이란?

---

- JPAQuery와 JPQLQuery 둘 다 QueryDSL에서 JPQL 기반의 쿼리를 작성할 때 사용되는 클래스
- JPQL은 JPA에서 제공하는 객체 지향 SQL로, 엔티티와 속성 이름을 사용하여 데이터베이스 쿼리를 작성
    - QueryDSL은 JPQL을 더 타입 안전하고, 코드로 작성하기 쉽게 만든 라이브러리로  Java 코드로 쿼리를 작성하면 JPQL 쿼리를 내부적으로 생성
    <br>
    <br>

### 👉 JPQLQuery

---

- JPQLQuery는 QueryDSL에서 제공하는 상위 **인터페이스**
    - JPQLQuery는 QueryDSL의 기본 인터페이스로, 구현체가 다양함
    - 실제로는 특정 JPA 구현체와 밀접히 연결(종속)되지 않으며, 독립적으로 쿼리 구조를 정의
- JPQL 기반 쿼리를 작성하기 위한 추상화된 기능을 제공
- 예시
    
    ```java
    JPQLQuery<User> query = new JPAQuery<>(); // EntityManager 불필요
    List<User> users = query.select(user)
                            .from(user)
                            .fetch();
    ```
    
- QueryDSL에서 추상적인 쿼리를 작성하려는 경우와 JPA를 직접 사용하지 않거나 특정 데이터베이스 구현체에 종속적이지 않은 코드가 필요할 때 사용
<br>
<br>

### 👉 JPAQuery

---

- JPAQuery는 JPQLQuery의 구현체
- JPA를 사용하는 환경에서 구체적인 JPQL 쿼리를 실행하는 데 사용
- JPA와 관련된 기능(e.g., 페이징 처리, flush, clear 등)을 포함
- JPA 환경에서 효율적으로 동작하며, JPA의 엔티티 매니저를 활용해 쿼리를 실행
- 예시
    
    ```java
    JPAQuery<User> query = new JPAQuery<>(entityManager);
    List<User> users = query.select(user)
                            .from(user)
                            .fetch();
    ```
    
- JPA 환경에서 EntityManager와 함께 QueryDSL을 사용할 때나, 쿼리 실행, 페이징, 트랜잭션 관리 등을 JPA와 연동해 처리할 때 사용
