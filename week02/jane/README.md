# 4. 엔티티 매핑



## @Entity

JPA를 사용하여 테이블과 매핑할 클래스에 사용

- name: JPA에서 사용할 엔티티의 이름 지정, 클래스 이름이 기본값, 다른 패키지에 같은 이름의 엔티티 클래스가 있다면 이름을 지정해서 충돌을 방지해야 함.
- public, protected 기본 생성자 필수
- final 클래스, enum, interface, inner 클래스 사용 불가
- 저장할 필드에 final 선언 불가

#### final이 안되는 이유

1. entity는 database로 persist될 수 있는 클래스인데, 디비에 레코드로 저장되는 값이 final field를 가지고 있는 것은 말이 안 된다.
2.  JPA는 DB에서 데이터를 조회한 후 지연 로딩 전략으로 엔티티를 생성하는데 지연 로딩을 위해 JPA는 엔티티를 상속해서 확장한 클래스인 프록시 객체를 생성한다. 그런데 final class는 상속이 안되므로 JPA가 프록시 객체를 생성할 수 없다. 따라서 final 클래스 사용이 불가하다. 
3. JPA가 프록시 객체를 생성할 때는 Reflection API를 이용하는데 리플렉션을 이용한 객체 생성에서는 기본 생성자를 이용한다. 이후 생성된 프록시 객체의 필드는 setter로 초기화가 이루어지는데 final 필드는 setter로 초기화할 수 없고 클래스 로딩 시점 또는 생성자를 이용해서만 초기화할 수 있다. 따라서 final 필드 사용이 불가하다.



## @Table

엔티티와 매핑할 테이블 지정, 생략 시 매핑한 엔티티 이름을 테이블 이름으로 사용

- name: 매핑할 테이블 이름
- catalog: catalog 기능이 있는 데이터베이스에서 catalog 매핑
- schema: schema 기능이 있는 데이터베이스에서 schema 매핑
- uniqueConstraints: DDL 생성 시에 유니크 제약 조건 생성, 2개 이상의 복합 유니크 제약 조건 설정 가능, 스키마 자동 생성을 사용해서 DDL 만들 때만 사용 됨
  - 컬럼 unique 설정할 수도 있지만 이 경우 이름이 마음대로 지정되어 버림

```java
@Table(uniqueConstraints = {
        @UniqueConstraint(columnNames = {"oauth_type"}, name = "UK_OAUTH_TYPE"),
        @UniqueConstraint(columnNames = {"social_service_id"}, name = "UK_SOCIAL_SERVICE_ID"),
})
```



## 데이터베이스 스키마 자동 생성

- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 &rarr; 객체 중심
- 클래스 매핑 정보와 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL을 만들어 줌 &rarr; 생성된 DDL 운영서버에서 바로 사용 금지

### hibernate.hbm2ddl.auto 속성

- create: drop + create
- create-drop: drop + create + drop
- update: 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정
- validate: 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고 후 애플리케이션 실행 x, DDL 수정 x
- none: 자동 생성 기능 사용 x
- 운영 서버는 validate 또는 none 사용



### hibernate.ejb.naming_strategy

- 이름 매핑 전략 변경

- 하이버네이트는 org.hibernate.cfg.ImprovedNamingStrategy 클래스 제공 &rarr; 테이블 명이나 컬럼 명이 생략되면 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑



## DDL 생성 기능

- DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않음.

- 따라서 스키마 자동 생성 기능을 사용하지 않는다면 사용할 이유가 없지만 애플리케이션 개발자가 제약 조건을 파악하기 쉬우므로 명시해주는 것이 좋음



## 기본 키 매핑

### 직접 할당

기본 키를 애플리케이션에서 직접 할당

- `@Id`
- 자바 기본형, 래퍼형, String, Date, BigDecimal, BigInteger 적용 가능
- em.persist()로 엔티티를 저장하기 전에 애플리케이션에서 기본 키 직접 할당
- 식별자 값 없이 저장하면 하이버네이트 예외인 IdentifierGenerationException 발생

### 자동 생성

대리키 사용

데이터베이스 벤더마다 지원 방식이 다르기 때문에 다양한 전략 존재

#### 전략

- IDENTITY: 기본 키 생성을 데이터베이스에 위임

  - MySQL, PostgresSQL, SQL Server, DB2
  - `@GeneratedValue(strategy = GenerationType.IDENTITY) `
  - JPA는 보통 트랜잭션 커밋 시점에 insert sql을 실행하지만 이 경우에는 em.persist() 시점에 insert sql 실행하여 db에서 식별자를 조회 (쓰기 지연 동작 x)
  - 데이터베이스에 엔티티를 저장하여 식별자 값을 획득한 후 영속성 컨텍스트에 저장
  - JDBC3에 추가된 Statement.getGeneratedKeys() 메서드로 데이터를 저장하는 동시에 생성된 기본 키 값을 얻어올 수 있으며, 하이버네이트는 해당 메서드를 사용하여 데이터베이스와 한 번만 통신한다. 

- SEQUENCE: 데이터베이스 시퀀스를 사용해서 기본 키 할당

  - H2, Oracle, PostgreSQL, DB2
  - 데이터베이스 시퀀스 오브젝트 사용 
    - 데이터베이스 시퀀스 오브젝트: 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트
  - 작동 방식
    1. em.persist()를 호출할 때 데이터베이스 시퀀스를 사용해서 식별자 조회
    2. 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장
    3. 트랜잭션 커밋 & 플러시로 엔티티를 데이터베이스에 저장
  - allocationSize(시퀀스 한 번 호출에 증가하는 수)가 기본 50으로 설정되어 있음 &rarr; 최적화
    - 50으로 설정 시 1~50까지 메모리에서 식별자 할당, 51이 되면 시퀀스 값을 100으로 증가시킨 후 51~100까지 메모리에서 식별자 할당
    - 시퀀스 값을 선점하기 때문에 여러 JVM이 동시에 동작해도 기본 키 값이 충돌하지 않음 &rarr; 동시성 이슈 없음
    - 데이터베이스에 직접 접근하여 데이터를 등록하면 시퀀스 값이 한 번에 많이 증가함

  ```java
  @Entity 
  @SequenceGenerator( 
      name = “MEMBER_SEQ_GENERATOR", 
      sequenceName = “MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름
      initialValue = 1, allocationSize = 1) 
  public class Member { 
      @Id 
      @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQ_GENERATOR") 
      private Long id; 
  }
  ```

  

- TABLE: 키 생성 테이블 사용

  - 시퀀스 대신 테이블을 사용하는 것을 빼면 시퀀스 전략과 동일

    &rarr; 데이터베이스 시퀀스 생성용 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장

  - select 쿼리로 값을 조회하고 값을 증가시키기 위해 update 쿼리 사용

  - 모든 데이터베이스에서 사용할 수 있으나 성능이 떨어짐

  - allocationSize로 최적화 가능

  ```java
  @Entity 
  @TableGenerator( 
      name = "MEMBER_SEQ_GENERATOR", 
      table = "MY_SEQUENCES", 
      pkColumnValue = “MEMBER_SEQ", allocationSize = 1) 
  public class Member { 
      @Id 
      @GeneratedValue(strategy = GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR") 
      private Long id;
  }
  ```

- AUTO: 데이터베이스 방언에 따라 위 세 가지 전략 중 하나 자동 선택



### 기본 키 조건

1. null 값 허용 x
2. 유일해야 한다.
3. 변해선 안 된다. 

- JPA에서는 모든 엔티티에서 대리 키 사용을 권장한다. 
  - 자연 키: 비즈니스에 의미가 있는 키 e.g. 주민등록번호, 이메일, 전화번호
  - 대리 키: 비즈니스와 관련 없는 임의로 만들어진 키 = 대체 키 e.g. 오라클 시퀀스, auto_increment, 키생성 테이블



## @Transient

객체에 임시로 어떤 값을 보관하고 싶을 때 사용

- 매핑 x
- 데이터베이스 저장 및 조회 x
- 메모리에서만 사용



## @Lob

데이터베이스 BLOB, CLOB 타입과 매핑

필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑



## @Access

JPA가 엔티티 데이터에 접근하는 방식 지정

설정 안 할 경우 `@Id`의 위치가 기준이 된다.

프로퍼티: 객체의 고유한 속성

필드: 속성의 실체가 담기는 곳

#### 필드 접근

- AccessType.FIELD

- private field도 접근 가능

#### 프로퍼티 접근

- AccessType.PROPERTY
- Getter 사용



---

***Source***

https://jithub.tistory.com/410

https://stackoverflow.com/questions/3472438/why-cant-entity-class-in-jpa-be-final/3472487

https://www.giorgiosironi.com/2009/12/how-orm-works.html

