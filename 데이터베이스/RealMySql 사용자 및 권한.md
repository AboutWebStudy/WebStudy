### 시스템 계정과 일반 계정

일반계정을 관리, 데이터베이스 서버 관리 등과 같은 중요작업(DBA가 관리하는 것)을 하는 **시스템계정**과 일반 응용프로그램을 확인하거나 개발자들이 활용하는 **일반 계정**으로 나뉘어져 있다.

구체적인 시스템계정의 역할

- 계정 관리(계정 생성 및 삭제, 계정의 권한 부여 및 제거)
- 다른 세션(Connection)또는 그 세션에서 실행 중인 쿼리를 강제 종료
- **스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정**

### 계정 생성

MySQL 8.0버전부터는 계정생성을 GRANT가 아닌, CREATE USER명령을 통해, 권한 부여는 GRANT 명령을 통해 실행되도록 구분했다.

첫 계정 생성시에 설정되는 옵션으로는

- 계정의 인증 방식과 비밀번호
- 비밀번호 관련 옵션(유효 기간, 이력 개수, 비밀번호 재사용 불가기간)
- 기본 역할
- SSL 옵션
- 계정 잠금 여부

등이 있으며, 작성예시는 다음과 같다

```sql
CREATE USER 'user'@'%'
IDENTIFIED WITH 'mysql_native_password' BY ' 'password'
REQUIRE NONE
PASSWORD EXPIRE INTERVAL 30 DAY
ACCOUNT UNLOCK
PASSWORD HISTORY DEFAULT
PASSWORD REQUIRE CURRENT DEFAULT;
```

위 옵션을 하나하나 보자면

### IDENTIFIED WITH

사용자 인증 방식과 비밀번호를 설정 방식에는 4가지가 대표적

- Native Pluggable Authentication : 비밀번호 해시 SHA-1알고리즘을 통한 값 저장
- Caching SHA-2 Pluggable Authentication : SHA-2(256비트)알고리즘을 통한 값 저장
- PAM Pluggable Authentication : 유닉스나 리눅스 패스워드 또는 LDAP같은 외부 인증을 사용할 수 있게 해주는 인증 방식으로, MySQL 엔터프라이즈 에디션에서만 사용 가능
- LDAP Pluggable Authentication : LDAP를 이용한 외부인증방식 이또한 MySQL 엔터프라이즈에서만 사용 가능

기본 인증방식은 Caching SHA-2 Authentication이며, 이는 SSL/TLS 또는 RSA 키페어를 필요로 하기 때문에 혹시나 Native Pluggable 인증방식으로 바꿀 때는 추가 명령어가 필요하다.

선택적으로 바꾸는 이유 : 암호화 및 보안에는 SHA-2가 더 좋지만 , 비밀번호 인증 과정에서 해시함수 확인을 위해 CPU의 자원을 많이 소모하기 때문

### REQUIRE

MySQL 서버에 접속할 때 SSL/TLS의 사용 여부

### PASSWORD EXPIRE

비밀번호의 유효기간 설정 PASSWORD EXPIRE 절에 설정 가능한 옵션은 다음과 같다.

- EXPIRE : 계정 생성과 동시에 비밀번호의 만료 처리
- NEVER : 비밀번호 만료기간 없음
- DEFAULT : default_password_lifetime : 시스템 변수에 저장된 기간
- INTERVAL n DAY : 비밀번호의 유효기간을 오늘부터 n일자로 설정

### PASSWORD HISTORY

한번 사용했던 비밀번호를 재사용 못하게 하는 옵션

- PASSWORD HISTORY DEFAULT : 시스템변수에 저장된 개수만큼
- n : 최근 n개까지만 비밀번호의 이력을 저장해 이력에 남아있는 비밀번호만 불가능

### PASSWORD REUSE INTERVAL

한번 사용했던 비밀번호의 재사용 금지**기간을 설정하는 옵션**

### PASSWORD REQUIRE

비밀번호가 만료되었을 때 현재 비밀번호를 필요로 할지 말지를 결정

### ACCOUNT LOCK / UNLOCK

계정 생성 시, ALTER USER 명령을 사용해 계정 정보를 변경할 때 계정을 잠글지 사용가능하게할지 여부 설정

### 비밀번호 관리

암호설정을 고수준으로 하도록 변경할 수도 있다.

고수준 암호 옵션은 세가지로 나뉜다.

- LOW : 비밀번호의 길이만 검증
- MEDIUM : 비밀번호의 길이를 검증, 숫자 대소문자, 특수문자 배합 검증
- STRONG : MEDIUM의 검증 포함하여 금칙어 검증

등이 있다.

### 권한

데이터베이스나 테이블과 객체에 적용되는 권한을 글로벌 권한이라고 하며, 데이터베이스나 테이블을 제어하는 데 필요한 권한을 객체 권한이라고 한다.

객체 권한은 GRANT 명령으로 부여할 때 특정 객체를 명시해야 하며, 글로벌 권한은 명시하지 말아야 한다.

예외적인 권한 부여 방법으로는 ALL 또는 ALL PRIVILEGES가 있는데, 이는 글로벌과 객체 두가지 용도 모두 사용될 수 있는데, 객체에 ALL이 부며되면 해당 객체에 적용될 수 있는 모든 권한을 부여받는 것이며, 글로벌로 ALL이 사용되면 글로벌 수준에서 가능한 모든 권한을 부여받는다.

### 역할

MySQL 8.0 버전부터는 권한을 묶어서 역할(ROLE)을 사용할 수 있게 되었다.

```sql
CREATE ROLE
role_emp_read,
role_emp_write;
```

처럼 역할을 정의할 수 있고 이러한 역할에 GRANT를 통해 실제 권한들을 부여해 관리할 수 있다.

```sql
GRANT SELECT ON employees.* TO role_emp_read;
GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```

role_emp_read 객체에는 employees DB의 모든 객체에 SELECT만 가능

관리하는 방법에 따라서 편하게 역할과 권한으로 지정해서 사용하면 될 것 같다.
