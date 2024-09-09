### 👉 궁금한 점

---

- DTO가 필요한 데이터만 다루기 위해 사용하는 게 주 목적이라고 생각했는데, 그러면 쿼리를 직접 매핑해서 사용하는 MyBatis에서는 크게 의미가 없는 게 아닐까? 라는 궁금증에서 찾아보게 됨
- [DTO, DAO, VO 정의에 대해 정리했던 글](https://velog.io/@reasonoflife39/JavaDTO-DAO-VO)
<br>
<br>

### 👉 DTO(Data Transfer Object) 사용 이유

---

- 애플리케이션 계층 간의 데이터를 캡슐화하여 전송하는 데 사용
    - 데이터의 무결성을 보호하고, 엔티티와 서비스 간의 결합도를 낮출 수 있음
    - 데이터베이스 엔티티와는 다른 용도
- 클라이언트에 필요한 데이터만을 선택적으로 제공할 수 있기 때문에 불필요한 데이터를 전송하지 않도록 할 수 있음
- 데이터를 전송하는 과정에서 특정 포맷으로 변환할 수 있게 함
    - 데이터베이스 엔티티의 필드 이름과 API 응답의 JSON 필드 이름이 다를 때 DTO를 사용하여 변환할 수 있음
    <br>
    <br>

### 👉 JPA와 DTO : 사용하는 이유와 이점

---

- JPA를 사용하면 엔티티 클래스가 데이터베이스 테이블과 직접적으로 매핑되며, 이는 엔티티 클래스가 데이터베이스의 모든 컬럼을 포함할 수 있음
- 따라서 엔티티를 그대로 사용하면 불필요한 데이터까지 모두 메모리에 로드하거나 네트워크를 통해 전송하게 됨
- 이런 상황에서 필요한 데이터만 포함한 DTO를 만들어 사용하면 성능 면에서 유리하고, 데이터 전송의 효율성을 높일 수 있음
- 그 외에는 아래의 MyBatis에서 사용하는 이유와 같음
<br>
<br>

### 👉 MyBatis와 DTO : 사용하는 이유와 이점

---

- DTO를 사용하면 데이터베이스 구조와 애플리케이션 구조를 분리할 수 있음
    - 애플리케이션 간의 결합도가 낮아짐 ⇒ DB 테이블 구조가 변경되더라도, 애플리케이션의 다른 계층에 영향을 최소화할 수 있음
- 각 경우마다 항상 쿼리를 작성하기 어려울 수 있음
    - 동일한 데이터베이스에서 데이터를 가져오는데, 각 기능마다 필요한 데이터가 다를 수 있기 때문에 DTO를 활용하면 매번 쿼리를 새로 짜는 상황을 개선할 수 있음
    - 매번 쿼리를 새로 짜서 매핑하는 것보다 DTO를 활용해서 필요한 데이터만 사용하면 유지보수 및 코드 가독성에 이점이 생김
- 데이터 캡슐화와 보안의 장점
    - 특정 필드가 외부에 노출되는 것을 방지하거나, 민감한 정보를 필터링하기 위해 DTO를 사용할 수 있음
- API의 응답 포맷이 데이터베이스 엔티티와 다를 때, DTO를 사용하여 변환할 수 있음
    - MyBatis는 데이터를 가져올 때 결과를 특정한 클래스로 매핑하는데 DTO를 사용하여 데이터베이스에서 가져온 데이터를 애플리케이션이 필요로 하는 형태로 변환할 수 있음
    - 예를 들어, TIMESTAMP 형식의 날짜를 필요에 따라 String 형식의 특정 날짜 형태로 변환해서 제공할 수 있음
    <br>
    <br>

### 👉 **MyBatis 사용 코드 예시**

---

```xml
<mapper namespace="com.example.mapper.UserMapper">

    <!-- 사용자 정보를 가져오는 쿼리 -->
    <select id="getUserById" resultType="com.example.dto.UserDTO">
        SELECT username, email, birth_date AS birthDate
        FROM users
        WHERE id = #{userId}
    </select>

</mapper>
```

```java
public class UserDTO {
    private String username;
    private String email;
    private LocalDate birthDate;

    public UserDTO(String username, String email, LocalDate birthDate) {
        this.username = username;
        this.email = email;
        this.birthDate = birthDate;
    }

    // ...
}
```

```java
public interface UserMapper {
    // 해당 Interface로 SQL 쿼리 결과를 DTO에 매핑하도록 정의
    UserDTO getUserById(Long id);
}
```

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public UserDTO getUserById(Long userId) {
        return userMapper.getUserById(userId);  // MyBatis가 SQL 쿼리를 실행하고, 결과를 UserDTO에 매핑
    }
}
```
