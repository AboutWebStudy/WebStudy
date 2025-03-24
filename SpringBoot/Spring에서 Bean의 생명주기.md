### **👉 Spring에서 Bean 생명주기**

---

- Spring에서 **Bean(빈)** 은 스프링 컨테이너에 의해 관리되는 객체를 의미
- 이 Bean이 생성되고, 초기화되고, 사용되다가 소멸되기까지의 과정을 **Bean의 생명주기**라고 함
<br>
<br>

### **👉 Spring Bean의 생명주기 단계**

---

**1️⃣ 객체 생성(Instantiation)**

- Bean이 처음으로 생성되는 단계
- XML 기반 설정에서는 `<bean>` 태그, Java 기반 설정에서는 `@Bean` 또는 `@Component` 로 등록한 객체가 대상이 됨
    
    ```java
    @Component
    public class MyBean {
        public MyBean() {
            System.out.println("1. Bean 객체 생성");
        }
    }
    ```
    
- Bean의 기본 스코프는 `singleton`이므로 스프링 컨테이너가 시작될 때 Bean이 미리 생성됨
<br>

**2️⃣ 의존성 주입(Dependency Injection)**

- Bean이 생성된 후, `@Autowired`를 통해 필요한 의존성을 주입 받음
    - 스프링이 실행될 때, `@Autowired`가 붙은 필드, 생성자, 또는 Setter 메서드를 보고 알맞은 Bean을 찾아서 주입
- 의존성을 주입 받는 방법은 필드 인젝션, Setter 인젝션, 생성자 인젝션이 있음
    - **필드 인젝션**
        
        ```java
        @Component
        public class MyService {
            @Autowired
            private MyBean myBean; // 필드 인젝션
        
            public void doSomething() {
                myBean.someMethod();
            }
        }
        ```
        
        - `@Autowired`를 필드에 직접 붙여서 의존성을 주입하는 방식
        - 코드가 간결하지만 단점이 많아 잘 쓰이지 않음
            - 필드가 private이라 Mock 객체를 주입하려면 리플렉션을 사용해야 해서 테스트가 어려움
            - 의존성을 주입하려면 반드시 Spring 컨테이너가 필요
            - 필드 인젝션은 의존 관계가 복잡할 때 문제를 파악하기 어려워 순환 참조 등의 오류가 발생하면 해결이 어려움
            <br>
    - **Setter 인젝션**
        
        ```java
        @Component
        public class MyService {
            private MyBean myBean;
        
            @Autowired
            public void setMyBean(MyBean myBean) { // Setter 인젝션
                this.myBean = myBean;
            }
        
            public void doSomething() {
                myBean.someMethod();
            }
        }
        ```
        
        - @Autowired를 Setter 메서드에 붙여서 의존성을 주입하는 방식
        - 코드가 길어지고, `setMyBean()`을 호출해서 언제든지 값을 바꿀 수 있음 → 불변 객체를 만들기 어려움
            - 불변 객체란, 한 번 값이 정해지면 절대 바뀌지 않는 객체
            - 값이 절대 바뀌지 않으니까 어느 순간 누가 값을 바꿨는지 찾아다닐 필요 없음 ⇒ 디버깅이 쉬워짐
            - 멀티스레드 환경에서 값이 바뀌지 않으니까 별도로 synchronized 처리 안 해도 됨
            - 의도하지 않은 변경을 방지해 버그 발생 가능성 감소
            <br>
    - **생성자 인젝션**
        
        ```java
        @Component
        public class MyService {
            private final MyBean myBean;
        
            @Autowired
            public MyService(MyBean myBean) {
                this.myBean = myBean;
                System.out.println("2. 의존성 주입");
            }
        }
        ```
        
        - `@Autowired`를 생성자에 붙여서 의존성을 주입하는 방식
        - 불변성을 유지할 수 있고, 테스트가 용이하므로 가장 많이 사용
        - Spring 컨테이너 없이도 사용 가능 → 일반 Java 객체처럼 사용할 수 있음
        - 테스트가 쉽고(생성자를 통해 직접 Mock 객체를 주입할 수 있음), 순환 참조 문제를 쉽게 발견할 수 있음
        - Spring 4.3 이후부터는 생성자가 하나인 경우 `@Autowired` 생략 가능
            
            ```java
            @Component
            public class MyService {
                private final MyBean myBean;
            
                public MyService(MyBean myBean) { // @Autowired 생략 가능
                    this.myBean = myBean;
                }
            }
            ```
            
            ```java
            @Component
            @RequiredArgsConstructor // 필수(final) 필드만 가지고 생성자 만들어주는 역할
            public class MyService {
                private final MyBean myBean;
            }
            ```
            
            - 롬복의 `@RequiredArgsConstructor`를 사용하면 생성자 코드도 생략 가능 ⇒ final이나 @NonNull이 붙은 필드만 골라서 생성자를 자동으로 만들어주는 롬복 어노테이션
            <br>

**3️⃣ 초기화(Initialization)**

- Bean이 생성된 후, 개발자가 초기화 작업을 수행할 수 있도록 제공하는 단계
- `@PostConstruct` 또는 `InitializingBean` 인터페이스를 사용할 수 있음
    
    ```java
    @Component
    public class MyBean {
        @PostConstruct
        public void init() {
            System.out.println("3. 초기화 로직");
        }
    }
    ```
    
    - 예를 들어, DB 연결이나 캐시 로딩 같은 작업을 여기서 하면 됨
    <br>

**4️⃣ 사용(Usage)**

- 애플리케이션이 실행되는 동안 Bean을 사용하게 됨
- 서비스 로직이 실행되는 동안 Bean이 정상적으로 동작하는 단계
<br>

**5️⃣ 소멸(Destruction)**

- 애플리케이션이 종료될 때, Bean이 소멸되는 단계
- `@PreDestroy` 또는 `DisposableBean` 인터페이스를 사용해서 정리 작업을 수행할 수 있음
    
    ```java
    @Component
    public class MyBean {
        @PreDestroy
        public void destroy() {
            System.out.println("4. 소멸 단계 - 리소스 정리");
        }
    }
    ```
    <br>
    <br>
    

### **👉 생명주기 콜백 메서드 설정 방법**

---

- Spring에서는 여러 가지 방법으로 Bean의 생명주기 콜백을 설정할 수 있음
    - 콜백 메서드란, Spring이 알아서 호출해주는 메서드
    <br>

**1️⃣ @PostConstruct, @PreDestroy 사용**

- 스프링 5 이후 가장 일반적으로 사용되는 방법
- 클래스 내부에 `@PostConstruct`(초기화)와 `@PreDestroy`(소멸) 메서드를 정의해서 사용
    
    ```java
    @Component
    public class MyBean {
        @PostConstruct
        public void init() {
            System.out.println("초기화 실행");
        }
    
        @PreDestroy
        public void cleanup() {
            System.out.println("소멸 실행");
        }
    }
    ```
    <br>
    

**2️⃣ InitializingBean, DisposableBean 인터페이스 구현**

- `InitializingBean`의 `afterPropertiesSet()` 메서드에서 초기화 로직을 작성할 수 있음
- `DisposableBean`의 `destroy()` 메서드에서 소멸 시 실행할 로직을 작성할 수 있
    
    ```java
    @Component
    public class MyBean implements InitializingBean, DisposableBean {
    
        @Override
        public void afterPropertiesSet() {
            System.out.println("초기화 실행!");
        }
    
        @Override
        public void destroy() {
            System.out.println("소멸 실행");
        }
    }
    ```
    <br>
    

**3️⃣ Bean 설정 파일에서 initMethod, destroyMethod 지정**

- `@Bean` 어노테이션을 사용할 때, `initMethod`와 `destroyMethod` 속성을 지정할 수도 있음
    
    ```java
    @Configuration
    public class AppConfig {
        @Bean(initMethod = "init", destroyMethod = "cleanup")
        public MyBean myBean() {
            return new MyBean();
        }
    }
    ```
    
    ```java
    public class MyBean {
        public void init() {
            System.out.println("초기화 실행");
        }
    
        public void cleanup() {
            System.out.println("소멸 실행");
        }
    }
    ```
    <br>
    

### **👉 Spring Bean의 스코프와 생명주기 차이**

---

- Bean의 생명주기는 스코프(Scope) 에 따라 달라질 수 있음
    
    
    | 스코프 | 설명 |
    | --- | --- |
    | `singleton` | 컨테이너 내에서 Bean이 한 개만 생성됨 ⇒ 기본값 |
    | `prototype` | 매번 새로운 Bean 인스턴스를 생성함 |
    | `request` | HTTP 요청마다 새로운 Bean을 생성 |
    | `session` | HTTP 세션마다 새로운 Bean을 생성 |
    <br>

**1️⃣ Singleton과 Prototype의 생명주기 차이**

```java
@Component
@Scope("prototype")
public class MyPrototypeBean {
    @PostConstruct
    public void init() {
        System.out.println("Prototype Bean 초기화");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Prototype Bean 소멸");
    }
}
```

- `singleton` 스코프의 경우 컨테이너가 시작될 때 Bean이 생성되지만, `prototype` 스코프는 매번 새로운 객체가 생성됨
- `prototype` 스코프의 Bean은 컨테이너가 직접 관리하지 않기 때문에 `@PreDestroy`가 실행되지 않음
    - ⇒ 정리 작업이 필요하면 수동으로 호출해야 함
        
        ```java
        @Scope("prototype")
        public class MyPrototypeBean {
            public void cleanup() {
                System.out.println("수동으로 정리 작업 실행");
            }
        }
        ```
        
    - `prototype` 빈이 싱글톤 빈에 의존성 주입이 되어 있으면 매번 새로운 빈이 생성되지 않음 ⇒ `ObjectProvider` 사용하면 매번 새로운 `prototype` 빈을 주입 받을 수 있음
        
        ```sql
        @Component
        public class SingletonBean {
            @Autowired
            private ObjectProvider<ProtoBean> protoBeanProvider;
        
            public void doWork() {
                ProtoBean protoBean = protoBeanProvider.getObject(); // 사용할 때마다 새로 생성
                System.out.println(protoBean);
            }
        }
        ```
        
        - `ObjectProvider`는 Spring에서 제공하는 지연 조회(필요할 때만 꺼내서 쓰는) 도구로, `prototype` 빈처럼 매번 새로운 객체가 필요할 때 유용
        <br>
        <br>

### **👉 실무에서 Bean 생명주기 활용 예시**

---

- **예제 : 데이터베이스 연결 관리**
    - Bean이 초기화될 때 DB 연결을 열고, 애플리케이션이 종료될 때 연결을 닫는 방식으로 활용 가능
        
        ```java
        @Component
        public class DatabaseConnection {
            private Connection connection;
        
            @PostConstruct
            public void connect() {
                // DB 연결 로직
                System.out.println("DB 연결 설정 완료");
            }
        
            @PreDestroy
            public void disconnect() {
                // DB 연결 해제 로직
                System.out.println("DB 연결 해제");
            }
        }
        ```
        <br>
        

- **예제 : 캐시 초기화 및 정리**
    - 애플리케이션 시작 시 캐시 데이터를 로딩하고, 종료 시 캐시를 정리하는 방식으로 활용 가능
        
        ```java
        @Component
        public class CacheManager {
            private Map<String, String> cache = new HashMap<>();
        
            @PostConstruct
            public void loadCache() {
                // 캐시 데이터 로딩
                System.out.println("캐시 데이터 로딩 완료");
            }
        
            @PreDestroy
            public void clearCache() {
                // 캐시 데이터 삭제
                cache.clear();
                System.out.println("캐시 정리 완료");
            }
        }
        ```
