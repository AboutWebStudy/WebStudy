### 👉 메서드 체이닝(Method Chaining)

---

- 여러 메서드 호출을 연속적으로 연결하여 하나의 표현식으로 작성하는 프로그래밍 기법
- 각 메서드가 객체 자신(`this`)을 반환하여 계속 이어서 메서드를 호출할 수 있음
    - 일반적인 `setter 메서드`는 `void`를 반환하기 때문에 메서드 체이닝이 불가능
    - `return this`를 통해 모든 메서드가 같은 객체를 대상으로 실행됨
    - `return this`가 메서드 연속 호출을 가능하게 하는 이유
        
        ```java
        // 순서 관점으로 이해하기
        
        Car car = new Car();
        
        // 1단계 : car.setColor("빨강") 실행
        Car temp1 = car.setColor("빨강");  // return this; 때문에 car 객체가 반환됨
        
        // 2단계 : temp1.setBrand("현대") 실행 (temp1은 car와 같은 객체)
        Car temp2 = temp1.setBrand("현대"); // return this; 때문에 car 객체가 반환됨
        
        // 실제로는 이렇게 한 줄로 쓸 수 있음
        Car result = car.setColor("빨강").setBrand("현대");
        ```
        
        ⇒ car, temp1, temp2, result는 전부 같은 객체
        
        ```java
        // 메모리 관점으로 이해하기
        
        Car car = new Car(); // 메모리 주소 : @7344699f
        
        // car.setColor("빨강")를 호출하면
        public Car setColor(String color) {
            this.color = color;  // @7344699f 객체의 color 설정
            return this;         // @7344699f 주소를 반환
        }
        
        // 그래서 car.setColor("빨강").setBrand("현대")는
        // @7344699f.setBrand("현대")와 같음
        ```
        
- 플루언트 인터페이스(Fluent Interface)를 구현할 수 있음
    - 자연어처럼 읽히는 API를 만드는 설계 방법
        
        ```java
        public class QueryBuilder {
            public static QueryBuilder newQuery() { return new QueryBuilder(); }
            public QueryBuilder select(String... columns) { return this; }
            public QueryBuilder from(String table) { return this; }
            public QueryBuilder where(String condition) { return this; }
            public QueryBuilder and(String condition) { return this; }
            public QueryBuilder or(String condition) { return this; }
            public QueryBuilder orderBy(String column) { return this; }
        }
        
        // 사용했을 때, SQL 문장처럼 읽힘!
        String sql = QueryBuilder.newQuery()
            .select("name", "age", "email")
            .from("users")
            .where("age > 18")
            .and("city = '서울'")
            .orderBy("name")
            .build();
        ```
        
- 예제
    - 메서드 체이닝 없이 사용할 때
        
        ```java
        public class Car {
            private String color;
            private String brand;
            private int speed;
            
            public void setColor(String color) {
                this.color = color;
            }
            
            public void setBrand(String brand) {
                this.brand = brand;
            }
            
            public void setSpeed(int speed) {
                this.speed = speed;
            }
        }
        
        // 사용할 때
        Car car = new Car();
        car.setColor("빨강");
        car.setBrand("현대");
        car.setSpeed(100);
        ```
        
    - 메서드 체이닝을 사용할 때
        
        ```java
        public class Car {
            private String color;
            private String brand;
            private int speed;
            
            public Car setColor(String color) {
                this.color = color;
                return this; // 자기 자신을 돌려줌
            }
            
            public Car setBrand(String brand) {
                this.brand = brand;
                return this; // 자기 자신을 돌려줌
            }
            
            public Car setSpeed(int speed) {
                this.speed = speed;
                return this; // 자기 자신을 돌려줌
            }
        }
        
        // 사용할 때 : 한 줄로 작성할 수 있음
        Car car = new Car()
            .setColor("빨강")
            .setBrand("현대")
            .setSpeed(100);
        ```
        <br>
        <br>
        

### 👉 빌더(Builder) 패턴

---

- 복잡한 객체의 생성 과정을 단계별로 나누어 객체를 생성하는 디자인 패턴
- 메서드 체이닝을 활용한 디자인 패턴으로 가독성이 높은 코드를 작성할 수 있음
- 불변 객체를 생성할 수 있음
- 생성자 오버로딩의 필요성을 줄이고, 선택적 매개변수를 직관적으로 처리할 수 있음
- 빌더 패턴은 클래스의 내부 클래스로 `static Builder 클래스`를 생성해서 구현
    - 해당 클래스의 객체를 만들기 위한 전용 도구 같은 것이라 논리적 연관성 때문에 내부 클래스로 구현
    - 객체를 생성할 때, 코드의 간결성 및 가독성이 좋음
        
        ```java
        // 좋은 예 : 연관성이 명확함
        Hamburger.Builder builder = new Hamburger.Builder();
        
        // 나쁜 예 : 별도 클래스로 만들면
        HamburgerBuilder builder = new HamburgerBuilder(); // 이름이 길어짐
        ```
        
    - 객체 생성을 `Builder`를 통해서만 하도록 강제하기 위해 생성자를 `private`으로 설정하는 것이 일반적인데, 내부 클래스인 경우 `Builder`에서 클래스의 `private` 생성자에 접근 가능
    - 일반 내부 클래스인 경우, 클래스의 객체를 생성해야 `Builder`에 대한 객체를 생성할 수 있기 때문에 `static 내부 클래스`로 선언
- 예제
    - 빌더 패턴을 적용 안 했을 때
        
        ```java
        // 생성자가 복잡함
        public class Hamburger {
            public Hamburger(String bun, String meat, String cheese, 
                            boolean lettuce, boolean tomato, boolean onion, 
                            boolean pickle, String sauce) {
                // ...
            }
        }
        
        // 사용할 때, 어떤 매개변수인지 알기 어려움 : 자료형이 같으면 순서가 잘못되어도 오류 없이 생성됨
        Hamburger burger = new Hamburger("참깨빵", "소고기", "체다치즈", 
                                        true, true, false, true, "마요네즈");
        ```
        
    - 빌더 패턴을 적용했을 때
        
        ```java
        public class Hamburger {
            private String bun;
            private String meat;
            private String cheese;
            private boolean lettuce;
            private boolean tomato;
            private boolean onion;
            private boolean pickle;
            private String sauce;
            
            // 생성자는 private으로 외부에서 직접 못 만들게 함
            private Hamburger(Builder builder) {
                this.bun = builder.bun;
                this.meat = builder.meat;
                this.cheese = builder.cheese;
                this.lettuce = builder.lettuce;
                this.tomato = builder.tomato;
                this.onion = builder.onion;
                this.pickle = builder.pickle;
                this.sauce = builder.sauce;
            }
            
            // Builder 클래스 : 햄버거를 만드는 도구
            public static class Builder {
                // 필수 재료
                private String bun;
                private String meat;
                
                // 선택 재료(기본값 설정)
                private String cheese = "";
                private boolean lettuce = false;
                private boolean tomato = false;
                private boolean onion = false;
                private boolean pickle = false;
                private String sauce = "";
                
                // 필수 재료는 생성자에서 받음
                public Builder(String bun, String meat) {
                    this.bun = bun;
                    this.meat = meat;
                }
                
                // 선택 재료들은 메서드로 처리
                public Builder cheese(String cheese) {
                    this.cheese = cheese;
                    return this; // 체이닝을 위해
                }
                
                public Builder lettuce(boolean lettuce) {
                    this.lettuce = lettuce;
                    return this;
                }
                
                public Builder tomato(boolean tomato) {
                    this.tomato = tomato;
                    return this;
                }
                
                public Builder onion(boolean onion) {
                    this.onion = onion;
                    return this;
                }
                
                public Builder pickle(boolean pickle) {
                    this.pickle = pickle;
                    return this;
                }
                
                public Builder sauce(String sauce) {
                    this.sauce = sauce;
                    return this;
                }
                
                // 최종적으로 햄버거 객체 생성
                public Hamburger build() {
                    return new Hamburger(this);
                }
            }
        }
        ```
        
        ```java
        // 어떤 매개변수에 어떤 값을 넣는지 명확하게 알 수 있음
        Hamburger myBurger = new Hamburger.Builder("참깨빵", "소고기") // new로 Builder 객체 생성
            .cheese("체다치즈") // Builder 객체의 메서드 호출
            .lettuce(true)    // Builder 객체의 메서드 호출
            .tomato(true)     // Builder 객체의 메서드 호출
            .sauce("마요네즈")  // Builder 객체의 메서드 호출
            .build();         // Hamburger의 최종 객체 생성
        
        // 간단한 햄버거 객체도 쉽게 생성 가능
        Hamburger simpleBurger = new Hamburger.Builder("일반빵", "닭고기")
            .build();
        ```
        <br>
        <br>
        

### 👉 @Builder 어노테이션

---

- 컴파일 타임에 `static 내부 Builder 클래스`를 자동으로 생성해주는 어노테이션
- 예시
    - 우리가 작성한 코드
        
        ```java
        import lombok.Builder;
        
        @Builder
        public class Pizza {
            private String dough;
            private String sauce;
            private String cheese;
            private boolean pepperoni;
            private boolean mushroom;
        }
        ```
        
    - 어노테이션으로 자동 생성된 코드
        
        ```java
        public class Pizza {
            private String dough;
            private String sauce;
            private String cheese;
            private boolean pepperoni;
            private boolean mushroom;
            
            // Lombok이 자동으로 생성하는 생성자
            Pizza(String dough, String sauce, String cheese, boolean pepperoni, boolean mushroom) {
                this.dough = dough;
                this.sauce = sauce;
                this.cheese = cheese;
                this.pepperoni = pepperoni;
                this.mushroom = mushroom;
            }
            
            // Lombok이 자동으로 생성하는 static 내부 Builder 클래스
            public static class PizzaBuilder {
                private String dough;
                private String sauce;
                private String cheese;
                private boolean pepperoni;
                private boolean mushroom;
                
                PizzaBuilder() {
                }
                
                public PizzaBuilder dough(String dough) {
                    this.dough = dough;
                    return this;
                }
                
                public PizzaBuilder sauce(String sauce) {
                    this.sauce = sauce;
                    return this;
                }
                
                public PizzaBuilder cheese(String cheese) {
                    this.cheese = cheese;
                    return this;
                }
                
                public PizzaBuilder pepperoni(boolean pepperoni) {
                    this.pepperoni = pepperoni;
                    return this;
                }
                
                public PizzaBuilder mushroom(boolean mushroom) {
                    this.mushroom = mushroom;
                    return this;
                }
                
                public Pizza build() {
                    return new Pizza(dough, sauce, cheese, pepperoni, mushroom);
                }
                
                public String toString() {
                    return "Pizza.PizzaBuilder(dough=" + this.dough + 
                           ", sauce=" + this.sauce + ", cheese=" + this.cheese + 
                           ", pepperoni=" + this.pepperoni + ", mushroom=" + this.mushroom + ")";
                }
            }
            
            // 빌더 팩토리 메서드도 자동 생성
            public static PizzaBuilder builder() {
                return new PizzaBuilder();
            }
        }
        ```
        
        ```java
        // 사용 - 두 가지 방법
        // 방법 1 : 팩토리 메서드 사용 (추천)
        Pizza pizza1 = Pizza.builder()
            .dough("씬크러스트")
            .sauce("토마토")
            .cheese("모짜렐라")
            .pepperoni(true)
            .build();
        
        // 방법 2 : 직접 Builder 생성
        Pizza pizza2 = new Pizza.PizzaBuilder()
            .dough("씬크러스트")
            .sauce("토마토")
            .build();
        ```
        
        - 팩토리 메서드란?
            - 객체를 직접 생성하는 대신, 객체 생성을 담당하는 메서드
            - 클래스명을 직접 몰라도 객체를 생성할 수 있다는 장점이 있음
            - 나중에 다른 `Builder`를 반환하도록 변경하는 등 구현을 바꿀 수 있음
- `@Builder`의 추가 기능들
    - 기본값 설정
        
        ```java
        @Builder
        public class Pizza {
            @Builder.Default
            private String dough = "일반도우";  // 기본값
            
            @Builder.Default
            private boolean pepperoni = false;  // 기본값
            
            private String sauce;
            private String cheese;
        }
        
        // 사용
        Pizza pizza = Pizza.builder()
            .sauce("토마토")
            .cheese("모짜렐라")
            .build();  // dough와 pepperoni는 기본값 사용
        ```
        
    - 필수 필드 지정
        
        ```java
        @Builder
        public class Pizza {
            @NonNull  // 필수 필드
            private String dough;
            
            @NonNull  // 필수 필드
            private String sauce;
            
            private String cheese;  // 선택 필드
        }
        ```
        
        - `@NonNull`은 Lombok에서 제공하는 어노테이션으로, 해당 필드가 `null`이면 `NullPointerException`을 발생시키는 코드를 자동 생성
    - 빌더 메서드명 커스텀
        
        ```java
        @Builder(builderMethodName = "createPizza")
        public class Pizza {
            private String dough;
            private String sauce;
        }
        
        // 사용
        Pizza pizza = Pizza.createPizza()  // builder() 대신 createPizza()
            .dough("씬크러스트")
            .sauce("토마토")
            .build();
        ```
