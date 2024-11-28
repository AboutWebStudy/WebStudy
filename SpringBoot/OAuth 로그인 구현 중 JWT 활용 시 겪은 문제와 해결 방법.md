### 👉 최초 OAuth 회원가입 방식

---

- OAuth 회원가입 시, OAuth 서버에서 받을 수 있는 정보 외에도 필수 입력 값들이 있었음
    - 회원가입이 안 된 사용자인 경우, OAuth에서 받은 정보들을 화면에 바디로 넘겨주고 회원가입 화면으로 리다이렉트 시킴
    - 넘겨준 값은 화면에서 고정하고, 추가 정보를 input으로 받아 서버에 POST 가입 요청을 보내는 걸로 구상
- ⇒ 프론트에서 바디의 값에 접근하지 못한다고 해서 방식 수정
    - 리다이렉트 시, 바디의 값에 접근하지 못하고 바로 페이지가 이동 된다고 함
    <br>
    <br>

### 👉 OAuth 회원가입에 추가 정보를 안 받는 걸로 변경

---

- 추가 정보를 받지 않고 OAuth 요청 시 회원이면 로그인, 비회원이면 가입 시키고 로그인 처리
    
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class OAuth2UserSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
    
        private final TokenProvider tokenProvider;
        private final UserRepository userRepository;
        private final RedisService redisService;
        private final ObjectMapper objectMapper;
    
        @Override
        public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            OAuth2CustomUser oAuth2CustomUser = (OAuth2CustomUser) authentication.getPrincipal();
    
            // 기존 유저인 경우 로그인 성공 응답 처리
            User user = userRepository.findByEmail(oAuth2CustomUser.getName()).orElseThrow(()
                    -> new NotFoundEmailException(NOT_FOUND_EMAIL_EXCEPTION));
    
            String accessToken = tokenProvider.generateAccessToken(authentication);
            String refreshToken = tokenProvider.generateRefreshToken(authentication);
    
            JwtTokenRes jwtTokenRes = JwtTokenRes.from(accessToken, refreshToken, user);
            redisService.saveValue(user.getEmail(), jwtTokenRes.getRefreshToken());
    
            createResponseHandler(response, jwtTokenRes);
    
            if (user.isDefaultNickname()) {
                response.setHeader("Location", "/nickname");
                response.setStatus(HttpServletResponse.SC_TEMPORARY_REDIRECT);
                return;
            }
    
            response.sendRedirect("/");
        }
    
        private void createResponseHandler(HttpServletResponse response, JwtTokenRes jwtTokenRes) throws IOException {
            response.addHeader(JwtFilter.ACCESS_AUTHORIZATION_HEADER, "Bearer " + jwtTokenRes.getAccessToken());
            response.addHeader(JwtFilter.REFRESH_AUTHORIZATION_HEADER, "Bearer " + jwtTokenRes.getRefreshToken());
        }
    }
    ```
    
- JWT AccessToken과 RefreshToken을 헤더에 추가
- 필수 값으로 생각했던 값(닉네임)은 DB에 Nullable로 변경하지 않고, OAuth 로그인 하면 `UUID.RandomUUID()`로 임의의 값을 넣어줌
    - `isDefaultNickname`이라는 컬럼을 추가해 임의의 값이 닉네임으로 설정된 경우 로그인 시에 닉네임 설정 페이지로 리다이렉트 되게 로직 추가
- ⇒ 프론트에서 해당 헤더에도 접근이 안 된다고 해서 다른 방식 구상
<br>
<br>

### 👉 AccessToken과 RefreshToken 쿠키에 담기

---

- 주변 개발자 분에게 여쭤보고 하니 AccessToken과 RefreshToken은 주로 쿠키에 담아서 넘겨주고, 프론트에서 해당 쿠키를 그대로 서버에 넘겨서 사용한다고 함
    - 이 방식으로 구현하기 위해서는 CORS 설정이 필요
        
        ```java
        @Configuration
        public class CorsConfig {
        
            @Bean
            public CorsFilter corsFilter() {
                UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
                CorsConfiguration config = new CorsConfiguration();
                config.setAllowCredentials(true);
                config.addAllowedOriginPattern("http://localhost:3000");
                config.addAllowedHeader("*");
                config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE"));
                config.addExposedHeader("Set-Cookie");
        
                source.registerCorsConfiguration("/**", config);
                return new CorsFilter(source);
            }
        }
        ```
        
        ```java
        private void createResponseHandler(HttpServletResponse response, JwtTokenRes jwtTokenRes) throws IOException {
            Cookie accessTokenCookie = new Cookie("AccessToken", jwtTokenRes.getAccessToken());
            Cookie refreshTokenCookie = new Cookie("RefreshToken", jwtTokenRes.getRefreshToken());
        
            accessTokenCookie.setHttpOnly(true);
            refreshTokenCookie.setHttpOnly(true);
        
            // path 설정
            accessTokenCookie.setPath("/");
            refreshTokenCookie.setPath("/");
        
            // https 통신용
            accessTokenCookie.setSecure(true);
            refreshTokenCookie.setSecure(true);
            
            // 서드파티 쿠키 차단 해제 ⇒ setSecure(true)랑 같이 쓰여야 함
            accessTokenCookie.setSameSite("None");
            refreshTokenCookie.setSameSite("None");
        
            accessTokenCookie.setMaxAge(24 * 60 * 7);
            refreshTokenCookie.setMaxAge(24 * 60 * 7);
        
            response.addCookie(accessTokenCookie);
            response.addCookie(refreshTokenCookie);
        }
        ```
        
    - 프론트에서도 `credential: true`설정이 필요
- 로컬에서 테스트 할 때는 문제 없었으나, 현재 프론트 테스트 방식이 `AWS 개발 서버로 요청` → `로그인 되면 localhost:3000으로 리다이렉트`
    - 도메인이 달라서 setSameSite를 None으로 설정해줘야 했는데 그러려면 HTTPS 통신이 필요
    - 테스트를 위해 HttpOnly(false) 등으로 변경하고 진행
    - 이렇게 해도 프론트에서 토큰에 접근이 불가능 하다고 함
    <br>
    <br>

### 👉 최종 방법 : 쿼리 스트링으로 임의의 식별 값을 넘기고 프론트에서 요청 받는 방식으로 변경

---

- 보안 이슈나 프론트와 원활한 작업을 위해서 토큰 넘기는 방식 변경
    - OAuth 로그인 성공 시, `UUID.RandomUUID`로 임의의 값과 사용자의 이메일을 Redis에 저장
    - 리다이렉트 하며 해당 임의의 값을 프론트에 쿼리 스트링으로 전달
    - 프론트에서 해당 값을 포함해 다시 서버로 요청
    - 프론트에서 넘긴 값이 Redis의 값과 같고 메일이 같으면 토큰 발행해서 쿠키로 넘김
- 구현 방식
    
    ```java
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class OAuth2UserSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
    
        private final UserRepository userRepository;
        private final RedisService redisService;
    
        @Override
        public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            OAuth2CustomUser oAuth2CustomUser = (OAuth2CustomUser) authentication.getPrincipal();
    
            // 기존 유저인 경우 로그인 성공 응답 처리
            User user = userRepository.findByEmail(oAuth2CustomUser.getName()).orElseThrow(()
                    -> new NotFoundEmailException(NOT_FOUND_EMAIL_EXCEPTION));
    
            // auth redis 저장
            String authToken = UUID.randomUUID().toString();
            redisService.saveAuthValue(authToken, user.getEmail());
    
            if (user.isDefaultNickname()) {
                String redirectUrl = String.format("/nickname?code=%s", authToken);
                response.setHeader("Location", redirectUrl);
                response.setStatus(HttpServletResponse.SC_TEMPORARY_REDIRECT);
                return;
            }
    
            String redirectUrl = String.format("http://localhost:3000/code=%s", authToken);
            response.sendRedirect(redirectUrl);
        }
    }
    ```
    
    ```java
    @Slf4j
    @RestController
    @RequestMapping("/tokens")
    @RequiredArgsConstructor
    public class TokenController implements TokenApi {
    
        private final TokenService tokenService;
    
        @PostMapping
        public ResponseEntity<JwtTokenRes> createOauthToken(@RequestBody CreateOauthToken dto, HttpServletResponse response) {
            JwtTokenRes res = tokenService.createOauthToken(dto);
    
            ResponseCookie accessTokenCookie = ResponseCookie.from("ACCESS_TOKEN", res.getAccessToken())
                    .httpOnly(true)
                    .secure(true)
                    .path("/")
                    .maxAge(24 * 60)
                    .sameSite("None")
                    .build();
    
            ResponseCookie refreshTokenCookie = ResponseCookie.from("REFRESH_TOKEN", res.getRefreshToken())
                    .httpOnly(true)
                    .secure(true)
                    .path("/")
                    .maxAge(24 * 60 * 7)
                    .sameSite("None")
                    .build();
    
            response.addHeader(HttpHeaders.SET_COOKIE, accessTokenCookie.toString());
            response.addHeader(HttpHeaders.SET_COOKIE, refreshTokenCookie.toString());
    
            return ResponseEntity.status(CREATED).build();
        }
    
        @PostMapping("/reissue")
        public ResponseEntity<AccessTokenRes> reGenerateToken(@AuthenticatedUser AuthenticatedUserReq auth,
                                                              @RequestHeader("RefreshToken") String refreshToken,
                                                              HttpServletResponse response) {
            AccessTokenRes res = tokenService.reissueToken(auth, refreshToken);
    
            ResponseCookie accessTokenCookie = ResponseCookie.from("ACCESS_TOKEN", res.getAccessToken())
                    .httpOnly(true)
                    .secure(true)
                    .path("/")
                    .maxAge(24 * 60)
                    .sameSite("None")
                    .build();
    
            response.addHeader(HttpHeaders.SET_COOKIE, accessTokenCookie.toString());
    
            return ResponseEntity.status(OK).build();
        }
    }
    ```
    
    ```java
    @Getter
    @Builder
    @NoArgsConstructor(access = PROTECTED)
    @AllArgsConstructor
    public class CreateOauthToken {
    
        private String authUUID;
    }
    ```
    
    ```java
    @Slf4j
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class TokenService {
    
        private final RedisService redisService;
        private final UserService userService;
        private final TokenProvider tokenProvider;
    
        /**
         * OAuth 용 Token 발급
         */
        public JwtTokenRes createOauthToken(CreateOauthToken dto) {
            User user = userService.findByEmail(redisService.getValue(dto.getAuthUUID()));
            // 조회 후 삭제
            redisService.deleteValue(dto.getAuthUUID());
    
            JwtTokenRes jwtTokenRes = tokenProvider.generateOAuthToken(user);
            redisService.saveValue(user.getEmail(), jwtTokenRes.getRefreshToken());
    
            return jwtTokenRes;
        }
    
        /**
         * accessToken 재발급
         */
        public AccessTokenRes reissueToken(AuthenticatedUserReq auth, String refreshToken) {
            String token = redisService.getValue(auth.getEmail());
    
            if(refreshToken.equals(token)){
                Authentication authentication = tokenProvider.getAuthentication(token);
                String accessToken = tokenProvider.generateAccessToken(authentication);
    
                return AccessTokenRes.from(accessToken);
            }
    
            throw new NotValidRefreshTokenException(NOT_VALID_REFRESH_TOKEN_EXCEPTION);
    
        }
    }
    ```
    <br>
    <br>
    

### 👉 느낀 점

---

- 프론트에 대한 지식 부족과 주로 어떤 방식으로 사용하는지 알지 못해 많이 헤맨 것 같지만 여러 방법을 고민해보고 적용하며 공부 할 수 있었다
