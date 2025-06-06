## **단일 장애 지점(Single Point Of Failure)**

SPOF 문제는 개발 인프라를 구성할 때 빼 놓을 수 없는 고려점이다.

**SPOF(Single Point of Failure)** 는 단일 구성 요소에 장애가 발생했을 때, 전체 시스템이 영향을 받아 서비스를 이용할 수 없게 되는 상황을 의미한다
예를 들어, 클라이언트가 요청을 보내는 서버가 하나뿐이라면, 해당 서버에 장애가 발생할 경우 서비스를 이용할 수 없게 된다.

다음은 웹 요청의 일반적인 흐름을 SPOF 관점에서 단순화한 예시이다.
<img src="https://github.com/user-attachments/assets/b9f0c08d-5bd8-4697-80b0-55773129be69" width="600"/>


모든 인프라에서 SPOF를 고려하기 위해 위와 같은 예시를 들었으며, 각 계층별로 SPOF는 반드시 고려해봐야 할 문제이다.

클라이언트가 서버에 요청을 진행하면, L4 로드밸런서가 요청을 받아 각 Proxy Server에 데이터를 넘기며, Proxy 서버는 요청을 내부의 WAS로 전달한다. WAS에서는 Database에 있는 값을 확인해 생성된 데이터를 Client에게 반환합니다.

### L4 Load Balancer

L4 로드밸런서는 TCP/UDP 레벨에서 요청을 분산 처리하는 역할을 한다.

해당 계층에서 SPOF를 방지하기 위한 방법은 다음과 같다.

- **이중화 구성 (Active-Active 또는 Active-Passive)**
- **VRRP(Virtual Router Redundancy Protocol)** 를 통한 고가용성 구성
- **클라우드 환경에서는 관리형 L4 (예: AWS NLB, Azure Load Balancer)** 를 활용해 내결함성 확보

**SPOF 예방 방안**

- L4 LB를 두 대 이상 배치
- 헬스 체크 기반의 자동 장애 전환(failover) 구성

### Proxy Server

Proxy 서버는 클라이언트 요청을 받아 내부 WAS로 전달하거나 캐싱, 인증, 보안 처리 등의 역할을 한다.

프록시가 하나일 경우 SPOF가 발생하므로 **수평 확장 및 L4 앞단 분산**이 필수다.

**SPOF 예방 방안**

- 여러 대의 프록시 서버를 L4 로드밸런서를 통해 분산
- Stateless 설계로 프록시 간 장애 영향을 최소화
- Proxy 레벨에서의 Failover 및 헬스 체크 기능 사용

### WAS

WAS는 실제 비즈니스 로직이 실행되는 계층이다. 하나의 WAS 인스턴스에 모든 처리를 맡길 경우, 서비스 전체 장애가 발생할 수 있다.

**SPOF 예방 방안**

- WAS 인스턴스를 **다중 구성(Auto Scaling 포함)**
- Proxy 또는 L7 Load Balancer에서 **로드밸런싱**
- **세션 클러스터링 또는 세션 스토리지 분리(Redis 등)** 를 통해 무상태 아키텍처 구현

### Database

데이터베이스는 서비스의 핵심 데이터를 저장하므로, 장애 시 영향도가 가장 크다.

DB는 전통적으로 SPOF가 발생하기 쉬운 계층이므로, 고가용성 구성이 필수이다.

**SPOF 예방 방안**

- **Master-Slave 또는 Master-Master 복제 구성**
- **리더-팔로워 구조 + 자동 Failover (예: RDS Multi-AZ, MySQL with MHA)**
- **샤딩 또는 분산 DB** 도입 고려
- 정기 백업 및 장애 복구 절차 수립

SPOF는 모든 인프라 구성에서 반드시 고려해야 할 요소이며, 계층별로 고가용성을 확보하는 구조 설계가 안정적인 시스템 운영의 핵심이다.
