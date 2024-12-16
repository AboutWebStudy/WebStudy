
### 이번 장의 핵심

MySQL의 동시성에 영향을 미치는 요소

- 잠금
- 트랜잭션
- 트랜잭션의 격리 수준

### 잠금 vs 트랜잭션

- 잠금은 **동시성**을 **제어**하는 것이고
- 트랜잭션은 **데이터의 정합성**을 **보장**하기 위한 기능이다.

# 트랜잭션

- MySQL에서는 **InnoDB 스토리지 엔진**만 트랜잭션을 지원한다 (MyISAM, MEMORY 지원x)
- 쿼리문의 작업 도중 문제가 생겼을 때 **문제가 생기기 이전의 데이터 상태로 되돌려주는 기능**이다.
- 쿼리문의 적용이 데이터에 **100% 반영되거나, 적용되지 않아야 함**을 보장한다.

## MySQL에서의 트랜잭션

```sql
CREATE TABLE tab_myisam (fdpk INT NOT NULL, PRIMARY KEY (fdpk)) ENGINE=MYISAM;
CREATE TABLE tab_innodb (fdpk INT NOT NULL, PRIMARY KEY (fdpk)) ENGINE=INNODB;
INSERT INTO tab_myisam (fdpk) VALUES (3);
-> SUCCESS;
INSERT INTO tab_innodb (fdpk) VALUES (3);
-> SUCCESS;

SET autocommit=ON;

INSERT INTO tab_myisam (fdpk) VALUES (1), (2), (3);
-> !ERROR 발생
INSERT INTO tab_innodb (fdpk) VALUES (1), (2), (3);
-> !ERROR 발생

SELECT * FROM tab_myisam;
fdpk = {1, 2, 3}

SELECT * FROM tab_innodb;
fdpk = {3}
```

| InnoDB | MyISAM |
| --- | --- |
| 오류 발생 후 쿼리문의 데이터반영 x | 오류 발생 후 발생 전까지의 데이터를 반영 |
| 트랜잭션 | 트랜잭션x |

### 트랜잭션을 사용할 때 주의사항

- 커넥션 풀의  개수는 무제한이 아니다
- 네트워크 통신이 포함된 문제가 생겼을 때 웹 서버 뿐만이 아니라 DBMS자체도 통신이 필수적으로 필요하기 때문에 문제가 생길 것이다.
- 논리적으로 필요로하는 데이터를 수정, 추가하는 일이 아닌 단순 조회는 트랜잭션의 범위에 들어갈 필요가 없다.

때문에 커넥션과 트랜잭션의 활성화 범위는 가능한 최소한으로 잡는 것이 가장 좋다. 또한 위의 주의사항을 지키지 않는다면, 성능의 치명적인 위험이 발생할 가능성이 높아질 것이다.
