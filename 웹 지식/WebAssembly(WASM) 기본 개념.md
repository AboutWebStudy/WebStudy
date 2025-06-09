### 👉 WebAssembly(WASM)란?

---

- 웹에서 아주 빠르게 실행되는 프로그램을 만들 수 있도록 도와주는 새로운 기술
- C언어나 Rust 같은 빠른 언어로 코드를 짜고, 이걸 웹에서도 실행되게 만든 기술
- Wasm은 안전한 샌드박스 환경에서 실행되기 때문에 해킹이나 메모리 침범 위험이 낮음
- 예전에는 데스크탑 앱으로만 가능하던 고성능 작업이 웹에서도 가능
- 한 번 컴파일하면 모든 브라우저에서 동일하게 실행
- 바이트코드 형태로 압축되어 네트워크 전송에 유리
- 모든 주요 브라우저에서 지원(Chrome, Firefox, Safari, Edge), 2017년부터 안정적으로 사용 가능
<br>
<br>

### 👉 WebAssembly(Wasm)의 탄생 배경

---

- 기존 웹의 한계
    - 자바스크립트는 모든 브라우저에서 동작하지만
    - 정적 타입이 없고, GC가 있으며, 인터프리터 언어이기 때문에 고성능 작업에는 한계가 있음
    - 특히 게임, 영상처리, 머신러닝 같은 CPU-intensive 작업에는 비효율적
- ⇒ 네이티브 언어처럼 빠르고, 안전하면서, 웹 브라우저에서도 실행 가능한 포맷이 필요해짐
    - 일반적으로 JavaScript 대비 10-100배 빠른 실행 속도
    - 특히 수학 연산, 이미지/비디오 처리에서 큰 성능 향상
    <br>
    <br>

### 👉 WebAssembly(Wasm)의 핵심 개념

---

| 개념 | 설명 |
| --- | --- |
| **바이트코드** | 사람이 읽기 힘든, 기계가 빠르게 읽을 수 있는 코드 |
| **스택 기반 가상 머신** | 명령어를 스택에서 처리하는 방식 |
| **호출 스택, 메모리 관리** | 직접 메모리 컨트롤 가능(자동 가비지 컬렉션이 없음, 수동 또는 Rust ownership 사용) |
| **브라우저 통합** | JS와 상호작용 가능(JS → Wasm 호출 / 반대 가능) |
- 지원 언어
    - 주요 언어 : C/C++, Rust
    - 기타 지원 언어 : Go, C#, AssemblyScript 등
    - AssemblyScript : TypeScript와 유사한 문법으로 WASM 개발이 가능한 언어
    <br>
    <br>

### 👉 동작 구조(예시 : C 코드 → Wasm → 브라우저)

---

```c
// hello.c
int add(int a, int b) {
  return a + b;
}
```

1. **컴파일**
    
    ```c
    emcc hello.c -s WASM=1 -o hello.js
    ```
    
    - 출력물
        - hello.wasm(바이트코드)
        - hello.js(Wasm을 브라우저에서 로드하는 래퍼)
    - 명령어 분석
        - emcc : Emscripten 컴파일러로 C/C++ 코드를 WebAssembly로 변환해주는 도구
        - hello.c : 우리가 컴파일할 원본 C 파일
        - -s WASM=1 : WebAssembly 출력 형식으로 컴파일하라는 의미(asm.js 대신 .wasm을 만듦)
        - -o hello.js : 출력 파일 이름을 hello.js로 지정. 이 JS 파일이 .wasm을 로딩하고 실행해주는 역할
2. **브라우저에서 실행**
    
    ```jsx
    const wasm = await WebAssembly.instantiateStreaming(fetch('hello.wasm'));
    const result = wasm.instance.exports.add(3, 4); // 7
    ```
    <br>
    <br>
    

### 👉 WebAssembly의 구성 요소

---

- **Module**
    - `.wasm` 파일 자체
    - 컴파일된 함수/메모리 포함
- **Instance**
    - 모듈을 실행 가능한 상태로 만든 것
- **Memory**
    - 모듈에서 사용하는 가상 메모리
    - `ArrayBuffer` 기반
- **Import/Export**
    - 외부에서 함수를 불러오거나, 내보낼 수 있음(JS와 연동 포인트)
    <br>
    <br>

### 👉 JavaScript와의 통합

---

- JS → Wasm 호출
    
    ```jsx
    const { instance } = await WebAssembly.instantiateStreaming(fetch("math.wasm"));
    instance.exports.square(5); // -> 25
    ```
    
- Wasm → JS 함수 사용(Import)
    
    ```c
    // C 코드에서 JS 함수를 호출
    extern void js_log(int x);
    void callLog(int a) {
      js_log(a);
    }
    ```
    
    - JS에서 js_log라는 함수를 정의해서 Wasm이 호출할 수 있음
    <br>
    <br>

### 👉 실제 사용 예시

---

- **Figma**
    - Rust로 렌더링 엔진 작성 → Wasm으로 웹에서 실행
- **AutoCAD Web**
    - C++로 작성된 핵심 로직을 Wasm으로 컴파일
- **TensorFlow.js**
    - 일부 모델은 WebAssembly backend로 훨씬 빠르게 추론 가능
    <br>
    <br>

### 👉 장단점

---

| 장점 | 단점 |
| --- | --- |
| 매우 빠름 (네이티브 수준) | 디버깅 어려움 |
| 언어 호환성 (C/C++/Rust 등) | 개발 초기엔 학습곡선 존재 |
| 안전성 확보 (샌드박스) | GC 없음(직접 메모리 관리 필요) |
| JS와 통합 가능 | 브라우저 제한 있음(파일 I/O, UI 등은 JS 필요) |
<br>
<br>

### 👉 실제 사용 시 고려사항

---

- 작은 작업에는 JavaScript가 더 효율적일 수 있음(오버헤드 때문)
- 주로 CPU 집약적인 작업에 적합
- 간단한 연산보다는 복잡한 알고리즘이나 대량의 데이터 처리에 효율적
