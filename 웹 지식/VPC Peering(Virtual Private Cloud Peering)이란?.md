### 👉 VPC Peering이란?

---

- 클라우드 환경에서 서로 다른 VPC(Virtual Private Cloud) 간의 네트워크 트래픽을 프라이빗하게 연결할 수 있는 네트워크 서비스
    - VPC란, 클라우드에서 사용하는 나만의 가상 네트워크 공간이라고 생각하면 됨
    - 클라우드 위에 있는 내 집 같은 네트워크라고 보면 됨 ⇒ 내가 원하는 대로 방(서브넷)을 나누고, 문(보안 규칙)을 만들 수 있음
    - VPC도 주소 체계(CIDR)를 설정해서 내부에 있는 컴퓨터들이 서로 소통할 수 있음
- 이 서비스를 통해 두 VPC 간에 인터넷 게이트웨이, NAT 게이트웨이, VPN 또는 전용 회선을 거치지 않고, 클라우드 제공자의 내부 네트워크를 통해 안전하게 통신할 수 있음
- AWS, GCP, Azure 등 주요 클라우드 제공자에서 비슷한 기능을 제공함
<br>
<br>

### 👉 VPC Peering의 주요 특징

---

- VPC Peering은 인터넷을 경유하지 않고, 클라우드 제공자의 백본 네트워크를 사용하여 VPC 간에 데이터를 전송
- VPC Peering을 설정하면 두 VPC 간에 상호 통신이 가능하며, 이를 통해 애플리케이션이나 서비스 간의 데이터 교환을 쉽게 할 수 있음
- 서로 다른 VPC에 있는 데이터베이스, 애플리케이션 서버, 파일 스토리지 등 리소스를 공유할 수 있음
- VPC Peering은 고성능의 안전한 통신을 제공하며, 필요에 따라 다양한 VPC 간 연결을 설정할 수 있음
<br>
<br>

### 👉 VPC Peering의 사용 사례

---

- 하나의 VPC는 개발 환경용이고, 다른 VPC는 운영 환경용일 때, 두 환경 간의 통신을 설정하여 개발자들이 운영 데이터에 접근할 수 있게 할 수 있음
- 조직에서 여러 AWS 계정을 사용하며, 각 계정의 VPC를 연결해 중앙화된 리소스를 공유하고 관리할 수 있음
- 서로 다른 리전에 있는 VPC 간 연결이 필요할 때 사용 가능합니다(AWS에서는 이를 리전 간 VPC Peering이라고 함)
<br>
<br>

### 👉 주의 사항

---

- VPC A와 VPC B, VPC B와 VPC C 간에 Peering이 설정되어 있어도, VPC A와 VPC C 간에는 직접 연결되지 않음
    - 필요하다면, A-C 간에도 별도의 Peering을 설정해야 함
- Peering 연결 후에는 양쪽 VPC의 라우팅 테이블에 경로를 추가해야 정상적으로 통신이 가능함
- 두 VPC의 CIDR 블록이 겹치면 VPC Peering을 설정할 수 없음
    - 네트워크를 나누고 주소를 할당할 때 사용하는 표기 방식으로, 네트워크의 주소 범위를 표기(서브넷 마스크)
    - VPC Peering은 두 VPC의 네트워크를 연결하는 것이기 때문에, 서로 다른 VPC의 주소 범위가 겹치면(중복되면) 데이터가 어디로 가야 할지 알 수 없기 때문에 중복 불가
    <br>
    <br>

### 👉 VPC Peering을 쉽게 이해하기

---

- VPC Peering을 쉽게 설명하자면, 두 개의 집(서로 다른 VPC)을 연결하는 전용 다리를 놓는 것이라고 생각하면 됨
- 각각의 집(VPC)은 고유한 주소(CIDR)와 담장이 있어서 자기만의 네트워크 환경을 가지고 있음
- 두 집이 서로 통신할 일이 생겼을 때, 외부 도로(인터넷)를 거치지 않고 바로 연결할 수 있는 다리를 놓는 것이 VPC Peering이라고 생각하면 됨
<br>
- 다리가 필요한 이유
    - 다리를 통해 연결하면 외부 인터넷으로 나가지 않아도 되니까 더 안전함
    - 직접 연결되니 데이터가 빠르게 오갈 수 있음
    - 한쪽 집에 있는 물건(예 : 데이터베이스)을 다른 쪽 집에서 쉽게 가져다 쓸 수 있음
    <br>
- 주의 사항
    - 두 집의 주소(CIDR 블록)가 겹치면 어디로 가야 할지 헷갈려서 다리를 놓을 수 없음
    - 한쪽만 원한다고 다리가 생기지 않아요. 양쪽 집 주인이 모두 동의해야 다리가 연결됨 ⇒ 라우팅 테이블에 경로 추가
    - 집 A와 집 B, 집 B와 집 C가 다리로 연결됐다고 해서 집 A와 집 C가 자동으로 연결되지는 않음
    <br>
    <br>

### 👉 Transit Gateway와의 차이

---

- Transit Gateway는 클라우드 환경에서 여러 VPC(Virtual Private Cloud)와 온프레미스 네트워크를 연결하는 중앙 허브 역할을 하는 네트워크 서비스
    - AWS를 기준으로 설명하면, VPC Peering이 두 VPC 간의 개별 연결을 설정하는 방식이라면, Transit Gateway는 여러 VPC와 네트워크를 중앙에서 통합 관리하고 연결할 수 있는 서비스
    <br>
- Transit Gateway의 주요 특징
    - 여러 VPC와 온프레미스 네트워크를 Transit Gateway를 통해 연결 가능
        - 별도의 Peering 연결 없이 모든 VPC와 네트워크를 허브처럼 연결해 통신 가능
        - ⇒ 중앙 허브 역할을 함
    - 최대 수백 개의 VPC를 연결할 수 있어 대규모 네트워크 설정에 적합
        - VPC Peering은 네트워크가 많아질수록 관리 복잡도가 증가하지만, Transit Gateway는 이를 간소화함
    - 라우팅 정책을 중앙에서 설정 및 관리 가능
        - 특정 VPC 간에만 통신을 허용하거나 제어할 수 있음
    - VPC 간 연결뿐만 아니라, 온프레미스 네트워크, Direct Connect, VPN 등도 연결 가능
        - 서로 다른 리전에 있는 VPC도 연결할 수 있음
        <br>
- 비교
    
    
    | 특징 | Transit Gateway | VPC Peering |
    | --- | --- | --- |
    | 연결 구조 | 중앙 허브를 통해 다수의 VPC와 네트워크 연결 | 두 VPC 간 개별 연결 |
    | 확장성 | 매우 높은 확장성 | VPC 수가 많아질수록 관리 복잡도 증가 |
    | 관리 용이성 | 중앙에서 라우팅 정책 관리 | 각 VPC의 라우팅 테이블을 개별 설정해야 함 |
    | 비용 | 사용량에 따라 비용 발생 (Transit Gateway 데이터 전송 비용) | 상대적으로 저렴한 편 |
    | 리전 간 연결 | 지원 | 제한적 (리전 간 VPC Peering 필요) |
    | 온프레미스 연결 | 지원 (VPN, Direct Connect 통합 가능) | 지원하지 않음 |
    <br>
    <br>

### 👉 AWS에서 VPC Peering 설정하기

---

- 다른 CIDR을 가진 VPC 2개 필요
- VPC Peering 요청 생성
    - AWS Management Console에 로그인
    - VPC 서비스로 이동
    - 왼쪽 메뉴에서 Peering Connections를 선택
    - Create Peering Connection 버튼을 클릭
    - Peering 연결 정보를 입력
        - Requester VPC: 요청을 보낼 VPC 선택
        - Accepter VPC: 연결 대상 VPC 선택
            - 같은 계정 내 VPC 연결: Accepter VPC 선택 가능
            - 다른 계정 연결: 상대 계정의 VPC ID를 입력
    - Create Peering Connection 버튼을 클릭하여 요청 생성
- Peering 요청 승인
    - 요청을 받은 쪽에서 AWS Management Console에 로그인
    - Peering Connections로 이동
    - 요청 목록에서 방금 생성된 Peering 요청을 찾음
    - 요청을 선택한 후, Actions → Accept Request를 클릭하여 승인
- 라우팅 테이블 업데이트
    - Peering 연결이 설정되면, 두 VPC 간 트래픽이 전달되도록 라우팅 테이블을 업데이트해야 함
    - VPC → Route Tables로 이동
    - 각 VPC에 연결된 라우팅 테이블을 선택
    - Routes 탭으로 이동하여 새 경로를 추가
        - Destination : 상대 VPC의 CIDR 블록
        - Target : 생성한 Peering Connection 선택
        - 예시
            - VPC A의 라우팅 테이블 : 192.168.0.0/16 → pcx-xxxxxxxx
            - VPC B의 라우팅 테이블 : 10.0.0.0/16 → pcx-xxxxxxxx
    - 저장
- 보안 그룹 설정
    - 보안 그룹은 기본적으로 트래픽을 차단하므로, 연결을 허용하려면 적절한 규칙을 추가해야 함
    - VPC → Security Groups로 이동
    - 연결된 인스턴스의 보안 그룹을 선택
    - Inbound Rules 및 Outbound Rules를 수정
        - Source : 상대 VPC의 CIDR 블록
        - Protocol/Port : 필요한 서비스 포트를 허용
    - 저장
