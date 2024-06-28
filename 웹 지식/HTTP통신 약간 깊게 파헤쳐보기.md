우리가 알고있는 HTTP통신은 OSI 7계층의 응용계층에 속한다. 이 통신을 하기 위해 어떤 동작이 이루어지는지 키워드를 갖고 간단하게 다뤄보자

# TCP(Transmission Controll Protocol)

TCP는 클라이언트와 서버간의 데이터 통신을 위해 사용하는 4계층에 속하는 프로토콜이다.
HTTP는 기본적으로 통신을 하기 위해 TCP를 활용해 **3-way-handshake** 과정을 거쳐 TCP 소켓 연결을 하고 데이터를 전송한다.이후 소켓 연결을 해제할 때는 **4-way-handshake** 과정을 거쳐 연결을 해제한다.

## 3-way-handshake

TCP통신은 기본적으로 3방향 핸드셰이크라는 방식으로 네트워크 소켓연결을 시도하는데
SYN → SYN-ACK → ACK 의 순서로 진행된다.

1. **SYN**(SYNchronize) : 클라이언트가 TCP SYN FLAG패킷을 서버로 송신
2. **SYN-ACK**(Synchronize-ACKnowledgement) : 서버는 SYN을 수신해 클라이언트에게 SYN-ACK를 송신
3. **ACK**(ACKnowledge) : 클라이언트에서 SYN-ACK를 수신하면 ACK를 서버에 송신

서버는 위의 과정을 거친 최종ACK를 수신받았을 때 TCP 소켓연결이 설정된다.

(그림첨부예정)

## 4-way-handshake

소켓연결 후 소켓을 끊을 때는 4-way-handshake 과정을 진행한다. 여기서 사용하는 주요 FLAG는 FIN, ACK이 있다.

해당 연결해제 절차는 클라이언트 측과 서버 측 모두 사용 가능하기에 A, B로 설명하자면,

1. A(FIN) → B
2. B(ACK) → A
3. B(FIN) → A
4. A(ACK) → B

위의 순서를 통해 A와 B의 소켓이 해제되는데 최초 **(1)**FIN요청을 받은 B는 A에게 **(2)**ACK신호를 줌과 동시에 연결을 끊는 작업을 시행하고 연결을 종료할 수 있는 준비작업이 완료되면 B가 A에게 최종종료되었다는 **(3)**FIN FLAG를 넘겨준다. 이를 받은 A는 최종확인 **(4)**ACK FLAG를 B에게 보냄으로써 소켓은 서로 닫히게 된다.

(그림첨부예정)

# HTTP 통신

HTTP통신으로 호출하기 위한 흐름은 기본적으로 TCP 소켓 연결을 기본으로 소켓연결이 된 후 Request & Response작업이 진행된다.

때문에 순서를 다시한번 적자면 이렇게 된다

3-way-handshake → Request → Response → 4-way-handshake의 순서로 통신 흐름을 이해할 수 있다.

(그림첨부예정)

# HTTPS 통신

우리가 평소에 사용하는 사이트들은 모두 HTTP가 아닌 HTTPS를 사용하고있다.
HTTPS는 Http에 S(Secure)가 붙어서 보안을 강화시킨 통신방식인데, 이 통신을 시행하면 중간에 데이터를 가로채가도 데이터는 암호화되어있기에 어떤 데이터를 가로챈건지 알 수 없게 되는 보안프로토콜이다.

이 통신방식은  Secure Sockets Layer(SSL)/Transport Layer Security (TLS) 등으로 불리우는 암호화 프로토콜통신(**SSL Handshake, TLS Negotiation**)을 HTTP통신의 데이터 교환에 앞서서 시행한다.

## SSL Handshake, TLS Negotiation

이 통신을 SSL Handshake/TLS Negotiation이라고도 불리며, 기존의 소켓연결인 3-way-handshake 직후에 시행하는 통신작업인데, 순서는 다음과 같다.

1. Client → ClientHello → Server
2. Server → ServerHello → Client
3. Client → PremasterSecret → Server
4. Server → PremasterSecret(decrypt) → Client
5. HTTPS 통신가능

위의 과정을 거쳐 진행되고 전체 진행순서를 총 정리하면

3-way-handshake → SSL Handshake/TLS Negotiation(소켓 연결 완료) → Request → Response → 4-way-handshake(연결 해제)

의 순서가 된다.

(그림첨부예정)

출처 : https://sh-safer.tistory.com/146

https://velog.io/@dyunge_100/Network-TCPIP-4계층에-대하여

https://velog.io/@tess/아니-왜-http를-써요-tcp를-쓰지

https://brunch.co.kr/@sangjinkang/38

https://sooolog.dev/HTTP-통신과-TCP-통신-그리고-웹-소켓에-대한-기본-개념-정리/
