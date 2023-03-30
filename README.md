# Real MySQL 스터디 정리
- [Real MySQL 링크] (http://www.yes24.com/Product/Goods/103415627)
- 우테캠 JPA 스터디원 중 참석율이 높은 분들과 스터디
- 스터디 방식: 주마다 돌아가면서 정리한 내용 gather에서 발표

## 2장. 설치와 설정

### 2.3 MySQL 서버 업그레이드
- MySQL을 업그레이드 하는 방법에는 아래 **두 가지 방법**을 고려할 수 있다.
  - MySQL 서버 데이터 파일을 그대로 두고 업그레이드 하는 방법 (In-Place 업그레이드)
  - 서버의 데이터를 SQL이나 데이터로 덤프 후, 새로 업그레이드된 MySQL 서버에서 데이터를 적재하는 방법 (논리적 업그레이드)
  - 인플레이스 업그레이드는 제약사항이 있지만 업그레이드 시간이 단축되며 논리적 업그레이드는 제약사항이 없지만 많은 시간이 소요된다.

### 2.4 서버 설정

#### 2.4.4 정적 변수와 동적 변수
- MySQL 서버의 시스템 변수는 서버가 기동중인 상태에서 변경 가능한지에 따라 동적 변수와 정적 변수로 구분된다.
- SET 명령어로 변경되는 시스템 변수 값은 설정 파일은 my.cnf (or my.ini)에 반영되는 것은 아니기 때문에 서버가
재 시작되면 설정값이 초기화 되기 때문에 **설정을 영구히 적용하려면 my.cnf 파일을 수정**해야 한다.
- **MySQL 8.0 버전**부터 `SET PERSIST` 명령을 통해 시스템 변수를 변경하면서 설정 파일로도 기록 가능하다.

#### 2.4.5 SET PERSIST
- MySQL 서버의 `max_connections` 변수는 동시에 접속하는 최대 커넥션 수를 제한하는 동적 변수다.
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
validate_password.check_user_name	   |  ON    |   
validate_password.dictionary_file	   |        |
validate_password.length	           |  8     |   비밀번호 길이
validate_password.mixed_case_count   |  1     |   최소 1개이상의 대소문자
validate_password.number_count	     |  1     |   최소 1개이상의 숫자
validate_password.policy	           |  MEDIUM|
validate_password.special_char_count | 1      |  최소 1개이상의 특수문자
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

## 4. 아키텍처
- MySQL 서버는 엔진과 스토리지 엔진으로 구분된다.
![mysql 전체구조](./images/mysql_structure.png)
- MySQL 엔진은 클라이언트의 접속 및 쿼리 요청을 처리하는 커넥션 핸들러, 최적화된 실행을 위한 옵티마이저로 구성된다.
- MySQL 엔진이 SQL 문장을 분석하고 최적화하는 두뇌라면 실제 데이터를 읽어오는 역할은 스토리지 엔진이 한다. 

### 4.1.2 MySQL 스레딩 구조
![mysql 스레딩구조](images/MySQL_%EC%8A%A4%EB%A0%88%EB%94%A9%EA%B5%AC%EC%A1%B0.png)
- MySQL 서버는 스레드 기반으로 동작하며 포그라운드와 백그라운드 스레드로 구분된다.
- 포그라운드 스레드는 최소 접속된 클라이언트 수만큼 존재하며 각 사용자가 요청하는 쿼리를 처리한다.
- 포그라운드 스레드는 데이터를 MySQL의 데이터 버퍼나 캐시로부터 가져오며 버퍼나 캐시에 없는 경우 스토리지 엔진에
따라 직접 가져오거나 백그라운드 스레드가 처리해준다.
- MyISAM 스토리지 엔진은 해당이 없지만 InnoDB 스토리지 엔진의 경우 여러 작업들이 백그라운드 스레드로 처리된다.
- 일반적인 DBMS는 쓰기 작업을 버퍼링해서 일괄처리하며 InnoDB도 이렇게 처리한다.

### 4.1.3 메모리 할당 및 사용구조
- MySQL에서 사용되는 메모리 공간은 크게 글로벌과 로컬 메모리 영역으로 구분된다.
- 글로벌 메모리 영역은 클라이언트 스레드 수와 무관하게 하나의 메모리 공간만 할당되며 모든 스레드에 의해 공유된다.
아래와 같은 글로벌 메모리 영역들이 있다.
  - 테이블 캐시
  - InnoDB 버퍼 풀
  - InnoDB 어댑티브 해시 인덱스
- 로컬 메모리 영역은 클라이언트 스레드가 쿼리를 처리하는데 사용하는 메모리 영역으로 대표적으로 커넥션 버퍼와 정렬 버퍼가 있다.
- MySQL 서버는 클라이언트 요청을 처리하기 위해 스레드를 하나씩 할당하게 되고 로컬 메모리는 각 클라이언트 스레드별로
독립적으로 할당되며 공유되지 않는다는 특징이 있다.
- 대표적인 로컬 메모리 영역은 다음과 같다.
  - 정렬 버퍼
  - 조인 버퍼
  - 바이너리 로그 캐시
  - 네트워크 버퍼

### 4.1.4 플러그인 스토리지 엔진 모델
- MySQL의 독특한 구조 중 대표적인 것이 플러그인 모델로 다른 회사나 사용자가 직접 스토리지 엔진을 개발하는 것도 가능하다.
- MySQL에서 MyISAM, InnoDB와 같이 다른 스토리지 엔진에 쿼리를 실행하더라도 MySQL 엔진의 처리내요은 비슷하며
단순히 데이터 읽기/쓰기 방식이 차이가 있다. 실질적인 GROUP BY나 ORDER BY등의 복잡한 처리는 스토리지 엔진 영역이
아니라 MySQL 엔진의 처리 영역인 `쿼리 실행기`에서 처리된다.

### 4.1.5 컴포넌트
- MySQL 8.0 부터 기존 플러그인 아키텍처의 단점을 보완하기 위해 컴포넌트 아키텍처가 지원된다. 플러그인의 단점은
  - 플러그인은 오직 MySQL 서버와 인터페이스 할 수 있고 플러그인들끼리 통신할 수 없음
  - 플러그인 상호 의존관계를 설정할 수 없어 관리가 어려움