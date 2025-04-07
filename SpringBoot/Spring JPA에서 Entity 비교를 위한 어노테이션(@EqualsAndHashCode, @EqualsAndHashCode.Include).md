### 👉 공부하게 된 이유

---

- JPA를 사용하여 구현하다 보면 Entity에 관례처럼 사용하는 어노테이션 중 @EqualsAndHashCode와 @EqualsAndHashCode.Include의 정확한 역할을 알아보고 싶어서 공부를 하게 됨
<br>
<br>

### 👉 Entity 예제

---

```java
// import문 생략

@Getter
@Entity
@Table(name = "notification")
@SuperBuilder
@AllArgsConstructor
@NoArgsConstructor(access = PROTECTED)
@EqualsAndHashCode(onlyExplicitlyIncluded = true, callSuper = true)
public class Notification extends BaseEntity {

    @Id
    @EqualsAndHashCode.Include
    @UuidGenerator(style = RANDOM)
    @JdbcTypeCode(SqlTypes.VARCHAR)
    @Column(name = "notification_id", columnDefinition = "VARCHAR(36)")
    private UUID id;
    
    @Column(name = "notification_type")
    @Enumerated(EnumType.STRING)
    private NotificationType notificationType;
    
    @JdbcTypeCode(SqlTypes.VARCHAR)
    @Column(name = "target_id", columnDefinition = "VARCHAR(36)")
    private UUID targetId;
    
    @Column(name = "is_read")
    private boolean isRead;
    
    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    public void updateIsRead() {
        this.isRead = true;
    }
}
```
<br>
<br>

### 👉 `@EqualsAndHashCode`의 역할

---

- 클래스의 모든 필드를 기준으로 `equals()`와 `hashCode()` 메서드를 자동 생성하는 어노테이션
    - `hashCode()`는 객체를 식별하기 위한 정수 값을 반환하는 메서드
        - `HashMap`, `HashSet`, `Hashtable` 같은 해시 기반 컬렉션에서 객체를 빠르게 찾기 위해 사용
        - 이 메서드는 `equals()`랑 같이 작동 해야 함 ⇒ 같다고 판단되는 객체(`equals()가 ture인 경우`)는 같은 hashCode를 가져야 하기 때문(자바 규칙)
            - 만약 동일하지 않으면 `user1`과 `user2`의 `equals()` 결과는 `true`인데 `hashCode`가 달라서  `HashSet`에 둘 다 저장되는 경우가 생길 수 있음
    - `equals(Object obj)`는 두 객체가 논리적으로 같은지를 비교하는 데 사용
        - 기본적으로 `Object.equals()`는 참조 주소를 비교
        - 하지만 보통 원하는 건, ‘값이 같으면 같다’고 판단하는 것이기 때문에 Override 해서 사용
- **`onlyExplicitlyIncluded = true`**
    - `onlyExplicitlyIncluded = true`를 사용하면 명시적으로 포함된 필드만을 기준으로 equals/hashCode를 생성
        - ⇒ 이 때 사용하는 어노테이션이 `@EqualsAndHashCode.Include`
    - id 필드에만 `@EqualsAndHashCode.Include`를 붙였기 때문에, `equals()`와 `hashCode()`는 오직 id만 기준으로 생성
- **`callSuper = true`**
    - 부모 클래스의 equals()와 hashCode()도 호출해서 포함시키라는 의미
    - 예제에서 상속 받고 있는 BaseEntity에 `@EqualsAndHashCode.Include` 가 붙은 필드가 있다면 그것 또한 비교에 포함시킬 수 있게 하는 것
        
        ```java
        // 예시, 실제 사용하는 코드 X
        
        @MappedSuperclass
        @EqualsAndHashCode(onlyExplicitlyIncluded = true)
        public abstract class BaseEntity {
        
            @EqualsAndHashCode.Include
            @CreatedDate
            private LocalDateTime createdAt;
        }
        ```
        
        - 상속 구조에서 부모 필드가 식별에 중요한 경우에만 써야 함
        <br>
        <br>

### 👉 왜 `@EqualsAndHashCode`를 사용해야 할까?

---

- JPA에서는 엔티티는 식별자(id)를 기준으로 동일성 비교를 해야 함
    - 식별자가 같으면 같은 객체로 볼 수 있기 때문
- 모든 필드를 비교 대상으로 두거나 `equals()`랑 `hashCode()`를 Override를 하지 않으면 지연 로딩 프록시 객체 비교 이슈, 무한 루프 (양방향 관계) 등의 문제가 생길 수 있음
    <br>
    - **✅ 프록시 객체와 비교 이슈(지연 로딩)**
        - JPA는 지연 로딩 시 진짜 엔티티 대신 프록시 객체를 리턴할 수 있음
            
            ```java
            User user1 = entityManager.find(User.class, 1L);        // 실제 객체
            User user2 = someEntity.getUser();                      // 프록시 객체
            ```
            
        - 이 두 객체는 다른 클래스일 수도 있어도 같은 DB row를 나타내는데, `equals()`가 두 객체는 다르다고 나옴
            - user2는 Hibernate가 만든 프록시 객체로 User가 아니라 User를 상속한 무명 서브 클래스일 수 있음
            - 영속성 컨텍스트나 컬렉션에서 같은 엔티티인데 중복 저장되거나 못 찾음 ⇒ 데이터 무결성 깨짐
        - 그렇게 되면 `Set.contains()`나 `List.remove()`가 동작 하지 않음 ⇒ 영속성 컨텍스트 내 캐시 무결성 깨짐
            - 그래서 `equals`에서 `id`만 비교하고, 클래스 비교를 `instanceof` 또는 `Hibernate.getClass(this)` 같은 방법으로 우회
            - 또는 Lombok + `@EqualsAndHashCode(onlyExplicitlyIncluded = true)`로 id 기준으로만 비교하게 함
            <br>
    - **✅ 양방향 연관관계 무한 루프**
        
        ```java
        // 예시, 실제 사용하는 코드 X
        
        class User {
            @OneToMany(mappedBy = "user")
            List<Notification> notifications;
        }
        
        class Notification {
            @ManyToOne
            User user;
        }
        ```
        
        - 이 상황에서 `equals()`나 `hashCode()`에 `user` 또는 `notifications`가 포함돼 있으면 무한 순환 참조가 발생하거나 스택오버플로우가 발생할 수 있음
            - 만약 equals가 아래와 같으면 `Notification.equals()` 호출 → `user.equals()` 호출 → `notifications.equals()` 호출 → 다시 `Notification.equals()` 호출… 과 같이 끝 없이 자신을 호출하게 됨
                
                ```java
                public boolean equals(Object o) {
                    return this.user.equals(o.user);
                }
                ```
                
        - 그래서 연관관계 필드는 `equals()`/`hashCode()` 대상에서 제외해야 함 ⇒ `onlyExplicitlyIncluded = true` 필요
        <br>
    - **✅ 엔티티 상태 변화 시 equals/hashCode 깨짐**
        - 전체 필드를 hashCode의 대상으로 설정하면 데이터가 수정되었을 때, hashCode가 바뀜
        - `HashSet`이나 `HashMap`에 넣었다가 못 찾는 현상 등 발생
        - `equals()`/`hashCode()`는 가능한 한 불변 필드 기준(식별자)으로만 만들어야 함
