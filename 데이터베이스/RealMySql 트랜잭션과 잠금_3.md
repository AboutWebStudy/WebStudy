# InnoDB 스토리지 엔진 잠금

InnoDB 스토리지 엔진은 MySQL에서 제공하는 잠금과는 별개로 스토리지 엔진 내부에서 레코드 기반의 잠금 방식을 탑재하고 있다. 

- 레코드 기반의 잠금을 하기 때문에 MyISAM보다 뛰어난 동시성 처리를 제공할 수 있다.
- MySQL 자체에서 떨어져나온 잠금 탓에 InnoDB잠금에 대한 정보는 MySQL명령을 이용해 접근이 어렵다.
- 상태 확인
    - InnoDB의 트랜잭션과 잠금은 MySQL 서버의 information_schema 데이터베이스의 `INNODB_TRX, INNODB_LOCKS, INNODB_LOCK_WAITS`
    - performance_schema의 `data_locks, data_lock_waits` 테이블로 대체되는중

## InnoDB 스토리지 엔진의 잠금

- **레코드 락**
    - 레코드 자체만을 잠그는 것을 레코드 락(Record lock, Record only lock)이라고 하며, 다른 상용 DBMS의 레코드 락과 동일한 역할을 한다. 한 가지 중요한 차이는 InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스의 레코드를 잠근다는 점이다.
- **갭 락**
    - 갭 락은 레코드 자체가 아니라 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 것을 의미한다. 갭 락의 역할은 레코드와 레코드 사이의 간격에 새로운 레코드가 생성되는 것을 제어하는 것이다.
- **넥스트 키 락**
    - 바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 실행될 때 소스 서버에서 만들어 낸 결과와 동일한 결과를 만들어내도록 보장하는 것이 주목적이다.
    - `레코드 락 + 갭 락` 형태의 잠금
    - MySQL 서버에서는 REPEATABLE READ 격리 수준을 사용
    - `innodb_locks_unsafe_for_binlog` 변수가 0이라면 변경을 위해 검색하는 레코드에는 넥스트 키 락 방식으로 잠금이 걸린다.
- **자동 증가 락**
    - `AUTO_INCREMENT` 칼럼 속성과 관련된 잠금
    - 각 레코드가 중복되지 않고 저장된 순서대로 증가하는 번호를 해당 락이 돕는다.
    - `innodb_autoinc_lock_mode`
        - 0 : 모든 INSERT 문장이 자동 증가 락을 사용
        - 1 : MySQL 서버가 INSERT되는 레코드의 건수를 정확히 예측할 수 있을 때는 락을 사용하지 않고, 훨씬 가볍고 빠른 래치(뮤텍스)를 이용해 처리
        - 2 : 래치(뮤텍스)만 사용 주의할 점으로는 **유니크한 값이 생성되는 것만 보장**

## 인덱스와 잠금

InnoDB의 특징인 **레코드 락**은 레코드가 아닌 **인덱스를 잠근다** 인덱스가 아닌 다른 칼럼을 사용한다면 해당 칼럼 관련한 레코드 모두가 잠기게 된다.

```sql
SELECT COUNT(*) FROM employees WHERE first_name='Georgi';

SELECT COUNT(*) FROM employees WHERE first_name='Georgi' AND last_name='Klassen';
```

## 레코드 수준의 잠금 확인 및 해제

레코드 수준의 잠금은 테이블의 레코드 각각에 잠금이 걸리므로 그 레코드가 자주 사용되지 않는다면 오랜 시간 동안 잠겨진 상태로 남아 있어도 잘 발견되지 않는다.

```sql
스레드 잠금 대기 확인 방법
mysql> SHOW PROCESSLIST;

스레드 잠금 상세 확인 방법
SELECT * FROM performance_schema.data_locks\G
```

# MySQL의 격리 수준

- **사용중인 트랜잭션의 격리 수준에 의해 실행하는 SQL문장이 어떤 결과를 가져오게 되는지를 정확하게 예측할 수 있어야 한다.**
- READ UNCOMMITTED(DIRTY READ)
    - 각 트랜잭션에서의 변경 내용이 COMMIT이나 ROLLBACK 여부에 상관없이 다른 트랜잭션에서 보인다.
    - Dirty read : 어떤 트랙잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있는 현상
    - 부정합 → DIRTY READ
- **READ COMMITTED**
    - 오라클 DBMS 기본 격리 수준
    - 트랜잭션이 걸려있을 시 언두 로그 데이터 반환
    - 부정합 → NON-REPEATABLE
- **REPEATABLE READ**
    - InnoDB 기본 격리 수준
    - MVCC(Multi Version Concurrency Control)을 통해 언두 영역에서 연관 있는 트랜잭션 번호의 구간을 모두 남겨둔다.
    - 부정합 → PHANTOM READ(다른 트랜잭션에서 처리한 값을 가져오는 것)
    - 유일하게 InnoDB에서 PHANTOM READ문제가 걸리지 않음 (FOR UPDATE , FOR SHERE제외)
    
    ```sql
    SELECT emp_no FROM employees WHERE emp_no>=499999 FOR UPDATE;
    
    INSERT INTO employees (emp_no, first_name, birth_date, last_name, hire_date) VALUES (500000, 'Georgi', '1990-01-01', 'GOGO','1990-01-01');
    ```
    
- SERIALIZABLE
    - 가장 엄격한 격리 수준
    - SELECT문도 공유 잠금 상태가 된다.
    - PHANTOM READ Safe

|  | DIRTY READ | NON-REPEATABLE READ | PHANTOM READ |
| --- | --- | --- | --- |
| READ UNCOMMITTED | O | O | O |
| READ COMMITTED | X | O | O |
| REPEATABLE READ | X | X | O(InnoDB는 x) |
| SERIALIZABLE | X | X | X |

READ UNCOMMITTED → … → SERIALIZABLE 뒤로 갈수록 격리 정도가 높아지고 동시 처리 성능은 떨어진다. 하지만 유의미한 성능 차이는 격리 수준이SERIALIZABLE인 경우에만 발생한다.
