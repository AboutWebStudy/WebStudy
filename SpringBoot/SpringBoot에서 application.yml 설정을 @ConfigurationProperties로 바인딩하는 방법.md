### 👉 공부하게 된 이유

---

- JWT 토큰 설정을 코드에서 관리하는 것보다 `application.yml`에 선언해서 관리하는 게 코드 유지보수나 보안적(깃허브 업로드 때 제외할 수 있음)으로도 좋을 것 같아서 적용하기 위해 알아보게 됨
<br>
<br>

### 👉 적용 방법

---

- Gradle에 의존성 추가
    
    ```java
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    ```
    
- application.yml에 필요한 값을 설정
    
    ```yaml
    jwt:
      header: Authorization
      secret: secretsecretsecretsecret123secretsecretsecretsecret123===
      access-validity-in-seconds: 500
      refresh-validity-in-seconds: 6048000
    ```
    
- `application.yml`의 값을 바인딩할 객체 생성하고 `@ConfigurationProperties` 어노테이션 추가
    
    ```java
    @Getter
    @Setter
    @ConfigurationProperties(prefix = "jwt")
    public class JwtProperties {
    
        private String header;
        private String secret;
        private long accessValidityInSeconds;
        private long refreshValidityInSeconds;
        private long authValidityInSeconds;
    }
    ```
    
    - `@Getter`는 해당 값들을 사용하기 위해 필요
    - `@Setter`는 Spring에서 `@ConfigurationProperties` 클래스를 사용할 때, setter 메서드로 값을 바인딩하는 방식이라 설정하지 않으면 값이 바인딩되지 않음(프로젝트 빌드 시 오류 발생)
- 실행 클래스에 `@ConfigurationPropertiesScan(basePackages = "com.example")` 어노테이션 추가
    
    ```java
    @SpringBootApplication
    @EnableJpaAuditing
    @ConfigurationPropertiesScan(basePackages = "com.example")
    public class TravelApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(TravelApplication.class, args);
    	}
    
    }
    ```
    
    - `@ConfigurationPropertiesScan`
        - `@ConfigurationProperties`로 선언된 클래스를 스캔해서 자동으로 빈으로 등록해주는 역할
        - `basePackages` 속성에 지정한 패키지 하위에서 `@ConfigurationProperties`가 붙은 클래스를 찾아 등록
        <br>
        <br>

### 👉 값이 바인딩되는 방식

---

- @ConfigurationProperties → POJO(자바 클래스)를 설정 바인딩 대상으로 지정
    
    ```java
    @Target({ElementType.TYPE, ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Indexed
    public @interface ConfigurationProperties {
        @AliasFor("prefix")
        String value() default "";
    
        @AliasFor("value")
        String prefix() default "";
    
        boolean ignoreInvalidFields() default false;
    
        boolean ignoreUnknownFields() default true;
    }
    ```
    
    - `@ConfigurationProperties`는 단순히 메타정보만 담고 있음
        - 말 그대로 ’얘는 어떤 설정값을 바인딩받는 클래스야’라는 표시
- Spring 내부에서 `ConfigurationPropertiesBindingPostProcessor`라는 빈 후처리기가 값을 바인딩
    - Spring이 애플리케이션 실행 중에 `@ConfigurationProperties` 어노테이션이 붙은 클래스를 스캔
    - 그런 다음, `@ConfigurationProperties`의 `prefix` 값을 기준으로 `application.yml`에서 매칭되는 값을 찾고, 그걸 해당 필드에 바인딩
    - 바인딩할 때는 리플렉션을 써서 자동으로 `setter` 호출하거나 생성자를 통해 값을 주입(생성자를 통해 넣으려면 별도 설정 등 필요)
- `@SpringBootApplication`이나 `@ConfigurationPropertiesScan` 같은 어노테이션을 통해 해당 클래스를 빈으로 등록하면, Spring이 실행 중에 `@ConfigurationProperties`가 붙은 클래스를 자동으로 감지해서 처리
<br>
<br>

### 👉 `ConfigurationPropertiesBindingPostProcessor` 내부

---

```java
public class ConfigurationPropertiesBindingPostProcessor implements BeanPostProcessor, PriorityOrdered, ApplicationContextAware, InitializingBean {
    public static final String BEAN_NAME = ConfigurationPropertiesBindingPostProcessor.class.getName();
    private ApplicationContext applicationContext;
    private BeanDefinitionRegistry registry;
    private ConfigurationPropertiesBinder binder;

    public ConfigurationPropertiesBindingPostProcessor() {
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public void afterPropertiesSet() throws Exception {
        this.registry = (BeanDefinitionRegistry)this.applicationContext.getAutowireCapableBeanFactory();
        this.binder = ConfigurationPropertiesBinder.get(this.applicationContext);
    }

    public int getOrder() {
        return -2147483647;
    }

    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (!this.hasBoundValueObject(beanName)) {
            this.bind(ConfigurationPropertiesBean.get(this.applicationContext, bean, beanName));
        }

        return bean;
    }

    private boolean hasBoundValueObject(String beanName) {
        return BindMethod.VALUE_OBJECT.equals(BindMethodAttribute.get(this.registry, beanName));
    }

    private void bind(ConfigurationPropertiesBean bean) {
        if (bean != null) {
            Assert.state(bean.asBindTarget().getBindMethod() != BindMethod.VALUE_OBJECT, "Cannot bind @ConfigurationProperties for bean '" + bean.getName() + "'. Ensure that @ConstructorBinding has not been applied to regular bean");

            try {
                this.binder.bind(bean);
            } catch (Exception var3) {
                Exception ex = var3;
                throw new ConfigurationPropertiesBindException(bean, ex);
            }
        }
    }

    public static void register(BeanDefinitionRegistry registry) {
        Assert.notNull(registry, "Registry must not be null");
        if (!registry.containsBeanDefinition(BEAN_NAME)) {
            BeanDefinition definition = BeanDefinitionBuilder.rootBeanDefinition(ConfigurationPropertiesBindingPostProcessor.class).getBeanDefinition();
            definition.setRole(2);
            registry.registerBeanDefinition(BEAN_NAME, definition);
        }

        ConfigurationPropertiesBinder.register(registry);
    }
}
```

- `setApplicationContext(ApplicationContext applicationContext)`
    - 스프링이 실행 중 이 클래스에 ApplicationContext를 주입
    - 이후 다른 작업들(빈 등록, 바인딩 등)에 이 ApplicationContext 사용
    <br>
- `afterPropertiesSet()`
    - 빈 생성이 완료되면 호출 ⇒ 바인딩할 준비를 마치는 단계
    - `registry` : 빈 정의를 등록/조회할 수 있는 객체(`BeanDefinitionRegistry`)
    - `binder` : 실제로 바인딩을 수행할 핵심 객체(`ConfigurationPropertiesBinder`)
    <br>
- `postProcessBeforeInitialization(Object bean, String beanName)`
    - Spring이 모든 빈을 초기화하기 전에 자동으로 호출
    - 해당 빈이 `@ConfigurationProperties`가 붙은 것인지 판단
    - 맞다면 설정파일(`application.yml`, `properties`)의 값을 해당 bean에 바인딩
    <br>
- `hasBoundValueObject(String beanName)`
    - 해당 bean이 이미 바인딩된 적 있는지 확인
    - `@ConstructorBinding`이 붙은 클래스인지 체크해서 중복 바인딩을 피하려는 목적도 있음
    <br>
- `bind(ConfigurationPropertiesBean bean)`
    - 여기서 진짜 YAML 파일을 Java 객체로 변환
    - `binder.bind(bean)` 호출이 핵심
        - 내부적으로 `Binder`라는 객체가 동작해서 `application.yml`의 값들을 가져오고, POJO의 setter에 주입(또는 생성자 기반이면 생성자로)
        <br>
- `register(BeanDefinitionRegistry registry)`
    - Spring이 이 `PostProcessor`를 Bean으로 등록하지 않았다면 자동 등록해주는 static 메서드
    - 즉, Spring Boot가 실행될 때 이걸 자동으로 등록해서 작동하게끔 보장
    <br>
- **📢 순서 정리**

    
    `@ConfigurationProperties`
  <br>
    ↓
  <br>
    Spring이 클래스 인식
  <br>
    ↓
  <br>
    `ConfigurationPropertiesBindingPostProcessor` 클래스
  <br>
    `postProcessBeforeInitialization()`
  <br>
    ↓
  <br>
    `binder.bind()` 호출
  <br>
    ↓
  <br>
    Binder가 `application.yml` 파싱 후
  <br>
    ↓
  <br>
    필드에 값 주입(setter 또는 생성자)
