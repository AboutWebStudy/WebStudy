## CGI(Common Gateway Interface)

**서버 사이드에서 사용자의 프로그램을 동작하도록 하기 위한 프로토콜**

CGI는 웹 서버에서 동적인 클라이언트의 요구에 응답하기 위해 만들어졌다.

### CGI의 역할 및 흐름

- 보통의 경우 클라이언트가 HTTP요청을 했을 때 웹 서버에서는 요청이 백엔드 서버까지 들어가서 프로그램을 실행해야 하는지, 정적인 콘텐츠(html, css, js, image 등)만을 Response할지를 정함
- 로직 처리 프로그램에 넘겨야 한다면 서버에서 CGI프로토콜로 HTTP요청을 변환하여 백엔드 서버로 넘김
- 백엔드 서버에서 요청을 처리 후 CGI프로토콜로 서버에 Response 반환
- 상당히 많은 언어가 이러한 프로토콜(FastCGI API)을 지원 ( 파이썬, PHP, JAVA, Node.js 등 )

## FastCGI

**기존 CGI의 변형된 버전으로 주요 목적은 웹 서버와 CGI 프로그램 간 통신 시 발생되는 부하를 줄이며, 더 많은 요청을 관리할 수 있게 하는 것이다.**

### FastCGI 특징

- 여러 웹 서버들이 FastCGI를 구현하며, 다양한 언어로 만들어진 프로그램을 api호출할 수 있다
- Nginx같은 웹 서버들은 이 FastCGI통신을 하기 위해 게이트웨이 역할(프로토콜 변환)을 포함한다.

## WebServer와의 상관관계

아래의 자료와 함께 Nginx Web Server를 예시로 설명을 하자면

![image](https://github.com/user-attachments/assets/554d704d-cca9-4b4c-89b5-33bcb0d5c298)


1. 클라이언트가 웹 서버에 보내는 HTTP 또는 HTTPS 요청
2. Nginx에서 HTTPS요청이였다면 HTTP로 변환 후 FastCGI프로토콜로 다시 담아 FastCGI API를 지원하는 언어에 요청
3. 애플리케이션 프로그램은 요청된 로직을 처리 후 FastCGI 프로토콜로 Nginx측에 반환
4. Nginx는 응답받은 FastCGI 데이터를 HTTPS였다면 HTTPS로, HTTP였다면 HTTP로 반환
5. 통신 종료

## 정리

- 위에서 Nginx는 WS의 정적 페이지 반환, 게이트웨이 역할, FastCGI API와 연계하며 WAS의 역할을 모두 수행
- CGI, FastCGI는 웹 서버에서 사용 언어의 제약없이 통신에 활용할 수 있는 프로토콜
- 현 시점에서는 CGI가 아닌 FastCGI로 동적 데이터 처리를 주로 하고있음

출처 : https://ko.wikipedia.org/wiki/FastCGI
