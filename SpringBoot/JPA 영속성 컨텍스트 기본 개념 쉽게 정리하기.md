### 👉 JPA란?

---

- JPA는 ‘Java Persistence API’의 줄임말
- 자바로 데이터베이스를 쉽게 다루게 해주는 기술
- 원래는 JDBC로 SQL을 직접 짜야 했는데, JPA는 객체(Entity)랑 테이블을 자동으로 매칭해서 개발자가 객체만 다루면 알아서 DB에 반영되게 도와줌
<br>
<br>

### 👉 영속성 컨텍스트(Persistence Context)란?

---

- ‘Persistence’는 데이터를 저장하는 걸 뜻함
- ‘Context’는 환경/상태 관리 공간이라고 생각하면 됨
- 영속성 컨텍스트란, JPA가 엔티티 객체(= DB의 레코드에 해당하는 자바 객체)를 '저장하고 관리하는 공간'
    - 이 공간은 메모리에 존재하고, DB랑 바로 연결되지 않고 중간 단계
    - 비영속 상태는 길 가던 사람이고, 영속 상태는 회사에 입사해서 사원증 받은 직원이라고 비유하면 쉬움
- 매번 DB에 접근하면 느리고 번거롭기 때문에, JPA는 메모리에 먼저 객체를 저장하고, 나중에 필요한 순간에 한 번에 DB에 반영하는데 이 과정을 위해 영속성 컨텍스트가 필요
<br>
<br>

### 👉 영속성 컨텍스트의 주요 특징

---

- 엔티티를 영속 상태로 만듦
    - `em.persist(entity)` 하면 엔티티가 영속성 컨텍스트에 저장 ⇒ 영속 상태
    - 이때부터는 엔티티를 따로 저장하거나 수정하라고 명령하지 않아도, JPA가 알아서 관리하는 상태
    - `persist()` 하면 JPA가 ‘얘는 내가 책임지고 관리할게!’ 상태가 되는 것
    <br>

**1️⃣ 1차 캐시(First-level Cache)**

- 영속성 컨텍스트는 1차 캐시 역할도 함
    - 같은 엔티티를 또 조회하면, DB에 다시 물어보지 않고 메모리에서 꺼냄
        
        ```java
        Member member1 = em.find(Member.class, 1L); // DB 조회
        Member member2 = em.find(Member.class, 1L); // DB 안 가고 메모리에서 가져옴
        ```
        
        - 여기서 member1 == member2는 true ⇒ 같은 객체
        <br>

**2️⃣ 변경 감지(Dirty Checking)**

- 영속성 컨텍스트에 저장된 엔티티를 수정만 해도, 별도 저장 명령 없이 트랜잭션 커밋할 때 자동으로 UPDATE 쿼리를 만들어서 DB에 반영
    
    ```java
    Member member = em.find(Member.class, 1L);
    member.setName("변경된 이름"); // 그냥 setter만 호출
    em.getTransaction().commit(); // commit할 때 UPDATE 쿼리 자동 생성
    ```
    
    - 따로 `update()`를 부르지 않아도 DB에 반영
    <br>

**3️⃣ 동일성(identity) 보장**

- 같은 트랜잭션 안에서는, 같은 ID를 가진 엔티티는 항상 같은 객체
    - ⇒ 1차 캐시를 통해 '엔티티 동일성'을 보장
    <br>

4️⃣ **플러시(Flush)**

- 영속성 컨텍스트에 저장해놨던 변경 사항들을 DB에 밀어 넣는 작업
- 트랜잭션을 `commit()`할 때 자동으로 flush 됨
- 수동으로 할 때 : `em.flush()`
- 주의할 점은, flush는 DB에 보내는 것이고, 트랜잭션을 끝내는 것은 아님
    - ⇒ 트랜잭션이 살아있으면 롤백도 가능
    <br>
    <br>

### 👉 JPA 엔티티 생명주기 : 비영속 ➔ 영속 ➔ 준영속 ➔ 삭제

---

- **1️⃣ 비영속(Transient)**
    
    ```java
    Member member = new Member(); // 비영속 상태
    ```
    
    - 엔티티 객체를 `new`로만 만들었을 때, 아직 아무런 관리도 안 되고 있는 상태
    - DB랑도 아무런 연결 없음
    - 그냥 메모리에만 있는 상태로 JPA는 신경 안 씀
    <br>

- **2️⃣ 영속(Persistent)**
    
    ```java
    em.persist(member); // 영속 상태
    ```
    
    - `em.persist(member);`를 호출하면 영속성 컨텍스트에 등록됨 ⇒ JPA가 관리 시작
    - 수정사항도 자동 추적(DIRTY CHECKING)하고, 나중에 자동으로 DB에 반영
    <br>

- **3️⃣ 준영속(Detached)**
    - 영속 상태였던 엔티티를 영속성 컨텍스트에서 떼어낸 상태 ⇒ JPA가 더 이상 관리하지 않음
    - 객체는 여전히 메모리에 살아있지만, DB 반영은 안 됨
    - 준영속이 되는 조건
        - `em.detach(member)`  : 특정 엔티티만 관리 끊기
        - `em.clear()`  : 영속성 컨텍스트를 통째로 초기화
        - `em.close()`  : EntityManager 자체를 닫아버리기
        <br>

- **4️⃣ 삭제(Removed)**
    - `em.remove(member);` 하면 삭제 상태가 됨
        - 이걸 호출하면 JPA 내부 메모리(영속성 컨텍스트) 안에서 `member` 객체가 "삭제 예정" 표시가 붙는다고 보면 됨
    - 트랜잭션 커밋하면 DB에서도 실제로 삭제됨
        - 트랜잭션을 커밋하는 순간, JPA가 "삭제 예정 표시가 있는 객체"를 보고 DELETE 쿼리를 만들어서 DB에 진짜로 삭제 명령을 날림
        <br>
        <br>

### 👉 실제 JPA를 사용할 때 동작 방법

---

- `persist()`를 직접 쓰는 경우 → 거의 없음
    - 직접 em.persist(entity)를 호출하는 경우는 거의 없음
    - 대신 스프링 데이터 JPA 같은 걸 쓰면, 예를 들어 `save()` 메서드를 호출하는 것만으로 내부에서 자동으로 `persist()`나 `merge()`가 처리됨
        
        ```java
        memberRepository.save(member);
        ```
        <br>
- `detach()`를 직접 쓰는 경우 → 매우 드묾
    - `detach()`는 일부러 객체를 영속성 컨텍스트에서 분리하고 싶을 때 쓰는데, 일반적인 CRUD에서는 잘 안 씀
    - 다만, 특정 성능 최적화나, 특별한 상황(ex : 대용량 조회 후 메모리 부담 줄이기)에서 쓸 수 있음
    <br>
- **⇒ 영속성 관리는 자동**
    - 스프링에서는 보통 한 메서드 안에서 트랜잭션을 열고, 메서드가 끝나면 트랜잭션이 자동으로 커밋되고, 그 과정에서 persist나 dirty checking이나 flush가 자동으로 일어남
    - 개발자는 ‘단순히 객체를 만들고 값만 바꾼다’는 식으로 개발
