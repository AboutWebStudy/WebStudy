### 👉 JWT(JSON Web Token)란?

---

- 웹 애플리케이션에서 사용자 인증 및 정보 교환을 위한 방식
- JWT는 일반적으로 사용자 인증을 위해 사용
- 서버와 클라이언트 간에 사용자 정보를 안전하게 전송하는 데 사용
    - 이를 통해 사용자 인증과 권한 부여를 수행할 수 있음
- JWT는 인코딩된 형태로 정보를 저장하며, 기본적으로는 암호화되지 않음
    - 민감한 정보는 포함하지 않거나, 필요한 경우 암호화된 JWT(JWE)를 사용
  <br>
  <br>

### 👉 JWT의 구성 요소

---

```
Header:
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload:
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}

Signature:
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  your-256-bit-secret
)
```

- **Header**
    - 토큰의 유형(JWT)과 서명 알고리즘(HMAC SHA256, RSA 등)을 포함
    <br>
        
- **Payload**
    - 토큰의 클레임(Claim)으로, 사용자 정보와 같은 데이터를 포함
        - 클레임은 특정 주체에 대한 속성을 기술하는 데이터
            
            ```
            sub : 주체(subject) 식별자
            name : 사용자 이름
            iat : 토큰 발행 시간(Issued At)
            ```
            
    - 이 부분은 base64로 인코딩 됨
    <br>
    
- **Signature**
    - 토큰의 무결성을 보장하기 위해 Header와 Payload를 서명한 것
    <br>
    
- ⇒ 이 세 부분은 점(.)으로 구분되어 하나의 문자열로 결합
    - Header
        
        ```
        {
          "alg": "HS256",
          "typ": "JWT"
        }
        ```
        
        ```
        Base64Url로 인코딩한 결과
        
        eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
        ```
        
    - Payload
        
        ```
        {
          "sub": "1234567890",
          "name": "John Doe",
          "iat": 1516239022
        }
        ```
        
        ```
        Base64Url로 인코딩한 결과
        
        eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
        ```
        
    - Signature
        - Header와 Payload를 `.` 으로 결합
            
            ```
            eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
            ```
            
        - 위 문자열을 비밀키와 함께 HMAC SHA256 알고리즘을 사용하여 서명
        - 비밀키가 your-256-bit-secret이라고 가정하면, 서명 결과는 아래와 같음
            
            ```
            SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
            ```
            
    - 최종적으로 이 세 문자열을 전부 합침
        
        ```
        eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
        ```
        <br>
        <br>
        

### 👉 JWT 사용 이유

---

- **무상태(Stateless) 인증**
    - JWT는 클라이언트 측에서 토큰을 저장하고 서버가 해당 토큰을 검증하여 사용자를 인증
    - 이는 서버가 상태를 저장하지 않아도 되므로, 서버 확장이 용이
    <br>
- **성능 향상**
    - 세션 기반 인증은 서버 메모리에 사용자 정보를 저장하므로, 서버 부하가 증가할 수 있음
    - 반면, JWT는 클라이언트 측에서 토큰을 저장하고 전송하므로 서버 부하를 줄일 수 있음
    <br>
- **다양한 클라이언트 지원**
    - JWT는 쿠키가 아닌 Authorization 헤더를 통해 전달되므로, 웹 뿐만 아니라 모바일 앱, 데스크탑 애플리케이션 등 다양한 클라이언트에서 사용할 수 있음
    <br>
- **보안**
    - JWT는 서명을 통해 무결성을 보장
    - 필요시 암호화를 통해 데이터의 기밀성을 보장할 수 있음
  <br>
  <br>

### ❓ 쿠키와 세션이 있는데 JWT를 사용하는 이유

---

- 쿠키와 세션은 주로 서버 사이드 렌더링(SSR)에서 사용
    - SSR 방식은 서버가 클라이언트의 요청에 따라 HTML을 생성하고, 브라우저에 전달(JSP, Thymeleaf)
    - 이때 사용자 인증은 주로 쿠키와 세션을 통해 이루어딤
    - 쿠키
        - 클라이언트 측에 저장
        - 서버와 클라이언트 간의 HTTP 요청/응답 시 자동으로 전송
    - 세션
        - 서버 측에 저장
        - 세션 ID를 클라이언트의 쿠키에 저장하고, 이 세션 ID를 통해 서버에서 사용자 정보를 참조
        - 서버 메모리에 세션 데이터를 저장하기 때문에 사용자 수가 많아지면 서버 부하가 증가할 수 있음
- 요즘에는 주로 API 서버를 사용
    - API 서버는 RESTful API를 통해 데이터를 주고받는 방식(클라이언트 사이드 랜더링/CSR) 사용
    - 서버와 화면의 분리
    - 이때 상태를 저장하지 않는 무상태(stateless) 방식이 일반적
    - JWT
        - 클라이언트는 로그인 시 JWT를 발급받아 저장
        - 이후 요청 시 이 토큰을 헤더에 포함하여 서버에 전송
        - 서버는 토큰을 검증하여 사용자 정보를 확인
- 정리
    - 쿠키/세션 방식 : 서버가 상태 정보를 저장하고 관리
    - JWT 방식 : 클라이언트가 상태 정보를 포함하는 토큰을 관리하고, 서버는 요청 시 토큰 검증
