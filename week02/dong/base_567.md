# JPA 정리

## 요약
- @Entity : Java Class (Entitiy객체) 와 DB SQL 테이블 간의 매핑
- @Table : 
- @Column : 필드와 컬럼 매핑
- @id : 기본 키 매핑 :
  
- 연관관계 매핑 : @ManyToOne,@JoinColumn





## JPA존재의미 : DB를 Java & OOP 속으로

- DB-table을 java class(Entity)로 대응
- DB의 Column 을 (java) Entitiy Class 의 필드변수 하나로 대응
- 결론 : 자바속으로 DB가 들어오도록
  - JPA를 안쓰면? : DB에게 자바프로젝트가 잡아먹혀 SQL 개발자가 된다





### @Entity

- 기본생성자가 꼭 필요함(아무런 파라미터가 없는 생성자), public/protected 접근제한자

- final 클래스(상속이 불가능한 클래스) 키워드사용 불가능

- enum, interface, inner(nested) 클래스 사용이 불가능

- 저장할 필드에 final 키워드를 사용할 수 없음

- name : JPA에서 사용할 엔티티 이름을 지정하는 어노테이션 필드


```java
@Entity(name = "item_table")
public class Item {
```





### @Table

- name : 매핑할 테이블 이름 엔티티 이름을사용 (default : 엔티티 이름 사용)

- catalog : 데이터베이스 catalog 매핑 (default : DB 이름)
  - 

- schema : 데이터베이스 schema 매핑

- uniqueConstraints : DDL 생성 시에 유니크 제약 조건 생성

***Name\*** : 매핑할 테이블 이름

***Catalog\*** : catalog 기능이 있는 

***Schema\*** : schema 기능이 있는 DB에서 schema를 매핑

***uniqueConstraints\*** : DDL 생성 시 유니크 제약조건을 만듦

   스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용



출처: https://data-make.tistory.com/610 [Data Makes Our Future]

```

```





## JPA 날짜와 시간에 관하여

- @Temporal : 어노테이션 템포럴 쓰면 됨

- 자바에서의 시간 : 8버전 이후 날짜+시간이 합쳐진 LocalDateTime이 있음
  - LocalDateTime
  - ZonedDateTime : 기준시각 맞추는 작업 필요 없고 무조건 zdt에서 타임존 변환
- 디비에서의 시간 : 3가지로 존재
  - 날짜만
  - 시간만
  - 날짜+시간

- LocalDateTime(Java) - DATETIME(RDB) 사이는 서로 자동맵핑되는 사이







## JPA에서 Enum사용하는법

- Enum시간에도 학습했지만, ordinal 사용은 금지 - 엄청난 서버장애 / 데이터 밀리는 현상 경험 가능
  - 기선 : ordinal을 숫자를 지정해서 (1, 10, 23, 77) 이런식으로 띄엄띄엄 배치하는 스킬로 쓸수는 있음
- 무조건! 꼭! OrdinalType.STRING 을 명시해줘야함
- 결론 : JPA(RDB)에는 VARCHAR로 저장됨!



### DB 마이그레이션..?

- 이미 라이브서버로 서비스는 돌아가고 있는데 갈아엎고 싶다..

- VARCHAR로 저장되는데 이 이름이 마음에 안들어(CARD >> IBK_CARD, SINHAN_CARD)







## 아주 큰 데이터 

### blob

### clob







## 엔티티에서 무시할(저장하지 않을) 필드변수

### @transient 

- 엔티티로 데이터베이스에 저장되지 않는 필드변수(컬럼이 생성되지 않음)
- 주로 메모리에서만 쓰고 저장하지 않을 변수들을 사용



### @JsonInclude

- Response DTO 에서 사용하는데 조회 요청시에서 그때마다 계산해서 줘야할 데이터들



```java
//1:1 관계의 판다 이름을 알고싶으면
@JsonProperty("panda_name")
public String getCompanyName() {
    return Optional.ofNullable(panda)
    	.map(PandaDto::getName)
      .map(ZooDtoReq::getName)
      .orElse(null);
}



//1:N 관계의 bamboo의 총합을 알고싶을때
@JsonProperty("totalPrice")
public BigDecimal getTotalContractPrice() {
	return purchases.stream()
    .filter(bamboo -> !bamboo.getIsDeleted()) // 필터로 삭제된건지 하나씩 체크한다음에
    .map(purchase -> BigDecimals.of(bamboo.getPricePerOne())  // 개당가격을
    .multiply(BigDecimal.valueOf(Optional.ofNullable(bamboo.getCount()).orElse(0))) // 개당가격 * 갯수 = 총합계
    .val())
    .reduce(BigDecimal.ZERO, BigDecimal::add);
}
    
```







## Unique 제약조건 거는법

### @Column 에서 유니크 제약조건 걸어주기

- 사용하지 않음! 그 이유는...
  - 이름 반영이 안됨 : 이상한 이름으로 들어감
  - 여러 필드를 조합한 복합유니크 키로 사용이 불가능 (이 기능은 테이블 어노테이션에서 사용)



### @Table 에서 유니크키 제약조건





## PrimaryKey 맵핑

- @Id : 어노테이션을 사용해서 pk(기본키)를 맵핑함

  - 직접 만들수도, 자동생성(auto incre) 모두 가능

- @GenVal 아시죠?

- 기본키는 4가지 방법이 있는데

  - Auto : DB에 위임, default
  - IDentity
  - Sequence

  - ??

- 기본키가 int가 아니라 Long인 이유 

  - 데이터 11억건이 넘어갈때의 위험성때문에
  - 성능상 int사용시의 이점이 크지 않음

- sequence Generator에서 성능최적화가 있는데 나중에 다룸
  - 테이블전략 : 키성능을 높이기 위해 전용의 테이블을 만들어 시퀀스를 흉내~ (뭔지 모르겠음..)



- 얼로케이션 사이즈 : 시퀀스전략 사용시 >> 시퀀스 테이블에서 미리 시퀀스id 생성해서 반영
  - 시나리오1 ; 시퀀스테이블에서 미리 만들어서 반영 : 테이블전략
  - 시나리오2 : 시퀀스전략, 시퀀스테이블 안쓰고 시퀀스 오브젝트 사용해서 
  - 시퀀스 내부에 테이블에 다른 오브젝트가 뭔가를 한다!?!?





DDL 옵션

- DDL은 개발 장비에서만 사용
- SQL과 테이블 중심 -> 객체 중심으로 관점 변화











 

## TMI - DBMS 시스템 카탈로그란?



### DBMS 시스템 카탈로그란?

- DBMS에서 시스템 카탈로그는 **DB 시스템 그 자체에 관련이 있는 다양한 객체에 관한 정보**를 포함하는 DB시스템 스스로를 을 위한 DataBase 입니다!

- 시스템 카탈로그 내의 각 테이블은 DB사용자를 포함하여 DBMS에서 지원하는 모든 **데이터 객체에 대한 정의나 명세에 관한 정보를 담기위해 만들어놓은 DB내부의 테이블** 이며 DB가 자체적으로 관리함
- **데이터 정의어(DDL)의 결과**로 구성되는 기본 테이블, 뷰, 인덱스, 패키지, 접근 권한 등의 데이터베이스 구조 및 통계 정보를 저장
- 카탈로그는 데이터베이를 위한 일종의 **메타 데이터** 
  - 정보를 저장하는 DB시스템만을 위한 
  - DB시스템을 위해 존재하는 데이터(메타-데이터)



### 카탈로그의 특징

- 카탈로그 시스템 자체도 일반 데이터들과 같은 테이블로 구성됨
  - 일반 이용자도 SQL 쿼리문으로 카탈로그 내부의 내용을 **검색** 가능

- 단,  INSERT, DELETE, UPDATE등의  **DML로 시스템 카탈로그를 직접 갱신은 불가능**
  - DML(Data Manipulation Language) - 데이터 조작어 :  INSERT, DELETE, UPDATE, SELECT
  - SELECT 문으로 조회는 가능

- 조작방법 (카탈로그의 update) : 사용자가 SQL문을 실행시켜 기본 테이블, 뷰, 인덱스 등에 변화를 주면 시스템이 자동으로 갱신된다
- 데이터 정의어(DDL)을 통해서만 시스템 카탈로그를 갱신할 수 있다.
- 데이터베이스 시스템 제품에 따라 천차만별
- 시스템 카탈로그는 **DBMS가 스스로 생성하고 유지** 되는 부분임









## TMI - DBMS 키워드 정리

- 스키마 : 데이터가 저장되는 형식을 결정하는 전반적인 구조 
  - 시간이 지나도 데이터베이스 스키마는 드물게 변경
-  인스턴스 : 특정 시점에서의 데이터베이스에 저장된 정보의 집합
- 데이터 모델
  - 사용 가능한 데이터 만을 선별하여 구조화된 데이터베이스에 저장, 사용하기 위한 과정 이며
  - 의미, 데이터 타입, 연산 및 제약조건을 명시하기 위해 사용할 수 있는 개념들의 집합 - 예: 관계형 모델, ER모델, 객체지향적 모델
  - 실세계의 일부분을 DBMS가 지원하는 데이터 모델의 형태로 나타내는 과정 

- 인스턴스 - 특정 시점에서의 데이터베이스에 저장된 정보 집합
  - 시간이 지남에 따라 데이터베이스 인스턴스는 지속적으로 변화 

- ER 모델 - 실세계 인식에 기초하여 실세계의 객체(object)를 나타내는 개체(entity)들과 개념들 간의 관계(relationship)로 구성 

- 관계형 모델(relational model) - 릴레이션이라고 하는 표 현태의 데이터 구조를 사용하여 데이터를 저장 
  - relation(릴레이션) : RDB의 Table, **``테이블 그 자체``** 를 의미함!!
  - relationship(릴레이션쉽) : 테이블사이에서의 **관계**
- 객체지향적 모델 : 객체들을 기반으로 데이터베이스를 구성 > 캡슐화를 유지, 프로그램과 데이터간의 불일치 감소 > 자유로운 사용자 정의 데이터 타입 - 빠른 DBMS 접근속도 - 객체는 유일한 OID로 구현 - DBMS는 OID를 이용 객체에 직접 접근
  - *OID*: *Object* Identifier : *객체* 식별자 , 일반적으로 PK 사용







## TMI - RDB 특징

* 데이터베이스 시스템의 자기 기술성
  * 데이터베이스시스템을 위한 정보는 데이터 데이터베이스 자체에(내부) 저장되어 있음
  * 데이터베이스가 내부적으로 쓰기 위한 정보들을 : 메타데이터(데이터를위한 데이터)
  * 메타데이터를 사용하는 특징을 : 자기기술성



- 프로그램과 데이터의 격리/분리 = 데이터 추상화 

  - APP : 처리, 흐름, 전체통합을 담당

  - DB : 데이터만 잘 저장하는 일을 담당

  - APP (웹이든 뭐든) 개발이 결국 데이터 잘 저장해서 공유, 시스템 복잡도를 낮춤

    

- 데이터의 공유와 다수 사용자 트랜잭션 처리
  - 데이터베이스는 공유 가능
    - 다수 사용자를 전제로 만들어진 DBMS
  -  동시에 수행되는 트랜잭션들이 상호 방해를 받지 않고 정확하고 효율적으로 수행되도륵 보장







## TMI - 데이터베이스 객체(OBJECT)

- DATABASE OBJECT 란?
  - 내에 존재하는 논리적인 저장 구조/구성 요소들
  - DBMS가 데이터를 관리&관리하기 위해 필요한 모든 논리적인 저장 구조를 만드는 데이터베이스 객체들
- DATABASE OBJECT 요소

| DataBase's Object   | EXPLAIN                                                      |
| ------------------- | ------------------------------------------------------------ |
| TABLE               | 데이터 담고 있는 객체                                        |
| VIEW                | 하나 이상의 테이블 연결해서 마치 테이블인 것처럼 사용하는 객체 |
| INDEX               | 테이블에 있는 데이터를 바르게 찾기 위한 객체                 |
| SYNONYM(시노님)     | 데이터베이스 객체에 대한 별칭을 부여한 객체                  |
| SEQUENCE            | 순차적으로 자동 증가하는 컬럼 객체                           |
| FUNCTION            | 특정 연산을 하고 값을 반환하는 객체                          |
| PROCEDURE(프로시져) | 함수와 비슷하지만 값을 반환하지 않는 객체                    |
| PACKAGE             | 용도에 맞게 함수나 프로시저 하나로 묶어 놓은 객체            |



### SEQUENCE (시퀀스 객체)

- 유일(UNIQUE)한 값을 생성해주는 오라클 객체이다.
- 시퀀스 오브젝트를 생성하면 순차적으로 자동 증가하는 컬럼을 생성한다
  - 주로 PK(기본키)의 Auto-Increment에 사용 
- 메모리에 Cache되었을 때 SEQUENCE 값의 액세스 효율이 증가
- SEQUENCE는 테이블과는 독립적으로 저장되고 생성됨
  - 따라서 하나의 SEQUENCE를 여러 테이블에서 쓸 수 있다

- JPA 시퀀스객체 관련한 내용
