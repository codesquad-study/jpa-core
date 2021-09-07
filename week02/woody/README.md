## << 엔티티 매핑 >>

## ⭐️ 엔티티

###  @Entity

- JPA가 관리하는, DB 테이블과 매핑되는 엔티티라는 뜻

**주의**

- JPA를 사용해서 테이블과 매핑할 클래스는 **@Entity** 필수
- **기본 생성자 필수**(파라미터가 없는 public 또는 protected 생성자)
- 저장할 필드에 **final 사용 X**

</br>

## ⭐️ 데이터베이스 스키마 자동 생성 == DDL 생성 기능

- (+) 객체중심으로 DDL을 생성할 수 있다는 장점

- (+) 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성

</br>

### DDL 자동 생성 속성

`hibernate.hbm2ddl.auto`

| 옵션        | 설명                                          |
| ----------- | --------------------------------------------- |
| create      | 기존 테이블 삭제 후 다시 생성 (DROP + CREATE) |
| create-drop | create과 같은 기능 + 종료 시점 DROP           |
| update      | 변경한 부분만 반영(운영 DB에서는 사용X)       |
| validate    | 엔티티와 테이블이 정상 매핑되었는지만 확인    |
| none        | 사용 안함                                     |



- 실행 시에는 영향을 주지 않고 초기에 DDL을 작성할 때만 영향을 준다.

  - 즉, JPA의 실행 로직에는 영향을 주지 않음

  

🚨 **주의하라~~**

- 자동 생성 DDL은 개발 장비에서만 사용
  - 운영 서버에서는 저얼대 create, create-drop, update 사용 X

</br>

## ⭐️ 매핑 어노테이션

```java
@Entity
 public class Member {
     @Id
     private Long id;
     @Column(name = "name")
     private String username;
     private Integer age;
     @Enumerated(EnumType.STRING)
     private RoleType roleType;
     @Temporal(TemporalType.TIMESTAMP)
     private Date createdDate;
     @Temporal(TemporalType.TIMESTAMP)
     private Date lastModifiedDate;
     @Lob
     private String description;
 }
```



### **@Column : 컬럼 매핑**

- name : 이름 지정
- insertable : 등록 여부 결정
- updatable : 변경 가능 여부 결정
- nullable(DDL) : null 여부 결정
- unique(DDL) : 유니크 제약 조건
- length(DDL) : 문자열 길이 제약 조건, only for String type
- precision, scale(DDL) : BigDecimal 혹은 BigInteger 타입에 사용
  - precision은 소수점을 포함한 전체 자릿수
  - scale은 소수의 자릿수
  - double, float 타입에는 적용되지 않는다
- columnDefinition (DDL) : 데이터 베이스 컬럼 제약조건을 직접 문자열로 입력 가능

### **@Temporal : 날짜 타입 매핑**

날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용

LocalDate, LocalDateTime을 사용할 때는 생략 가능

- TemporalType.DATE : 데이터베이스 date 타입 (2021-09-07)
- TemporalType.TIME : 데이터베이스 time 타입 (14:33:11)
- TemporalType.TIMESTAMP : 데이터베이스 timestamp 타입 (2021-09-07 14:33:11)

### **@Enumerated : enum 타입 매핑**

자바 enum 타입을 매핑할 때 사용

- 기본값이 `EnumType.ORDINAL`이므로 꼭 `EnumType.STRING`으로 변경하자

### **@Lob : BLOB, CLOB 매핑**

- 필드 타입이 문자일 경우 **CLOB**
- 나머지 **BLOB**

### **@Transient : 특정 필드를 컬럼에 매핑하지 않음(매핑 무시)**

- 필드 매핑X → 데이터베이스에 저장X, 조회X
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

</br>

## ⭐️ 기본 키 매핑 방법

직접 할당: **@Id만 사용**자동 생성(**@GeneratedValue**)

- **IDENTITY**: 데이터베이스에 위임, MYSQL

- SEQUENCE

  : 데이터베이스 시퀀스 오브젝트 사용, ORACLE

  - @SequenceGenerator 필요

- TABLE

  : 키 생성용 테이블 사용, 모든 DB에서 사용

  - @TableGenerator 필요**AUTO**: 방언에 따라 자동 지정, 기본값

### IDENTITY 전략

- 기본 키 생성을 데이터베이스에 위임
  - 데이터베이스에 인서트를 해봐야 ID 값을 알 수 있다
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
- 그래서 IDENTITY 전략 사용하면 em.persist() 호출 시점에 즉시 INSERT 쿼리문을 실행하고 DB에서 식별자를 조회해 가지고 온다

### SEQUENCE 전략

- 값을 저장할 때 시퀀스 테이블에서 id 값만 생성한뒤 반환
- 커밋 시점이 되어서야 만들어진 id값과 함께 엔티티를 저장하는 쿼리문이 날아간다
  - insert 쿼리문 버퍼링 가능(한꺼번에 보낼 수 있다)
  - insert 쿼리문이 날아가기 전에는 시퀀스 테이블에 id만 만들어 저장하고 반환받으면 됨

### SEQUENCE 전략 - 최적화 기능

- `@SequenceGenerator` 옵션에서 allocationSize를 50으로 맞춰놓으면 시퀀스 테이블에서 50개의 시퀀스 id를 미리 생성하고 리턴한다
- 미리 만들어놓은 시퀀스 id를 다 사용했을 때 다시 시퀀스 테이블에 50개의 시퀀스 id를 생성 및 반환한다(이 과정 반복)
  - ⭐️  네트워크 통신량이 적어진다
  - id 값을 한 번에 받아오기 때문

### !! 식별자 전략

- 변하지 않는 자연키 거의 없다 (중간에 PK 바뀌는 상황이 오면 ... ㅠ)
- auto increment / identity 옵션, UUID 혹은 랜덤 문자열을 통해 최대한 대체키를 사용하자

비즈니스 관련 데이터를 키로 가지고 오는 것은 적절하지 못하다!! 

</br>

## << 연관관계 매핑 >>



## ⭐️ 단방향 연관관계

### If, 객체를 테이블에 맞추어 모델링한다면..😥

- 참조 대신 왜래키를 그대로 사용한다.

  ```java
  @Entity
  public class Member {
  
  		private Long id; 
  		
  		@Column(name = "USERNAME") 
  		private String name;
  		
  		@Column(name = "TEAM_ID")
  		private Long teamId; // 👈👈
  		 
  }
  
  @Entity
  public class Team {
  		@Id @GeneratedValue
  		private Long id; 
  
  		private String name;
  }
  ```



### 🚨 이때 발생하는 문제

- 연관관계가 없기 때문에 id 값을 직접 다뤄야 한다.

```java
//팀 저장
Team team = new Team();
team.setName("TeamA"); 
em.persist(team);

//회원 저장
Member member = new Member();
member.setName("member1"); 
member.setTeamId(team.getId()); // setTeam()처럼 객체를 다루지 않고, team의 id 값을 직접 다룬다
em.persist(member);

//조회
Member findMember = em.find(Member.class, member.getId());

//연관관계가 없음
Team findTeam = em.find(Team.class, team.getId()); // Member와 별개로 조회해야 한다
```



### ✅ 객체 지향 모델링(객체의 참조와 테이블의 외래 키를 매핑)

```java
@Entity
public class Member {

private Long id; 
@Column(name = "USERNAME") private String name; private int age;

@ManyToOne
@JoinColumn(name = "TEAM_ID") // Team과 조인시 어떤 컬럼을 기준으로 조인해야하나? TEAM_ID!!
private Team team;
```



```java
//팀 저장
Team team = new Team();
team.setName("TeamA"); 
em.persist(team);

//회원 저장
Member member = new Member();
member.setName("member1"); 
member.setTeamId(team.getId()); // setTeam()처럼 객체를 다루지 않고, team의 id 값을 직접 다룬다
em.persist(member);

//조회
Member findMember = em.find(Member.class, member.getId());

//연관관계가 있기 때문에 
// Member와 Team을 조인해서 둘 다 한꺼번에 들고온다
Team findTeam = findMember.getTeam();
```

</br>

## ⭐️ 양방향 연관관계 

### 객체 연관관계는 2개

- 회원 → 팀 연관관계 (단방향)
- 팀 → 회원 연관관계 (단방향)

👉  객체의 양방향 관계는 서로 다른 단방향 관계 2개



### 테이블 연관관계는 1개

- 회원 ↔  팀 (양방향)

외래키 하나로 두 테이블의 연관관계 관리

예를 들어 MEMBER.TEAM_ID 왜래키 하나로 두 테이블에 접근 가능

```sql
select * from member m join team t on m.team_id = t.team_id

select * from team t join member m on t.team_id = m.team_id
```



### Member와 Team 객체가 MEMBER 테이블을 둘다 매핑? NOPE 둘 중 하나로 외래키를 관리해야 한다

- 연관관계의 주인 Member (외래 키가 있는 곳이 주인)
- Team에서 members는 조회용! (mappedBy)





### 순수한 객체 관계를 고려하면, 양쪽 다 값을 입력해주어야 한다

```java
Team team = new Team(); 
team.setName("TeamA"); 
em.persist(team);

Member member = new Member(); 
member.setName("member1");

team.getMembers().add(member); //👈👈 연관관계의 주인에 값 설정
member.setTeam(team); //👈👈 
em.persist(member);
```



- 순수 객체 상태(DB와 싱크 되기 전의 상태)를 고려해서 항상 양쪽에 값을 설정하자
- 연관관계 편의 메소드를 생성하자
- 단! 양방향 매핑시에 무한 루프를 조심하자
  - toString(), lombok : 왠만하면 쓰지마
  - JSON 생성 라이브러리 : 컨트롤러에서 엔티티 절대 반환하지 마! 그럼 문제 생기지 않는다!

```java
// Member 클래스 내부 //

// << 연관관계 편의 메소드 >>
// 하나의 메소드에서 양방향 세팅을 한꺼번에 // 양방향 세팅 과정에서 원자성 보장
public void setTeam(Team team) { 
		this.team = team;
		team.getMembers().add(this);
}
```

- getter/setter와 헷갈리지 않게 하기위해서 setTeam() 대신 changeTeam() 등 다른 네이밍을 사용

- [참고] 원자성 : 트랜잭션의 작업이 부분적으로 실행되거나 중단되지 않는 것을 보장



### 언제 양방향? 언제 단방향?

- 단방향 매핑만으로도 이미 연관관계 매핑은 완료
- 단방향 매핑을 잘 하고 양방향은 필요할 때 추가해도 됨 (테이블에 영향을 주지 않음)