### 👉  세션 인증 방식

---

- **1️⃣ 개념**
    - 사용자가 로그인 → 서버는 세션을 생성하고, 세션 ID를 클라이언트(브라우저)에 쿠키로 저장해 보냄
    - 이후 요청마다 브라우저가 세션 ID 쿠키를 자동으로 전송해서 인증을 처리
        
        ```java
        [클라이언트]
          ↓ (요청 + JSESSIONID 쿠키)
        [서버]
          ↓
        HttpSession → SecurityContext → Authentication → UserDetails
        ```
        
        - 브라우저는 자동으로 `Cookie: JSESSIONID=xxxx`를 요청에 포함
        - 서버는 이 세션 ID로 세션을 조회하고, 거기서 `SecurityContext`를 꺼냄
        - `SecurityContext.getAuthentication()`으로 로그인 여부와 사용자 정보를 확인
    - 세션 정보는 서버 메모리(혹은 DB)에 저장되어 있음
    <br>
- **2️⃣ 특징**
    - 상태 기반 인증 = 서버가 사용자의 상태를 기억
    - 서버에 세션 저장소 필요
    - CSRF 공격에 취약 → 토큰으로 방지 가능
    - 주로 브라우저 기반 웹 애플리케이션에서 사용(주로 서버 사이드 랜더링에서 사용)
    <br>
- **3️⃣ 구조 요약**
    
    ```java
    [Client]
       │
       ▼
    [GET /login] - 로그인 페이지 요청
       │
       ▼
    [스프링 시큐리티 로그인 폼 렌더링]
       │
       ▼
    [POST /login] - 로그인 정보 전송 (username, password)
       │
       ▼
    [UsernamePasswordAuthenticationFilter] (스프링 시큐리티 내장)
       │
       ▼
    [UserDetailsService] → 사용자 검증 → Authentication 객체 생성
       │
       ▼
    [SecurityContextHolder]에 인증 객체 저장
       │
       ▼
    [HttpSession] 생성 → 세션 ID(`JSESSIONID`) 쿠키로 클라이언트에 전달
       │
       ▼
    [Client: 이후 모든 요청에 자동으로 JSESSIONID 쿠키 포함]
       │
       ▼
    [서버는 JSESSIONID로 세션 조회 → SecurityContextHolder 복원]
       │
       ▼
    [Controller에서 @AuthenticationPrincipal로 사용자 정보 접근]
    ```
    <br>
    <br>
    

### 👉 세션 인증 방식 로그인 예제

---

- SecurityConfig 설정
    
    ```java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {
    
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            http
                .csrf().disable()
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/login", "/register").permitAll()
                    .anyRequest().authenticated()
                )
                .formLogin(form -> form
                    .loginPage("/login")
                    .defaultSuccessUrl("/home", true)
                )
                .logout(logout -> logout
                    .logoutSuccessUrl("/login")
                );
    
            return http.build();
        }
    }
    ```
    
    - `formLogin()`을 설정하고 로그인이 성공하면, 서블릿 컨테이너은 자동으로 세션을 생성하고 `JSESSIONID`라는 쿠키를 브라우저에 전송 → 그 쿠키는 이후 요청마다 자동으로 함께 전송
        - `/login` 폼으로 로그인 정보 전송(POST)
        - 인증 성공 시 스프링 시큐리티가 `HttpSession` 생성
        - 톰캣이 `JSESSIONID`라는 쿠키를 생성해서 응답 헤더에 포함
        - 클라이언트는 `Set-Cookie` 헤더를 통해 세션 ID 쿠키를 받아 저장
        - 이후 요청마다 자동으로 `Cookie: JSESSIONID=...` 헤더 포함
    - 🐙 로그인 한 유저인지는 세션 ID로 어떻게 확인 하는지?
- UserDetailsService 구현 : 사용자 조회
    
    ```java
    @Service
    public class CustomUserDetailsService implements UserDetailsService {
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            // DB에서 사용자 조회
            return User.builder()
                    .username(username)
                    .password("password") // 비밀번호 인코딩 주의
                    .roles("USER")
                    .build();
        }
    }
    ```
    
- Controller 구현
    
    ```java
    @Controller
    public class LoginController {
    
        @GetMapping("/login")
        public String loginPage() {
            return "login"; // login.html 반환
        }
    
        @GetMapping("/home")
        public String home() {
            return "home"; // 로그인 성공 시 보여줄 페이지
        }
    }
    ```
    

### 👉  토큰(JWT) 인증 방식

---

- **1️⃣ 개념**
    - 사용자가 로그인 → 서버는 사용자 정보를 기반으로 JWT 토큰을 생성해 클라이언트에 반환
    - 클라이언트는 이 토큰을 이후 모든 요청의 Authorization 헤더에 포함해서 보냄
    - 서버는 이 토큰을 검증해서 사용자 인증을 수행
    <br>
- **2️⃣ 특징**
    - 무상태(stateless) 인증 = 서버가 사용자 상태를 저장하지 않음
    - 서버 확장성 높음(세션 저장 필요 없음)
    - 주로 SPA, 모바일 앱에서 사용
    - 토큰 탈취에 주의 필요(HTTPS 필수)
    <br>
- **3️⃣ 구조 요약**
    
    ```java
    [Client]
       │
       ▼
    [POST /login] - 로그인 요청
       │
       ▼
    [AuthController] → [UserDetailsService] → 사용자 검증 → JWT 발급
       │
       ▼
    [클라이언트는 JWT 저장 후 모든 요청에 Authorization 헤더로 보냄]
       │
       ▼
    [JwtAuthenticationFilter] → JWT 검증 → SecurityContext에 인증 객체 설정
       │
       ▼
    [Controller에서 @AuthenticationPrincipal로 사용자 정보 접근]
    ```
    <br>
    <br>
    

### 👉 토큰(JWT) 인증 방식 예제

---

- User 엔티티
    
    ```java
    @Getter
    @Entity
    public class User {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String username;
        private String password;
        private String role; // 예: ROLE_USER, ROLE_ADMIN
    }
    
    ```
    
- UserDetailsService 구현 : 사용자 조회
    
    ```java
    @Service
    public class CustomUserDetailsService implements UserDetailsService {
    
        private final UserRepository userRepository;
    
        public CustomUserDetailsService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
    
            return new CustomUserDetails(user);
        }
    }
    ```
    
- JWT 유틸 클래스
    
    ```java
    @Component
    public class JwtUtil {
    
        private final String secret = "mySecretKey"; // 실제로는 환경변수로 관리
    
        public String generateToken(UserDetails userDetails) {
            return Jwts.builder()
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 3600000)) // 1시간
                .signWith(SignatureAlgorithm.HS256, secret)
                .compact();
        }
    
        public String extractUsername(String token) {
            return Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
        }
    
        public boolean validateToken(String token, UserDetails userDetails) {
            String username = extractUsername(token);
            return username.equals(userDetails.getUsername());
        }
    }
    ```
    
- JWT 필터 생성
    
    ```java
    public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
        private final JwtUtil jwtUtil;
        private final CustomUserDetailsService userDetailsService;
    
        public JwtAuthenticationFilter(JwtUtil jwtUtil, CustomUserDetailsService userDetailsService) {
            this.jwtUtil = jwtUtil;
            this.userDetailsService = userDetailsService;
        }
    
        @Override
        protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                        FilterChain filterChain) throws ServletException, IOException {
    
            String header = request.getHeader("Authorization");
    
            if (header != null && header.startsWith("Bearer ")) {
                String token = header.substring(7);
    
                try {
                    String username = jwtUtil.extractUsername(token);
                    UserDetails userDetails = userDetailsService.loadUserByUsername(username);
    
                    if (jwtUtil.validateToken(token, userDetails)) {
                        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                            userDetails, null, userDetails.getAuthorities());
                        SecurityContextHolder.getContext().setAuthentication(authToken);
                    }
    
                } catch (Exception e) {
                    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    return;
                }
            }
    
            filterChain.doFilter(request, response);
        }
    }
    ```
    
    - 필터에서 토큰을 확인해서 로그인한 사용자면 `SecurityContext`에 사용자 정보를 넣음 ⇒ 토큰 검증, 사용자 인증 수행
- SecurityConfig 설정
    
    ```java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {
    
        private final JwtUtil jwtUtil;
        private final CustomUserDetailsService userDetailsService;
    
        public SecurityConfig(JwtUtil jwtUtil, CustomUserDetailsService userDetailsService) {
            this.jwtUtil = jwtUtil;
            this.userDetailsService = userDetailsService;
        }
    
        @Bean
        public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {
            return http.getSharedObject(AuthenticationManagerBuilder.class)
                .userDetailsService(userDetailsService)
                .passwordEncoder(passwordEncoder())
                .and()
                .build();
        }
    
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
    
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            JwtAuthenticationFilter jwtFilter = new JwtAuthenticationFilter(jwtUtil, userDetailsService);
    
            http
                .csrf().disable()
                .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/login").permitAll()
                    .anyRequest().authenticated()
                )
                .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    
            return http.build();
        }
    }
    ```
    
- Controller 구현
    
    ```java
    @RestController
    public class AuthController {
    
        private final AuthenticationManager authenticationManager;
        private final JwtUtil jwtUtil;
        private final CustomUserDetailsService userDetailsService;
    
        public AuthController(AuthenticationManager authenticationManager, JwtUtil jwtUtil, CustomUserDetailsService userDetailsService) {
            this.authenticationManager = authenticationManager;
            this.jwtUtil = jwtUtil;
            this.userDetailsService = userDetailsService;
        }
    
        @PostMapping("/login")
        public ResponseEntity<Map<String, String>> login(@RequestBody Map<String, String> request) {
            String username = request.get("username");
            String password = request.get("password");
    
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(username, password)
            );
    
            SecurityContextHolder.getContext().setAuthentication(authentication);
    
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            String token = jwtUtil.generateToken(userDetails);
    
            return ResponseEntity.ok(Collections.singletonMap("token", token));
        }
    }
    ```
    <br>
    <br>
    

### 👉 사용자의 로그인 여부 검증 : `@AuthenticationPrincipal`

---

```java
@GetMapping("/mypage")
public String myPage(@AuthenticationPrincipal UserDetails userDetails) {
    String username = userDetails.getUsername();
    return "안녕하세요, " + username + "님!";
}
```

- 권한 제어는 `@PreAuthorize` 사용
    
    ```java
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/admin")
    public String adminPage() {
        return "관리자 페이지";
    }
    ```
