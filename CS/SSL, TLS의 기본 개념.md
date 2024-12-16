### SSL(Secure Sockets Layer)이란?

---

- 인터넷 상에서 데이터를 안전하게 전송하기 위해 설계된 보안 프로토콜
- 사용자가 흔히 웹사이트의 URL에서 "https://"를 볼 때, 그 "s"가 SSL(또는 그 후속 버전인 TLS)을 사용하고 있다는 의미
<br>
<br>

### SSL의 목적

---

- 암호화를 통해 데이터를 읽거나 해석할 수 없도록 보호
- 인증을 통해 통신 상대방의 신원을 확인
- 데이터가 전송 중에 변경되지 않았음을 보장하는 데이터 무결성
<br>
<br>

### SSL 인증서

---

- SSL 인증서는 서버의 신원을 증명하기 위해 필요한 디지털 서명 파일
- 공인 인증 기관(CA, Certificate Authority)에서 발급하며, 서버가 진짜임을 보장하고 암호화된 통신을 가능하게 함
<br>
<br>

### SSL 동작 방식

---

![](https://velog.velcdn.com/images/reasonoflife39/post/bd3eb3e5-5ce2-4d75-9974-fe73e1242c65/image.png)
<br>
<br>

### SSL의 후속 기술 : TLS

---

- 현재 SSL은 더 이상 사용되지 않고, 이를 대체하는 TLS(Transport Layer Security) 사용
- TLS는 SSL의 개선된 버전으로, SSL과 TLS는 비슷하게 사용되지만, 최신 보안 기술은 TLS를 기반으로 함
<br>
<br>

### 실생활 예시

---

- 온라인 쇼핑몰에서 신용카드 정보를 입력할 때 HTTPS가 없다면, 해커가 쉽게 정보를 탈취할 수 있음
- 이메일, 은행 웹사이트, 정부 웹사이트 등 모든 중요 서비스는 SSL/TLS를 사용하여 보안을 유지
<br>
<br>

### HTTPS : SSL/TLS로 강화된 HTTP

---

- HTTP는 원래 데이터를 평문(Plain Text)으로 전송하기 때문에 제3자가 내용을 쉽게 엿볼 수 있음
- SSL/TLS를 추가하면, HTTP 통신이 암호화된 채로 이루어지며 HTTPS로 바뀜
- 클라이언트와 서버는 HTTP 요청을 보내기 전에 SSL/TLS 핸드셰이크를 통해 안전한 연결을 설정
    - 핸드셰이크가 성공적으로 끝나면 HTTP 데이터는 암호화된 터널 안에서 전송됨
    - 핸드셰이크는 HTTP와 독립적으로 작동하며, 한번 안전한 터널이 만들어지면 그 위에서 평소처럼 HTTP 요청과 응답이 오감
    - 즉, HTTP가 핸드셰이크를 통해 요청을 시작하지 않음
        - 핸드셰이크는 SSL/TLS의 역할이며, HTTP는 이미 만들어진 안전한 통로 위에서 동작
- HTTPS는 "HTTP over SSL/TLS"를 의미
    - HTTP 프로토콜 위에 SSL/TLS를 추가하여 보안을 강화한 것
    - "HTTP"가 열린 대문이라면, "HTTPS"는 자물쇠가 걸린 대문
