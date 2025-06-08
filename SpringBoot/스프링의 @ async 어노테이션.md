`@Async` 어노테이션은 **Spring Framework**에서 비동기 처리를 지원하기 위해 사용되는 기능

주로 시간이 오래 걸리는 작업을 별도의 쓰레드에서 실행하여 **메인 쓰레드의 응답성을 향상**시키기 위해 사용

### 기본 개념

- `@Async`는 **메서드를 비동기로 실행**하게 만들어주는 어노테이션
- Spring이 관리하는 **TaskExecutor**를 사용해 별도의 쓰레드에서 메서드를 실행
- 메서드를 호출한 쪽은 즉시 반환되며, 실제 작업은 백그라운드에서 진행

### 사용 방법

1. **설정 활성화**

```jsx
@Configuration
@EnableAsync
public class AsyncConfig {
    // 커스텀 TaskExecutor 설정도 가능
}
```

1. **비동기로 실행할 메서드에 `@Async` 추가**

```java
@Service
public class MyService {

    @Async
    public void asyncMethod() {
        // 오래 걸리는 작업
        System.out.println("비동기 작업 수행 중...");
    }
}
```

1. **호출 예시**

```java
@RestController
public class MyController {

    @Autowired
    private MyService myService;

    @GetMapping("/start")
    public String startAsyncTask() {
        myService.asyncMethod(); // 비동기 호출
        return "요청 처리 완료 (비동기 작업 중)";
    }
}
```

---

### 반환값이 있는 경우

- `Future<T>`, `CompletableFuture<T>` 또는 `ListenableFuture<T>`로 감싸서 리턴 가능

```java
@Async
public CompletableFuture<String> asyncReturnMethod() {
    return CompletableFuture.completedFuture("비동기 결과");
}
```

### 주의사항

- `@Async` 메서드는 **프록시 기반**이므로 **같은 클래스 내부에서 호출하면 비동기로 동작하지 않음**
    
    → 자기 자신을 `@Autowired`로 주입받아 사용하거나 구조 분리를 권장
    
- 기본 쓰레드 풀은 제한적이므로 **커스텀 TaskExecutor**를 정의해주는 것이 좋음
- `@Async`는 `public` 메서드에서만 제대로 작동함

### 커스텀 Executor 왜 사용?

**기본 Executor는 `SimpleAsyncTaskExecutor`를 사용**

- Spring은 `@Async`를 사용할 때 별도 설정이 없으면 `SimpleAsyncTaskExecutor`를 사용
- 이 Executor는 **쓰레드 풀을 사용하지 않고**, 호출할 때마다 **새로운 쓰레드를 생성**

⇒ 따라서 쓰레드 새로 무한정 생성 및 재사용 불가능

- **쓰레드 재사용이 안 됨**
- **많은 요청이 들어오면 OutOfMemoryError 위험**
- **성능과 안정성 모두 낮음**

**커스텀 Executor 사용 예시**

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("AsyncExecutor-");
        executor.initialize();
        return executor;
    }
}
```
