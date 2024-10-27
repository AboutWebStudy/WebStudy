### 👉 소켓(Socket) 파일의 개념

---

- 소켓 파일은 컴퓨터 내에서 프로세스 간 통신을 위한 인터페이스를 제공하는 파일 형태의 통신 채널
- 주로 리눅스 환경에서 `*.sock` 확장자로 나타나며, 로컬에서 프로세스 간 네트워크 통신 없이도 데이터를 주고받을 수 있도록 함
<br>
<br>

### 👉 Docker에서 소켓 파일의 역할

---

- Docker에서 소켓 파일은 Docker 데몬(Docker의 백그라운드 프로세스로 모든 Docker 관련 작업 처리)과 클라이언트(명령어 도구 또는 API) 간의 통신을 가능하게 해주는 파일
- Docker 명령어를 실행하거나 API 호출을 통해 컨테이너를 관리할 때 사용
- 기본적으로 Docker는 Unix 소켓 파일을 사용하여 데몬(dockerd)과 클라이언트(docker 명령어 등) 간의 통신을 처리
    - Unix 소켓 파일은 네트워크 프로토콜 없이 로컬 시스템에서 통신을 가능하게 함 → 네트워크를 거치지 않아 성능이 더 좋음
- 주요 역할
    - 사용자가 입력한 명령어를 클라이언트에서 Docker 데몬으로 전달
    - 데몬이 전달 받은 명령어를 수행
    - 데몬이 수행한 결과를 다시 클라이언트로 반환
- 예시
    - 파일 이름이 `docker.sock`이라면, 이는 Docker 데몬과 클라이언트 간의 통신을 담당하는 소켓 파일
    - Docker 명령어 (`docker ps`, `docker run` 등)를 입력할 때, 이 명령어들이 `docker.sock` 파일을 통해 Docker 데몬에 전달 → 데몬이 해당 명령을 수행한 뒤 결과를 다시 클라이언트로 전달하는 구조
    <br>
    <br>

### 👉 Docker 소켓 파일 사용 예시

---

- Docker CLI
    
    ```docker
    sudo docker run hello-world
    ```
    
    - Docker는 내부적으로 /var/run/docker.sock을 통해 데몬에 hello-world 이미지를 실행하라는 요청을 보냄
    - 데몬이 이 요청을 수행하고 결과를 다시 반환
    <br>
- Docker API(파이썬 코드)
    
    ```python
    import docker
    client = docker.DockerClient(base_url='unix://var/run/docker.sock')
    container = client.containers.run("hello-world", detach=True)
    print(container.logs())
    ```
    
    - `base_url='unix://var/run/docker.sock'` 부분이 Docker 소켓 파일을 통해 데몬에 접근하겠다는 의미
  <br>
  <br>

### 👉 Docker에서 소켓 파일 마운트의 의미

---

- Docker를 사용할 때 소켓 파일을 컨테이너에 마운트하는 경우가 있음
- 이를 통해 컨테이너 내부에서 호스트의 Docker 데몬에 접근할 수 있게 되는데, 이렇게 하면 컨테이너가 Docker 데몬과 직접 통신할 수 있게 됨
    - 컨테이너 내부에서 또 다른 컨테이너를 생성하거나 삭제할 수 있는 권한이 생기게 되는 것
- 예시
    
    ```docker
    docker run -v /var/run/docker.sock:/var/run/docker.sock -it my-container
    ```
    
    - 호스트의 `/var/run/docker.sock` 파일을 컨테이너 내부의 `/var/run/docker.sock` 경로에 마운트
    - 이렇게 하면 my-container라는 컨테이너 내부에서 Docker CLI를 통해 호스트의 Docker 데몬에 접근할 수 있음
    <br>
    <br>

### 👉 Docker에서 소켓 파일이 없다면?

---

- `/var/run/docker.sock` 파일이 없거나 접근이 불가능한 경우에는 Docker 명령어를 실행할 때 `Cannot connect to the Docker daemon`이라는 오류 발생
<br>
<br>

### 👉 Docker에서 소켓 파일을 통한 통신 과정

---

1. Docker 데몬(dockerd)이 실행되면 /var/run/docker.sock 파일이 생성
    - 소켓 파일은 Unix 소켓을 통해 클라이언트와 데몬 간 통신을 담당하는 일종의 엔드포인트 역할
2. 사용자가 docker 명령어를 입력하면, Docker 클라이언트는 이 명령을 HTTP 요청으로 변환하여 소켓 파일을 통해 데몬에게 전달
    - Docker는 HTTP 기반 REST API를 사용하여 명령을 처리하며, 이 요청이 TCP 대신 로컬 Unix 소켓 파일을 통해 전달
        - HTTP 통신 방식은 클라이언트가 HTTP 요청을 보내고, 서버가 HTTP 응답을 반환하는 패턴
        - 여기서 `HTTP 기반`이라는 것은 명령어가 HTTP 프로토콜의 형식을 따르는 것을 의미하며, 실제 네트워크를 통한 HTTP 통신(TCP/IP)을 사용하지 않음
        - Unix 소켓은 네트워크 없이 로컬에서 클라이언트와 데몬 간에 데이터 전송을 처리함
    - 예시
        - `docker ps`라는 명령어 실행 시, 클라이언트가 /containers/json이라는 HTTP 요청을 보내는 것으로 변환
            
            ```docker
            GET /containers/json HTTP/1.1
            Host: /var/run/docker.sock
            ```
            
        - 이 요청은 /var/run/docker.sock 파일을 통해 데몬에게 전달
3. Docker 데몬은 클라이언트로부터 받은 HTTP 요청을 처리
4. 데몬이 요청을 처리한 후, 그 결과는 다시 소켓 파일을 통해 클라이언트에게 전달
    - 예를 들어, `docker ps` 명령의 결과로는 실행 중인 컨테이너 목록이 JSON 형식으로 반환
        
        ```json
        HTTP/1.1 200 OK
        Content-Type: application/json
        
        [
          {
            "Id": "8dfafdbc3a40",
            "Names": ["/cool_name"],
            "Image": "redis",
            "State": "running"
          }
        ]
        ```
        
    - 클라이언트는 이 결과를 받아 사용자가 보기 쉽게 출력
