`volatile`은 자바에서 **멀티스레딩 환경**에서 자주 사용되는 키워드로, 변수에 대한 **가시성(visibility)** 을 보장해 주는 기능

---

### `volatile`이란?

`volatile` 키워드는 **변수 선언 앞에 붙여서** 사용되며, 해당 변수가 **여러 스레드에서 동시에 접근될 수 있음을 나타냄**

```java

private volatile boolean running = true;
```

---

### 필요한 상황

자바에서 멀티스레딩을 할 때, **각 스레드는 자신만의 캐시**에 변수 값을 저장할 수 있음

이때, 한 스레드가 변경한 값이 **다른 스레드에 보이도록 캐시 업데이트 진행**

예시:

```java
class Example implements Runnable {
    private boolean running = true;

    public void run() {
        while (running) {
            // do something
        }
    }

    public void stop() {
        running = false;
    }
}

```

이 코드는 `stop()` 메서드에서 `running`을 `false`로 바꾸더라도, `run()` 내부의 스레드는 `true`만 계속 보고 멈추지 않음

```java

private volatile boolean running = true;
```

---

### `volatile`의 효과

1. **가시성 보장 (Visibility)**
    - 어떤 스레드가 값을 바꾸면, 다른 모든 스레드는 그 변경을 즉시 볼 수 있음
2. **원자성은 보장하지 않음 (No Atomicity)**
    - `volatile`은 **읽기/쓰기 자체는 원자적**이지만, **복합 연산(++, += 등)은 원자적이지 않음.**
    - 예: `count++`는 `volatile`로도 안전하지 않음

---

### `volatile` vs `synchronized`

| 특징 | `volatile` | `synchronized` |
| --- | --- | --- |
| 가시성 | O | O |
| 원자성 | X | O |
| 성능 | 빠름 | 상대적으로 느림 (락 걸림) |
| 용도 | 단순 플래그, 읽기/쓰기 변수 | 복잡한 상태 공유, 연산 등 |

---

### 🔸 사용 예시

```java
public class FlagExample {
    private volatile boolean flag = false;

    public void setFlagTrue() {
        flag = true;
    }

    public void waitForFlag() {
        while (!flag) {
            // 대기 중
        }
        System.out.println("Flag detected!");
    }
}

```

요약

- volatile은 멀티 스레드 환경에서 **변수 값의 변경이 다른 스레드에 보이지 않는 문제(가시성 문제)** 를 해결하기 위해 사용된다.
- 일반적으로 CPU는 변수 값을 **자신의 캐시에 저장**하고 사용하기 때문에, 한 스레드에서 변경한 값이 **다른 스레드의 캐시에 반영되지 않아** 변경이 전파되지 않는 문제가 발생할 수 있다.
- `volatile` 키워드를 사용하면, 변수의 **값을 항상 메인 메모리에서 읽고 쓰게 되어** 최신 값이 모든 스레드에 보이도록 보장할 수 있다.

```jsx
        [Thread A]                  [Thread B]
      ┌─────────────┐            ┌─────────────┐
      │ local cache │            │ local cache │
      │  running=T  │            │  running=F  │  ← stop() 호출됨
      └──────┬──────┘            └──────┬──────┘
             │                          │
             ▼                          ▼
         계속 루프 돈다          stop()에서 running = false로 설정

```

- Thread A는 여전히 `true`인 줄 알고 계속 루프를 돔
- Thread B는 메모리에서 `false`로 바꿨지만, A는 캐시된 값을 사용 중

```
    [Thread A]                  [Thread B]
  ┌─────────────┐            ┌─────────────┐
  │ local cache │            │ local cache │
  │             │            │             │
  └──────┬──────┘            └──────┬──────┘
         │                          │
         ▼                          ▼
  Main Memory (running)
           ↓
   running = false  ← 모든 스레드는 이 값을 직접 봄

```

- Thread B가 `false`로 바꾸면
- Thread A도 즉시 최신 값을 보고 루프 종료
