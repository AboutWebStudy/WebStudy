### 👉 오류 내용

---

- Jenkins에서 Spring Boot 프로젝트를 배포하는 파이프라인을 실행하던 중, 다음과 같은 오류가 발생
    
    ```java
    java.io.IOException: error=0, Failed to exec spawn helper: pid: 3762312, exit value: 1
    
    (중략)
    
    Also:   org.jenkinsci.plugins.workflow.actions.ErrorAction$ErrorId: 642da921-c992-4410-9dc4-2b47a72bb197
    Caused: java.io.IOException: Cannot run program "nohup" (in directory "/var/lib/jenkins/workspace/example"): error=0, Failed to exec spawn helper: pid: 3762312, exit value: 1
    ```
    
    - 이 오류는 sh 'mvn -version' 또는 sh 'mvn package' 등의 스크립트 실행 단계에서 발생
    - 이후 단계인 git clone, build, deploy 등이 전부 "skipped due to earlier failure(s)" 상태로 넘어가 파이프라인 전체가 실패 됨
    <br>
    <br>

### 👉 원인 분석

---

- Jenkins가 파이프라인 내 sh 명령어를 실행할 때 내부적으로 nohup을 호출하려다 실패할 때 발생
- Jenkins는 sh 명령을 실행할 때 `durable-task 플러그인`을 통해 백그라운드 프로세스를 생성하며, 이 과정에서 nohup을 사용하는데, `spawn helper` 오류는 리눅스에서 프로세스를 생성(fork)할 수 없는 환경일 때 나타남
    - durable-task 플러그인이란?
        - Jenkins에서 sh, bat, powershell 등 외부 명령어를 안정적으로 실행하기 위해 사용하는 핵심 플러그인
        - Jenkins 파이프라인의 sh, bat 명령어를 실행할 때, 실행 도중 Jenkins가 잠시 중단되거나 끊겨도, 명령어 실행을 지속할 수 있도록 관리
        - 명령어가 백그라운드에서 실행되도록 cookie 파일, nohup, watchdog 등을 이용해서 프로세스를 추적
        - 보통 sh 'mvn package' 같은 명령어가 실행되면 Jenkins가 해당 명령을 임시 스크립트로 저장
            - → durable-task 플러그인이 그 스크립트를 nohup bash script.sh & 형태로 백그라운드에서 실행
            <br>
            <br>

### 👉 시도한 방법

---

- **`nohup` 명령어 존재 여부 확인**
    
    ```bash
    which nohup
    ls -l /usr/bin/nohup
    ```
    
    → `/usr/bin/nohup` 존재했고, `-rwxr-xr-x` 실행 권한도 정상
    <br>
    
- **`jenkins` 사용자로 직접 `nohup` 테스트**
    
    ```bash
    sudo su - jenkins
    nohup bash -c "echo hello" &
    ```
    
    → 백그라운드 실행도 정상, `nohup.out` 파일도 생성됨
    <br>
    
- **시스템 자원 제한 확인 (`ulimit`)**
    
    ```bash
    cat /proc/$(pgrep -f jenkins)/limits
    ```
    
    → `Max processes`, `Max open files` 모두 충분히 높게 설정됨(문제 없음)
    <br>
    <br>
    

### 👉 해결 방법

---

- nohup 자체는 문제 없어서, Jenkins가 외부 프로세스를 생성할 때 사용하는 Java의 fork 메커니즘을 변경하여 해결
<br>

- **1️⃣ `/etc/default/jenkins` 파일 수정**
    
    ```java
    sudo vi /etc/default/jenkins
    ```
    <br>
    
- **2️⃣ 파일의 맨 아래 아래 내용을 추가**
    
    ```java
    JAVA_ARGS="-Djava.awt.headless=true -Djdk.lang.Process.launchMechanism=vfork"
    ```
    
    - `Djava.awt.headless=true` : GUI 없는 서버 환경에서 기본 설정
        - Java에는 GUI를 다루는 `java.awt`라는 라이브러리가 있는데, 보통 마우스, 키보드, 모니터 같은 디스플레이 장치가 존재하는 환경을 전제로 동작
        - 서버나 Jenkins 같은 환경은 **디스플레이 없는(headless) 환경**이라 만약 GUI 기능을 사용하는 라이브러리가 존재한다면, GUI가 없는데도 이를 사용하려고 해서 오류가 날 수 있음
        - Jenkins는 GUI 없이 CLI와 웹에서 동작하기 때문에 기본적으로 항상 이 설정을 켜는 게 안정적
    - `Djdk.lang.Process.launchMechanism=vfork` : Java가 프로세스를 만들 때 `fork()` 대신 **`vfork()`** 사용 (리눅스에서 spawn 오류 방지)
        - Unix/Linux 시스템에서 새로운 프로세스를 만들 때 사용하는 기본 메커니즘이 fork()
            - `fork()`는 현재 프로세스(부모)의 복사본을 만드는 시스템 호출 → 복사한 후, 거기서 `exec()`를 호출해 새로운 프로그램을 실행
            - `fork()`는 부모 프로세스를 메모리까지 복사하기 때문에 메모리 소모가 큼
            - Jenkins처럼 메모리를 많이 쓰는 자바 애플리케이션에서 `fork()`로 sh, nohup, mvn 같은 외부 명령을 실행하는 경우에는 시스템 자원이 부족하거나 `spawn helper` 실패 같은 문제가 발생할 수 있음
        - `vfork()`는 `fork()`보다 가볍고 빠른 대안
            - 부모 프로세스를 복사하지 않고, 자식 프로세스가 exec() 호출할 때까지 메모리를 공유하면서 기다림
            - Java는 `vfork()`를 사용할 수 있도록 선택지를 열어두고 있는데, 기본적으로는 `fork()`를 사용
            <br>
- **3️⃣ Jenkins 재시작**
    
    ```java
    sudo systemctl restart jenkins
    ```
    
    - ⇒ 동일한 파이프라인을 다시 실행했을 때, sh, git, mvn package, deploy 전 단계가 모두 정상 수행됨
    - ⇒ nohup 관련 오류가 더 이상 발생하지 않음
