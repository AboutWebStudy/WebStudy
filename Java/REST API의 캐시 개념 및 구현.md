### 👉 궁금해진 이유

---

- REST API에서 REST의 특징 중 Chacheable이 있는데 한 번도 다뤄보거나 알아본 적이 없어서 알아 보게 됨
<br>
<br>

### 👉 REST API에서 캐시란?

---

- 캐시는 자주 사용되는 데이터나 결과를 일시적으로 저장하여 다음 요청에서 재활용하는 기술
<br>
<br>

### 👉 캐시의 기본적인 흐름

---

- 클라이언트가 요청을 보냄
- 서버는 해당 요청에 대한 응답을 생성하여 클라이언트에 전달
- 클라이언트 또는 중간 캐시 계층(브라우저, 프록시, CDN 등)이 응답 데이터를 저장
    - 서버가 클라이언트나 중간 계층에게 "이 데이터를 얼마나 오래, 어떤 조건으로 캐싱해도 되는지"를 알려주는 역할을 해야 하기 때문에 서버의 설정이 필요
- 동일한 요청이 발생하면 서버에 요청을 보내지 않고 저장된 데이터를 재사용
<br>
<br>

### 👉 HTTP 헤더로 캐시를 구현하는 방법

---

- 주로 클라이언트(브라우저) 또는 중간 계층(프록시, CDN)에 적용할 때 사용
- 서버가 클라이언트와 중간 캐시 계층에게 데이터를 얼마나 오래 저장할지, 데이터가 변경되었는지 등을 알려줌
- 클라이언트(브라우저 캐시) 또는 프록시/중간 서버에 캐싱
- 특징
    - REST API 응답에 캐싱 정책을 포함시켜 네트워크 요청 자체를 줄임
    - Cache-Control로 데이터 만료 시간 지정
    - ETag로 데이터 변경 여부 확인
    - 클라이언트가 캐싱 여부를 결정하도록 제어함
    <br>
    <br>

### 👉 HTTP 헤더를 활용한 캐시 구성 요소

---

- `Cache-Control` : 캐싱 동작을 제어하는 데 사용
    - public : 응답을 모든 캐시(클라이언트, 프록시 등)가 저장 가능
    - private : 응답을 클라이언트만 캐시 가능
    - no-store : 캐시 금지(로그인 같은 보안 데이터)
    - max-age=<seconds> : 응답 데이터의 유효 기간을 설정
    - 예시
        
        ```
        Cache-Control: max-age=3600
        ```
  <br>
        

- `ETag` : 데이터 변경 여부를 확인하는 데 사용
    - 클라이언트는 `If-None-Match` 헤더와 함께 이전 ETag 값을 전송
    - 서버는 변경이 없으면 304 Not Modified를 반환하여 데이터 재전송을 피함
    - 예시
        
        ```
        HTTP/1.1 200 OK
        Content-Type: application/json
        ETag: "abcd1234"
        ```
        
        - 서버는 요청을 처리하고, 응답에 ETag 값을 포함시켜 클라이언트에게 보냄
            - 서버는 클라이언트에게 "이 데이터의 고유한 버전(해시값 같은 것)"을 알려줌
        - 클라이언트는 응답을 로컬 캐시에 저장하면서, ETag 값도 함께 저장
        
        ```
        GET /api/data HTTP/1.1
        Host: example.com
        If-None-Match: "abcd1234"
        ```
        
        - 이후 클라이언트가 동일한 데이터를 요청하려고 할 때, If-None-Match 헤더를 포함하여 서버에 요청을 보냄
        - 서버는 클라이언트가 보낸 ETag 값과 현재 데이터의 ETag 값을 비교
        - 서버는 데이터가 변경되지 않았으면 304 Not Modified를 응답하고, 클라이언트는 기존 캐시 데이터를 그대로 사용
            
            ```
            HTTP/1.1 304 Not Modified
            ```
            
            - 만약 변경 되었다면 서버는 새 데이터를 전달하며, 새로운 ETag 값을 포함
                
                ```
                HTTP/1.1 200 OK
                Content-Type: application/json
                ETag: "efgh5678"
                ```
  <br>
                

- `Expires` :  응답이 만료되는 시점을 명시(`Cache-Control: max-age`가 더 우선 적용됨)
    - 이건 데이터의 유효 기간을 명시적으로 알려주는 헤더
    - 예시
        
        ```
        Expires: Wed, 25 Nov 2024 12:00:00 GMT
        ```
        
        - 특정 시간까지 캐시된 데이터를 사용할 수 있다는 뜻이지만 보통 `Cache-Control: max-age`가 우선권을 가짐
  <br>
  <br>

### 👉 Spring Boot로 HTTP 캐시 구현하기

---

- **Cache-Control 설정**
    - 간단한 설정(단일 컨트롤러)
        
        ```java
        import org.springframework.http.CacheControl;
        import org.springframework.http.ResponseEntity;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        import java.util.concurrent.TimeUnit;
        
        @RestController
        public class MyController {
        
            @GetMapping("/api/data")
            public ResponseEntity<String> getData() {
                return ResponseEntity.ok()
                        .cacheControl(CacheControl.maxAge(1, TimeUnit.HOURS).cachePublic()) // 1시간 캐싱, public
                        .body("Hello, World!");
            }
        }
        ```
        
        - **`maxAge`** : 캐싱 기간을 초 단위로 설정 (`TimeUnit`으로 가독성 있게 작성 가능)
        - **`cachePublic`** : 모든 클라이언트가 캐시 데이터를 사용할 수 있도록 설정
            - 말고, `cachePrivate`도 있는데 이는 브라우저와 같은 특정 클라이언트에서만 캐싱한다는 뜻
    - 글로벌 설정
        
        ```java
        import org.springframework.context.annotation.Configuration;
        import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
        import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
        
        @Configuration
        public class WebConfig implements WebMvcConfigurer {
        
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                registry.addResourceHandler("/static/**")
                        .setCacheControl(CacheControl.maxAge(1, TimeUnit.HOURS).cachePublic());
            }
        }
        ```
  <br>
        

- **ETag 설정**
    - 직접 구현(컨트롤러나 필터 중 택 1)
        - 컨트롤러
            
            ```java
            import org.springframework.http.ResponseEntity;
            import org.springframework.web.bind.annotation.GetMapping;
            import org.springframework.web.bind.annotation.PathVariable;
            import org.springframework.web.bind.annotation.RestController;
            
            @RestController
            public class ItemController {
            
                @GetMapping("/api/items/{id}")
                public ResponseEntity<Item> getItem(@PathVariable Long id) {
                    // 데이터 조회
                    Item item = findItemById(id); // 데이터베이스에서 조회(원래는 서비스 호출)
            
                    // 데이터의 수정 시간을 기반으로 ETag 생성
                    String eTag = Integer.toHexString((item.getId() + item.getUpdatedAt().toString()).hashCode());
            
                    // 클라이언트가 보낸 ETag 확인
                    String clientETag = RequestContextHolder.getRequestAttributes()
                                            .getRequest().getHeader("If-None-Match");
            
                    // 클라이언트 ETag와 서버 ETag가 같다면 변경 없음
                    if (clientETag != null && clientETag.replace("\"", "").equals(eTag)) {
                    if (eTag.equals(clientETag)) {
                        return ResponseEntity.status(304).build(); // 304 Not Modified
                    }
            
                    // 데이터와 새로운 ETag 반환
                    return ResponseEntity.ok()
                            .eTag(eTag)
                            .body(item);
                }
            
                private Item findItemById(Long id) {
                    // 예제 데이터
                    return new Item(id, "Sample Item", LocalDateTime.now());
                }
            }
            ```
            
        - Filter 활용
            
            ```java
            import javax.servlet.Filter;
            import javax.servlet.FilterChain;
            import javax.servlet.ServletRequest;
            import javax.servlet.ServletResponse;
            import javax.servlet.http.HttpServletRequest;
            import javax.servlet.http.HttpServletResponse;
            
            import org.springframework.stereotype.Component;
            
            @Component
            public class ETagFilter implements Filter {
            
                @Override
                public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
                    HttpServletRequest httpRequest = (HttpServletRequest) request;
                    HttpServletResponse httpResponse = (HttpServletResponse) response;
            
                    // 데이터를 기반으로 ETag 생성 (간단히 URL 해시로 예제 구현)
                    String eTag = Integer.toHexString(httpRequest.getRequestURI().hashCode());
            
                    // 클라이언트의 If-None-Match 헤더 확인
                    String clientETag = httpRequest.getHeader("If-None-Match");
                    if (eTag.equals(clientETag)) {
                        // ETag가 동일하면 304 Not Modified
                        httpResponse.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                        return;
                    }
            
                    // ETag 헤더 추가
                    httpResponse.setHeader("ETag", eTag);
            
                    // 다음 필터로 요청 전달
                    chain.doFilter(request, response);
                }
            }
            ```
            
    - SpringBoot의 `ShallowEtagHeaderFilter` 활성화하는 방법
        
        ```java
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        import org.springframework.web.filter.ShallowEtagHeaderFilter;
        
        import javax.servlet.Filter;
        
        @Configuration
        public class WebConfig {
        
            @Bean
            public Filter shallowEtagHeaderFilter() {
                return new ShallowEtagHeaderFilter();
            }
        }
        ```
  	
  	- ShallowEtagHeaderFilter는 응답 본문을 기반으로 ETag를 자동 생성하며, 클라이언트가 보내는 If-None-Match 헤더와 비교해 304 응답을 반환
  
  	- 대규모 데이터 처리에는 추가적인 성능 비용이 발생할 수 있음
  
  <br>
  <br>

- **Expires 설정**
    
    ```java
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.time.LocalDateTime;
    import java.time.ZoneId;
    import java.util.Date;
    
    @RestController
    public class MyController {
    
        @GetMapping("/api/data")
        public ResponseEntity<String> getData() {
            HttpHeaders headers = new HttpHeaders();
            headers.setExpires(Date.from(LocalDateTime.now().plusHours(1)
                    .atZone(ZoneId.systemDefault()).toInstant()).getTime()); // 1시간 후 만료
    
            return ResponseEntity.ok()
                    .headers(headers)
                    .body("Hello, World!");
        }
    }
    ```
