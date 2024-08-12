### 👉 Java 파일 빌드와 배포의 기본 흐름

---

1. 소스 코드(`.java` 파일)를 컴파일
2. jar 파일 생성
    - ⇒ jar 파일은 모든 의존성을 포함, 실행할 수 있는 파일
3. jar 파일 실행
<br>
<br>

### 👉 빌드 도구 없이 Java 파일 직접 빌드 및 배포 방법

---

1. **소스코드 컴파일**
    - **소스 코드 컴파일(의존성이 없는 경우)**
        - .java 파일 컴파일
            - ⇒ 컴파일러가 바이트코드라는 중간 코드로 변환하며, `.class` 파일로 저장
                - 후에 바이트코드는 JVM에서 실행되고, JVM은 바이트코드를 읽고 해석하여 실제 기계어로 변환 → CPU가 해당 코드 실행
        - src/main/java 디렉토리에 Java 파일이 있다면 아래와 같은 명령어로 컴파일 가능
            
            ```bash
            javac -d out src/main/java/**/*.java
            ```
            
            - `-d out` : 컴파일된 클래스 파일을 저장할 디렉토리를 지정 (out 폴더)
            - `src/main/java/**/*.java` : 모든 Java 소스 파일을 포함
    - **소스코드 컴파일(의존성이 있는 경우)**
        - 빌드 도구가 없을 경우 의존성(라이브러리)을 직접 관리해야 함
        - 프로젝트에서 필요한 모든 외부 라이브러리를 수동으로 다운로드하고, 클래스패스에 추가 필요
            
            ```bash
            javac -d out -cp "libs/*" src/main/java/**/*.java
            ```
            
            - `-cp "libs/*"` : 모든 라이브러리를 클래스패스에 포함
        - ➕ 클래스패스(Classpath)는 JVM이 클래스 파일(.class)을 찾을 때 사용하는 경로
            - 클래스패스를 지정하지 않으면 기본적으로 자바 런타임 환경(JRE) 내부의 표준 라이브러리만을 참조
            - 따라서, 추가적인 라이브러리를 사용하려면 이 경로를 명시적으로 지정 필요
            <br>
2. **실행 가능한 JAR 파일 만들기(JAR 파일 패키징)**
    - `META-INF/MANIFEST.MF` 파일이 있어야 JAR 파일 실행 가능
        - 이 파일에서 애플리케이션의 진입점(Main-Class) 지정
            
            ```
            Main-Class: com.example.YourMainClass
            Class-Path: libs/
            ```
            
            - `META-INF/MANIFEST.MF` 예시
                - 해당 파일에 클래스패스를 지정하는 것은 JAR 파일을 실행할 때 적용되는 것이지, 컴파일 단계에서 적용되는 것은 아님
    - 컴파일된 클래스 파일을 JAR 파일로 패키징(JAR 파일 만들 때, `MANIFEST.MF` 파일 포함)
        
        ```bash
        jar cvfm app.jar META-INF/MANIFEST.MF -C out/ .
        ```
        
        - `-C out/ .` : out 디렉토리의 모든 파일을 JAR 파일에 포함
        - `cvfm` : 새로운 JAR 파일을 생성하고(c), JAR 파일 생성 과정에서 포함된 파일 목록을 자세히 출력(v), 파일 이름 지정(f), MANIFEST 파일 추가(m)
        <br>
3. **애플리케이션 실행**
    - JAR 파일 실행
        
        ```bash
        java -jar app.jar
        ```
        <br>
        <br>
        

### 👉 SpringBoot에서 Gradle을 사용한 빌드 및 배포 방법

---

1. **Gradle 파일(build.gradle)을 통해 필요한 의존성과 빌드 설정 관리**
    
    ```java
    plugins {
        id 'org.springframework.boot' version '3.0.1'
        id 'io.spring.dependency-management' version '1.0.15.RELEASE'
        id 'java'
    }
    
    group = 'com.example'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '17'
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
    
    tasks.named('test') {
        useJUnitPlatform()
    }
    
    ```
    <br>
    
2. **컴파일과 빌드**
    - 아래 명령어로 프로젝트를 빌드하고 컴파일
        
        ```bash
        ./gradlew build
        ```
        
    - 명령어 입력 시, 아래 기능 수행
        - 프로젝트 소스 코드를 컴파일
        - 테스트를 실행
        - 애플리케이션을 실행 가능한 jar 파일로 패키징
            - build/libs/ 디렉토리에 `프로젝트이름-버전.jar` 파일이 생성
            <br>
3. **애플리케이션 실행**
    - 아래 명령어로 JAR 파일 실행
        
        ```bash
        java -jar build/libs/프로젝트이름-버전.jar
        ```
        
        - 명령어 실행 시, 내장된 톰캣 서버가 시작되고 SpringBoot 애플리케이션 실행
            - `spring-boot-starter-web` 의존성에  내장 톰캣 서버를 실행시키는 데 필요한 라이브러리와 구성 요소들이 포함되어 있기 때문
