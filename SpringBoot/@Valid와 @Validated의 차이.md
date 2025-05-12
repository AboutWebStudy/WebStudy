### 👉 @Valid와 @Validated의 역할

---

- Spring Boot에서 유효성 검사(Validation)를 수행할 때 사용하는 애너테이션
- 둘 다 Java Bean Validation(JSR-303, JSR-380)을 기반
    - JSR-303은 자바에서 객체 유효성 검사(Validation)를 위한 공식 표준 명세(= 전 세계 자바 개발자들이 함께 약속한 유효성 검사 규칙)
- 컨트롤러에서 요청 바디나 파라미터에 대해 검증을 수행할 때 사용
<br>
<br>

### 👉 @Valid

---

- @Valid는 JSR-303을 따르는 기본 애너테이션으로, 자바 진영에서 범용적으로 사용
- 중첩 객체의 유효성 검사 지원
    - 예를 들어, UserDto 안에 AddressDto라는 또 다른 객체가 있다면, @Valid는 그 내부 객체까지 따라가서 검사
        
        ```java
        public class UserDto {
            @NotBlank
            private String name;
        
            @Valid  // 내부 객체도 검사하라고 명시해야 함!
            private AddressDto address;
        }
        
        public class AddressDto {
            @NotBlank
            private String city;
        }
        ```
        
- 그룹(Group) 기능은 지원하지 않음
- 필드나 파라미터 수준에서 유효성 검사 수행
- List로 파라미터가 들어올 때는 `List <@Valid UserDto> userDto` 같은 형태로 작성
- 사용 예제
    
    ```java
    @PostMapping("/user")
    public ResponseEntity<?> createUser(@RequestBody @Valid UserDto userDto) {
        // userDto에 정의된 제약 조건을 검사함
    }
    ```
    <br>
    <br>
    

### 👉 @Validated

---

- Spring Framework에서 제공하는 확장된 애너테이션
- @Valid의 모든 기능 + 그룹 기능(Group Validation)을 지원
    - 그룹(Group) 기능은 검사 대상에 따라 유효성 검사 규칙을 다르게 적용하고 싶을 때 사용하는 기능
    - 사용자 정보를 등록할 때와 수정할 때 검사해야 할 필드가 다를 수 있음
        - 등록(Create) : username, password, email은 필수
        - 수정(Update) : email만 수정 가능하고 나머지는 필수가 아님
    - 이런 경우에 하나의 DTO로 유효성 검사 조건만 작업별로 나누어 사용할 수 있음
        - 그룹 인터페이스 정의
            
            ```java
            public interface OnCreate {}
            public interface OnUpdate {}
            ```
            
        - 제약 조건에 그룹 지정
            
            ```java
            public class UserDto {
            
                @NotBlank(groups = OnCreate.class) // 등록 시 필수
                private String username;
            
                @NotBlank(groups = OnCreate.class)
                private String password;
            
                @Email(groups = {OnCreate.class, OnUpdate.class}) // 둘 다 검사
                private String email;
            
                // getters/setters 생략
            }
            ```
            
        - 컨트롤러에서 @Validated로 그룹 지정
            
            ```java
            @PostMapping("/create")
            public ResponseEntity<?> createUser(@RequestBody @Validated(OnCreate.class) UserDto userDto) {
                // OnCreate 그룹에 해당하는 제약 조건만 검사
                ...
            }
            
            @PutMapping("/update")
            public ResponseEntity<?> updateUser(@RequestBody @Validated(OnUpdate.class) UserDto userDto) {
                // OnUpdate 그룹만 검사
                ...
            }
            ```
            
- 클래스나 메서드 수준에서 유효성 검사를 지정할 수 있음
    - 서비스 레이어에서도 메서드에 유효성 검사 적용 가능
        
        ```java
        @Service
        @Validated  // 클래스에 걸면 이 클래스 안의 메서드 파라미터에 대해 검사 가능
        public class UserService {
        
            public void saveUser(@Validated(OnCreate.class) UserDto userDto) {
                // 유효성 검사 실행됨
            }
        }
        ```
        
    - AOP 기반의 유효성 검사 등에 사용
- 사용 예제
    
    ```java
    @PostMapping("/user")
    public ResponseEntity<?> createUser(@RequestBody @Validated(OnCreate.class) UserDto userDto) {
        // 그룹(OnCreate)에 해당하는 제약 조건만 검사함
    }
    ```
