# 스프링 부트 쇼핑몰 프로젝트 with JPA
## 2.3.1 상품 엔티티 설계하기

### mysql docker-compose 로 설치
1. docker-compse.yml 파일 작성
```yaml
version: '3' # 파일 규격 버전

services: # 이 항목 밑에 실행하려는 컨테이너 들을 정의
  mysql:
    container_name: mysql # 컨테이너 이름 설정
    image: mysql:latest # 사용할 이미지
    restart: always
    ports:
    - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: shop
      MYSQL_USER: shop
      MYSQL_PASSWORD: 1234
      TZ: Asia/Seoul
    volumes: # -v 옵션 (다렉토리 마운트 설정)
    #- /etc/timezone:/etc/timezone:ro
    #- /etc/localtime:/etc/localtime:ro
    - ./data:/var/lib/mysql
    command:
    - --character-set-server=utf8mb4
    - --collation-server=utf8mb4_unicode_ci
```

2. 도커컴포즈 실행
```bash
docker-compose -d
```

3. shop 유저 권한 주기
컨테이너 외부에서 MySQL에 로그인도 가능해야 하므로 jmlim@localhost에서 localhost 대신 %를 사용한다
```bash
docker exec -it mysql bash
root@f3af78fa6428:/#mysql -u root -p

mysql> GRANT ALL PRIVILEGES ON *.* TO 'shop'@'%';
mysql> flush privileges;
mysql> quit

exit
```


### 엔티티 매핑 관련 어노테이션
| 어노테이션            | 설명                                          |
| -------------------- | --------------------------------------------- |
| `@Entity`            | 클래스를 엔티티로 선언                        |
| `@Table`             | 엔티티와 매핑할 테이블을 지정                 |
| `@Id`                | 테이블의 기본키에 사용할 속성을 지정          |
| `@GeneratedValue`    | 키 값을 생성하는 전략 명시                    |
| `@Column`            | 필드와 컬럼 매핑                              |
| `@Lob`               | BLOB, CLOB 타입 매핑(용어 설명 참조)          |
| `@CreationTimestamp` | Insert 시 시간 자동 저장                      |
| `@UpdateTimestamp`   | Update 시 시간 자동 저장                      |
| `@Enumerated`        | enum 타입 매핑                                |
| `@Transient`         | 해당 필드 데이터베이스 매핑 무시              |
| `@Temporal`          | 날짜 타입 매핑                                |
| `@CreateDate`        | 엔티티가 생성되어 저장될 때 시간 자동 저장    |
| `@LastModifiedDate`  | 조회한 엔티티의 값을 변경할 때 시간 자동 저장 |

### CLOB과 BLOB의 의미
CLOB이란 사이즈가 큰 데이터를 외부 파일로 저장하기 위한 데이터 타입이다
문자형 대용량 파일을 저장하는데 사용하는 데이터 타입이라고 생각하면 된다
BLOB은 바이너리 데이터를 DB 외부에 저장하기 위한 타입이다
이미지, 사운드, 비디오 같은 멀티미디어 데이터를 다룰 때 사용할 수 있다

### `@Column` 어노테이션 추가 속성
| 속성                  | 설명                                                         | 기본값           |
| --------------------- | ------------------------------------------------------------ | ---------------- |
| name                  | 필드와 매핑할 컬럼의 이름 설정                               | 객체의 필드 이름 |
| unique(DDL)           | 유니크 제약 조건 설정                                        |                  |
| insertable            | Insert 가능 여부                                             | true             |
| updatable             | update 가능 여부                                             | true             |
| length                | String 타입의 문자 길이 제약조건 설정                        | 255              |
| nullable(DDL)         | null 값의 허용 여부 설정<br />false 설정 시 DDL 생성 시에 not null 제약조건 추가 |                  |
| columnDefinition      | 데이터베이스 컬럼 정보 직접 기술<br />예) `@Column(columnDefinition = "varchar(5) default'10' not null")` |                  |
| precision, scale(DDL) | BigDecimal 타입에서 사용(BigInteger 가능) precision은 소수점을 포함한 전체 자리수이고, scale은 소수점 자리수<br />Double과 float 타입에는 적용되지 않음 |                  |

### `@GeneratedValue` 어노테이션을 통한 기본키 생성 전략
| 생성 전략                     | 설명                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `GenerationType.AUTO` (default) | JPA 구현체가 자동으로 생성 전략 결정                         |
| `GenerationType.IDENTITY`       | 기본키 생성을 데이터베이스에 위임<br />예) MySQL 데이터베이스의 경우 `AUTO_INCREMENT`를 사용하여 기본키 생성 |
| `GenerationType.SEQUENCE`       | 데이터베이스 시퀀스 오브젝트를 이용한 기본키 생성<br />`@SequenceGenerator`를 사용하여 시퀀스 등록 필요 |
| `GenerationType.TABLE`          | 키 생성용 테이블 사용<br />`@TableGenerator` 필요              |