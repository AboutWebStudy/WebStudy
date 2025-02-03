### 👉 작성했던 기본 Entity

---

```java
@Getter
@SuperBuilder
@MappedSuperclass
@NoArgsConstructor(access = PROTECTED)
@EntityListeners(AuditingEntityListener.class)
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class BaseEntity {

    @CreatedDate
    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "modified_at")
    private LocalDateTime modifiedAt;
}
```

- 생성된 일시를 저장하는 `created_at` 컬럼과 수정된 일시를 저장하는 `modified_at` 컬럼이 별도 코드를 작성하지 않아도 자동으로 세팅 되어 저장됨
- 해당 필드 위에 적은 `@CreatedDate`와 `@LastModifiedDate` 덕분에 자동 세팅 되는 것
<br>
<br>

### 👉 `@CreatedDate`와 `@LastModifiedDate` 동작 분석

---

Spring Data JPA의 Auditing 기능은 `@CreatedDate`와 `@LastModifiedDate`를 자동으로 감지하고, 엔티티의 생성 및 수정 시 해당 필드 값을 채움
<br>

1. **🔍 엔티티 감지 및 이벤트 트리거**
    
    ```java
    package org.springframework.data.jpa.domain.support;
    
    import jakarta.persistence.PrePersist;
    import jakarta.persistence.PreUpdate;
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.runtime.reflect.Factory;
    import org.springframework.beans.factory.ObjectFactory;
    import org.springframework.beans.factory.annotation.Configurable;
    import org.springframework.beans.factory.aspectj.AbstractDependencyInjectionAspect;
    import org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect;
    import org.springframework.beans.factory.aspectj.ConfigurableObject;
    import org.springframework.data.auditing.AuditingHandler;
    import org.springframework.lang.Nullable;
    import org.springframework.util.Assert;
    
    @Configurable
    public class AuditingEntityListener implements ConfigurableObject {
    
        // ... 중략
    
        @PrePersist
        public void touchForCreate(Object target) {
            Assert.notNull(target, "Entity must not be null");
            if (this.handler != null) {
                AuditingHandler object = (AuditingHandler)this.handler.getObject();
                if (object != null) {
                    object.markCreated(target); // AuditingHandler의 markCreated() 호출
                }
            }
    
        }
    
        @PreUpdate
        public void touchForUpdate(Object target) {
            Assert.notNull(target, "Entity must not be null");
            if (this.handler != null) {
                AuditingHandler object = (AuditingHandler)this.handler.getObject();
                if (object != null) {
                    object.markModified(target); // AuditingHandler의 markModified() 호출
                }
            }
    
        }
    }
    ```
    
    - `AuditingEntityListener` 클래스는 JPA 이벤트 리스너
        - `AuditingEntityListener`는 JPA 엔티티의 변경 사항을 감지하고, `@CreatedDate` 및 `@LastModifiedDate`를 자동으로 설정하는 역할 등을 함
    - 엔티티가 persist 또는 update 될 때(`@PrePersist`, `@PreUpdate`) JPA 이벤트 리스너가 동작하여 `@CreatedDate`와 `@LastModifiedDate`를 처리
        - persist : 엔티티를 영속성 컨텍스트(Persistence Context)에 등록하고, 이후 트랜잭션이 커밋될 때 데이터베이스에 저장되도록 하는 JPA 메서드
    - 동작 순서
        - 엔티티가 처음 저장될 때 → `@PrePersist`가 호출되고 `touchForCreate()` 실행 → `AuditingHandler`의 `markCreated()` 실행
        - 엔티티가 수정될 때 → `@PreUpdate`가 호출되고 `touchForUpdate()` 실행 →  `AuditingHandler`의 `markModified()` 실행
        - `AuditingHandler`를 이용하여 해당 엔티티의 `@CreatedDate` 및 `@LastModifiedDate` 값을 설정
        <br>

1. **🔍 `AuditingHandler`를 통한 필드 값 설정**
    
    ```java
    package org.springframework.data.auditing;
    
    import java.util.Optional;
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.springframework.beans.factory.InitializingBean;
    import org.springframework.data.domain.AuditorAware;
    import org.springframework.data.mapping.context.MappingContext;
    import org.springframework.data.mapping.context.PersistentEntities;
    import org.springframework.util.Assert;
    
    public class AuditingHandler extends AuditingHandlerSupport implements InitializingBean {
    
    		// ... 중략
    
        public <T> T markCreated(T source) {
            Assert.notNull(source, "Entity must not be null");
            return this.markCreated(this.getAuditor(), source);
        }
    
        public <T> T markModified(T source) {
            Assert.notNull(source, "Entity must not be null");
            return this.markModified(this.getAuditor(), source);
        }
    
        Auditor<?> getAuditor() {
            return (Auditor)this.auditorAware.map(AuditorAware::getCurrentAuditor)
    	        .map(Auditor::ofOptional)
    	        .orElse(Auditor.none());
        }
    }
    ```
    
    - 동작 순서
        - `markCreated(target)` 실행 시 → 내부적으로 `AuditingHandlerSupport`의 `markCreated(getAuditor(), target)` 호출
        - `markModified(target)` 실행 시 → 내부적으로 `AuditingHandlerSupport`의  `markModified(getAuditor(), target)` 호출
        - `getAuditor()` 호출을 통해 현재 사용자 정보를 가져옴 (`AuditorAware` 사용)
            - 사용자 정보를 가져오는 이유는 ‘누가 생성했는지(`@CreatedBy`), 누가 수정했는지(`@LastModifiedBy`)’ 를 추적하고 필요하면 해당 필드에 저장하기 위해
        - `markCreated()` 및 `markModified()` 내부적으로 `touch()` 메서드를 실행하여 엔티티 필드 값을 설정
        <br>

1. **🔍 `@CreatedDate` 및 `@LastModifiedDate` 필드 값 설정**
    
    ```java
    package org.springframework.data.auditing;
    
    import java.time.temporal.TemporalAccessor;
    import java.util.Objects;
    import java.util.Optional;
    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.springframework.core.log.LogMessage;
    import org.springframework.data.mapping.context.PersistentEntities;
    import org.springframework.lang.Nullable;
    import org.springframework.util.Assert;
    
    public abstract class AuditingHandlerSupport {
    
    		// ... 중략
    
    		// touch() 호출
        <T> T markCreated(Auditor<?> auditor, T source) {
            Assert.notNull(source, "Source entity must not be null");
            return this.touch(auditor, source, true);
        }
        
    		// touch() 호출
        <T> T markModified(Auditor<?> auditor, T source) {
            Assert.notNull(source, "Source entity must not be null");
            return this.touch(auditor, source, false);
        }
    
        private <T> T touch(Auditor<?> auditor, T target, boolean isNew) {
            Optional<AuditableBeanWrapper<T>> wrapper = this.factory.getBeanWrapperFor(target);
            return wrapper.map((it) -> {
                this.touchAuditor(auditor, it, isNew);
                Optional<TemporalAccessor> now = this.dateTimeForNow ? this.touchDate(it, isNew) : Optional.empty();
                if (logger.isDebugEnabled()) {
                    Object defaultedNow = now.map(Object::toString).orElse("not set");
                    Object defaultedAuditor = auditor.isPresent() ? auditor.toString() : "unknown";
                    logger.debug(LogMessage.format("Touched %s - Last modification at %s by %s", target, defaultedNow, defaultedAuditor));
                }
    
                return it.getBean();
            }).orElse(target);
        }
    
        private Optional<TemporalAccessor> touchDate(AuditableBeanWrapper<?> wrapper, boolean isNew) {
            Assert.notNull(wrapper, "AuditableBeanWrapper must not be null");
            Optional<TemporalAccessor> now = this.dateTimeProvider.getNow();
            Assert.notNull(now, () -> {
                return String.format("Now must not be null Returned by: %s", this.dateTimeProvider.getClass());
            });
            Optional var10000 = now.filter((__) -> {
                return isNew;
            });
            Objects.requireNonNull(wrapper);
            var10000.ifPresent(wrapper::setCreatedDate);
            var10000 = now.filter((__) -> {
                return !isNew || this.modifyOnCreation;
            });
            Objects.requireNonNull(wrapper);
            var10000.ifPresent(wrapper::setLastModifiedDate);
            return now;
        }
    }
    ```
    
    - 동작 순서
        - `touch()` 실행 시 `touchDate()` 호출
        - `CurrentDateTimeProvider.INSTANCE.getNow()`를 통해 현재 시간 가져옴
        - 엔티티가 새로 생성된 경우 → `setCreatedDate()`를 호출하여 `@CreatedDate` 값 설정
        - 엔티티가 수정된 경우 → `setLastModifiedDate()`를 호출하여 `@LastModifiedDate` 값 설정
        <br>
        <br>

### 👉 `@CreatedDate`와 `@LastModifiedDate` 동작 요약

---

- **`@CreatedDate` : 엔티티가 `persist`될 때 (`@PrePersist`) ⇒ 엔티티가 처음 저장될 때 한 번만 설정됨**
    1. `AuditingEntityListener.touchForCreate()` 실행
    2. `AuditingHandler.markCreated(target)` 실행
    3. `AuditingHandlerSupport.touch()` 실행
    4. `touchDate()`를 통해 `createdAt`과 `modifiedAt`에 현재 시간 저장
    <br>

- **`@LastModifiedDate` : 엔티티가 `update`될 때 (`@PreUpdate`) ⇒ 엔티티가 변경될 때마다 업데이트됨**
    1. `AuditingEntityListener.touchForUpdate()` 실행
    2. `AuditingHandler.markModified(target)` 실행
    3. `AuditingHandlerSupport.touch()` 실행
    4. `touchDate()`를 통해 `modifiedAt`을 현재 시간으로 업데이트
