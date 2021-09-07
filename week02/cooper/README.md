# 1. 엔티티 매핑

- 객체와 테이블 매핑 : `@Entity`, `@Table`
- 필드와 컬럼 매핑 : `@Column`
- 기본 키 매핑 : `@Id`
- 연관 관계 매핑 : `@ManyToOne`, `@OneToMany`

<br>

## 1. 객체와 테이블 매핑

### 1. @Entity

- JPA가 관리하는 엔티티를 정의
- 주의 사항
  1. 기본 생성자는 필수 (public, protected)
  2. final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 속성 (attribute)
  - `name` : 엔티티 이름 지정 (default : 클래스 이름)

<br>

### 2. @Table

- 속성 (attribute)

  - `name` : 매핑할 테이블 이름 (default : 엔티티 이름)

  - `catalog` : catalog 기능이 있는 DB에서 catalog를 매핑

  - `schema` : schema 기능이 있는 DB에서 schema를 매핑

  - `uniqueConstraints` : DDL 생성 시, 유니크 제약 조건을 만든다.

    (복합키 유니크 제약조건도 만들 수 있다.)

    ```java
    @Entity
    @Table(name="MEMBER", uniqueConstraints = {@uniqueConstraints(
    	name = "NAME_AGE_NIQUE",
    	columnNames = {"NAME", "AGE"})})
    public class Member {...}
    ```

<br>

### 3. @Id

- `IDENTITY` : 기본 키 생성을 DB에 위임합니다. 이 방법의 단점은 PK값을 알기 위해서는 DB를 조회해야 합니다.
- `SEQUENCE` : DB sequence를 사용해서 기본 키를 할당합니다.
- `TABLE` : 키 생성 테이블을 생성합니다.

<br>

### 4. @Column

```java
public class Member {

	//nullable, length는 DDL 생성 시 사용하고, JPA 실행 로직에 영향을 미치치 않는다.
	@Column(name="NAME", nullable = false, length = 10)
	private String name;
}
```

**속성(attribute)**

- `name` : 필드와 매핑할 테이블의 컬럼 이름
- `table` : 하나의 엔티티를 두 개이상 의 테이블에 매핑할 때 사용한다.
- `unique` : 컬럼에 제약 조건을 설정할 때 사용한다.
- `length` : 문자 길이 제약 조건(String 타입에만 사용)
- `columnDefinition` : 데이터베이스 컬럼 정보를 직접 줄 수 있다.
- `precision, scale` : BigDecimal(BigInteger) 타입에서 사용

<br>

**주의! 기본타입에 @Column을 추가하는 경우**

```java
public class Member {
	// 생성 DDL : data1 integer not null 
	private int data1;

	// 생성 DDL : data2 integer
	private Integer data2;

	// 생성 DDL : data3 integer
	@Column
	private int data3;

	// 생성 DDL : data4 integer not null
	@Column(nullable = false)
	private int data4;

}
```

자바의 기본 타입은 null을 선언할 수 없고 JPA는 별도의 @Column annotaion을 추가하지 않으면 DDL 생성 시, not null 제약 조건을 추가합니다. 참조 타입의 경우, null을 선언할 수 없고 JPA 또한 nullable = true가 기본 설정입니다. 즉, not null 제약 조건을 추가하지 않습니다.

하지만 만약 **기본 타입에 @Column 에노테이션 선언 시**, @Column 에노테이션의 기본 설정이 nullable = true 이므로 **not null 제약 조건을 생성하지 않습니다.** 그러므로 기본 타입에 @Column 에노테이션을 선언할 경우, `nullable = false`를 선언해야 합니다.

<br>

### 5. @Enumerated

자바의 enum 타입을 매핑할 때, 사용한다.

```java
Enum Grade {
	USER, ADMIN
}

public class Member {
	//DB에 숫자 1로 저장
	@Enumerated(EnumType.ORDINAL)
	private Grade grade = ADMIN;

	//DB에 문자 ADMIN으로 저장
	@Enumerated(EnumType.STRING)
	private Grade grade2 = ADMIN;
}
```

<br>

**EnumType.STRING으로 선언해서 관리하자!!**

ORDINAL의 경우, ENUM의 정렬이 변경될 경우, 값이 틀어질 수도 있습니다. 이를 방지하기 위해 꼭 EnumType.STRING을 사용하도록 합니다.

```java
Enum Grade {
	USER, //(0)
	GRADE //(1)
}

/**
 *    DB
 * | Grade |
 * |   0   | -> USER 반환
 * |   1   | -> GRADE 반환
 *
 */

//순서가 바뀌면 다른 값을 반환한다.
Enum Grade {
	GRADE, //(0)
	USER //(1)
}
/**
 *    DB
 * | Grade |
 * |   0   | -> GRADE 반환
 * |   1   | -> USER 반환
 *
 */
```

<br>

### 6. @Temporal

java.util.Date, java.util.Calender을 매핑할 때 사용하는 에노테이션이다. Java8의 LocalDate와 LocalDateTime의 경우, @Temporal을 선언할 필요가 없다.

```java
@Entity
public class TimeInfo {
	@Temporal(Temporal.DATE) // 날짜
	private Date date;

	@Temporal(Temporal.TIME) // 시간
	private Date time;

	@Temporal(Temporal.TIMESTAMP) //날짜와 시간
	private Date timestamp;
}
```

그리고 데이터베이스 방언을 사용하지만 JPA에서 별도의 코드 변경없이 지원합니다.

- datetime : MySQL
- timestamp : H2, Oracle, PostgreSQL

<br>

### 7. @Transient

필드를 매핑하지 않을 때 사용하는 에노테이션. DB에 저장하지 않고 객체에 임시로 값을 보관하고 싶을 때, 사용한다.

```java
@Transient
private Integer temp;
```

<br><br>

## 2. DB schema 자동 생성

### 1. `hibernate.hbm2ddl.auto` 속성

- 개발 초기 단계 : `create` & `update`
- 테스트 서버 : `update` & `validate`
- 운영 서버 : `validate` & `none`

```yaml
## 1. 기존 DDL 삭제 및 새로 DDL 생성
hibernate.hbm2ddl.auto=create

## 2. 새로 생성 후, 어플리케이션 종료 시 DDL 제거
hibernate.hbm2ddl.auto=create-delete

## 3. DB table와 Entity를 비교해 변경 사항만 수정
hibernate.hbm2ddl.auto=udpate

## 4. DB table과 Entity를 비교해 차이가 있으면 실행 금지
hibernate.hbm2ddl.auto=validate

## 5. 생성 기능 사용X
hibernate.hbm2ddl.auto=none
```

<br>

### 2. hibernate에서 query 확인

```yaml
hibernate.show_sql=true
```

<br><br>

## 2. 연관 관계

### 1. 객체 vs 테이블 연관 관계

객체는 **단뱡향 관계**인 반면에 테이블 연관 관계는 양방향 **연관관계** 입니다. 예를 들면

- 객체의 경우, Member.getTeam()을 통해 팀을 알 수 있지만, Team을 통해서 Member를 알 수 없습니다.

  ```java
  class Member {
  	private Team team;
  	public Team getTeam() {}
  }
  
  class Team {}
  
  member.getTeam();
  // member를 통해서 team 조회 가능
  //team -> member 불가능ㅠ
  ```

- 테이블의 경우, Meber의 TEAM_ID(FK)를 통해서 Member와 Team 정보를 모두 알 수 있습니다.

  ```sql
  SELECT * FROM MEMBER M
  	JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID;
  
  SELECT * FROM TEAM T
  	JOIN MEMBER M ON M.TEAM_ID = T.TEAM_ID;
  ```

<br>

### 2. 양방향 연관관계

- 객체의 양방향 연관관계는 결국 **서로 다른 단방향 관계가 2개**입니다.

- `객체 그래프 탐색` : 객체 참조를 통해 연관관계를 탐색하는 방법을 말합니다.

  ```java
  member.getTeam()
  ```

<br>

### 3. @ManyToOne & @JoinColumn

### `@ManyToOne`

- 다대일 관계에 선언하는 에노테이션이며, 주로 FK를 가지는 주체입니다. (연관관계 주인)
- 속성
  - `optional` : false 설정시 연관된 엔티티가 항상 있어야 한다.
  - `fetch` : 글로벌 패치 절략 설정 (`FetchType.EAGER`, `FetchType.LAZY`)
  - `cascade` : 영속성 전이 기능을 사용

<br>

### `@JoinColumn`

- 외래키 매핑 시, 사용하는 에노테이션
- 속성
  1. `name` : 매핑할 외래키 이름
  2. `referencedColumnName` : 외래키가 참조하는 대상 테이블의 컬럼명
  3. `foreinKey` : 외래 키 제약 조건을 직접 지정할 수 있다.

<br>

## 2. 연관 관계 주인

`연관 관계 주인`은 **외래키(FK)를 관리하는 주체**를 말합니다. 반대로 연관 관계의 주인이 아닌 쪽(`mappedBy`)은 읽기만 가능합니다.  변경 사항을 반영하고 싶은 경우, 연관 관계 주인을 통해서 해야하며 아닌쪽에서 진행할 경우, 변경 사항을 반영하지 않습니다.

하지만 주의사항이 있습니다. 비록 연관관계 주인을 통해서 변경사항을 반영해야 하지만 **연관 관계 주인이 아닌 쪽(`mappedBy`)에 어플리케이션의 데이터 일관성을 위해 변경**해주어야 합니다. 이를 편하게 구현하기 위한 방법이 연관 관계 편의 메서드를 사용합니다. 연관 관계 편의 메서드의 경우, 주인과 주인이 아닌 쪽 모두 설정할 수 있습니다. 하지만 나중에 헷갈리지 않기 위해 일관성 있게 처리하는 것을 추천합니다.

```java
@Entity
public class Member {
	...
	@ManyToOne
	private Team team;

	//연관관계 주인쪽 : 연관 관계 편의 메서드
	public setTeam(Team team) {
		if(this.team != null) {
			Member member = this.team.getMember();
			member.remove(this);
		}
		this.team = team;
		team.getMember().add(this);
	}
}

@Entity
public class Team {
	@OneToMany(mappedBy = "team")
	private List<Member> members = new ArrayList<>(); 

	//연관관계 주인 아닌 쪽 : 연관 관계 편의 메서드
	public addMember(Member member) {
		members.add(member);
		member.setTeam(this);
	}
}
```