### 👉 궁금해진 이유

---

- `Post post = postService.create(dto, user)` 로 post 생성
- `Post findPost = postService.findById(post.getId())`로 findPost 생성
- `post.toString()` 과 `findPost.toString()`으로 확인해보니 두 객체 주소가 같음
- ⇒ 새로 생성된 객체라면 값은 같더라도 주소는 달라야 하는데 어디선가 같은 값인지 확인하고, 같다면 이미 생성된 객체의 주소를 return 해주고 있다고 짐작하게 됨
<br>
<br>

### 👉 원인 : JPA 1차 캐시의 작동 원리

---

- JPA에서 제공하는 1차 캐시(영속성 컨텍스트)의 동작 방식 때문이라는 부분 확인
    - 영속성 컨텍스트는 같은 엔티티를 다시 조회할 때 데이터베이스에 재쿼리를 보내지 않고, 이미 로드된 엔티티가 있으면 그 객체를 반환
    - ⇒ 스프링에서 동일한 트랜잭션 내에서는 1차 캐시(영속성 컨텍스트)에 의해 동일한 엔티티 객체가 반환될 수 있음
    <br>
- 1차 캐시는 세션 단위로 작동하는데, 같은 세션 안에서 동일한 엔티티를 조회하거나 생성하면, 동일한 객체의 참조를 반환
    - 여기서 세션은 웹의 세션과 조금 다른 개념으로 데이터베이스와 상호작용하는 일종의 단일 연결
    - 세션은 데이터베이스와의 작업을 처리하는 동안 엔티티 객체를 관리하고, 데이터베이스의 변경사항을 추적하여 필요 시 자동으로 저장(Flush)하거나, 데이터베이스에서 데이터를 가져오는 역할
    - 기본적으로는 세션은 하나의 트랜잭션과 동일한 기간 동안 유지
        - 트랜잭션이 끝나면 세션도 종료되고, 이때 영속성 컨텍스트(즉, 1차 캐시)도 사라짐
        - ⇒ 하지만 항상 그런 것은 아니며, 트랜잭션이 종료된 후에도 세션이 완전히 종료되지 않을 수 있음
            - Spring의 세션 관리 방식에 따라 달라짐
            - 특히 Open Session in View 패턴(OSIV)이 활성화되어 있는 경우(아래에 내용 추가)
            <br>
- 1차 캐시 동작 방식
    - 엔티티 저장
        - postService.create()를 통해 새로운 Post 엔티티를 저장할 때, Hibernate는 해당 엔티티를 데이터베이스에 저장하는 것과 동시에 1차 캐시에 저장
    - 엔티티 조회
        - `postService.findById(id)`로 동일한 엔티티를 조회할 때, Hibernate는 먼저 1차 캐시에서 해당 엔티티가 있는지 확인하고, 있으면 데이터베이스를 조회하지 않고 영속성 컨텍스트에서 캐시된 객체를 반환
        - ⇒ `findById()` 호출 전에 영속성 컨텍스트를 비우면 동일한 객체를 불러오지 않음
        <br>
        <br>

### 👉 Hibernate(JPA 구현체)의 코드 확인

---

- findById가 정의 되어 있는 CrudRepository 인터페이스를 SimpleJpaRepository가 구현
    
    ```java
    // SimpleJpaRepository 일부
    public Optional<T> findById(ID id) {
        Assert.notNull(id, "The given id must not be null");
        Class<T> domainType = this.getDomainClass();
        if (this.metadata == null) {
            return Optional.ofNullable(this.entityManager.find(domainType, id));
        } else {
            LockModeType type = this.metadata.getLockModeType();
            Map<String, Object> hints = this.getHints();
            return Optional.ofNullable(type == null ? this.entityManager.find(domainType, id, hints) : this.entityManager.find(domainType, id, type, hints));
        }
    }
    ```
    <br>
    
- 위 로직에 있는 EntityManager 인터페이스를 Hibernate의 Session 인터페이스가 상속 받고, 이 Session 인터페이스를 SessionImpl 클래스가 구현
    
    ```java
    // SessionImpl 일부
    public <T> T find(Class<T> entityClass, Object primaryKey) {
        return this.find(entityClass, primaryKey, (LockModeType)null, (Map)null);
    }
    
    public <T> T find(Class<T> entityClass, Object primaryKey, LockModeType lockModeType, Map<String, Object> properties) {
        this.checkOpen();
        LockOptions lockOptions = null;
    
        String entityName;
        try {
    				// LockModeType은 enum으로 READ, WIRTE, OPTIMISTIC 등 락 모드가 선언 되어 있음
            if (lockModeType != null) {
                this.checkTransactionNeededForLock(lockModeType);
                lockOptions = this.buildLockOptions(lockModeType, properties);
            }
    
            EffectiveEntityGraph effectiveEntityGraph = this.loadQueryInfluencers.getEffectiveEntityGraph();
            effectiveEntityGraph.applyConfiguredGraph(properties);
            this.loadQueryInfluencers.setReadOnly(getReadOnlyHint(properties));
            if (effectiveEntityGraph.getSemantic() == GraphSemantic.FETCH) {
                this.setEnforcingFetchGraph(true);
            }
    
    				// 이 부분에서 캐시를 먼저 확인한 후, 해당 엔티티가 캐시에 존재하지 않으면 데이터베이스에서 로드
            Object var29 = this.byId(entityClass).with(this.determineAppropriateLocalCacheMode(properties)).with(lockOptions).load(primaryKey);
            return var29;
        } catch (EntityNotFoundException var17) {
            EntityNotFoundException enfe = var17;
            if (enfe instanceof FetchNotFoundException) {
                throw enfe;
            }
    
            if (log.isDebugEnabled()) {
                entityName = entityClass != null ? entityClass.getName() : null;
                String identifierValue = primaryKey != null ? primaryKey.toString() : null;
                log.ignoringEntityNotFound(entityName, identifierValue);
            }
    
            entityName = null;
            return entityName;
        } catch (ObjectDeletedException var18) {
            entityName = null;
        } catch (ObjectNotFoundException var19) {
            ObjectNotFoundException e = var19;
            throw new IllegalArgumentException(e.getMessage(), e);
        } catch (TypeMismatchException | ClassCastException | MappingException var20) {
            RuntimeException e = var20;
            throw this.getExceptionConverter().convert(new IllegalArgumentException(((RuntimeException)e).getMessage(), e));
        } catch (JDBCException var21) {
            JDBCException e = var21;
            if (this.accessTransaction().isActive() && this.accessTransaction().getRollbackOnly()) {
                if (log.isDebugEnabled()) {
                    log.debug("JDBCException was thrown for a transaction marked for rollback; this is probably due to an operation failing fast due to the transaction marked for rollback.", e);
                }
    
                entityName = null;
                return entityName;
            }
    
            throw this.getExceptionConverter().convert(e, lockOptions);
        } catch (RuntimeException var22) {
            RuntimeException e = var22;
            throw this.getExceptionConverter().convert(e, lockOptions);
        } finally {
            this.loadQueryInfluencers.getEffectiveEntityGraph().clear();
            this.loadQueryInfluencers.setReadOnly((Boolean)null);
            this.setEnforcingFetchGraph(false);
        }
    
        return entityName;
    }
    ```
    
    - 위에 주석으로 표기한 것처럼 `Object var29 = this.byId(entityClass).with(this.determineAppropriateLocalCacheMode(properties)).with(lockOptions).load(primaryKey);` 가 캐시된 객체를 반환하거나 객체를 새로 로드하는 부분
        - `this.byId(entityClass)` : 특정 엔티티 클래스를 식별자(PK)로 조회하기 위한 준비 단계
        - `determineAppropriateLocalCacheMode(properties)` : 특정 조회 요청에서 사용할 캐시 모드를 결정하는 메서드
        - `with(lockOptions)` : 엔티티를 로드할 때 동시성 제어를 위한 락 옵션 설정
        - `load(primaryKey)`  : 아래 코드와 같이 구현 되어 있어, primariKey에 대한 객체가 있으면 해당 객체를 반환하고 아니면 새로 로드
        <br>
- `load(primaryKey)` 부분 로직
    
    ```java
    // IdentifierLoadAccessImpl 일부
    public final T load(Object id) {
        return this.perform(() -> { // 캐시 관련 처리를 하는 곳
            return this.doLoad(id); // 최종적으로 이 메서드에서 객체 확인 후 처리
        });
    }
    
    protected final T doLoad(Object id) {
        SessionImplementor session = this.context.getSession();
        Object result = this.load(this.coerceId(id, session.getFactory()), session.asEventSource(), this.entityPersister.getEntityName(), this.isReadOnly(session));
        this.initializeIfNecessary(result);
        return result;
    }
    
    private Object load(Object id, EventSource eventSource, String entityName, Boolean readOnly) {
    		final LoadEvent event;
    		if ( lockOptions != null ) {
    			event = new LoadEvent( id, entityName, lockOptions, eventSource, readOnly );
    			context.fireLoad( event, LoadEventListener.GET );
    		}
    		else {
    			event = new LoadEvent( id, entityName, false, eventSource, readOnly );
    			boolean success = false;
    			try {
    				context.fireLoad( event, LoadEventListener.GET );
    				success = true;
    			}
    			catch (ObjectNotFoundException e) {
    				// 엔티티가 없는 경우 처리 (세션 캐시에 프록시가 있는데 실제 엔티티가 없을 때)
    			}
    			finally {
    				context.afterOperation( success );
    			}
    		}
    		return event.getResult();
    	}
    ```
    
    - `doLoad()`에서 `load()` 파라미터들의 역할
        - `this.coerceId(id, session.getFactory())` : 주어진 식별자 `id`를 올바른 형식으로 변환하는 메서드로, 데이터베이스의 식별자 타입과 일치하는 형식으로 변환
        - `session.asEventSource()` : 세션을 이벤트 소스(Event Source)로 사용하여 데이터베이스에 접근하고, 필요한 데이터를 로드
            
            ```java
            @Override
            public EventSource asEventSource() {
                return this;
            }
            ```
            
            - 이때, 세션은 엔티티 조회와 관련된 다양한 이벤트를 관리하며, 1차 캐시에서 먼저 엔티티를 확인한 후, 캐시에 없을 경우 데이터베이스에서 조회를 시도
            - `asEventSource()` 메서드는 단순히 현재 세션(SessionImple의 객체)을 반환하지만, 이 세션은 Hibernate에서 이벤트 관리를 위한 핵심적인 역할을 함
            - `SessionImpl` 클래스가 `EventSource` 인터페이스를 구현하고 있기 때문에, 세션 자체가 이벤트 소스 역할을 수행하고, 엔티티 로딩, 캐싱, 데이터베이스 접근 등과 관련된 이벤트를 처리할 수 있음
        - `this.entityPersister.getEntityName()` : 해당 엔티티의 이름을 가져오는 메서드
        - `this.isReadOnly(session)` : 세션이 읽기 전용 모드인지 여부를 확인하는 메서드
    - `load()`
        - `fireLoad()`  : Hibernate의 이벤트 시스템을 통해 LoadEvent를 발행
            - `fireLoad()`는 LoadEvent를 발생시키고, 해당 이벤트를 리스너가 처리할 수 있도록 전달
            - `LoadEventListener`는 `fireLoad()` 메서드에서 호출되는 인터페이스로, 이 메서드를 통해 캐시에서 엔티티를 먼저 조회하고, 없을 경우 데이터베이스에서 조회
            - 이벤트 리스너(`DefaultLoadEventListener`)의 아래 코드의 `doOnLoad()` 메서드를 통해 캐싱된 엔티티가 있는지 확인
            <br>
- 이벤트 리스너(`DefaultLoadEventListener`) 로직
    
    ```java
    // DefaultLoadEventListener 일부
    
    @Override
    public void onLoad(LoadEvent event, LoadType loadType) throws HibernateException {
        // 엔티티의 메타데이터(EntityPersister)를 가져옴
        final EntityPersister persister = getPersister(event);
        
        // 만약 퍼시스터가 없다면, 예외 던짐
        if (persister == null) {
            throw new HibernateException("Unable to locate persister: " + event.getEntityClassName());
        }
    
        // 엔티티의 식별자(ID)를 확인하는 로직을 실행
        checkId(event, loadType, persister);
        
        // 실제로 엔티티를 로드하는 메서드를 호출
        doOnLoad(persister, event, loadType);
    }
    
    private void doOnLoad(EntityPersister persister, LoadEvent event, LoadType loadType) {
        try {
            // 엔티티의 키(EntityKey)를 생성하여 캐시 및 데이터베이스 조회에 사용
            final EntityKey keyToLoad = event.getSession().generateEntityKey(event.getEntityId(), persister);
    
            if (loadType.isNakedEntityReturned()) {
                // 프록시 없이 실제 엔티티를 반환
                event.setResult(load(event, persister, keyToLoad, loadType));
            } else {
                // 프록시를 반환하거나, 락 모드에 따라 실제 엔티티를 반환
                final Object result = event.getLockMode() == LockMode.NONE
                        ? proxyOrLoad(event, persister, keyToLoad, loadType)
                        : lockAndLoad(event, persister, keyToLoad, loadType);
                event.setResult(result);
            }
        } catch (HibernateException e) {
            // 로드 실패 시 예외를 로깅하고 던짐
            LOG.unableToLoadCommand(e);
            throw e;
        }
    }
    
    private Object proxyOrLoad(LoadEvent event, EntityPersister persister, EntityKey keyToLoad, LoadType options) {
        // 트레이스 로깅 활성화 시, 엔티티 로드 로깅
        if (LOG.isTraceEnabled()) {
            LOG.tracev("Loading entity: {0}", infoString(persister, event.getEntityId(), persister.getFactory()));
        }
    
        // 바이트코드 프록시가 있으면 바이트코드 프록시로 로드(지연 로딩 프록시)
        if (hasBytecodeProxy(persister, options)) {
            return loadWithBytecodeProxy(event, persister, keyToLoad, options);
        } 
        // 일반 프록시가 있으면 프록시로 로드(지연 로딩 프록시)
        else if (persister.hasProxy()) {
            return loadWithRegularProxy(event, persister, keyToLoad, options);
        } 
        // 엔티티를 실제로 데이터베이스에서 가져오며, Hibernate의 지연 로딩 프록시를 사용하지 않고 바로 로드
        else {
            return load(event, persister, keyToLoad, options);
        }
    }
    
    private Object loadWithBytecodeProxy(LoadEvent event, EntityPersister persister, EntityKey keyToLoad, LoadType options) {
        // 세션과 영속성 컨텍스트를 가져옴
        final EventSource session = event.getSession();
        final PersistenceContext persistenceContext = session.getPersistenceContextInternal();
        
        // 이미 로드된 엔티티가 있으면 그것을 반환
        final EntityHolder holder = persistenceContext.getEntityHolder(keyToLoad);
        final Object managed = holder == null ? null : holder.getEntity();
    
        if (managed != null) {
            // 삭제 여부 확인 후, 삭제된 경우 null 반환
            return options.isCheckDeleted() && wasDeleted(persistenceContext, managed) ? null : managed;
        } 
        // HibernateProxy가 있는 경우, 프록시 팩토리를 통해 로드
        else if (persister.getRepresentationStrategy().getProxyFactory() != null) {
            return loadWithProxyFactory(event, persister, keyToLoad, holder);
        } 
        // 서브클래스가 있는 경우 데이터베이스에서 직접 로드
        else if (persister.hasSubclasses()) {
            return load(event, persister, keyToLoad, options);
        } 
        // 엔티티를 실제로 데이터베이스에서 가져오며, Hibernate의 지연 로딩 프록시를 사용하지 않고 바로 로드
        else {
            return createBatchLoadableEnhancedProxy(persister, keyToLoad, session);
        }
    }
    ```
    
    - `doOnLoad()`
        - `keyToLoad` : 엔티티를 식별하는 키(EntityKey)를 생성, 이 키는 캐시나 데이터베이스에서 해당 엔티티를 찾기 위한 고유 식별자
        - `isNakedEntityReturned()` : 이 조건은 프록시를 사용하지 않고 실제 엔티티를 반환할지 여부를 결정하며 만약 프록시 없이 직접 엔티티가 필요하다면 `load()` 메서드로 실제 엔티티를 로드
        - `proxyOrLoad()` : 프록시 객체를 사용할 수 있는 경우 프록시 객체를 반환하거나, 그렇지 않으면 실제 엔티티를 로드
        - `lockAndLoad()` : 특정 락 모드를 사용하는 경우, 데이터베이스에 락을 설정한 후 엔티티를 로드
        <br>
        <br>

### 👉 Open Session in View (OSIV) 패턴

---

- Spring Boot에서는 기본적으로 OSIV 패턴이 활성화되어 있음
- 이 패턴은 HTTP 요청이 처리되는 동안 Hibernate 세션을 열어 두는 방식
- 이는 조회 작업을 요청의 끝까지 처리할 수 있게 해 주지만, 동시에 트랜잭션이 종료된 후에도 세션이 유지되어 1차 캐시가 살아 있는 상태로 남아있게 됨
    - 일반적으로 성능 향상을 위해 사용되지만, 1차 캐시가 유지되는 동안 동일한 객체가 계속해서 반환된다는 특성을 갖고 있음
