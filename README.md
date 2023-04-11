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
- MySQL 서버는 **엔진과 스토리지 엔진**으로 구분된다.
![mysql 전체구조](./images/mysql_structure.png)
- MySQL 엔진은 클라이언트의 접속 및 쿼리 요청을 처리하는 `커넥션 핸들러`, 최적화된 실행을 위한 `옵티마이저로` 구성된다.
- MySQL 엔진이 SQL 문장을 분석하고 최적화하는 두뇌라면 실제 데이터를 읽어오는 역할은 `스토리지 엔진`이 한다. 

#### 4.1.2 MySQL 스레딩 구조
![mysql 스레딩구조](images/MySQL_%EC%8A%A4%EB%A0%88%EB%94%A9%EA%B5%AC%EC%A1%B0.png)
- MySQL 서버는 스레드 기반으로 동작하며 `포그라운드`와 `백그라운드` 스레드로 구분된다.
- 포그라운드 스레드는 최소 접속된 클라이언트 수만큼 존재하며 각 사용자가 요청하는 쿼리를 처리한다.
- 포그라운드 스레드는 데이터를 MySQL의 데이터 버퍼나 캐시로부터 가져오며 버퍼나 캐시에 없는 경우 스토리지 엔진에
따라 직접 가져오거나 백그라운드 스레드가 처리해준다.
- MyISAM 스토리지 엔진은 해당이 없지만 InnoDB 스토리지 엔진의 경우 여러 작업들이 백그라운드 스레드로 처리된다.
- 일반적인 DBMS는 쓰기 작업을 버퍼링해서 일괄처리하며 InnoDB도 이렇게 처리한다.

#### 4.1.3 메모리 할당 및 사용구조
- MySQL에서 사용되는 메모리 공간은 크게 `글로벌과 로컬 메모리 영역`으로 구분된다.
- `글로벌 메모리 영역`은 클라이언트 스레드 수와 무관하게 `하나의 메모리 공간만 할당되며 모든 스레드에 의해 공유`된다.
아래와 같은 **글로벌 메모리 영역**들이 있다.
  - 테이블 캐시
  - InnoDB 버퍼 풀
  - InnoDB 어댑티브 해시 인덱스
- 로컬 메모리 영역은 클라이언트 스레드가 쿼리를 처리하는데 사용하는 메모리 영역으로 대표적으로 커넥션 버퍼와 정렬 버퍼가 있다.
- MySQL 서버는 클라이언트 요청을 처리하기 위해 스레드를 하나씩 할당하게 되고 로컬 메모리는 각 클라이언트 스레드별로
독립적으로 할당되며 공유되지 않는다는 특징이 있다.
- 대표적인 **로컬 메모리 영역**은 다음과 같다.
  - 정렬 버퍼
  - 조인 버퍼
  - 바이너리 로그 캐시
  - 네트워크 버퍼

#### 4.1.4 플러그인 스토리지 엔진 모델
- MySQL의 독특한 구조 중 대표적인 것이 플러그인 모델로 다른 회사나 사용자가 직접 스토리지 엔진을 개발하는 것도 가능하다.
- MySQL에서 MyISAM, InnoDB와 같이 다른 스토리지 엔진에 쿼리를 실행하더라도 MySQL 엔진의 처리내용은 비슷하며
단순히 데이터 읽기/쓰기 방식이 차이가 있다. 실질적인 GROUP BY나 ORDER BY등의 복잡한 처리는 스토리지 엔진 영역이
아니라 MySQL 엔진의 처리 영역인 `쿼리 실행기`에서 처리된다.

#### 4.1.5 컴포넌트
- MySQL 8.0 부터 기존 플러그인 아키텍처의 단점을 보완하기 위해 컴포넌트 아키텍처가 지원된다. 플러그인의 단점은
  - 플러그인은 오직 MySQL 서버와 인터페이스 할 수 있고 플러그인들끼리 통신할 수 없음
  - 플러그인 상호 의존관계를 설정할 수 없어 관리가 어려움

#### 4.1.6 쿼리 실행구조
- 쿼리를 실행하는 관점에서 MySQL 구조는 아래와 같이 나눌 수 있다.
![쿼리 실행구조](./images/%EC%BF%BC%EB%A6%AC%EC%8B%A4%ED%96%89%EA%B5%AC%EC%A1%B0.png)
- **쿼리파서** (쿼리 문장을 분해해 트리 형태의 구조로 만들어내는 작업, 문법 오류는 이과정에서 발견됨)
- **전처리기** (각 테이블의 존재여부 및 권한 확인)
- **옵티마이저** (들어온 쿼리를 어떻게 가장 빠르게 처리할지를 결정하는 역할)
- **실행 엔진** (핸들러에게 요청해서 받은 결과를 연결하는 역할)
- **핸들러** (스토리지 엔진, 실행엔진 요청에 따라 디스크에 저장하고 읽어온다.)

#### 4.1.8 캐시
- 쿼리 캐시가 성능 저하와 많은 버그로 인해 MySQL 8.0 부터 **쿼리 캐시는 기능에서 제거**되었다.

### 4.2 InnoDB 스토리지 엔진 아키텍처
- InnoDB는 MySQL에서 사용하는 스토리지 엔진 중 유일하게 레코드 기반의 잠금을 제공하며 그 때문에 높은 동시성 처리가
가능하고 안정적이며 성능이 뛰어나다.
- 아래 그림은 InnoDB 아키텍처를 간단히 보여주는데 각 특징들을 하나씩 살펴보자
![InnoDB 구조](./images/innodb_구조.png)

#### 4.2.1 프라이머리 키에 의한 클러스터링
- InnoDB의 모든 테이블은 기본적으로 프라이머리 키를 기준으로 클러스터링 되어 저장된다.
- 즉 프라이머리 키 값의 순서대로 디스크에 저장되며 모든 세컨더리 인덱스는 레코드의 주소 대신 프라이머리 키값을
논리적인 주소로 사용한다. 프라이머리 키가 클러스터링 인덱스이기 때문에 프라이머리 키를 이용한 레인지 스캔은
상당히 빨리 처리될 수 있다.
- 결과적으로 쿼리 실행계획에서 `프라이머리 키`는 다른 보조 인덱스에 비해 **높은 우선순위**를 가지게 될 확률이 높다.

#### 4.2.2 외래 키 지원
- 외래키는 InnoDB 스토리지 엔진 레벨에서 지원하는 기능이지만 **서버 운영의 불편함으로 실무에서는 사용하지 않는 경우**가 많다.
- InnoDB에서 외래키는 부모/자식 테이블 모두 해당 컬럼에 인덱스를 생성하고, 변경시 부모/자식 테이블에 데이터가 있는지 체크하는 작업이 필요하므로 잠금이 여러 테이블로 전파되고 그로인한 데드락이 발생할 때가 있으니 주의해야 한다.
- 또한 수동으로 데이터에 쓰기 작업을 할 때 부모/자식 테이블 관계를 명확히 파악하지 못하면 작업이 실패할 수도 있다.
- 특히 긴급하게 조치를 해야할 경우 더욱 문제가 될 수 있다.

#### 4.2.3 MVCC (Multi Version Concurrency Control)
- `MVCC`의 가장 큰 목적은 `잠금을 사용하지 않는 일관된 읽기`를 제공하는데 있다. InnoDB는 Undo Log(언두 로그)를
이용해 이 기능을 구현한다. 
- 여기서 `멀티 버전`이라 함은 하나의 레코드에 여러개의 버전이 동시에 존재한다는 의미다. 
- 예를 들어 `insert` 쿼리로 데이터를 쓰고 `update` 쿼리로 데이터를 변경한다고 가정하자. `update` 쿼리가 아직 `commit이나`
`rollback`이 되기전에 해당 데이터를 조회하면 원본 데이터가 조회될까? 아니면 메모리상에 수정된 데이터가 조회될까? 
- 결과는 MySQL 서버에 설정된 `격리 수준(Isolation Level)`에 따라 다르다. 격리수준이 `READ_UNCOMMITED`인 경우 메모리상에
수정된 데이터를 읽어서 반환하는 반면 `READ_COMMITED`나 그 이상의 격리수준인 경우 아직 `commit` 되지 않았기 때문에 변경되기
이전 데이터를 보관하는 undo 영역의 데이터를 반환환다.
- 이렇게 하나의 레코드에 대해 여러 버전을 유지하는 것을 `MVCC`라고 표현한다. 

#### 4.2.4 잠금없는 일관된 읽기 (Non-Locking Consistent READ)
- InnoDB 스토리지 엔진은 `MVCC` 기술을 이용해 잠금을 걸지 않고 읽기 작업을 수행한다. 즉, 다른 트랜잭션의 변경 작업과
관계없이 항상 잠금을 대기하지 않고 바로 실행된다.
- 예를 들어 특정 사용자가 레코드를 변경하고 아직 커밋을 수행하지 않았다 하더라도 이 트랜잭션이 다른 사용자의 `SELECT`
작업을 방해하지 않는다.
- 이를 잠금없는 일관된 읽기라고 표현하며 InnoDB에서는 변경되기 전의 데이터를 읽기 위해 언두 로그를 사용한다.

#### 4.2.5 자동 데드락 감지
- InnoDB 스토리지 엔진은 내부적으로 `데드락`을 감지해 주기적으로 하나를 강제 종료한다. 이때 어느 트랜잭션을 종료할지
판단 기준은 트랜잭션의 언두 로그 양이며, 언두 로그가 적을수록 롤백을 해도 언두처리를 해야 할 내용이 적다는 뜻이다.
- 동시 처리량이 증가하거나 잠금 개수가 많아지면 데드락 감지 스레드가 느려지는데 이를 위해 MySQL 서버는
`innodb-deadlock_detect` 변수를 제공하며 OFF일 경우 데드락 감지 스레드는 작동하지 않게 된다.
- 또한 `innodb-lock-wait-timeout` 시스템 변수를 활성화하면 데드락 상황에서 일정시간이 지나면 자동으로 요청이 실패하도록
할 수 있다. 따라서 `innodb-deadlock_detect` 변수가 OFF인 상황이라면 `innodb-deadlock_detect` 변수르 기본값보다 낮게 설정해서 사용할 것을 권한다. 

#### 4.2.7 InnoDB 버퍼 풀
- InnoDB 스토리지 엔진에서 가장 핵심적인 부분으로 데이터나 인덱스 정보를 메모리에 **캐시해두는** 공간이다.
- 쓰기 작업을 지연시켜 일괄 작업으로 처리할 수 있게 해주는 버퍼 역할도 같이 한다.

#### 4.2.9 언두 로그
- InnoDB 스토리지 엔진은 트랜잭션과 격리 수준을 보장하기 위해 데이터가 변경되기 전 이전 버전의 데이터를 별도로 백업한다.
- 이렇게 백업된 데이터를 `Undo Log`라고 하는데 어떻게 사용되는지 간단히 살펴보면
  - **트랜잭션 보장**
    - 트랜잭션이 롤백되면 변경전 데이터로 복구해야 하는데, 이때 언두 로그에 백업해둔 이전 버전의 데이터를 이용해 복구
  - **격리 수준보장**
    - 특정 커넥션에서 데이터를 변경하는 도중 다른 커넥션에서 데이터를 조회하면 트랜잭션 격리 수준에 맞게 언두 로그에 백업해둔 데이터를 읽어서 반환하기도 한다.

#### 4.2.11 리두 로그
- MySQL 서버가 SW/HW 적인 문제로 비정상 종료되었을 때 데이터 파일에 기록하지 못한 데이터를 잃지 않기 위한 안전장치로 `리두 로그`를 사용한다.
- MySQL 서버를 포함한 DB 서버는 데이터 변경 내용을 로그로 먼저 기록한다. 모든 DBMS는 읽기 성능을 고려한 자료구조를
가지고 있기 때문에 데이터 파일 쓰기는 디스크의 랜덤 엑세스가 필요하다.
- 따라서 데이터 쓰기는 상대적으로 큰 비용이 필요한데 이로 인한 성능저하를 마기 위해 쓰기 비용이 낮은 자료 구조를 가진
리두 로그를 가지고 있으며 비정상 종료시, 리두 로그의 내용을 이용해 데이터를 서버가 종료되기 직전 상태로 복구한다.

#### 4.2.13 InnoDB와 MyISAM, MEMEORY 스토리지 엔진 비교
- MySQL 5.5 부터는 InnoDB 스토리지 엔진이 기본 스토리지 엔진으로 채택됐지만 시스템 테이블은 여전히 MyISAM 테이블을 사용했다.
- MySQL 8.0 부터는 기존 MyISAM이 지원하던 특정기능을 InnoDB가 모두 지원하게 되서 이후 버전에서는 MyISAM 스토리지 엔진은 없어질 것으로 예상된다.
- MyISAM이나 MEMORY 스토리지 엔진의 장점은 MySQL 5.x 버전이라면 의미가 있는 비교겠지만 8.0에서는 더이상 무의미한 
비교이고 8.0 버전에서는 MySQL의 모든 기능이 InnoDB 스토리지 엔진 기반으로 재편됐다.

## 5장. 트랜잭션과 잠금
  
### 5.1 트랜잭션

#### 5.1.1 MySQL에서의 트랜잭션
- 트랜잭션은 쿼리가 실행될때 **적용되거나 아무것도 적용되지 않도록** 보장해주는 메커니즘
- MyISAM과 InnoDB 쿼리 결과
  ![MyISAM과 InnoDB](./images/presentation/myisam_쿼리결과.png)
- 새삼 트랜잭션의 소중함을 깨달음.. 트랜잭션이 없다면 데이터의 정합성을 맞추기가 어렵다.

#### 5.1.2 주의사항
- 프로그램 코드에서 트랜잭션의 범위를 최소화하는게 좋다.
```text
bad. 사용자 로그인 -> 질문등록 -> 트랜잭션 시작 -> 질문유효성 검사 -> DB에 저장 -> 원작자에게 메일 전송 -> COMMIT
good. 사용자 로그인 -> 질문등록 -> 질문유효성 검사 -> 트랜잭션 시작 -> DB에 저장 -> COMMIT -> 원작자에게 메일전송
```

### 5.2 MySQL 엔진의 잠금
- MySQL 엔진레벨과 스토리지 엔진 레벨이 있다.

#### 5.2.1 글로벌 락
- 주로 MyISAM이나 MEMORY 엔진에서 백업시 사용.
- MySQL 8.0 부터는 InnoDB가 기본스토리지 엔진으로 채택되면서 기존의 글로벌 락은 사용하지 않음
- MySQL 8.0 부터는 조금도 가벼운 글로벌 락인 백업 락이 도입.
  - DB 스키마나 인증 정보를 변경할 순 없지만 데이터의 변경은 허용됨
  
#### 5.2.2 테이블 락
- InnoDB 스토리지 엔진에서 레코드 Lock을 제공하기 때문에 스키마를 변경하는 쿼리(DDL)에만 테이블 락이 설정됨.

#### 5.2.3 네임드 락
- 사용자가 지정한 문자열에 대해 락을 획득 및 반납한다.. 그러하다.

#### 5.2.4 메타 데이터 락
- DB 객체의 이름이나 구조를 변경하는 경우 자동으로 획득
- `mysql> RENAME TABLE users To user_back;` 쿼리 실행시 자동획득

### 5.3 InnoDB 스토리지 엔진 잠금
- InnoDB 스토리지엔진은 레코드 기반의 잠금 방식을 사용해서 뛰어난 동시성 처리를 제공한다.
- MySQL 서버의 `information_schema` DB에 존재하는 `INNODB_TRX`, `INNODB_LOCKS`, `INNODB_LOCK_WAITS` 테이블을 조인해서
조회하면 어떤 트랜잭션이 어떤 잠금을 가지고 있고, 어떤 잠금에 의해 대기되고 있는지 정보 파악 가능
![information_scheam](./images/presentation/information_schema.png)
![음](./images/presentation/innodb_locks.png)

### 5.3.1 InnoDB 스토리지 엔진의 잠금

#### 5.3.1.1 레코드 락
- 레코드를 잠그는 것을 레코드락이라고 하며 InnoDB는 레코드 자체가 아니라 인덱스의 레코드를 잠근다는 특징
- 인덱스를 설정하지 않더라도 내부적으로 자동생성된 클러스터 인덱스를 이용해 잠금을 설정

#### 5.3.1.2 갭락
- 레코드와 인접한 레코드 사이의 간격을 잠그는 것.
- 인접한 두 레코드 사이에 새로운 레코드가 생성되는 것을 제어한다.

#### 5.3.1.3 넥스트 키락
- 레코드락 + 갭락을 합쳐놓은 형태의 잠금을 넥스트 키락이라고 한다.
- 갭락이나 넥스트 키락은 MySQL 서버와 복제간의 동일한 데이터를 만들기 위해 사용하며 가능하다면 
바이너리 로그 포맷을 ROW 형태로 바꿔서 넥스트 키락이나 갭락을 줄이는게 좋다.
- MySQL 8.0에서는ROW 포맷의 바이너리 로그가 기본으로 설정됨

#### 5.3.1.4 자동 증가 락(AUTO_INCREMENT 락)
- MySQL에는 AUTO_INCREMENT 칼럼이 사용된 테이블에 동시에 여러 레코드가 삽입되는 경우를 위해
내부적으로 AUTO_INCREMENT 락이라고 하는 테이블 수준의 잠금을 사용한다.
- AUTO_INCREMENT 락은 트랜잭션과 관계없이 INSERT나 REPLACE 문장에서 AUTO_INCREMENT 값을 가져오는 순간에만
락이 걸렸다가 즉시 해제된다. 
- MySQL 5.1 버전 이상부터 innodb_autoinc_lock_mode 시스템 변수를 이용해 자동 증가락의 작동방식을 변경할 수 있다.
  - innodb_autoinc_lock_mode = 0
    - 모든 INSERT 문장은 자동 증가 락을 사용
  - innodb_autoinc_lock_mode = 1
    - INSERT 되는 개수를 정확히 예측이 가능할 때 락대신 훨씬 가볍고 빠른 래치를 이용해 처리.
    - 대량 INSERT 되면 여러개의 자동증가 값을 한번에 할당받아 INSERT 되는 레코드에 사용한다.
      - INSERT 된 대량의 데이터는 자동 증가값이 누락되지 않고 연속적
      - 이후에 INSERT 되는 레코드의 자동증가 값은 연속되지 않고 누락된 값이 발생할 수 있다.
- innodb_autoinc_lock_mode = 2
  - 항상 lock을 사용하지 않고 경량화된 래치를 사용.
  - 이떄의 자동 증가 값은 unique를 보장하지만 연속된 값을 보장하진 않는다.
- `MySQL 5.7 버전에서는 기본값이 1이지만 8.0 버전부터는 기본값이 2로 바뀌었다.`