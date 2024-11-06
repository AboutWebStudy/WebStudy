### 👉 Docker의 브릿지 네트워크란?

---

- Docker 컨테이너들 간의 기본 네트워킹 모드 중 하나로, 기본적으로 Docker 데몬이 컨테이너를 시작할 때 자동으로 할당
- 브리지 네트워크를 통해 동일한 호스트 내에 있는 Docker 컨테이너들이 서로 통신할 수 있으며, 네트워크 격리를 제공하여 외부 네트워크와의 연결을 통제할 수 있음
<br>
<br>

### 👉 Docker 브릿지 네트워크의 주요 특징

---

- Docker에서 bridge 네트워크는 Docker가 설치될 때 자동으로 생성되는 기본 네트워크
    - 사용자가 별도로 설정하지 않는 한 컨테이너는 기본적으로 브리지 네트워크에 연결됨
- 동일한 브리지 네트워크에 연결된 컨테이너들은 IP 주소 또는 컨테이너 이름을 통해 서로 통신할 수 있음
    - 이는 단일 호스트 내부에서 컨테이너 간의 통신을 용이하게 함
- 브리지 네트워크에 연결된 컨테이너들은 외부 네트워크로부터 격리됨
    - 외부에 노출하려면 포트를 노출하거나 -p 옵션을 사용해 호스트의 포트를 매핑해야 함
    - 이를 통해 네트워크 보안을 강화할 수 있음
- 브리지 네트워크는 주로 로컬 개발 환경에서 여러 개의 컨테이너를 실행하고 서로 통신하게 할 때 사용
- 필요에 따라 사용자가 `docker network create --driver bridge <네트워크 이름>` 명령어로 새 브리지 네트워크를 생성할 수도 있음
    - 커스텀 브리지 네트워크는 기본 브리지보다 향상된 DNS 지원을 제공하므로 컨테이너들이 서로 이름으로 쉽게 접근할 수 있음
    <br>
    <br>

### 👉 기본 명령어

---

- 네트워크 목록 보기 : `docker network ls`
- 커스텀 브릿지 네트워크 생성 : `docker network create --driver bridge my-bridge`
- 네트워크에 컨테이너 연결 : `docker run --network my-bridge --name my-container my-image`
<br>
<br>

### 👉 Docker 브릿지 네트워크 사용 예시

---

- **아래 두 개의 Docker 컨테이너가 있다고 가정**
    - 웹 서버 컨테이너
    - 데이터베이스 컨테이너
    <br>
- **브릿지 네트워크 만들기**
    
    ```docker
    docker network create --driver bridge my-bridge
    ```
    
    - `my-bridge`라는 새로운 가상의 네트워크를 만듦
    <br>
- **컨테이너를 브릿지 네트워크에 연결하기**
    
    ```docker
    # 웹 서버 컨테이너 실행
    docker run -d --name web-server --network my-bridge nginx
    
    # 데이터베이스 컨테이너 실행
    docker run -d --name database --network my-bridge mysql
    ```
    
    - 두 컨테이너는 같은 my-bridge 네트워크에 연결되어 서로 통신할 수 있게 됨
    <br>
- **컨테이너끼리 이름으로 통신하기**
    
    ```docker
    # 웹 서버에서 데이터베이스에 접근할 때
    mysql -h database -u user -p
    ```
    
    - 브리지 네트워크에 연결된 컨테이너들은 서로 IP 주소를 몰라도, 이름으로 바로 찾을 수 있음
    - 위 명령어는 `web-server` 컨테이너가 데이터베이스에 접근할 때 IP 주소 대신 `database`라는 이름으로 접근하는 예시
