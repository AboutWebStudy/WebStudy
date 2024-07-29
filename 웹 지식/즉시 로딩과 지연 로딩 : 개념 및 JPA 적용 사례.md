### 👉 즉시 로딩(Eager Loading)과 지연 로딩(Lazy Loading) 역할

---

- 웹 페이지나 애플리케이션의 리소스를 언제 불러오는지에 대한 전략
<br>
<br>

### 👉 즉시 로딩(Eager Loading)

---

- 페이지나 애플리케이션이 처음 로드될 때 필요한 모든 데이터를 한 번에 불러오는 방식
- 이 방식은 초기 로딩 시간이 길어질 수 있지만, 이후의 사용자 경험이 빠르고 매끄러움
- ⇒ 사용자 경험이 한 번에 완성되어야 하는 경우, 예를 들어 첫 인상이 중요한 랜딩 페이지나 자주 사용되는 데이터를 즉시 보여줘야 하는 경우에 사용
- 예시
    - 사용자가 웹 페이지를 처음 방문할 때, 페이지에 필요한 모든 이미지, CSS, JavaScript 등을 한 번에 다 불러오는 경우
    - 데이터베이스에서 관련된 모든 데이터를 한 번에 불러오는 경우
    <br>
    <br>

### 👉 지연 로딩(Lazy Loading)

---

- 필요한 리소스나 데이터를 사용자에게 즉시 보여줄 필요가 있을 때만 불러오는 방식
- 초기 로딩 시간을 단축시킬 수 있으며, 사용자가 실제로 필요로 하는 시점에만 데이터를 불러오기 때문에 효율적
- ⇒ 애플리케이션의 초기 로딩 시간을 줄이고, 네트워크와 서버 자원을 절약하고 싶을 때 사용
    - 사용자들이 모든 콘텐츠를 다 보지 않을 가능성이 높은 경우 유용
- 예시
    - 사용자가 스크롤을 내릴 때, 화면에 보이는 이미지들만 불러오는 경우
    - 사용자가 페이지 아래로 스크롤할 때마다 추가 콘텐츠를 불러오는 경우(무한 스크롤)
    - 데이터베이스에서 특정 데이터를 필요로 할 때 그때그때 불러오는 것
    <br>
    <br>

### 👉 Spring JPA 에서의 즉시 로딩과 지연 로딩

---

- **즉시 로딩**
    - 즉시 로딩은 엔티티를 조회할 때 관련된 모든 연관 데이터를 함께 불러오는 방식
        - 엔티티를 불러올 때 그 엔티티와 연관된 다른 모든 데이터도 즉시 데이터베이스에서 가져 옴
    - 초기 로딩 시간이 길어지지만, 모든 연관 데이터가 이미 로드되어 있기 때문에 바로 사용할 수 있음
    - 예시
        
        ```java
        @Entity
        public class Author {
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
            private String name;
        
            @OneToMany(mappedBy = "author", fetch = FetchType.EAGER)
            private List<Book> books = new ArrayList<>();
        }
        ```
        
        ```java
        Author author = entityManager.find(Author.class, 1L);
        // 이 시점에서 Author와 연관된 모든 Book 엔티티들이 데이터베이스에서 불러와짐
        ```
        
        - Author 엔티티와 Book 엔티티가 OneToMany 관계인 경우
        - 즉시 로딩을 사용하면, Author 엔티티를 조회할 때 그 Author와 연관된 모든 Book 엔티티들도 한 번에 불러옴
        <br>
- **지연 로딩**
    - 엔티티를 조회할 때 연관 데이터를 바로 불러오지 않고, 해당 데이터가 실제로 필요할 때(접근할 때) 불러오는 방식
    - 초기 로딩 시간이 짧아지지만, 연관 데이터를 실제로 접근할 때 추가로 데이터베이스에서 불러오는 작업이 발생
    - 예시
        
        ```java
        @Entity
        public class Author {
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
            private String name;
        
            @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
            private List<Book> books = new ArrayList<>();
        }
        ```
        
        ```java
        Author author = entityManager.find(Author.class, 1L);
        // 이 시점에서는 Author만 불러와짐
        
        List<Book> books = author.getBooks();
        // 이 시점에서야 관련된 Book 엔티티들이 데이터베이스에서 불러와짐
        ```
        
        - Author를 조회할 때는 Author 데이터만 불러오고, Book 리스트는 실제로 접근할 때 불러옴
        <br>
- **설정 방법**
    - `@OneToMany` : 기본적으로 지연 로딩이 적용
        - `@OneToMany(fetch = FetchType.LAZY)`
        - `fetch = FetchType.LAZY`가 지연 로딩, `fetch = FetchType.EAGER`가 즉시 로딩
    - `@ManyToOne` : 기본적으로 즉시 로딩이 적용되지만, 지연 로딩으로 변경할 수 있음
        - `@ManyToOne(fetch = FetchType.LAZY)`
    - `@OneToOne 및 @ManyToMany` : 기본적으로 지연 로딩이 적용
