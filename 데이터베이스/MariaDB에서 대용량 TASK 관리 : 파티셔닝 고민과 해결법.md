### 👉 파티셔닝을 고민한 이유

---

```sql
CREATE TABLE `USER` (
	`user_id`	        VARCHAR(36)	  NOT NULL,
	`email`	          VARCHAR(255)  NOT NULL,
	`password`	      VARCHAR(255)  NOT NULL,
	`password_count`  INT           NOT NULL	DEFAULT 0,
	`role`	          VARCHAR(10)   NOT NULL	DEFAULT 'USER',
	`state`	          VARCHAR(10)   NOT NULL	DEFAULT 'ACTIVE',
	`social`	        VARCHAR(10)   NULL        DEFAULT NULL,
	`created_at`	    TIMESTAMP	    NOT NULL,
	`modified_at`	    TIMESTAMP	    NOT NULL,
	`deleted_at`      TIMESTAMP	    NULL        DEFAULT NULL,
	`is_deleted`      BOOLEAN	      NOT NULL	DEFAULT FALSE,
    PRIMARY KEY(`user_id`)
);

CREATE TABLE `TASK` (
	`task_id`          VARCHAR(36)  NOT NULL,
	`task_title`       TEXT         NOT NULL,
	`task_detail`      TEXT         NULL,
	`task_status`      VARCHAR(10)  NOT NULL  DEFAULT 'TODO',
	`task_date`        DATE         NOT NULL,
	`task_time`        TIMESTAMP    NULL,
	`task_order_index` INT          NOT NULL,
	`created_at`       TIMESTAMP    NOT NULL,
	`modified_at`      TIMESTAMP    NOT NULL,
	`user_id`	       VARCHAR(36)  NOT NULL,
    PRIMARY KEY(`task_id`),
    FOREIGN KEY (`user_id`) REFERENCES `USER`(`user_id`)
);

CREATE TABLE `CHECKLIST` (
	`checklist_id`    VARCHAR(36)   NOT NULL,
	`checklist_text`  VARCHAR(500)  NOT NULL,
	`is_checked`      BOOLEAN       NOT NULL	DEFAULT FALSE,
	`created_at`      TIMESTAMP     NOT NULL,
	`modified_at`     TIMESTAMP     NULL,
	`task_id`         VARCHAR(36)   NOT NULL,
    PRIMARY KEY(`checklist_id`),
    FOREIGN KEY (`task_id`) REFERENCES `TASK`(`task_id`)
);
```

- TO DO 서비스를 개발하기 위해 위처럼 테이블을 설계
- TASK 테이블에 모든 USER의 TASK가 저장되고, 반복되는 TASK들도 새 행(Row)로 저장될 테니 데이터가 방대해질 것으로 예상(사용자가 있다는 가정 하에)
- 데이터가 너무 많으면 조회 등에 부하가 걸릴 것으로 예상하고 이를 최소화 하기 위해 파티셔닝을 하기로 결정(수평적 파티셔닝)
    - TASK 같은 경우는 구조가 다른 USER의 TASK를 볼 일이 없으니 `user_id`로 파티셔닝 하면 부하가 최소화 되고, 그로 인해 `user_id`로 TASK를 조회하는 쿼리의 성능이 향상 될 것으로 예상
    <br>
    <br>

### 👉 파티셔닝이란?

---

- 큰 테이블을 논리적으로 작은 단위로 나누어 저장하는 것
- 데이터가 방대해질 경우 쿼리 성능을 개선하고 관리 효율성을 높이기 위해 사용
- 파티셔닝에는 수평적 파티셔닝과 수직적 파티셔닝이 있음
	<br>
    - **수평적 파티셔닝(Horizontal Partitioning)**
        - 데이터를 행(Row) 단위로 나누는 방식
        - 테이블의 컬럼 구조는 그대로 유지되지만, 특정 행들이 서로 다른 파티션에 저장됨
        - 주로 데이터 크기가 너무 커져서 성능 저하를 방지할 때 사용
        - 예를 들어, `user_id`를 기준으로 6개의 파티션으로 파티셔닝 한다면, `user_id` 값을 해싱하여 6개의 파티션으로 분산 저장
            - 같은 `user_id` 값은 항상 동일한 파티션에 저장됨
            - 수평적 파티셔닝 데이터 분배 방식 예시
                
                
                | Partition | 데이터 (task_id, user_id) |
                | --- | --- |
                | `p1` | (`task_1`, `user_001`), (`task_5`, `user_014`) |
                | `p2` | (`task_2`, `user_002`), (`task_6`, `user_022`) |
                | `p3` | (`task_3`, `user_003`), (`task_7`, `user_032`) |
                | `p4` | (`task_4`, `user_004`), (`task_8`, `user_041`) |
                | `p5` | (`task_9`, `user_005`), (`task_11`, `user_056`) |
                | `p6`  | (`task_10`, `user_006`), (`task_12`, `user_058`) |
                <br>
    - **수직적 파티셔닝**
        - 데이터를 컬럼(Column) 단위로 나누는 방식
        - 테이블을 여러 개로 분리하여, 자주 사용되는 컬럼과 그렇지 않은 컬럼을 분리하는 방식
        - 주로 테이블이 너무 넓어(컬럼이 많아)지고, 특정 쿼리의 성능이 저하될 때 사용
        <br>
        <br>

### 👉 수평적 파티셔닝 종류

---

- **파티셔닝의 종류**
    - **1️⃣ HASH 파티셔닝**
        - MOD(N) 연산을 사용하여 데이터를 균등하게 분산하는 방식
        - INT 타입(정수형) 컬럼만 사용 가능
            
            ```sql
            CREATE TABLE SALES (
                sale_id INT NOT NULL,
                amount DECIMAL(10,2) NOT NULL,
                store_id INT NOT NULL,
                PRIMARY KEY (sale_id, store_id)
            ) PARTITION BY HASH(store_id) PARTITIONS 4;
            ```
            
            - `store_id` 값을 해싱하여 4개의 파티션 중 하나에 저장
            - `store_id % 4` 연산을 사용하여 데이터를 분산
        - VARCHAR 같은 형태는 HASH 파티셔닝이 불가 ⇒ KEY 파티셔닝 사용
            - 내부적으로 해시(Hash) 값을 생성하여 자동 분배하기 때문에 VARCHAR 같은 문자열 데이터도 사용할 수 있음
                
                ```sql
                CREATE TABLE USERS (
                    user_id VARCHAR(36) NOT NULL,  -- UUID 또는 사용자 ID
                    username VARCHAR(255) NOT NULL,
                    email VARCHAR(255) NOT NULL,
                    PRIMARY KEY (user_id)
                )
                PARTITION BY KEY(user_id) PARTITIONS 6;
                ```
                
                - `user_id`를 내부적으로 해시 변환하여 6개의 파티션으로 분배
                - 특정 파티션에 데이터가 몰릴 가능성이 있다는 단점이 이있음
    - **2️⃣ LIST 파티셔닝**
        - 미리 지정된 특정 값(리스트)을 기준으로 데이터를 분배하는 방식
        - 주어진 컬럼 값이 특정 리스트에 포함되는지 여부에 따라 데이터를 각 파티션에 저장
        - 카테고리성 데이터나 특정 그룹을 나눌 때 유용
        - WHERE 조건에 특정 카테고리나 그룹으로 조회할 때 성능 향상
            
            ```sql
            CREATE TABLE ORDERS (
                order_id INT NOT NULL AUTO_INCREMENT,
                customer_id INT NOT NULL,
                order_status VARCHAR(20) NOT NULL,
                order_date DATETIME NOT NULL,
                total_amount DECIMAL(10,2) NOT NULL,
                PRIMARY KEY (order_id, order_status)
            )
            PARTITION BY LIST COLUMNS(order_status) (
                PARTITION p_pending VALUES IN ('PENDING'),     -- 대기
                PARTITION p_shipped VALUES IN ('SHIPPED'),     -- 배송 중
                PARTITION p_delivered VALUES IN ('DELIVERED'), -- 배송 완료
                PARTITION p_cancelled VALUES IN ('CANCELLED')  -- 취소됨
            );
            ```
            
            - 새로운 상태가 추가될 경우(예시 : RETURNED(반품)), 새로운 파티션을 추가해야 함
    - **3️⃣ RANGE 파티셔닝**
        - 숫자 또는 날짜의 ‘범위’를 기준으로 데이터를 분배하는 방식
        - LIST와 달리, 특정 값이 아니라 범위를 기준으로 데이터가 들어갈 파티션을 결정
        - 시간 또는 숫자 데이터가 많을 때 적합
        - WHERE 조건에서 특정 범위를 검색할 때 성능 향상
            
            ```sql
            CREATE TABLE ORDERS (
                order_id INT NOT NULL AUTO_INCREMENT,
                customer_id INT NOT NULL,
                order_date DATE NOT NULL,
                total_amount DECIMAL(10,2) NOT NULL,
                PRIMARY KEY (order_id, order_date)
            )
            PARTITION BY RANGE(TO_DAYS(order_date)) (
                PARTITION p2022 VALUES LESS THAN (TO_DAYS('2023-01-01')), -- 2022년 데이터
                PARTITION p2023 VALUES LESS THAN (TO_DAYS('2024-01-01')), -- 2023년 데이터
                PARTITION p2024 VALUES LESS THAN (TO_DAYS('2025-01-01')), -- 2024년 데이터
                PARTITION pmax VALUES LESS THAN MAXVALUE -- 나머지 모든 데이터
            );
            
            ```
            

- **적절한 파티션 개수 계산 방법**
    - **1️⃣ HASH 파티셔닝**
        - `파티션 개수 = min(서버 CPU 코어 수 × 2, 예상 행 개수 / 100만)`
            - 파티션 개수를 ‘CPU 코어 수’ 또는 ‘데이터 개수’ 기반으로 계산
                - 보통 1개 파티션당 100만 개 데이터를 초과하지 않는 것이 좋음
            - 고르게 데이터를 분산하는 것이 목표
            - 해시 충돌을 최소화하면서도, 너무 많은 파티션이 생기지 않도록 설정해야 함
    - **2️⃣ LIST 파티셔닝**
        - `파티션 개수 = min(카테고리 개수, 100)`
            - LIST 파티셔닝에서는 ‘카테고리 개수’에 맞게 파티션을 설정
            - 각 파티션이 특정 값(목록)에 해당하는 데이터를 포함
            - 너무 적으면 쿼리 속도가 저하되고, 너무 많으면 관리 오버헤드 발생
    - **3️⃣ RANGE 파티셔닝**
        - `파티션 개수 = (총 데이터 개수 / 100만) × (1년당 예상 파티션 개수)`
            - RANGE 파티셔닝에서는 ‘시간(날짜) 범위’를 기준으로 파티션을 나눔
                - 연도별(1년당 1개 파티션) → 약 10년치 데이터라면 10개 파티션
                - 월별(1년당 12개 파티션) → 10년치 데이터라면 120개 파티션
                - 일별(1년당 365개) → 너무 많아질 수 있음 (조정 필요)
            - 연도별, 월별, 주별 등 적절한 범위를 선택해야 함
            - 너무 많은 파티션을 생성하면 성능이 저하될 수 있음
            <br>
            <br>

### 👉 적용할 때 발생한 문제

---

- `SQL Error [1506] [HY000]: (conn=23) Partitioned tables do not support FOREIGN KEY`
    
    ```sql
    CREATE TABLE `TASK` (
    	`task_id`          VARCHAR(36)  NOT NULL,
    	`task_title`       TEXT         NOT NULL,
    	`task_detail`      TEXT         NULL,
    	`task_status`      VARCHAR(10)  NOT NULL  DEFAULT 'TODO',
    	`task_date`        DATE         NOT NULL,
    	`task_time`        TIMESTAMP    NULL,
    	`task_order_index` INT          NOT NULL,
    	`created_at`       TIMESTAMP    NOT NULL,
    	`modified_at`      TIMESTAMP    NOT NULL,
    	`user_id`	         VARCHAR(36)  NOT NULL,
        PRIMARY KEY(`task_id`),
        FOREIGN KEY (`user_id`) REFERENCES `USER`(`user_id`)
    ) PARTITION BY KEY(user_id) PARTITIONS 6; -- CPU 코어가 4개(최대 파티션 8개 권장)이고 예상 데이터를 고려해서 6으로 설정
    ```
    
    - MariaDB에서는 파티션된 테이블이 FK를 지원하지 않음
    - 이를 해결하기 위해 FK를 제거
    <br>
- `SQL Error [1503] [HY000]: (conn=23) A PRIMARY KEY must include all columns in the table's partitioning function`
    
    ```sql
    CREATE TABLE `TASK` (
    	`task_id`          VARCHAR(36)  NOT NULL,
    	`task_title`       TEXT         NOT NULL,
    	`task_detail`      TEXT         NULL,
    	`task_status`      VARCHAR(10)  NOT NULL  DEFAULT 'TODO',
    	`task_date`        DATE         NOT NULL,
    	`task_time`        TIMESTAMP    NULL,
    	`task_order_index` INT          NOT NULL,
    	`created_at`       TIMESTAMP    NOT NULL,
    	`modified_at`      TIMESTAMP    NOT NULL,
    	`user_id`	         VARCHAR(36)  NOT NULL,
        PRIMARY KEY(`task_id`),
        INDEX TASK_1 (user_id, task_date)
    ) PARTITION BY KEY(user_id) PARTITIONS 6;
    ```
    
    - `PARTITION BY KEY(컬럼명)`을 사용할 경우, 해당 컬럼이 PK에 포함되어야 함
    - `task_id`가 단독 PK인데, 파티션 키는 `user_id`라서 MariaDB가 일관성을 유지할 방법이 없기 때문에 오류 발생
    - 이를 해결하기 위해 `user_id`를 PK에 추가
    <br>
    <br>

### 👉 해결 방법

---

```sql
CREATE TABLE `USER` (
	`user_id`	        VARCHAR(36)	  NOT NULL,
	`email`	          VARCHAR(255)  NOT NULL,
	`password`	      VARCHAR(255)  NOT NULL,
	`password_count`  INT           NOT NULL	DEFAULT 0,
	`role`	          VARCHAR(10)   NOT NULL	DEFAULT 'USER',
	`state`	          VARCHAR(10)   NOT NULL	DEFAULT 'ACTIVE',
	`social`	        VARCHAR(10)   NULL        DEFAULT NULL,
	`created_at`	    TIMESTAMP	    NOT NULL,
	`modified_at`	    TIMESTAMP	    NOT NULL,
	`deleted_at`      TIMESTAMP	    NULL        DEFAULT NULL,
	`is_deleted`      BOOLEAN	      NOT NULL    DEFAULT FALSE,
    PRIMARY KEY(`user_id`)
);

CREATE TABLE `TASK` (
	`task_id`          VARCHAR(36)  NOT NULL,
	`task_title`       TEXT         NOT NULL,
	`task_detail`      TEXT         NULL,
	`task_status`      VARCHAR(10)  NOT NULL  DEFAULT 'TODO',
	`task_date`        DATE         NOT NULL,
	`task_time`        TIMESTAMP    NULL,
	`task_order_index` INT          NOT NULL,
	`created_at`       TIMESTAMP    NOT NULL,
	`modified_at`      TIMESTAMP    NOT NULL,
	`user_id`	         VARCHAR(36)  NOT NULL,
    PRIMARY KEY(`user_id`, `task_id`), -- 파티셔닝을 위해 user_id를 첫 번째 PK로 설정
    INDEX TASK_1 (task_id), -- JOIN용 인덱스
	  INDEX TASK_2 (task_date) -- 조회 최적화를 위한 인덱스
) PARTITION BY KEY(user_id) PARTITIONS 6;

CREATE TABLE `CHECKLIST` (
	`checklist_id`    VARCHAR(36)   NOT NULL,
	`checklist_text`  VARCHAR(500)  NOT NULL,
	`is_checked`      BOOLEAN       NOT NULL	DEFAULT FALSE,
	`created_at`      TIMESTAMP     NOT NULL,
	`modified_at`     TIMESTAMP     NULL,
	`task_id`         VARCHAR(36)   NOT NULL,
    PRIMARY KEY(`checklist_id`),
    INDEX CHECKLIST_1 (`task_id`)
);
```

- **TASK 테이블의 PK를 `user_id, task_id`로 하기로 결정**
    - `user_id`가 앞으로 온 이유
        - 1️⃣ 개념적으로 USER의 TASK이기 때문
        - 2️⃣ 조회가 일어날 때 WHERE 조건의 순서가 `user_id = '유저 아이디' AND task_date = 'YYYY-mm-dd'` 인 경우가 많기 때문에 인덱스를 활용할 수 있게 하기 위해
        - 3️⃣ `task_id, user_id`로 PK를 설정하면 파티션 프루닝(Partition Pruning)은 적용되지만, 파티션 내부에서 `task_id, user_id`순으로 정렬 ⇒  `user_id = ‘유저 아이디’` 조회 시 풀스캔이 발생할 수 있음
            - `PK(task_id, user_id) + INDEX(user_id)`를 설정해도 `PK(user_id, task_id) + INDEX(task_id)`보다 성능이 더 느릴 수 있음
                - PK는 기본적으로 클러스터형 인덱스로 동작하며, 데이터가 물리적으로 PK를 기준으로 정렬
                    - `PK(user_id, task_id)`라면, `WHERE user_id = '유저 아이디'` 하면 데이터가 연속된 위치에 있어서 빠르게 조회 가능
                - `PK(task_id, user_id) + INDEX(user_id)`인 경우에 `INDEX(user_id)`는 보조 인덱스로 동작하기 때문에 `user_id`만으로 조회하면 보조 인덱스를 탐색한 후, 다시 PK를 조회하는 과정이 필요
            - 🤔 파티션 프루닝이란?
                - MariaDB에서 `WHERE user_id = '유저 아이디'` 검색 시 성능 최적화를 위해 특정 파티션만 조회할 수 있도록 파티션 프루닝(Partition Pruning)이라는 최적화 기법 사용
                <br>
- **개념적 FK를 적용하기로 결정**
    - 실제로 DB에서 FK를 적용하지 않고, FK에 대한 INDEX를 설정해서 JOIN 시 INDEX로 최적화 가능하게 설정
    - PK를 `user_id, task_id`로 하면 다른 테이블과 JOIN 시에 `task_id`로 INDEX를 사용할 수 없어서 별도 INDEX 설정
    <br>
- **INDEX 설정 테이블**
    - TASK 테이블에 `task_id`에 대한 INDEX 설정
    - TASK 테이블을 참조하는 테이블에  `task_id` INDEX 설정
    <br>
- **필요 시, 오래된 데이터는 아카이브(Archive) TASK 테이블을 만들어서 저장 예정**
    - 특정 기간이 지난 후에(1-2년) 배치로 아카이브 TASK 테이블에 저장
    <br>
- **주의할 점 : `task_id`의 FK 참조와 TASK 테이블에서 `user_id`를 FK로 참조하는 것이 불가**
    - TASK 테이블에 데이터를 INSERT 등을 할 때, `user_id`가 실제 존재하는지 검증 필요
    - TASK 테이블을 참조하는 테이블에 INSERT 할 때, 해당 `task_id`가 실제 존재하는지 검증 필요
    - TASK의 데이터가 DELETE 될 때, 해당 TASK를 참조하는 데이터들을 필요 시에 수동으로 DELETE 해줘야 함(CASCADE 불가)
    - ⇒ 이 부분들은 어플리케이션에서 처리 필요
