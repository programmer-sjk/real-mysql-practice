# Real MySQL 스터디 정리
- [Real MySQL 링크] (http://www.yes24.com/Product/Goods/103415627)
- 우테캠 JPA 스터디원 중 참석율이 높은 분들과 스터디
- 스터디 방식: 주마다 돌아가면서 정리한 내용 gather에서 발표

## 2장. 설치와 설정

### 2.3 MySQL 서버 업그레이드
- MySQL을 업그레이드 하는 방법에는 아래 두 가지 방법을 고려할 수 있다.
  - MySQL 서버 데이터 파일을 그대로 두고 업그레이드 하는 방법 (In-Place 업그레이드)
  - 서버의 데이터를 SQL이나 데이터로 덤프 후, 새로 업그레이드된 MySQL 서버에서 데이터를 적재하는 방법 (논리적 업그레이드)
  - 인플레이스 업그레이드는 제약사항이 있지만 업그레이드 시간이 단축되며 논리적 업그레이드는 제약사항이 없지만 많은 시간이 소요된다.

### 2.4 서버 설정

#### 2.4.4 정적 변수와 동적 변수
- MySQL 서버의 시스템 변수는 서버가 기동중인 상태에서 변경 가능한지에 따라 동적 변수와 정적 변수로 구분된다.
- SET 명령어로 변경되는 시스템 변수 값은 설정 파일은 my.cnf (or my.ini)에 반영되는 것은 아니기 때문에 서버가
재 시작되면 설정값이 초기화 되기 때문에 설정을 영구히 적용하려면 my.cnf 파일을 수정해야 한다.
- **MySQL 8.0 버전**부터 `SET PERSIST` 명령을 통해 시스템 변수를 변경하면서 설정 파일로도 기록 가능하다.

#### 2.4.5 SET PERSIST
- MySQL 서버의 max_connections 변수는 동시에 접속하는 최대 커넥션 수를 제한하는 동적 변수다.
- SET 명령으로 설정을 수정하고 설정파일에 반영하지 않으면 서버 재시작시에 예전의 변수 값으로 MySQL 서버가 실행되고
이를 통해 장애가 발생할 수 있다. 
- `SET PERSIST max=connections=5000;` 과 같이 시스템 변수를 변경하면 MySQL 서버는 변경된 값을 즉시 적용함과
동시에 별도의 설정파일(mysqld-auto.cnf)에 변경 내용을 추가로 기록한다.
- MySQL 서버가 다시 시작될 때 기본 설정파일(my.cnf) 뿐만 아니라 자동 생성된 mysqld-auto.cnf 파일을 같이 참조해서 시스템 변수를 적용한다.
- SET PERSIST로 추가된 시스템 변수를 삭제해야 하는 경우 아래와 같은 명령어를 사용하는게 안전하다.
```sql
# 특정 시스템 변수만 삭제
RESET PERSIST max_connections;

# mysqld-auto.cnf 파일의 모든 시스템 변수를 삭제
RESET PERSIST;
```

## 3장. 사용자 및 권한
- MySQL 8.0 버전부터 권한을 묶어서 관리하는 역할(Role)의 개념이 도입

### 3.1 사용자 식별
- MySQL은 사용자 계정과 더불어 클라이언트 접속 지점(IP or 도메인)도 계정의 일부가 된다.
따라서 아래와 같이 계정이 중복될 수 있다.
```sql
'svc_id'@'192.168.0.10' (비밀번호 123)
'svc_id'@'%'            (비밀번호 456)
```
- IP 주소가 `192.168.0.10`인 PC에서 svc_id로 접속하면 MySQL은 더 좁은 범위의 계정을 선택하기 때문에 비밀번호 123이 아닌 456을 입력하면 실패하게 된다.

### 3.2 사용자 계정 관리

#### 3.2.1 시스템 계정과 일반 계정
- MySQL 8.0부터 계정은 SYSTEM_USER 권한을 가진 시스템 계정과 일반 계정으로 구분된다.
- 보통 시스템 계정은 서버 관리자를 위한 계정이고 일반 계정은 응용 프로그램이나 개발자를 위한 계정이다.
- 시스템 계정은 시스템 계정과 일반 계정을 관리할 수 있지만 일반 계정은 시스템 계정을 관리할 수 없다.

#### 3.2.2 계정 생성
- MySQL 5.7 버전까지는 GRANT 명령으로 권한의 부여와 동시에 계정 생성이 가능.
- 하지만 8.0 버전부터 계정의 생성은 `CREATE USER` 명령으로 권한 부여는 `GRANT` 명령으로 구분한다.
- 사용자 인증과 비밀번호 측면에서 MySQL 5.7은 패스워드에 대한 해시값을 저장하고 비교하는 방식이라면
8.0 버전부터는 `Cashing SHA-2 Authentication`이 기본 인증으로 채택되었다. 이방식은 SSL/TLS 또는 RSA 키페어가
필요하기 때문에 상황에 따라 8.0 버전에서도 예전의 패스워드 비교 방식을 설정할 수 있다.

### 3.3 비밀번호 관리
- MySQL 서버의 비밀번호 유효기간, 이력 관리를 통한 재사용 금지기능, 비밀번호의 강도를 설정할 수 있다.
```sql
INSTALL COMPONENT 'file://component_validate_password';
select * from mysql.component;
show GLOBAL VARIABLES LIKE "validate_password%";

# Variable_name                      | value  | 설명
validate_password.check_user_name	     ON       
validate_password.dictionary_file	
validate_password.length	             8        비밀번호 길이
validate_password.mixed_case_count     1        최소 1개이상의 대소문자
validate_password.number_count	       1        최소 1개이상의 숫자
validate_password.policy	             MEDIUM
validate_password.special_char_count	 1        최소 1개이상의 특수문자
```
- 비밀번호 정책은 크게 3가지 중 선택할 수 있으며 기본값은 MEDIUM으로 자동 설정된다.
    - LOW: 비밀번호 길이만 검증
    - MEDIUM: 길이와 숫자와 대소문자와 특수문자의 배합을 검증
    - STRONG: MEDIUM 레벨의 검증 + 금칙어 포함여부 검증

### 3.4 권한
- 사용자에게 권한을 부여할 때는 GRANT 명령을 사용한다. 아래 여러 예시를 보자.
```sql
# 글로벌 권한 (글로벌 권한은 특정 DB나 테이블에 부여할수 없기 때문에 *.*를 사용)
GRANT SUPER ON *.* TO 'user'@'localhost';

# DB 권한
GRANT EVENT ON *.* TO 'user'@'localhost';
GRANT EVENT ON employees.* TO 'user'@'localhost';

# 테이블 권한 (순서대로 전체, 특정 DB, 특정 DB의 테이블 권한 부여가능)
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.department TO 'user'@'localhost';
```

### 3.5 역할
- MySQL 8.0 버전부터 권한을 묶어 역할(Role)을 사용할 수 있게 되었다. 우선 역할을 생성해보자
```sql
CREATE ROLE role_emp_read, role_emp_write;
```
- CREATE ROLE 명령은 역할만 정의한 것으로 역할이 가질 실질적인 권한을 부여해야 한다.
```sql
GRANT SELECT ON employees.* TO role_emp_read;
GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```
- 역할은 그 자체로 사용될 수 없고 계정에 적용되므로 reader, writer 계정을 생성한 뒤에 역할을 부여한다.
```sql
CREATE USER reader@'127.0.0.1' IDENTIFIED BY 'password';
CREATE USER writer@'127.0.0.1' IDENTIFIED BY 'password';

GRANT role_emp_read TO reader@'127.0.0.1';
GRANT role_emp_read, role_emp_write TO writer@'127.0.0.1';
```
- 지금까지 역할을 생성해 권한을 부여하고, 이 역할을 계정에 반영했지만 추가적으로 생성된 역할을 활성화 해야 한다.
```sql
SET ROLE 'role_emp_read';
```
- 계정과 역할은 MySQL 서버 내부적으로 차이는 없으며 그럼에도 `CREATE ROLE, CREATE USER` 명령을 구분해서 지원하는
이유는 보안을 강화하는 용도이다.