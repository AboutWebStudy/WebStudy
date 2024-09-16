### 👉 직렬화와 역직렬화 개념 정리

---

- 데이터나 객체를 전송하거나 저장할 때 중요한 개념
- 웹 개발에서 객체 데이터를 외부로 전송하거나 저장할 때 자주 사용
- 직렬화는 데이터를 저장하거나 전송할 때 객체 상태를 유지하고, 효율적으로 데이터를 주고받기 위해 사용하는 방법
    - ⇒ 직렬화를 사용하지 않아도 전송/저장이 가능하지만 여러 필요 때문에 직렬화 사용(아래에서 설명)
    <br>
    <br>

### 👉 직렬화(Serialization)

---

- 직렬화는 객체를 이진 데이터(바이트 스트림)으로 변환하는 과정
    - 바이트 스트림이란, 이진수로 변환하는 것
        
        ```java
        "Hello" : 01001000 01100101 01101100 01101100 01101111
        ```
        
- 이렇게 하면 객체를 파일로 저장하거나 네트워크를 통해 전송할 수 있음
- Java에서는 객체를 직렬화하기 위해 Serializable 인터페이스를 구현해야 함
    
    ```java
    import java.io.Serializable;
    
    public class Employee implements Serializable {
        private static final long serialVersionUID = 1L;  // 클래스 버전을 명시하는 필드
        private String name;
        private int age;
    }
    ```
    
    - 직렬화 가능한 객체는 JVM 메모리 외부로 출력될 수 있음
    - `serialVersionUID`는 직렬화된 객체의 버전을 식별하는 ID
        - 직렬화된 객체를 나중에 역직렬화할 때, 이 serialVersionUID 값이 맞지 않으면 "이 객체는 더 이상 원래 클래스와 맞지 않다"고 판단하여 역직렬화를 실패하게 됨
        - 서로 같은 버전의 객체끼리만 직렬화하고 역직렬화할 수 있도록 하는 안전장치
        - `serialVersionUID`를 명시하지 않으면 Java가 자동으로 생성
            - 클래스 구조가 변경되면 Java가 자동으로 새로운 `serialVersionUID` 값을 생성할 수 있으며, 이는 직렬화된 객체와 호환되지 않을 수 있음
            - ⇒ 때문에 명시적으로 지정하여 수동으로 관리하는 것이 좋음
- 기본적인 Java 직렬화 외에도 JSON이나 XML 형식으로 직렬화하는 외부 라이브러리(Jackson, Gson 등)를 사용하면 보다 효율적
<br>
<br>

### 👉 직렬화 코드 예시

---

```java
// Employee 객체 생성, "John"이라는 이름과 30이라는 나이를 가진 Employee 객체
Employee emp = new Employee("John", 30);

// FileOutputStream을 생성하여 "employee.ser" 파일을 쓰기 모드로 열음
// 이 파일에 직렬화된 데이터를 저장할 것임
FileOutputStream fileOut = new FileOutputStream("employee.ser");

// ObjectOutputStream을 생성하여, Employee 객체를 직렬화하기 위해 사용
// ObjectOutputStream은 바이트 스트림을 통해 객체를 직렬화하는데, 그 바이트 스트림을 FileOutputStream에 전달하여 파일에 기록함
ObjectOutputStream out = new ObjectOutputStream(fileOut);

// Employee 객체를 직렬화하여 "employee.ser" 파일로 출력
// 즉, emp 객체의 상태를 바이트 스트림으로 변환해 파일에 기록
out.writeObject(emp);  // 직렬화 수행

// ObjectOutputStream을 닫아 자원을 해제
out.close();

// FileOutputStream도 닫아 파일을 닫고 자원을 해제
fileOut.close();
```
<br>
<br>

### 👉 역직렬화

---

- 역직렬화는 직렬화된 바이트 스트림을 다시 객체로 변환하는 과정
    - ⇒ 외부 저장소에서 읽은 이진 데이터를 Java 객체로 변환하는 것
- 예를 들어, 파일에서 읽거나 네트워크로부터 받은 데이터를 객체로 변환하는 과정
<br>
<br>

### 👉 역직렬화 코드 예시

---

```java
// 파일로부터 직렬화된 데이터를 읽기 위한 FileInputStream을 생성
FileInputStream fileIn = new FileInputStream("employee.ser");

// ObjectInputStream을 통해 파일에서 직렬화된 객체를 읽을 준비
ObjectInputStream in = new ObjectInputStream(fileIn);

// 파일에 직렬화되어 저장된 Employee 객체를 읽어와 역직렬화하여 Employee 객체로 복원
Employee emp = (Employee) in.readObject();  // 역직렬화 수행

// ObjectInputStream을 닫아 자원 해제
in.close();

// FileInputStream을 닫아 파일 자원을 해제
fileIn.close();

// 역직렬화된 Employee 객체의 name 필드를 출력
System.out.println(emp.getName());
```

```java
@RestController
public class EmployeeController {
    
    @PostMapping("/employee")
    public void saveEmployee(@RequestBody Employee employee) {
        // 요청 본문에서 JSON 데이터를 받아 Employee 객체로 역직렬화
        System.out.println(employee.getName());
    }
    
    @GetMapping("/employee")
    public Employee getEmployee() {
        // Employee 객체를 JSON으로 직렬화하여 클라이언트로 반환
        return new Employee("Jane", 28);
    }
}
```

- Java에서 JSON 직렬화는 매우 많이 사용
    - 특히 RESTful API 개발에서 중요한 역할을 함
    - 스프링부트는 Jackson 라이브러리를 사용하여 자동으로 객체를 JSON으로 직렬화하고 역직렬화 수행
    - ⇒ 웹 프레임워크에서 JSON 직렬화는 주로 serialVersionUID와 상관없이 동작
    <br>
    <br>

### 👉 직렬화와 역직렬화 사용 예시

---

- **HTTP 응답/요청**
    - 객체를 JSON이나 XML 형식으로 직렬화하여 전송하고, 역직렬화하여 다시 객체로 변환
    <br>
- **데이터베이스 저장**
    - 객체를 직렬화하여 데이터베이스에 저장하고, 역직렬화하여 가져옴
    <br>
- **캐싱**
    - 객체를 직렬화하여 Redis와 같은 캐시 저장소에 저장하고 필요할 때 역직렬화하여 다시 사용
    <br>
- **버전 호환성을 유지해야 하는 시스템**
    - 장기적으로 데이터를 보관하거나 여러 서버에서 동일 객체를 주고받는 환경
    <br>
    <br>

### 👉 직렬화를 사용하는 이유

---

- 직렬화의 가장 큰 목적은 객체의 상태를 유지하는 것
    - 객체를 바이트 스트림으로 변환하면 객체가 가지고 있던 모든 필드 값, 참조 관계 등을 그대로 유지하면서 다른 시스템으로 전송할 수 있음
        - 직렬화는 객체의 상태를 보존한 채 다른 환경에서도 동일한 객체로 사용할 수 있게 해줌
    - 예를 들어, 메모리에 있는 객체를 파일에 저장하고 나중에 다시 그 객체를 복원해야 하는 경우
        - 메모리는 프로그램이 종료되면 데이터가 사라지기 때문에, 객체를 저장하거나 다른 시스템으로 전송할 수 있어야 함
        - 직렬화를 통해 객체의 상태를 바이트 스트림으로 변환하면 객체의 모든 필드 값과 상태를 유지한 채로 파일이나 네트워크를 통해 주고받을 수 있음
        - 그 후, 다시 그 데이터를 객체로 복원할 수 있게 됨
        <br>
- 네트워크 통신에서 객체 전송
    - 분산 시스템이나 네트워크를 통해 객체를 다른 서버로 전송해야 할 때도 직렬화가 유용
        - Java 객체는 메모리 상에서만 존재하기 때문에, 이를 전송 가능한 형태로 변환해야 함
        - 직렬화를 통해 객체를 바이트 스트림으로 만들어 네트워크를 통해 전송할 수 있으며, 수신 측에서는 이를 역직렬화하여 객체로 복원할 수 있음
    - 예를 들어, RMI(Remote Method Invocation)나 EJB(Enterprise Java Beans) 같은 Java의 분산 시스템 기술은 객체를 네트워크 상에서 주고받을 때 직렬화를 사용
        - 이 과정에서 객체는 메모리에서 바이트 스트림으로 변환되어 네트워크로 전송되고, 수신한 쪽에서 다시 역직렬화되어 객체로 복원됨
        <br>
- 캐싱 및 데이터 영속화
    - 객체 데이터를 캐시나 데이터베이스에 저장할 때도 직렬화가 필요할 수 있음
    - 예를 들어, Redis와 같은 캐시 시스템에서는 객체 데이터를 바이트 스트림으로 변환해 캐시에 저장하고, 이를 나중에 다시 객체로 복원 ⇒ 이때도 직렬화가 필요
    - 또한, 파일 시스템이나 데이터베이스에 객체를 저장할 때도 직렬화를 사용해 객체의 상태를 저장하고, 나중에 다시 불러올 수 있음
    <br>
    <br>

### 👉 직렬화의 장점과 단점

---

- 장점
    - 객체를 바이트 스트림으로 변환하면 파일 시스템이나 네트워크를 통해 데이터를 쉽게 전송할 수 있음
    - 직렬화를 통해 객체의 상태를 저장/보존하고 필요 시 복원할 수 있음
    <br>
- 단점
    - 직렬화와 역직렬화는 성능을 저하시킬 수 있음. 특히 객체가 매우 복잡하거나 큰 경우 처리 속도가 느려질 수 있음
    - 클래스가 변경되었을 때 serialVersionUID 값이 다르면 역직렬화에 실패할 수 있음
        - 클래스의 직렬화 버전을 명시해야 함
        - 버전 관리 없이 클래스가 변경되면 역직렬화에 실패할 수 있음
    - 직렬화된 데이터는 쉽게 조작될 수 있으므로 민감한 정보를 직렬화할 때는 주의해야 함
        - Java 직렬화에서 민감한 필드는 transient 키워드를 사용하여 직렬화에서 제외할 수 있음
        <br>
        <br>

### ➕ serialVersionUID에 대한 궁금증 : 실제로 버전을 바꿔 사용하는 경우를 본 경우가 없음

---

- **`serialVersionUID`를 사용하게 되는 경우**
    - 직렬화된 객체를 나중에 복원(역직렬화)하려고 할 때, 클래스 구조가 변경된 경우 `serialVersionUID`가 중요하게 작동
    - 예를 들어, 한 시스템에서 객체를 직렬화한 후 그 객체를 다른 시스템에서 역직렬화하려고 하는 경우, 두 시스템 간의 객체 버전이 다르면 역직렬화가 실패할 수 있음
    - 다만, 웹 프레임워크는 데이터를 주고받을 때 주로 JSON을 사용하여 직렬화 및 역직렬화를 수행하고, 이는 serialVersionUID와 상관없이 잘 동작해서 직접적으로 유의미하게 사용하는 경우가 적은 것
    <br>
- **실제로 `serialVersionUID`가 중요한 경우**
    - 클래스 구조가 변경된 경우
        - 만약 클래스에 새로운 필드를 추가하거나 기존 필드를 제거한다면, 클래스의 구조가 바뀌게 되는데 이때 직렬화된 객체를 읽어들일 때 필드가 일치하지 않으면 역직렬화 오류가 발생할 수 있음
        - 이 문제를 해결하려면 클래스가 변경될 때마다 serialVersionUID를 명시적으로 관리해야 함
        - 새 필드를 추가하거나 제거할 때 serialVersionUID를 변경하지 않으면 기존 객체와의 호환성을 유지할 수 없기 때문
    - 배포 환경에서의 호환성
        - 예를 들어, 서버 간의 데이터를 주고받거나 여러 시스템에서 동일한 객체를 사용해야 하는 분산 시스템에서 한 시스템이 객체를 직렬화하고 다른 시스템에서 역직렬화하려 할 때, 각 시스템의 클래스 버전이 다르면 문제가 생길 수 있음
        - 이 경우 serialVersionUID를 변경하거나, 특정 구조 변경을 반영하여 새로운 버전을 만들어줘야 함
    - Java의 자동 생성
        - serialVersionUID를 명시하지 않으면, Java가 자동으로 생성해주는데, 이 값은 클래스 구조에 따라 달라짐
        - 만약 클래스 구조를 조금이라도 변경하면 자동 생성된 값이 달라지기 때문에 역직렬화에 실패할 수 있음
        - 이러한 문제를 방지하기 위해 명시적으로 serialVersionUID를 선언하는 것이 일반적
        <br>
- **자동 생성으로 인해 `serialVersionUID`가 변경되어 역직렬화가 되지 않는 경우 해결 방법**
    - serialVersionUID 고정
        - 우선, serialVersionUID를 명시적으로 설정하고 고정하는 것이 첫 번째 방법
        - 클래스에 변경이 있어도 serialVersionUID 값을 변경하지 않으면, 이전에 직렬화된 객체를 역직렬화할 수 있음
        - 주의할 점은 클래스 구조가 바뀌더라도 역직렬화를 허용하기 때문에 새로운 필드 값은 null이 되거나 기본값으로 설정됨
        - 이 방법은 호환성을 유지하지만, 데이터를 손실할 가능성이 있음
        <br>
    - 기본값 제공 (readObject 사용)
        
        ```java
        public class Employee implements Serializable {
            private static final long serialVersionUID = 1L;
            
            private String name;
            private int age;
            private String department;
        
            // 기본값 제공을 위한 readObject 메소드
            private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
                ois.defaultReadObject();  // 기존 필드 읽기
                if (department == null) {  // 만약 새 필드가 직렬화된 데이터에 없다면
                    department = "Unknown";  // 기본값 설정
                }
            }
        }
        ```
        
        - 새로운 필드가 추가되었을 때 기본값을 설정해주고 싶다면, readObject 메소드를 오버라이딩하여 새 필드에 대해 기본값을 제공할 수 있음
        - 이 방법은 새로운 필드가 포함된 경우에도 역직렬화할 수 있게 함
        <br>
    - 필드 호환성을 유지하는 방법
        
        ```java
        public class Employee implements Serializable {
            private static final long serialVersionUID = 1L;
            
            private String name;
            private int age;
            // 더 이상 사용하지 않지만 삭제하지 않음
            private String legacyField;  // 구 버전과 호환성을 유지하기 위한 필드
            private String department;  // 새로운 필드
        }
        ```
        
        - 직렬화 시 객체의 필드를 삭제하지 않고 남겨두는 것으로 기존 필드를 삭제하는 대신, 더 이상 사용하지 않더라도 계속 유지하는 방법
        - 새 필드를 추가해도 역직렬화 시 기존 필드에 대한 데이터는 여전히 처리됨
            - 클래스를 수정할 때 기존 직렬화된 객체와의 호환성을 유지하면서 새로운 기능을 추가할 수 있음
            <br>
    - `Externalizable` 인터페이스를 사용하여 완전한 제어
        
        ```java
        public class Employee implements Externalizable {
            private String name;
            private int age;
            private String department;
            
            public Employee() {
                // 기본 생성자 필요
            }
        
            @Override
            public void writeExternal(ObjectOutput out) throws IOException {
                out.writeObject(name);
                out.writeInt(age);
                out.writeObject(department);
            }
        
            @Override
            public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
                name = (String) in.readObject();
                age = in.readInt();
                department = (String) in.readObject();
                if (department == null) {
                    department = "Unknown";  // 기본값 설정
                }
            }
        }
        ```
        
        - Serializable은 직렬화 과정을 자동으로 처리하지만, Externalizable은 직렬화와 역직렬화 과정을 완전히 제어할 수 있게 해줌
        <br>
    - 버전 관리를 통한 호환성 유지
        - 대규모 프로젝트에서는 객체 직렬화를 사용하는 경우, 호환성 있는 버전을 유지하기 위한 버전 관리를 함
        - 새 버전을 추가하더라도 기존 시스템과의 호환성을 유지하는 것이 중요하기 때문에, 직렬화된 데이터의 버전 정보와 클래스를 대응시켜 관리
