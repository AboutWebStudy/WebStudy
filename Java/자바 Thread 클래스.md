**Thread 클래스**는 **멀티스레딩**을 구현할 때 사용하는 클래스

### `Thread` 클래스란?

- `java.lang.Thread` 클래스는 **하나의 독립적인 작업 흐름**(스레드)을 만들기 위해 사용됨
- 자바 프로그램은 기본적으로 하나의 메인 스레드에서 실행되는데, `Thread` 클래스를 이용하면 여러 작업을 **동시에 병렬로 실행 가능**

---

### 사용하는 방법 두 가지

1. **`Thread` 클래스를 상속**해서 사용
    
    ```java
    class MyThread extends Thread {
        public void run() {
            System.out.println("Thread 실행 중!");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            MyThread t1 = new MyThread();
            t1.start(); // run()이 아닌 start() 호출해야 새로운 스레드에서 실행됨
        }
    }
    ```
    
2. **`Runnable` 인터페이스를 구현**해서 사용
    
    ```java
    class MyRunnable implements Runnable {
        public void run() {
            System.out.println("Runnable 실행 중!");
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Thread t1 = new Thread(new MyRunnable());
            t1.start();
        }
    }
    ```
    

---

### 핵심 포인트

- `run()` 메서드는 스레드가 실행할 작업을 정의.
- `start()` 메서드를 호출해야 **새로운 스레드**가 만들어져서 `run()`이 실행됨.
- `Thread`를 상속하거나 `Runnable`을 구현하면, 멀티스레딩 프로그래밍이 가능해져서 여러 작업을 동시에 수행할 수 있어.
    - 현대에서는 Thread 자체를 전부 상속받는 것 보다 Runnable 인터페이스를 통해 실행하는 것이 일반적

### 1. **`sleep(long millis)`**

- 일정 시간 동안 현재 실행 중인 스레드를 **멈춤** (ms 단위).
- 예: `Thread.sleep(1000);` → 1초간 대기
- 예외 처리 필요: `InterruptedException`

```java
try {
    Thread.sleep(1000); // 1초 잠자기
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

---

### 2. **`join()`**

- 다른 스레드가 **끝날 때까지 기다림**.
- 예: `thread.join();` → 해당 스레드가 끝날 때까지 현재 스레드 일시 중지

```java
Thread t = new Thread(() -> {
    System.out.println("작업 중...");
});

t.start();
t.join(); // 메인 스레드는 t가 끝날 때까지 대기
System.out.println("모든 작업 완료");
```

---

### 3. **`interrupt()`**

- 스레드에게 **“깨워!”** 신호를 줌 (직접 멈추진 않음)
- 일반적으로 `sleep()` 중이거나, 반복문 체크로 반응함.

```java
Thread t = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        // 작업 수행
    }
    System.out.println("인터럽트 감지됨, 종료");
});

t.start();
t.interrupt(); // t 스레드에게 인터럽트 신호 보냄
```

---

### 4. **`currentThread()`**

- 현재 실행 중인 **스레드 참조**를 반환
- 예: `Thread t = Thread.currentThread();`

```java
System.out.println("현재 스레드 이름: " + Thread.currentThread().getName());
```

---

### 5. **`isAlive()`**

- 스레드가 **아직 실행 중인지** 확인
- 반환값: `true`면 아직 끝나지 않음

```java
if (t.isAlive()) {
    System.out.println("아직 실행 중");
}
```

---

### 6. **`setName(String name)` / `getName()`**

- 스레드 이름 설정 및 조회

```java
Thread t = new Thread(() -> {});
t.setName("Worker-1");
System.out.println(t.getName()); // Worker-1
```

---

### 7. **`setPriority(int priority)`**

- 스레드의 우선순위 설정 (`1~10`, 기본은 5)
- 단, OS마다 우선순위 반영은 다름

```java
t.setPriority(Thread.MAX_PRIORITY); // 우선순위 10
```
