### 👉 익명 클래스란?

---

- 이름이 없는 클래스
- 일반적으로 한 번만 사용하기 위해 정의된 클래스
- 주로 인터페이스나 추상 클래스를 구현할 때 사용
- 익명 클래스는 기존 클래스나 인터페이스를 기반으로 즉석에서 새로운 객체를 생성하고, 이 객체에 고유한 기능을 추가하거나 오버라이드할 때 유용
- 익명 클래스는 람다 표현식과 유사한 목적으로 사용될 수 있지만, 람다는 함수형 인터페이스를 구현할 때 사용
    - ⇒ 익명 클래스는 더 복잡한 구현이 필요하거나 인터페이스가 함수형이 아닐 때 유용
  <br>
  <br>

### 👉 사용 사례

---

- 무작위로 업로드 되는 파일에서 특정 내용들을 추출하는 경우
    - 파일의 양식이 다양하고 정보를 추출한 뒤에는 해당 파일을 재사용할 필요가 없음
    - 파일 형식에 따라 데이터를 추출하는 로직이 달라질 수 있기 때문에, 매번 클래스를 새로 구현하는 것은 비효율적
    - ⇒ 이런 경우에 익명 클래스 사용
  <br>
  <br>

### 👉 익명 클래스의 특징

---

- 클래스의 이름이 없음
- 코드를 작성하는 도중에 정의되어, 새로운 파일을 만들 필요가 없고 선언과 동시에 객체를 생성
- 주로 한 번만 사용되는 경우에 적합
- 짧고 간단한 코드를 구현할 때 유용
    - 길거나 복잡한 코드에는 적합하지 않음
  <br>
  <br>

### 👉 예시

---

- 익명 클래스는 주로 인터페이스를 구현하거나 추상 클래스를 확장할 때 사용
- 기본 예시
    
    ```java
    // 인터페이스 정의
    interface Greeting {
        void sayHello();
    }
    
    // 익명 클래스 사용
    public class Main {
        public static void main(String[] args) {
            // Greeting 인터페이스를 구현하는 익명 클래스 생성
            Greeting greeting = new Greeting() {
                @Override
                public void sayHello() {
                    System.out.println("Hello, World!");
                }
            };
    
            // 메서드 호출
            greeting.sayHello();
        }
    }
    
    ```
    
- 실용적인 예시 : 익명 클래스는 이벤트 핸들러, 콜백 메서드, 스레드 구현 등에서 많이 사용
    
    ```java
    import javax.swing.JButton;
    import javax.swing.JFrame;
    import java.awt.event.ActionEvent;
    import java.awt.event.ActionListener;
    
    public class Main {
        public static void main(String[] args) {
            JFrame frame = new JFrame("Button Example");
            JButton button = new JButton("Click Me");
    
            // 버튼에 액션 리스너 추가
            button.addActionListener(new ActionListener() {
                @Override
                public void actionPerformed(ActionEvent e) {
                    System.out.println("Button clicked!");
                }
            });
    
            frame.add(button);
            frame.setSize(200, 200);
            frame.setVisible(true);
        }
    }
    
    ```
    
    - ActionListener 인터페이스를 구현하는 익명 클래스를 사용하여 버튼 클릭 이벤트를 처리
