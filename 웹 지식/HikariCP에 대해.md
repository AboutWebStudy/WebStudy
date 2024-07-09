### 👉 HikariCP란?

---

- Java 애플리케이션에서 데이터베이스 커넥션 풀링(connection pooling)을 관리하는 매우 빠르고, 간단하며, 경량의 JDBC Connection Pool 라이브러리
  <br>
  <br>

### 👉 주요 역할 및 특징

---

- **커넥션 풀링**
    - 데이터베이스 연결 풀링은 미리 일정 수의 데이터베이스 연결을 생성해 두고 애플리케이션이 필요할 때마다 이 연결들을 재사용하는 방식
    - 이렇게 하면 매번 새로운 연결을 생성하고 닫는 데 드는 오버헤드를 줄일 수 있어 성능이 크게 향상됨
    <br>
- **성능 최적화**
    - HikariCP는 빠르고 경량으로 설계되어 있어 매우 효율적인 연결 풀링을 제공 ⇒ 높은 성능과 낮은 지연 시간
        - 커넥션 풀의 크기 최적화
        - 비동기 I/O 처리로, 데이터베이스와의 통신을 최적화
        - 스레드 관리 최적화
        <br>
- **안정성**
    - 안정적이고 신뢰성 있는 연결 풀링을 제공
        - 안정적인 연결 유지를 위한 메커니즘을 갖고 있음
        - 오류 감지를 모니터링 해서 문제가 발생하면 연결을 제거하고, 새로운 연결로 교체
        - 연결이 필요 없을 때는 유휴 상태로 두어 자원 절약
    - 다양한 구성 옵션을 제공하여 필요에 맞게 최적화할 수 있음
    <br>
- **스프링과의 통합**
    - 스프링 부트는 기본적으로 HikariCP를 데이터베이스 연결 풀링 라이브러리로 사용
    - 별도의 설정 없이도 스프링 부트에서 HikariCP를 쉽게 사용할 수 있도록 통합되어 있음
  <br>
  <br>

### 👉 설정 방법

---

- 스프링 부트는 기본적으로 HikariCP를 사용하도록 설정되어 있기 때문에 `spring-boot-starter-jdbc`나 `spring-boot-starter-data-jpa` 종속성을 추가하기만 하면 됨
- 필요에 따른 추가 설정
    
    ```yaml
    spring:
      datasource:
        url: jdbc:mariadb://localhost:3306/database
        username: username
        password: password
        driver-class-name: org.mariadb.jdbc.Driver
        hikari:
          maximum-pool-size: 10   # 연결 풀에서 유지할 수 있는 최대 커넥션 수
          minimum-idle: 5         # 연결 풀에서 유지할 수 있는 최소 유휴 연결 수
          idle-timeout: 30000     # 유휴 상태의 연결이 풀에서 제거되기까지 대기하는 시간(밀리초)
          pool-name: MyHikariCP   # 연결 풀의 이름
          max-lifetime: 1800000   # 풀에서 살아있을 수 있는 최대 연결 수명(밀리초)
          connection-timeout: 30000  # 연결을 얻기 위해 기다릴 최대 시간(밀리초)
    ```
