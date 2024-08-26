### 👉 가상 호스팅이란?

---

- 하나의 물리적 서버에서 여러 개의 웹사이트를 호스팅하는 기술
- 가상 호스팅을 사용하면 각 웹사이트가 독립적으로 운영되는 것처럼 보이지만 실제로는 같은 서버 자원을 공유함
- 서버 자원을 효율적으로 사용하고, 비용을 절감하며, 관리의 편의성을 높인다는 장점이 있음
<br>
<br>

### 👉 가상 호스팅의 종류 : 서버 자원의 사용과 비용에 따른 분류

---

- **공유 호스팅(Shared Hosting)**
    - 여러 웹사이트가 하나의 물리적 서버와 운영 체제를 공유하는 형태
    - 각 웹사이트는 별도 디렉토리로 분리되지만, 같은 서버 자원을 사용
        - ⇒ 한 아파트 건물에 여러 가정이 사는 것과 비슷. 각 가정이 따로 있지만 모두 같은 전기, 수도 사용
        - ⇒ 서버 안에서 각 사용자에게 별도의 공간(디렉토리)을 할당하는 방식
    - 공유 호스팅 같은 경우는 개인 블로그, 포트폴리오 웹 사이트 등에 사용
    - 가격이 저렴하고 설정이 간단하여, 소규모 웹사이트나 개인 블로그 운영에 적합
    - 같은 서버를 쓰는 다른 웹사이트가 트래픽이 많아지면 내 웹사이트 속도가 느려질 수 있다는 단점이 있음
    <br>
- **가상 사설 서버(VPS, Virtual Private Server) 호스팅**
    - 하나의 물리적 서버를 여러 개의 가상 서버로 분할하여, 각 가상 서버가 독립된 환경을 가진 것처럼 운영
    - 각 가상 서버(VPS)는 독립적인 운영 체제와 자원을 가지고 있어서, 다른 가상 서버의 영향을 덜 받음
    - 가상 사설 서버(VPS)를 제공하는 업체는 AWS, Google Cloud 등이 있음
    - 가상 사설 서버(VPS)는 [하이퍼바이저(Hypervisor)라는 소프트웨어를 사용](https://velog.io/@reasonoflife39/%EC%84%9C%EB%B2%84-%EA%B0%80%EC%83%81%ED%99%94-%EC%9C%A0%ED%98%95)하여 각 가상 서버가 독립된 운영 체제(OS)를 실행할 수 있도록 함
    - 공유 호스팅보다 가격이 높지만, 더 나은 성능과 보안, 높은 설정 자유도를 제공하기 때문에 중소 규모의 웹사이트에 적합
    <br>
    <br>

### 👉 가상 호스팅의 기술적 방식

---

- **이름 기반(Name-Based) 가상 호스팅**
    - 이름 기반 가상 호스팅은 하나의 IP 주소를 사용하여 여러 개의 도메인 이름을 호스팅하는 방식
    - 클라이언트가 웹 서버에 요청을 보낼 때, 서버는 요청 헤더의 호스트 이름(Host Header)을 기반으로 어떤 도메인을 요청했는지 식별하고, 해당 도메인에 맞는 웹 콘텐츠를 반환
    - 하나의 IP 주소로 여러 도메인을 호스팅할 수 있기 때문에 IP 주소의 절약이 가능
    - 설정이 간단하고, 대부분의 현대적인 웹 서버에서 지원
    - 모든 클라이언트가 HTTP/1.1 이상을 지원해야 사용 가능 (HTTP/1.1부터 Host Header를 지원)
    - 동일한 IP 주소와 포트를 사용하는 모든 도메인에 대해 개별 SSL/TLS 인증서를 사용할 수 없기 때문에 SAN(SUBJECT ALTERNATIVE NAME) 인증서나 와일드카드 인증서를 사용해야 함
        - ⇒ SAN 인증서나 와일드카드 인증서를 사용하면 여러 도메인에 대한 SSL/TLS 보안을 한 번에 처리할 수 있으며, 이는 관리의 복잡성을 줄여 줌
        <br>
- **IP 기반(IP-Based) 가상 호스팅**
    - IP 기반 가상 호스팅은 각 도메인마다 고유한 IP 주소를 할당하여 웹사이트를 호스팅하는 방식
    - 웹 서버는 클라이언트가 접속한 IP 주소를 기반으로 어떤 도메인을 요청했는지 식별
    - SSL/TLS를 사용하여 각 도메인에 대해 개별 인증서를 사용할 수 있음
    - 클라이언트가 HTTP/1.0만 지원해도 작동할 수 있음(Host Header를 사용하지 않기 때문)
    - 각 도메인마다 고유한 IP 주소가 필요하므로 IP 주소가 부족할 수 있고, IP 주소의 추가로 인해 비용이 증가할 수 있으며, 설정이 복잡하고, 더 많은 네트워크 자원이 필요
    <br>
- **⇒ 각 방식을 사용하는 경우**
    - 이름 기반 가상 호스팅은 대부분의 웹 호스팅 환경에서 일반적으로 사용
    - IP 기반 가상 호스팅은 SSL/TLS 인증서를 각각의 도메인에 대해 개별적으로 사용해야 하는 경우, 혹은 네트워크 인프라상 특정 IP 주소가 요구되는 경우에 사용
    <br>
    <br>

### 👉 가상 호스팅의 기술적 방식 적용 방법

---

- **이름 기반(Name-Based) 가상 호스팅**
    - Apache 웹 서버 설치 후, 각 가상 호스트의 설정 파일 생성(example1.com과 example2.com)
        
        ```bash
        sudo nano /etc/apache2/sites-available/example1.com.conf
        ```
        
        ```bash
        <VirtualHost *:80> // 같은 IP라 이 부분은 동일하게 사용할 수 있음
            ServerAdmin webmaster@example1.com
            ServerName example1.com
            ServerAlias www.example1.com
            DocumentRoot /var/www/example1.com
            ErrorLog ${APACHE_LOG_DIR}/example1.com-error.log
            CustomLog ${APACHE_LOG_DIR}/example1.com-access.log combined
        </VirtualHost>
        ```
        
    - 각 도메인에 대한 웹 파일을 저장할 디렉토리를 생성하고, 가상 호스트 활성화
        
        ```bash
        sudo mkdir -p /var/www/example1.com
        sudo mkdir -p /var/www/example2.com
        ```
        
        ```bash
        sudo a2ensite example1.com.conf
        sudo a2ensite example2.com.conf
        ```
        <br>
        
        
- **IP 기반(IP-Based) 가상 호스팅**
    - 여러 개의 IP 주소를 준비 해야 함
        - 서버의 네트워크 인터페이스 설정 수정 필요
    - Apache 웹 서버 설치 후, 각 가상 호스트의 설정 파일 생성(example1.com과 example2.com)
        
        ```bash
        sudo nano /etc/apache2/sites-available/example1.com.conf
        ```
        
        ```bash
        <VirtualHost 192.168.1.100:80> // IP 주소에 맞게 수정
            ServerAdmin webmaster@example1.com
            ServerName example1.com
            DocumentRoot /var/www/example1.com
            ErrorLog ${APACHE_LOG_DIR}/example1.com-error.log
            CustomLog ${APACHE_LOG_DIR}/example1.com-access.log combined
        </VirtualHost>
        ```
        
- 각 도메인에 대한 웹 파일을 저장할 디렉토리를 생성하고, 가상 호스트 활성화
    
    ```bash
    sudo mkdir -p /var/www/example1.com
    sudo mkdir -p /var/www/example2.com
    ```
    
    ```bash
    sudo a2ensite example1.com.conf
    sudo a2ensite example2.com.conf
    ```
    <br>
    <br>
    

### 👉 가상 호스팅의 장점

---

- 여러 웹사이트가 하나의 서버를 공유하기 때문에 서버 운영 비용을 절감할 수 있음
- 서버 자원을 여러 웹사이트가 효율적으로 사용할 수 있어 낭비를 줄임
- 여러 웹사이트를 한 서버에서 관리할 수 있어 관리가 용이
<br>
<br>

### 👉 가상 호스팅의 단점

---

- 같은 서버 자원을 공유하기 때문에 다른 웹사이트의 활동에 의해 성능이 영향을 받을 수 있음
- 공유 호스팅은 같은 서버에서 여러 웹사이트가 운영되므로, 한 웹사이트의 보안 문제가 다른 웹사이트에 영향을 미칠 수 있음. 또, 서버 설정에 대한 제어 권한이 제한적일 수 있음
