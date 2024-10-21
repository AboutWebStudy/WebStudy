### 👉 Docker 데몬(Docker Daemon)이란?

---

- Docker 데몬(dockerd)은 Docker 시스템의 백그라운드에서 실행되며, 모든 Docker 관련 작업을 실제로 처리하는 프로그램
- 데몬은 컨테이너를 시작하고 관리하는 것뿐만 아니라, 이미지 빌드, 네트워크 설정, 볼륨 관리 등의 역할을 수행
<br>
<br>

### 👉 Docker 데몬과 클라이언트의 역할

---

- Docker는 클라이언트-서버 아키텍처로 설계
    - 시스템에 설치된 서비스로, Linux 같은 운영체제에서는 `systemd` 같은 서비스 관리자를 통해 시작 및 관리 됨
    - 데몬이 실행되면, 사용자가 요청할 때마다 필요한 작업을 수행하는 구조
- Docker 클라이언트
    - 사용자가 상호작용하는 도구
    - 사용자가 터미널에서 `docker run` 같은 명령어를 입력하면, 이 클라이언트는 해당 명령어를 해석하고 데몬에 요청을 보냄
- Docker 데몬
    - 요청을 받아 실제 작업을 수행하는 서버
    - 클라이언트가 요청한 작업을 처리하고, 결과를 클라이언트로 반환
    <br>
    <br>

### 👉 Docker 데몬과 클라이언트의 통신 방식

---

- Docker 데몬과 클라이언트는 보통 Unix 소켓이나 TCP 포트를 통해 통신함
    - Unix 소켓 파일이 바로 `/var/run/docker.sock`
    - 주요 내용은 [🔗Docker의 소켓 파일 : 데몬과 클라이언트 통신](https://velog.io/@reasonoflife39/Docker%EC%9D%98-%EC%86%8C%EC%BC%93-%ED%8C%8C%EC%9D%BC-%EB%8D%B0%EB%AA%AC%EA%B3%BC-%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8-%ED%86%B5%EC%8B%A0) 참고
    <br>
    <br>

### 👉 Docker 데몬의 주요 기능

---

- Docker 이미지를 빌드, 가져오기(pull), 저장(push) 등의 작업을 수행
    - 이미지가 컨테이너의 '템플릿' 역할을 하는데, 데몬이 이를 관리
- `docker run` 명령으로 컨테이너를 실행하거나 `docker stop`, `docker rm` 같은 명령으로 컨테이너를 멈추거나 삭제하는 작업을 수행
- 각 컨테이너 간의 네트워크 연결을 관리함
    - 서로 다른 컨테이너가 통신할 수 있도록 브릿지 네트워크를 설정
    - 여러 호스트 간 컨테이너가 통신하도록 오버레이 네트워크를 구성
- 컨테이너의 데이터를 유지하고 관리하는 볼륨을 생성하거나 삭제하는 등의 작업
- Docker 데몬은 각 컨테이너와 시스템의 상태를 기록하고, 모니터링 도구와 통합되어 다양한 시스템 로그를 관리
<br>
<br>

### 👉 Docker 데몬의 REST API를 통한 제어

---

- Docker 데몬은 REST API를 제공
- 클라이언트는 이 API를 통해 Docker 데몬과 소통
- 터미널 명령어 외에도 HTTP를 사용해 원격으로 Docker 데몬을 제어할 수도 있음
<br>
<br>

### 👉 Docker 데몬 실행 및 관리

---

- Docker 데몬은 시스템 부팅 시 자동으로 시작
- 관리자는 주로 `systemctl` 명령어로 데몬을 제어할 수 있음
    
    ```bash
    # Docker 데몬 시작
    sudo systemctl start docker
    
    # Docker 데몬 상태 확인
    sudo systemctl status docker
    ```
