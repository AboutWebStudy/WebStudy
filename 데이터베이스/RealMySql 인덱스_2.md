## B-Tree 인덱스를 통한 데이터 읽기

---

### 인덱스 레인지 스캔

- 인덱스 접근방법중 가장 대표적
- `검색해야 할 인덱스의 범위가 결정됐을 때 사용`
- 인덱스 레인지 스캔 3단계
    1. 인덱스 탐색(index seek) : 인덱스에서 조건을 만족하는 값이 저장된 위치를 찾는다.
    2. 인덱스 스캔(index scan) : 최초 탐색 된 위치부터 필요한 만큼 인덱스를 차례대로 읽는다.
    3. 스캔 된 인덱스 키와 레코드 주소를 이용해 레코드가 저장된 페이지를 가져오고 최종 레코드를 읽어온다.
- 3번 과정은 쿼리가 해당 인덱스 키를 가진 필드 데이터가 필요하지 않다면 디스크의 레코드를 읽지 않을 수 있는데,  이를 `커버링 인덱스`라고 한다.

### 인덱스 풀 스캔

- 인덱스의 처음부터 끝까지 모두 읽는 방식
- 인덱스의 데이터 레코드까지 읽어야 한다면 테이블 풀 스캔이 동작
- 인덱스 리프 노드의 제일 앞 또는 뒤로 이동 후 `링크드리스트`를 따라서 처음부터 끝까지 스캔하는 방식
- 인덱스 레인지 → 인덱스 풀 → 테이블 풀 순서로 효율이 좋음

### 루스 인덱스 스캔

- MySQL 8.0부터 최적화
- 여러 조건이 만족되어야 옵티마이저에서 활용 ( GROUP BY 사용, …)
- 인덱스 레인지처럼 순차스캔을 하지만 중간에 필요 없는 인덱스 키 값은 무시
⇒ 일반적으로 GROUP BY 또는 MAX 또는 MIN 함수에 대해 최적화할 때 사용된다.

```sql
dept_no, emp_no 둘다 인덱스가 생성되어 있다고 가정했을 때
SELECT dept_no, MIN(emp_no)
	FROM dept_emp
	WHERE dep_no BETWEEN 'd002' AND 'd004'
	GROUP BY dept_no;
-> WHERE MIN()함수를 사용 and 인덱싱되어있는 정렬된 데이터
=> 조건의 범위를 다 돌 필요가 없다.
=> 첫 값만 읽고 끝
```

### 인덱스 스킵 스캔

- 핵심은 값이 정렬돼 있다는 것이며, 인덱스를 구성하는 칼럼의 순서가 매우 중요하다.
- ex) `ALTER TABLE employees ADD INDEX ix_gender_birthdate (gender, birth_date);`
    - 이 복합 인덱스가 사용되려면 WHERE 조건절에 gender 칼럼에 대한 비교 조건이 필수
    
    ```sql
    SELECT * FROM employees WHERE birth_date>='1965-02-01'; -> X
    
    SELECT * FROM employees WHERE gender='M' AND birth_date>='1965-02-01'; -> O
    ```
    
- 8.0 버전 이후 - `SET optimizer_switch=’skip_scan=on’;`
- **skip_scan - on**
    
    ![image](https://github.com/user-attachments/assets/122c2083-34f5-44f1-b939-3e7cc5232d1d)

    
    - type : `range` - **필요한 부분만 읽음**
    
    ```sql
    SELECT gender, birth_date FROM employees WHERE gender='M' AND birth_date>='1965-02-01';
    SELECT gender, birth_date FROM employees WHERE gender='F' AND birth_date>='1965-02-01';
    위의 쿼리문과 비슷하게 한번 최적화 작업을 거침
    ```
    
- **skip_scan - off**
    
    ![image](https://github.com/user-attachments/assets/0560704e-4c04-4787-b793-4314268ec682)

    
    - type : `index` - **인덱스 테이블 전체를 읽음(풀 인덱스 스캔)**
- 인덱스 스킵 스캔을 사용하기 위한 조건
    - WHERE 조건절에 조건이 없는 인덱스의 선행 칼럼의 유니크한 값의 개수가 적어야 함
    - 쿼리가 인덱스에 존재하는 칼럼만으로 처리 가능해야 함(커버링 인덱스)
 
## 다중 칼럼(Multi-column) 인덱스

- 2개 이상의 칼럼을 포함하는 인덱스 (Concatenated Index)
- 앞의 칼럼을 의존해서 정렬된다.
⇒ 인덱스 내의 칼럼 위치(순서)를 신중히 결정해야 한다.
