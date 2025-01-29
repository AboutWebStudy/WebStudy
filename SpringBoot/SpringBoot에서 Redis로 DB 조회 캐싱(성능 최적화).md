### 👉 Redis를 활용한 캐시의 사용 목적

---

- Spring Boot에서 Redis를 활용한 캐시는 주로 애플리케이션의 성능을 최적화하고 데이터베이스의 부하를 줄이는 데 사용
    - DB 조회 결과가 같은 경우면 매번 DB에 접근하지 않고, 캐시를 활용해 값을 가져올 수 있음
- Redis는 메모리 기반의 데이터 저장소로 빠른 데이터 접근 속도를 제공하며, 캐시용으로 매우 적합
<br>
<br>

### 👉 Redis 캐시를 주로 사용하는 곳

---

- 자주 조회되지만 잘 변하지 않는 데이터(예 : 제품 정보, 사용자 프로필 등)를 캐싱하여 데이터베이스 요청을 줄이는 경우
- 분산 환경에서 사용자 세션을 Redis에 저장하여 서버 간 세션 동기화 문제를 해결
- 외부 API 호출 결과를 캐싱하여 호출 비용 및 응답 시간을 줄임
- 계산 비용이 높은 연산 결과를 캐싱하여 재사용
<br>
<br>

### 👉 Spring의 @Cacheable과 @CacheEvict

---

- **`@Cacheable`**
    - `@Cacheable`은 메서드의 결과를 캐싱하거나, 캐시된 데이터를 반환하는 데 사용
    - 동작 방식
        - 메서드 호출 시, 먼저 지정된 캐시에서 결과를 조회
        - 캐시에 데이터가 없으면 메서드를 실행하여 결과를 생성한 뒤, 이를 캐시에 저장
            - ⇒ 캐시 미스(Cache Miss)
        - 이후 동일한 요청이 오면 캐시에 저장된 결과를 반환
            - ⇒ 캐시 히트(Cache Hit)
    - 속성
        
        
        | 속성 | 설명 |
        | --- | --- |
        | value | 캐시 이름을 지정. 여러 캐시를 사용할 수 있으며 배열 형태로도 정의 가능 |
        | key | 캐시 키를 지정(SpEL을 사용하여 키를 동적으로 설정할 수 있음) |
        | condition | 조건을 만족할 때만 캐싱을 수행(SpEL로 조건을 정의) |
        | unless | 조건을 만족할 경우 캐시하지 않음(SpEL로 조건을 정의) |
        | sync | true로 설정 시, 여러 스레드가 동시에 메서드를 호출하면 첫 번째 호출이 완료될 때까지 대기 |
        <br>
- **`@CacheEvict`**
    - @CacheEvict는 캐시에 저장된 데이터를 삭제하거나 무효화하는 데 사용
    - 동작 방식
        - 지정된 캐시에서 특정 키 또는 모든 데이터를 삭제
        - 데이터 삭제 후, 다음 요청이 오면 `@Cacheable`을 통해 캐시에 다시 저장
    - 속성
        
        
        | 속성 | 설명 |
        | --- | --- |
        | value | 캐시 이름을 지정 |
        | key | 삭제할 캐시 키를 지정(SpEL로 키를 동적으로 설정할 수 있음) |
        | allEntries | true로 설정하면 지정된 캐시의 모든 데이터를 삭제 |
        | condition | 조건을 만족할 때만 캐시를 무효화(SpEL로 조건을 정의) |
        | beforeInvocation | true로 설정하면 메서드 실행 전에 캐시를 삭제(기본값 false) |
        <br>
        <br>

### 👉 SpringBoot에서 Redis 캐시 사용 예시

---

- Redis 의존성 추가
    
    ```java
    dependencies {
    		...
        implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    }
    ```
    <br>
    
- `application.yml`에 Redis 설정 추가
    
    ```java
    spring:
      datasource:
        url: jdbc:mariadb://localhost:3306/sample
        username: root
        password: password
        driver-class-name: org.mariadb.jdbc.Driver
    
      jpa:
        hibernate:
          ddl-auto: update
        show-sql: true
    
      redis:
        host: localhost
        port: 6379
        cache:
          time-to-live: 60000 # 캐시 TTL (밀리초 단위, 60초)
    ```
    <br>
    
- Controller
    
    ```java
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;
    
    @RestController
    @RequestMapping("/products")
    public class ProductController {
    
        private final ProductService productService;
    
        public ProductController(ProductService productService) {
            this.productService = productService;
        }
    
        // 상품 조회 (캐싱 적용)
        @GetMapping("/{id}")
        public ResponseEntity<Product> getProductById(@PathVariable Long id) {
            Product product = productService.getProductById(id);
            return ResponseEntity.ok(product);
        }
    
        // 상품 생성
        @PostMapping
        public ResponseEntity<Product> createProduct(@RequestBody Product product) {
            Product savedProduct = productService.saveProduct(product);
            return ResponseEntity.status(HttpStatus.CREATED).body(savedProduct);
        }
    
        // 상품 캐시 삭제
        @DeleteMapping("/{id}/cache")
        public ResponseEntity<Void> evictProductCache(@PathVariable Long id) {
            productService.deleteProductCache(id);
            return ResponseEntity.noContent().build();
        }
    }
    
    ```
    <br>
    
- Entity
    
    ```java
    import jakarta.persistence.Entity;
    import jakarta.persistence.Id;
    import lombok.AllArgsConstructor;
    import lombok.Getter;
    import lombok.NoArgsConstructor;
    
    import static lombok.AccessLevel.PROTECTED;
    
    @Entity
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor(access = PROTECTED)
    public class Product {
        @Id
        private Long id;
        private String name;
        private int price;
    }
    
    ```
    <br>
    
- Repository
    
    ```java
    import org.springframework.data.jpa.repository.JpaRepository;
    
    public interface ProductRepository extends JpaRepository<Product, Long> {
    }
    ```
    <br>
    
- Serivce
    
    ```java
    import org.springframework.cache.annotation.CacheEvict;
    import org.springframework.cache.annotation.Cacheable;
    import org.springframework.stereotype.Service;
    
    @Service
    public class ProductService {
    
        private final ProductRepository productRepository;
    
        public ProductService(ProductRepository productRepository) {
            this.productRepository = productRepository;
        }
    
        // 캐싱된 데이터 조회
        @Cacheable(value = "products", key = "#id")
        public Product getProductById(Long id) {
            System.out.println("최초 조회는 MariaDB에서 조회"); // 캐시에 데이터가 존재할 경우, 메서드는 실행되지 않음
            return productRepository.findById(id)
                    .orElseThrow(() -> new RuntimeException("Product not found"));
        }
    
        // 데이터 저장 (MariaDB)
        public Product saveProduct(Product product) {
            return productRepository.save(product);
        }
    
        // 캐시 무효화
        @CacheEvict(value = "products", key = "#id") // @CacheEvict의 메서드는 항상 실행 됨
        public void deleteProductCache(Long id) {
            System.out.println("캐시 삭제");
        }
    }
    ```
    
    - 데이터는 MariaDB에 저장되고, 조회된 데이터는 Redis에 캐시 됨
    - 캐시 데이터는 설정된 TTL 동안 유지되거나, 수동으로 삭제 가능
    <br>
- RedisConfig
    
    ```java
    import org.springframework.cache.CacheManager;
    import org.springframework.cache.annotation.EnableCaching;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.data.redis.cache.RedisCacheConfiguration;
    import org.springframework.data.redis.cache.RedisCacheManager;
    import org.springframework.data.redis.connection.RedisConnectionFactory;
    
    import java.time.Duration;
    
    @EnableCaching
    @Configuration
    public class RedisConfig {
    
        @Bean
        public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
            RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                    .entryTtl(Duration.ofMinutes(10)) // 캐시 TTL 설정 (10분)
                    .disableCachingNullValues();
    
            return RedisCacheManager.builder(connectionFactory)
                    .cacheDefaults(config)
                    .build();
        }
    }
    ```
