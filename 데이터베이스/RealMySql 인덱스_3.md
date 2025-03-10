## B-Tree 인덱스의 정렬 및 스캔 방향

---

### 인덱스의 정렬

- 인덱스를 생성할 때 정렬을 오름차순 또는 내림차순으로 설정이 가능하고, 8.0 버전 부터는 혼합해서 생성할 수도 있다.

```sql
CREATE INDEX ix_teamname_userscore ON employees (team_name ASC, user_score DESC);
```

**인덱스 스캔 방향**

- 정렬이 되어 있기 때문에 인덱스 페이지의 위치를 끝값이나 첫값위치로3 옵티마이저가 부여

**내림차순 인덱스**

- 복합 인덱스에서 내림차순과 오름차순이 혼합되었을 때는 **8.0버전의 내림차순 인덱스**로만 해결가능
    
    ![image](https://github.com/user-attachments/assets/e3150b17-42e9-40ea-8b1e-762fb8f1d408)

    
- 역순이 더 느린 이유
    - 페이지 잠금이 인덱스 정순 스캔(Forward index scan)에 적합한 구조
    - 페이지 내에서 인덱스 레코드가 단방향으로만 연결된 구조
        
        ![image](https://github.com/user-attachments/assets/08434d6f-fcb3-441f-934a-0b532de4cb66)

        

## B-Tree 인덱스의 가용성과 효율성

WHERE 조건이나 GROUP BY, ORDER BY 절이 

### 비교 조건의 종류와 효율성

```sql
SELECT * FROM dept_emp
		WHERE dept_no='d002' AND emp_no >= 10114;

위의 쿼리문 기준 좋은 인덱스 선언 방법
Case A : INDEX (dept_no, emp_no)
dept_no가 동일한 모든 칼럼을 기준으로 emp_no를 찾음

Case B : INDEX (emp_no, dept_no)
emp_no 범위에 맞는 칼럼을 모두 찾음 -> 이후 dept_no가 다른 것을 제외
```

- 작업 범위 결정 조건 : ≥ 와 같은 범위가 필요한 조건
    - 많을 수록 쿼리 처리 성능을 높임
- 필터링 조건 : ‘참’, ‘거짓’을 탐색하는 조건
    - 쿼리의 처리 성능을 높이지 못함
    - 느려지는 경우가 더 많음
    - 최종적으로 가져오는 레코드는 작을 수 있음

### 인덱스의 가용성

B-Tree인덱스의 특징은 **왼쪽 값 기준**으로 오른쪽 값 정렬

```sql
Case A : INDEX (first_name)

Case B : INDEX (dept_no, emp_no)

SELECT * FROM employees WHERE first_name LIKE '%mer';

SELECT * FROM dept_emp WHERE emp_no>=10144;
```

- `SELECT * FROM employees WHERE first_name LIKE '%mer';`
=> %mer는 왼쪽 값이 상관없다고 기준을 잡는데 왼쪽 값 기준 정렬형식인 B-Tree에 적합치 않다.
- `SELECT * FROM dept_emp WHERE emp_no>=10144;`
=> 인덱스가 dept_no, emp_no 순으로 선언되면, dept_no의 기준으로 emp_no범위가 정해져 있을텐데, emp_no의 조건을 사용하지 않으면 인덱스의 왼쪽 값을 버리게 되므로 효율적이지 않다.
- 인덱스의 왼쪽 값 기준 규칙은 `WHERE`, `GROUP BY`, `ORDER BY` 절에 모두 적용됨

### 가용성과 효율성 판단

B-Tree 특성을 잘 활용하지 못하는 경우(**작업 범위 결정 조건이 아닌 것**)

- `NOT-EQUAL`로 비교된 경우(”<>”, “NOT IN”, “NOT BETWEEN”, “IS NOT NULL”)
- `LIKE ’%??’` (앞부분이 아닌 뒷부분 일치) 형태로 문자열 패턴이 비교된 경우
- 스토어드 함수나 다른 연산자로 인덱스 칼럼이 변형된 후 비교된 경우
- NOT-DETERMINISTIC 속성의 스토어드 함수가 비교 조건에 사용된 경우
- 데이터 타입이 서로 다른 비교(인덱스 칼럼의 타입을 변환해야 비교가 가능한 경우)
- 문자열 데이터 타입의 콜레이션이 다른 경우

**다중 복합 인덱스의 가용성**

- 작업 범위 결정 조건으로 활용 불가
    - 제일 앞의 칼럼에 대한 조건이 없을 때
    - 제일 앞의 칼럼의 비교 조건이 B-Tree 작업 범위 결정 조건이 아닌 경우
- 작업 범위 결정 조건으로 사용하는 경우
    - 동등 비교(”=” 또는 “IN”)
    - 크다 작다 형태(”>” 또는 “<”)
    - LIKE로 좌측 일치(LIKE ‘승환%’)
