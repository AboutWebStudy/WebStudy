### 👉 스프링부트의 이벤트 기반 프로그래밍(이벤트 리스너)

---

- 스프링 프레임워크에서는 이벤트 기반 프로그래밍을 지원
    - 애플리케이션 내에서 특정 이벤트가 발생했을 때, 해당 이벤트를 비동기적으로 처리하거나 후속 작업을 분리하여 처리할 수 있음
- 이벤트 리스너를 사용하여 메인 비즈니스 로직에서 부수적인 처리를 분리
    - 이벤트 리스너는 메인 프로세스와 독립적으로 실행할 수 있음 ⇒ 비동기 처리
    - 서비스 간의 결합도가 낮아짐
        - `UserService`에서 사용자를 생성할 때 `NotificationService`를 직접 호출하는 대신, 이벤트를 발행하고 NotificationListener에서 해당 이벤트를 처리하도록 함
- 사용 사례
    - 게시글에 댓글이 달리면 알림 보내기
    - 로그 기록하기
    - 이메일 전송
- 적용 예시
    - 이벤트 클래스 생성
        
        ```java
        @Getter
        @ArgsConstructor
        public class PostCreatedEvent {
            private final Long postId;
            private final Long userId;
        }
        ```
        
    - 서비스에서 이벤트 발행
        
        ```java
        @Service
        public class PostService {
            private final ApplicationEventPublisher eventPublisher;
        
            @Transactional
            public void createPost(Post post) {
                // 게시글 저장
                postRepository.save(post);
        
                // 게시글 작성 이벤트 발행
                eventPublisher.publishEvent(new PostCreatedEvent(post.getId(), post.getUserId()));
            }
        }
        ```
        
    - 이벤트 리스너 구현(알림 전송)
        
        ```java
        
        @Component
        public class NotificationEventListener {
        
            @TransactionalEventListener
            public void onPostCreated(PostCreatedEvent event) {
                System.out.println("Sending notification for post: " + event.getPostId());
                // 알림 전송 로직 (예: 이메일, 푸시 알림 등)
                notificationService.create(post.getId(), post.getUserId()));
            }
        }
        ```
        
    - 동작 방식
        - 사용자가 게시글을 작성하면 `PostService`에서 이벤트가 발행됨
        - `NotificationEventListener`는 트랜잭션이 성공적으로 커밋된 후(AFTER_COMMIT) 해당 이벤트를 수신하여 알림을 전송
            - ⇒ `@TransactionalEventListener`의 기본 설정은 AFTER_COMMIT으로 커밋이 정상적으로 실행 되어야 이벤트가 발행 됨
        - 위 같은 방식으로 인해 게시글 작성 작업과 알림 전송 작업이 분리되며, 만약 트랜잭션이 롤백될 경우 알림 전송도 발생하지 않음
        <br>
        <br>

### 👉 @TransactionalEventListener와 @EventListener

---

- **@TransactionalEventListener**
    - 트랜잭션의 상태에 따라 이벤트를 처리할 수 있도록 함
        - 트랜잭션이 성공했을 때만 후속 작업을 실행해야 하는 경우
    - 이벤트가 발생한 시점이 아닌, 트랜잭션이 특정 상태일 때 이벤트 리스너가 동작하도록 설정할 수 있음
    - 설정 가능한 속성(`phase` : 트랜잭션의 상태에 따라 이벤트 리스너가 호출되는 시점)
        - `AFTER_COMMIT (기본값)` : 트랜잭션이 커밋된 후에 이벤트가 실행
            - `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)`
        - `AFTER_ROLLBACK` : 트랜잭션이 롤백된 후에 이벤트가 실행
            - 예를 들면, 결제 중 오류가 걸려 롤백 되었는데 그 과정에서 따로 DB에 저장한 게 있다면 이벤트 삭제 진행 등
        - `AFTER_COMPLETION` : 트랜잭션이 완료된 후(성공 또는 실패)에 이벤트가 실행
        - `BEFORE_COMMIT` : 트랜잭션이 커밋되기 직전에 이벤트가 실행
        <br>
- **@EventListener**
    - 이벤트가 발생하면 즉시 리스너 메서드가 호출
    - 트랜잭션과 관계없이 즉시 이벤트를 처리하고 싶을 때 사용
    - 기본적으로 동기적으로 작동하며, 이벤트 발생 시 이벤트 핸들러 메서드가 블록(block)되어 실행
        - 필요에 따라 @Async를 사용하여 비동기적으로 실행할 수 있음
        <br>
- **비교 정리**
    
    
    | 특성 | @EventListener | @TransactionalEventListener |
    | --- | --- | --- |
    | 트랜잭션 의존성 | 트랜잭션과 무관하게 즉시 실행 | 트랜잭션 상태에 따라 실행 시점 제어 가능 |
    | 기본 실행 시점 | 이벤트 발생 시 바로 실행 | 트랜잭션이 커밋된 후(`AFTER_COMMIT`) |
    | 비동기 처리 지원 | `@Async`와 함께 사용 가능 | 주로 트랜잭션 종료 후 처리하므로 비동기로 동작 가능 |
    | 주 사용 사례 | 비즈니스 로직 외의 일반 이벤트 | 데이터 일관성 보장 필요 시, 트랜잭션 후처리 시 사용 |
    <br>
    <br>

### 👉 @TransactionalEventListener 적용 후 알림이 저장되지 않는 문제 발생

---

- **오류 내용**
    - 게시글에 댓글을 작성 하면 NotificationService의 create 메서드로 인해 Notification 테이블에 내용이 저장되어야 함
    - 실제 테스트 했을 때, 게시글의 댓글은 저장 되는데 Notification에는 저장 되지 않음
    <br>
- **작성 코드**
    
    ```java
    @Component
    @RequiredArgsConstructor
    @Slf4j
    public class NotificationEventListener {
    
        private final NotificationService notificationService;
    
        @TransactionalEventListener
        public void handlePostCommentEvent(PostCommentEvent event) {
            notificationService.create(event);
        }
    
        @TransactionalEventListener
        public void handlePostLikeEvent(PostLikeEvent event) {
            notificationService.create(event);
        }
    
        @TransactionalEventListener
        public void handleReviewScoreEvent(ReviewScoreEvent event) {
            notificationService.create(event);
        }
    }
    ```
    
    ```java
    public class NotificationService {
    
        private final NotificationRepository notificationRepository;
        private final ReviewService reviewService;
        private final PostService postService;
    
        /**
         * 알림 생성
         */
        @Transactional
        public void create(NotificationEvent event) {
            User user = findUserByTargetIdAndType(event.getTargetId(), event.getNotificationType());
    
            notificationRepository.save(event.toEntity(user));
        }
        
        // ... 중략
    }
    ```
    
    ```java
    @Slf4j
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class PostCommentService {
    
        private final UserService userService;
        private final PostService postService;
        private final PostCommentRepository postCommentRepository;
        private final ApplicationEventPublisher eventPublisher;
    
        /**
         * 댓글 생성
         */
        @Transactional
        public PostComment create(UUID postId, CreatePostCommentReq dto, AuthenticatedUserReq auth) {
            User user = userService.findByEmail(auth.getEmail());
            Post post = postService.findByIdNotDeletedPost(postId);
            
            PostComment postComment = postCommentRepository.save(CreatePostCommentReq.toEntity(dto, user, post));
            
            eventPublisher.publishEvent(new PostCommentEvent(post.getId()));
    
            return postComment;
        }
    }
    ```
    
    ```java
    @Builder
    @AllArgsConstructor
    public class PostCommentEvent implements NotificationEvent {
        private final UUID postId;
    
        @Override
        public UUID getTargetId() {
            return postId;
        }
    
        @Override
        public NotificationType getNotificationType() {
            return NotificationType.POST_COMMENT;
        }
    
        @Override
        public Notification toEntity(User user) {
            return Notification.builder()
                    .notificationType(NotificationType.POST_COMMENT)
                    .targetId(postId)
                    .isRead(false)
                    .user(user)
                    .build();
        }
    }
    ```
    <br>
    <br>
    

### 👉 오류 원인 및 해결

---

- **오류 원인**
    - `PostCommentService`의 create에 `@Transactional`이 설정 되어 있고, NotificationService의 create에도 `@Transactional`이 설정 되어 있음
        - @Transactional이 적용된 메서드는 현재 실행 중인 트랜잭션이 존재하면 그 트랜잭션을 재사용함
    - 위의 이유로 이벤트 핸들러가 실행될 때, 기존 트랜잭션을 재사용
        - 새로운 트랜잭션을 생성하지 않고, 원래의 트랜잭션을 그대로 사용
    - 이벤트 핸들러는 트랜잭션이 커밋된 후에 실행(`TransactionPhase.AFTER_COMMIT`)
    - 기존 트랜잭션은 이미 커밋이 됐기 때문에 추가적으로 커밋이 발생되지 않음
    - 결과적으로, `notificationRepository.save()`가 호출되더라도, 커밋이 일어나지 않아 알림이 DB에 저장되지 않음
        - `@TransactionalEventListener`는 커밋이 끝나면 실행 되기 때문
        <br>
- **해결 방법**
    - `NotificationService.create()`의 `@Transactional`에 `(propagation = Propagation.REQUIRES_NEW)`를 설정해서 새로운 트랜잭션을 생성
        - REQUIRES_NEW는 현재 트랜잭션을 일시 중지하고, 새로운 독립적인 트랜잭션을 시작함
        - 새로운 트랜잭션 내에서 notificationRepository.save()가 호출되면, 해당 트랜잭션에서 독립적으로 커밋됨
        - 따라서 원래 트랜잭션이 성공적으로 커밋되든, 롤백되든 관계없이 알림은 별도의 트랜잭션에서 안전하게 저장됨
        <br>
- **수정한 코드**
    
    ```java
    public class NotificationService {
    
        private final NotificationRepository notificationRepository;
        private final ReviewService reviewService;
        private final PostService postService;
    
        /**
         * 알림 생성
         */
        @Transactional(propagation = Propagation.REQUIRES_NEW)
        public void create(NotificationEvent event) {
            User user = findUserByTargetIdAndType(event.getTargetId(), event.getNotificationType());
    
            notificationRepository.save(event.toEntity(user));
        }
        
        // ... 중략
    }
    ```
    
    - 트랜잭션 옵션을 준 이후에는 Notification이 정상적으로 잘 저장됨
