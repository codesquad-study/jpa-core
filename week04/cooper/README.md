# 자바 ORM 표준 JPA 프로그래밍 섹션 9~11

# chapter9. 값 타입

## 1. 엔티티 타입 & 값 타입

### 1. 엔티티 타입

1. @Entity로 정의하는 객체

2. 데이터가 변해도 식별자로 지속해서 추적 가능

   ex)  회원 엔티티 나이가 변해도 **식별자로 인식** 가능

<br>

### 2. 값 타입

1. 값으로 사용하는 자바 기본 타입이나 객체
2. 식별자가 없고 값만 존재해서 변경시 추적 불가

ex) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체

<br>

## 2. 값 타입 분류

### 1. 기본 값 타입

- 자바 기본 타입, 래퍼 클래스, String

- 생명주기를 엔티티에 의존

- 값 타입을 공유하면X

  → 회원 이름 변경시, 다른 회원 이름도 함께 변경되면 안됨!

  ```java
  public static void main(String[] args) {
  		Integer a = new Integer(5);
  		Integer b = a;
  
  		b = 30; //a도 30으로 변경된다.
  }
  ```



<br>

### 2. 임베디드 타입(embedded type, 복합 값 타입)



<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/d57c3fd2-c6d0-4746-95c3-a642fd06dd76/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T095803Z&X-Amz-Expires=86400&X-Amz-Signature=9e0194f9bba844b038c4ae1574896a8baeed30306725475473fafb35f4f9e54c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="50%"></center>



- 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편 번호를 사용한다.
- 근무 시작일, 근무 종료일 : 객체 하나로 사용하면 되지 않을까?
- 주소 도시, 주소 번지, 주소 우편 번호 : 객체 하나로 사용하면 되지 않을까?

<br>

## 3. 임베디드 타입의 장점

1. 재사용
2. 높은 응집도
3. 값 타입만 사용하는 의미있는 메소드 선언 가능.
   - 용어 코드 공통화할 수 있어 장점이 있다.

<br>

## 4. 임베디드 예시

### (1) 예시 1

```java
@Entity
public class Member {
	@Embedded
	private Period workPeriod;

	@Embedded
	private Address homeAddress;
}

@Embeddable
public class Period {
	private LocalDateTime startTime;
	private LocalDateTime endTime;
}

@Embeddable
public class Address {
	private String city;
	private String street;
	private String zipcode;
}
```

<br>

### (2) 예시 2 : value type도 entity를 가질 수 있다.  (FK만 가지고 있으면 되기 때문)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/809aabc3-6747-4fb7-a346-855adf07f503/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T100148Z&X-Amz-Expires=86400&X-Amz-Signature=e15118b7a5cdd8b80a9dad8c82f1fc0df0500356d08f19c86992851f58bec92d&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22) 

```java
@Embeddable
public class Address {
	//member entity가 가능하다.
	private Member member;
}
```

<br>

### (3) 두 Addrss를가질 때, 해결법

```Java
@Entity
public class Member {
	
	@Embedded
		@AttributeOverrides({name="city",
			column=@Column("WORK_CITY")),
		@AttributeOverride(name="street",
			column=@Column("WORK_STREET")),
		@AttributeOverride(name="zipcode",
			column=@Column("WRO"))
	})
	private Address homeAddress;
	private Address awayAddrss;
}
```

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/425c6ec9-77bb-480b-96f5-4eb2fb7c53c4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T100307Z&X-Amz-Expires=86400&X-Amz-Signature=9a7e7b252d2730cfaebcf8432b31da7cae1a82bb003effb39153a5506c70f6a0&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

<br>

## 5. 값 타입 공유 참조

- 임베디드 타입 같은 `값 타입을 여러 엔티티에서 공유하면 안됨.`

- 만약 같은 값으로 공유하고 싶을 경우, `Entity로 선언`해야 한다.

- side effect가 발생하고 에러를 발견하기 어렵다.

- 이를 해결하기 위해서는 `값을 복사해서 사용`해야 한다.

  ```java
  Address address = new Address("city", "street", "zipcode");
  Address copiedAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());
  
  Member member2 = new Member();
  member2.setHomeAddress(copiedAddress);
  ```

<br>

## 6. 객체 타입의 한계

- 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없어 객체의 공유 참조는 피할 수 없다.

  ```java
  //primitive type
  int a = 10
  int b = a;
  b = 4;
  
  Address a = new Address("Old");
  Address b = a;
  b.setCity("New");
  //a 주소 => New
  //b 주소 => New
  ```

- 이를 해결하기 위해서는 `불변 객체`를 사용한다!

<br>

## 7. 불변 객체

- 객체 타입을 수정할 수 없게 만들면 공유 참조와 같은 부작용을 사전에 차단할 수 있다.

- 값 타입은 꼭 불변 객체(immutable object)로 설계해야 한다.

- 불변 객체로 선언 시, 값을 공유하더라도 부작용이 발생하지 않는다.

- **`불변 객체` : 생성 시점 이후, 절대 값을 변경할 수 없는 객체**

  - 생성자로만 값을 설정하고 수정자를 선언하지 않는다.
  - Integer, String이 자바에서 제공하는 대표적인 불변 객체

- 값을 변경하고 싶은 경우, 객체를 새로 만들어야 한다.

  ```java
  Address address = new Address(address.getCity(), address.getStreet(), address.getZipcode());
  member.setHomeAddress(newHomeAddress);
  ```

<br>

## 8. 값 타입 컬렉션

### 1_ 값 타입 컬렉션의 고민

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8ddcc95f-27ae-4dfe-b973-e8c3f2df525c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T100744Z&X-Amz-Expires=86400&X-Amz-Signature=1e8831273c41e50581b4429e94e98165562117d6b699ac0ca5023c166ec5a173&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="50%"></center>

- DB 테이블로 구현할 때가 문제다.

  - 하나일 때는 Column으로 추가하면 되지만,
  - 컬렉션 형태로 추가가 불가능하므로 이를 해결하기 위해서는 별도의 테이블을 뽑아야 한다. → 이 때의 경우, 값 타입을 다 묶어서 `PK로 선언한 테이블`을 뽑아내면 된다.

  <br>

### 2_ 값 타입 컬렉션의 예시

```java
@Entity
public class Member {

	@ElementCollection
	@CollectionTable(name = "FAVORITE_FOOD", joinColumns =
		@JoinColumn(name = "MEMBER_ID")
	@Column(name = "FOOD_NAME")//해당 값은 값이 하나기 때문에 예외적으로 선언할 수 있다.
	private Set<String> favoriteFoods = new HashSet<>();

	@ElementCollection
	@CollectionTable(name = "ADDRESS", joinColumns = 
		@JoinColumn(name = "MEMBER_ID")
	private List<Address> addresseHistory = new ArrayList<>();
}
```

### 1_값 타입 추가

```java
try {
	member.getHomeAddress(new Address("homeCity", "street", "zipcode");

	member.getFavoriteFoods().add("치킨"); //insert into FAVORITE_FOOD (MEMBER_ID, FOOD_NAME) values (?, ?);
	member.getFavoriteFoods().add("피자"); //insert into FAVORITE_FOOD (MEMBER_ID, FOOD_NAME) values (?, ?);
	member.getFavoriteFoods().add("족발"); //insert into FAVORITE_FOOD (MEMBER_ID, FOOD_NAME) values (?, ?);

	member.getAddressHistory().add(new Address("old1", "old_street1", "old_zip1"); //insert into ADDRESS_HISTORY (MEMBER_ID, CITY, STREET, ZIPCODE) values (?, ?, ?, ?);
	member.getAddressHistory().add(new Address("old2", "old_street2", "old_zip2"); //insert into ADDRESS_HISTORY (MEMBER_ID, CITY, STREET, ZIPCODE) values (?, ?, ?, ?);
	member.getAddressHistory().add(new Address("old3", "old_street3", "old_zip3"); //insert into ADDRESS_HISTORY (MEMBER_ID, CITY, STREET, ZIPCODE) values (?, ?, ?, ?);

} catch (Exception e) {
}
```

- 값 타입은 독자적인 라이프 사이클을 가지고 있지 않는다. 현재 케이스의 경우, Member Entity의 라이프 사이클에 의존적이므로 Member의 변경 사항을 인지해 값 타입의 객체가 생성, 삭제된다.
- `cascade = CascadeType.ALL, orphanRemove=true` 와 같다.

<br>

### 2_값 타입 조회

- @Embedded로 선언된 객체의 경우, 즉시 로딩을 지원하는 반면,
- @ElementCollection은 지연 로딩을 지원한다.

<br>

### 3_값 타입 수정

- 값 타입은 불변으로 관리해야 한다.

- 수정하기 위해서는 새로운 객체로 값 객체를 변경해야 한다.

- 객체 하나의 경우, 새로운 객체로 변경해서 사용하도록 하고

  ```java
  findMember.setHomeAddress(new Address("newCity", a.getStreet(), a.getZipcode());
  ```

- 컬렉션의 경우, Collection 원소를 제거하고 다시 추가해야 한다.

  ```java
  findMember.getFavoriteFoods().remove("치킨");
  findMember.getFavoriteFoods().add("한식");
  ```

- 컬렉션은 remove 제거 메서드 호출시, 기본적으로 동등성 비교를 통해 원소를 제거하므로 `equals ovveride`를 주의하자.

<br>

### 4_값 타입 수정 시, 수정 쿼리 빌런

- Collection 원소를 제거, 추가방식의 로직으로 코드를 작성해서 확인해보니 AddressHistory(Collection)을 통째로 제거하고 다시 추가하는 현상이 발견되었다.

- 이 원인은 값 타입은 식별자 개념이 존재하지 않아 값 수정시 추적이 어려워 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.

- `@OrderColumn`을 사용할 수 있지만 이것은 사용하지 말자.

- 해결법! : 값 타입 컬렉션을 매핑하는 테이블은 **`모든 컬럼을 묶어서 기본키를 구성`**해야 한다 : null입력 X, 중복 저장 X

- 실무 : 값 타입 컬렉션 대신 값 타입을 엔티티로 승격시켜 일대다 관계로 관리한다.

  ```java
  @Entity
  public class AddressEntity {
  	@Id
  	@GeneratedValue
  	private Long Id;
  
  	private Address address;
  }
  ```

  

  ```java
  @Entity
  public class Member {
  
  	@Id
  	@GeneratedValue
  	private Long id;
  	
  	//영속성 전이(Cascade) + 고아 객체 제거를 사용해서 해결
  	@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
  	private List<AddressEntity> addressHistory = new ArrayList<>();
  }
  ```

<br>

### 5_값 타입을 언제 사용하지

- `다중 선택`할 때, 사용한다. (ex. `해쉬태그 다중 선택`, `주소 이력`, `좌표`)
- 웬만하면 엔티티로 사용하도록 하자.

<br><br>

# 10. 객체 지향 쿼리 언어 1 - 기본 문법

## 1. JPQL

- 검색을 할 때도 테이블이 아닌 `엔티티 객체를 대상으로 검색`해야 한다.
- JQPL은 SQL을 추상화하므로 `특정 데이터베이스 SQL에 의존하지 않는다.`
- JPQL은 `엔티티` 객체 / SQL은 `DB` 대상으로 쿼리를 작성한다고 생각하면 된다.
- 동작 방식 : 먼저 JPQL이 실행하고 이를 번역한 SQL을 생성해 실행된다.

예시

```java
String jpql = "select m from Member m where m.name like '%kim%'";
List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```

<br>

## 2. 기본 문법

### (1) TypeQuery, Query

- `TypeQuery` : 반환 타입이 명확할 때 사용
- `Query`  : 반환 타입이 명확하지 않을 때 사용 (ex. `username : String`, `age : int`)

```java
TypeQuery<Member> query = em.createQuery("select m from Member m", Member.class);
Query query  = em.createQuery("select m.username, m.age from Member m");
```

<br>

### (2)결과 조회

- `query.getResultList()` : 결과가 하나 이상일 때, 리스트 반환
  - 결과가 없으면 빈리스트 반환.
- `query.getSingleResult()` : 결과가 정확히 하나, 단일 객체 반환
  - 결과가 없으면 `javax.perssitence.NoresultException`
  - 둘 이상이면 `javax.perssitence.NonUniqueResultException`

<br>

### (3) 파라미터 바인딩 - 이름 기준, 위치 기준

```java
Member result = em.createQuery("selct m from Member m where m.username = :username", Member.class)
				.setParameter("username" "member1") //메서드 체이닝으로 제공
				.getSingleResult();
```

<br>

## 3. 프로젝션

- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상 : `엔티티`, `임베디드 타입`, `스칼라 타입`(숫자, 문자등 기본 데이터 타입)
- `SELECT m FROM Member m` → 엔티티 프로젝션
- `SELECT m.team From Member m` → 엔티티 프로젝션
- `SELECT m.addrss FROM Member m` → 임베디드 타입 프로젝션
  - 임베디드 타입의 경우, 소속된 엔티티가 존재해야 하므로 엔티티에 종속적인 한계가 있다.
- SELECT m.username, m.age FROM Member m → 스칼라 타입 프로젝션

Q. em.createQuery를 실행하면 영속성 컨텍스트에서 관리하는 것인가?

→ `entity projection`을 선언하면 `EntityContext`에서 관리한다.

주의! **Foreign key**를 사용해야 하는 경우, `명시적 조인`을 하도록 하자!

```java
em.createQuery("select t from Member m join m.team t", Team.class).getResultList();
```

<br>

### (2) 프로젝션 - 여러 값 조회

```sql
SELECT m.username, m.age FROM Member m
```

1. query타입으로 조회
2. Object[] 타입으로 조회
3. new 명령어로 조회

<br>

```java
List<UserDTO> resultList = (SELECT new jpabootk.jpql.UserDTO(m.username, m.age) FROM Member m).getResultList();
```

- 단순 값을 DTO로 바로 조회
- 패키지 명을 포함한 전체 클래스 명 입력.
- 순서와 타입이 일치하는 생성자 필요

<br>

## 4. 페이징 API

- JPA는 페이징을 다음 두 API로 추상화
- `setFirstResult(int startPosition)` : 조회 시작 위치 (0부터 시작)
- `setMaxResults(int maxResult)` : 조회할 데이터 수

```java
String jpql = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
		.setFirstReulst(10)
		.setMaxResults(20)
		.getResultList();
```

- 동작 방식

  → DB 방언 방식을 작동한다. = DB마다 작성 방식이 다르다.

  - 주로 MySQL을 사용하므로 MySQL 방언만 기억하고 있자!

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7775e447-d88b-403d-809b-84d2ca980142/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T102015Z&X-Amz-Expires=86400&X-Amz-Signature=24f5ace8de3a615155dbf1cc82e91a642ddb462aac2a52d74581fe2277537437&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="50%"></center>

<br>

## 5. 조인

### (1) 조인 종류

- 내부 조인

  ```java
  SELECT m FROM Member m INNER JOIN m.team t;
  ```

- 외부 조인

  ```java
  SELECT m FROM Member m LEFT JOIN m.team t;
  ```

- 세타 조인 : 연관 관계없는 조인을 모든 조인을 다 조인한다. → `cross join`

  ```java
  SELECT count(m) FROM Member m, Team t WHERE m.username = t.name;
  ```

<br>

### (2) 조인 - ON절

- ON절은 활용한 조인(JPA 2.1 ~)

1. 조인 대상 필터링
2. 연관관계 없는 에티티 외부 조인 (hibernate 5.1 ~)

### 1. 조인 대상 필터링

ex) 회원과 팀을 조인하면서, 팀이름이 A인 팀만 조인

- JPQL

  ```java
  SELECT m, t FROM Member m LEFT JOIN m.team t ON t.name = 'A'
  ```

- SQL

  ```java
  SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id AND t.name='A';
  ```

<br>

### (3) 연관관계 없는 엔티티 외부 조인

ex) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

- JPQL

  ```java
  SELCT m, t FROM Member m LEFT JOIN Team t ON m.username = t.name;
  ```

- SQL

  ```java
  SELECT m.*, t.* FROM Member m LEFT JOIN t ON m.username = t.name;
  ```

<br>

## 6. 서브 쿼리

### (1) 서브 쿼리

- 서브 쿼리 : 쿼리 내부에 추가적인 쿼리를 추가하는 것을 의미.
- 메인 쿼리와 서브 쿼리와 연관성이 없어야 성능이 잘나온다

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/98d771a5-22e1-42c1-bc65-ca97afca3e86/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T102335Z&X-Amz-Expires=86400&X-Amz-Signature=2955d1f2622dc7f5591384f0a9ef3910fbfb2550cc9362c1d038bbec305d86c2&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="50%">

<br>

### (2) 서브 쿼리 지원 함수

- `EXISTS` : 서브쿼리에 결과가 존재하는 참
- {ALL | ANY | SOME} (subquery)
  - `ALL` : 모두 만족하면 참
  - `ANY`, `SOME` : 같은 의미, 조건을 하나라도 만족하면 참
- [NOT] `IN` (subquery) : 서브쿼리 결과 중 하나라도 같은 것이 있으면 참

<br>

### (3) JPA 서브 쿼리 한계

- JPA는 WHERE, HAVING절에서만 서브 쿼리 사용 가능

- SELECT에서 가능 (하이버네이트에서 지원)

  ```java
  SELECT (SELECT AVG(m1.age) FROM Member m1) | FROM Member m join Team t ON m.username = t.name;
  ```

- `FROM절의 서브 쿼리`는 현재 JPQL은 불가능

  - `조인`으로 풀 수 있으면 풀어서 해결 (혹은 쿼리를 두개를 합치는 경우)

<br><br>

## 7. JPQL 타입 표현

- `문자` : 'HELLO', 'She is'

- `숫자` :  10L(Long), 10D(Double), 10F(Float)

- `Booolean` : TRUE, FALSE

- `ENUM` : jpabook.MemberType.Admin (패키지명 포함)

  → QueryDSL은 import만 받아서 사용하면 됨.

- `엔티티 타입` : TYPE(m) = Member (상속 관계에서 사용)

**예시**

```java
String query = "SELECT m.username, 'HELLO', true FROM Member " + 
				"WHERE m.type = jpql.MemberType.ADMIN";
List<Object[]> result = em.createQuery(query).getResultList();
```

<br>

### JPQL 기타

- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, ≥, <, ≤, <>
- BETWEEN, LIKE, IS NULL

<br><br>

# 11. 객체 지향 쿼리 언어 2 - 중급 문법

## 1. 경로 표현식

### (1) 경로 표현식

- (점)을 찍어 객체 그래프를 탐색하는 것

  ```sql
  SELECT m.username, -- 상태 필드
  	FROM Member m
  	JOIN m.team t -- 단일 값 연관 필드
  	JOIN m.orders o -- 컬렉션 값 연관 필드
  	WHERE t.name = '팀A';
  ```

<br>

### (2) 경로 표현식 용어 정리

- 상태 필드(state filed) : 단순히 값을 저장하기 위한 필드
- 연관 필드(association field) : 연관관계를 위한 필드
  - 단일 값 연관 필드 : `@ManyToOne`, `@OneToOne`, 대상이 엔티티(ex. m.team)
  - 컬렉션 값 연관 필드 : `@OneToMany`, `@ManyToMany`, 대상이 컬렉션(ex. m.orders)
- 단일 값 연관 경로 : 묵시적 내부 조인(inner join) 발생, 탐색O
- 컬렉션 값 연관 경로 : 묵시적 내부 조인 발생, 탐색X

<br>

### (3) 경로 표현식 특징

- **상태 필드** : 경로 탐색 끝, 탐색 X

- **단일 값 연관 경로** : 묵시적 내부 조인(inner join) 발생, 탐색 O

- 컬렉션 값 연관 경로

   : 묵시적 내부 조인 발생, 탐색X

  - FROM 절에서 `명시적 조인`을 통해 별칭을 얻으면 별칭을 통해서 탐색 가능

    ```sql
    Integer result = em.createQuery(query, Integer.class).getSingleResult();
    ```

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4928c8ec-3eee-43d4-8fea-cbfde1649d6e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T102933Z&X-Amz-Expires=86400&X-Amz-Signature=ace40ed5fe816d99ff9dca71f6784d5d7cdc1c172b156da6b99e2c38c9480825&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="50%"></center>



### (4) 실무 조언

- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움으로 가급적 명시적 조인을 사용하자.
- 조인은 SQL 튜닝에 포인트

<br>

## 2. 페치 조인(fetch join)

### ` 실무에서 엄청 중요하다`

### (1) 페치 조인(fetch join)

- SQL 조인 종류 X
- JPQL에서 **성능 최적화**를 위해 제공하는 기능.
- 연관된 엔티티나 컬렉션을 **SQL 한번에 함께 조회**하는 기능 → `한방쿼리`로 풀어낼 때 사용
- join fetch 명령어 사용
- 페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로

<br>

### (2) 엔티티 페치 조인

- 회원을 조회하면서 연관된 팀도 함께 조회(`SQL 한번에`!)

- SQL을 보면 회원 뿐만 아니라 **팀(T.*)**도 함께 **SELECT**

- JPQL → Member m을 명시했지만

  ```sql
  SELECT m FROM Member m JOIN FETCH m.team;
  ```

- SQL → 실제 쿼리는 `EAGER LOADING`형태로 데이터를 호출한다.

  ```sql
  SELECT M.*, T.* FROM Member M INNER JOIN TEAM T ON M.TEAM_ID=T.ID;
  ```

<br>

### (3) 예시

<center><img src="https://user-images.githubusercontent.com/48561660/135071227-f7375526-96e5-437d-b10a-cd074164b384.png" width="50%"></center>

- `SELECT m FROM Member m` 실행 시

```sql
---------< 이름1 출력 부분>---------
Hibernate: 
    select
        team0_.team_id as team_id1_3_0_,
        team0_.name as name2_3_0_ 
    from
        Team team0_ 
    where
        team0_.team_id=?
member = 이름1 팀1

---------< 이름2 출력 부분>---------
member = 이름2 팀1

---------< 이름3 출력 부분>---------
Hibernate: 
    select
        team0_.team_id as team_id1_3_0_,
        team0_.name as name2_3_0_ 
    from
        Team team0_ 
    where
        team0_.team_id=?
member = 이름3 팀2
```

- `SELECT m FROM Member m JOIN FETCH m.team`

  ```sql
  =======================
  Hibernate: 
      /* SELECT
          m 
      FROM
          Member m 
      JOIN
          FETCH m.team */ select
              member0_.id as id1_0_0_,
              team1_.team_id as team_id1_3_1_,
              member0_.age as age2_0_0_,
              member0_.name as name3_0_0_,
              member0_.team_id as team_id4_0_0_,
              team1_.name as name2_3_1_ 
          from
              Member member0_ 
          inner join
              Team team1_ 
                  on member0_.team_id=team1_.team_id
  ---------< 이름1 출력 부분>---------
  member = 이름1 팀1
  
  ---------< 이름2 출력 부분>---------
  member = 이름2 팀1
  
  ---------< 이름3 출력 부분>---------
  member = 이름3 팀2
  ```

<br>

### (4) 1:N 관계(Collection) 데이터를 조회 시, 주의사항

**문제 상황. Team에서 member를 조회하고 싶은 경우**

```java
@Entity
public class Team {
		...
    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Member> members = new ArrayList<>();
		...
}
```

- collection의 경우, 같은 정보가 여러개 받아와 데이터 중복이 이루어진다

  ```java
  Hibernate: 
      /* SELECT
          t 
      FROM
          Team t 
      JOIN
          FETCH t.members */ select
              team0_.team_id as team_id1_3_0_,
              members1_.id as id1_0_1_,
              team0_.name as name2_3_0_,
              members1_.age as age2_0_1_,
              members1_.name as name3_0_1_,
              members1_.team_id as team_id4_0_1_,
              members1_.team_id as team_id4_0_0__,
              members1_.id as id1_0_0__ 
          from
              Team team0_ 
          inner join
              Member members1_ 
                  on team0_.team_id=members1_.team_id
  ---------< 팀1 출력 부분>---------
  member = 팀1 2
  member = Member{id=4, name='이름1', age='null', team=jpa.basic.domain.Team@6d467c87}
  member = Member{id=5, name='이름2', age='null', team=jpa.basic.domain.Team@6d467c87}
  
  ---------< 팀1 출력 부분>---------
  member = 팀1 2
  member = Member{id=4, name='이름1', age='null', team=jpa.basic.domain.Team@6d467c87}
  member = Member{id=5, name='이름2', age='null', team=jpa.basic.domain.Team@6d467c87}
  
  ---------< 팀2 출력 부분>---------
  member = 팀2 1
  member = Member{id=6, name='이름3', age='null', team=jpa.basic.domain.Team@a567e72}
  ```

  

- 이를 해결하기 위해서는 `distinct`를 통해서 해결한다.

<br>

### (5) 페치 조인과 DISTINCT

- SQL의 DISTINCT는 중복된 결과를 제거하는 명령

- JPQL의 DISTINC 2가지 기능 제공

  1. SQL에 DISTINC를 추가
  2. 애플리케이션에서 `엔티티 중복` 제거 → 어플리케이션 레벨에서 처리한다.

- 결과

  ```java
  ---------< 팀1 출력 부분>---------
  member = 팀1 2
  member = Member{id=4, name='이름1', age='null', team=jpa.basic.domain.Team@6d467c87}
  member = Member{id=5, name='이름2', age='null', team=jpa.basic.domain.Team@6d467c87}
  
  ---------< 팀2 출력 부분>---------
  member = 팀2 1
  member = Member{id=6, name='이름3', age='null', team=jpa.basic.domain.Team@a567e72}
  ```

- SELECT DISTINCT t FROM Team t JOIN FETCH t.members

  - SQL에 DISTINCTㄷ를 추가하지만 **`데이터가 다르므로 SQL 결과에서 중복제거 실패`**

  - DISTINCT가 추가적으로 어플리케이션에서 중복 제거 시도 → 같은 식별자를 가진 Team엔티티 제거

    <center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a272a13e-0ac1-4c92-85dd-efc3f6e5a782/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210928%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210928T104012Z&X-Amz-Expires=86400&X-Amz-Signature=22a3bdd4300d293e35aaadefa2b1d1ba51e3bc74812c21366fc856348b0cc1d7&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="50%"></center>

    

<br>

### (6) 페치 조인과 일반 조인의 차이

- 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않음.

- 페치 조인 실행시 `연관된 엔티티를 함께 조회`해온다.

  - 일반 조인 사용시

    ```sql
    Hibernate: 
        select
            team0_.team_id as team_id1_3_0_,
            team0_.name as name2_3_0_ 
        from
            Team team0_ 
        where
            team0_.team_id=?
    ```

  - 페치 조인 사용시

    ```sql
    Hibernate: 
        /* SELECT
            m 
        FROM
            Member m 
        JOIN
            FETCH m.team */ select
                member0_.id as id1_0_0_,
                team1_.team_id as team_id1_3_1_,
                member0_.age as age2_0_0_,
                member0_.name as name3_0_0_,
                member0_.team_id as team_id4_0_0_,
                team1_.name as name2_3_1_ 
            from
                Member member0_ 
            inner join
                Team team1_ 
                    on member0_.team_id=team1_.team_id
    ```

    

- 페치 조인을 사용할 때만 연관된 엔티팉도 함꼐 조회(즉시 로딩)

- **`페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념`**

<br>

### (7) 페치 조인의 특징과 한계

- `별칭 사용 금지` : SELECT DISTINCT t FROM Team t JOIN FETCH t.members `as m` → 금지

- 부분 조회 시, fetch join은 사용하지 않는다.

  (hibernate의 사상은 연관관계의 데이터를 한번에 모든 테이터를 가지고 오기 위함이다.)

- `둘 이상의 컬렉션`은 페치조인을 할 수 없다.

  - (team → members, orders를 가지고 있는 경우. members, orders 페치조인 금지)

- 컬렉션 페치조인 시, `페이징 API`를 사용할 수 없다. → 일대다 관계에서는 작동하지 않는다.

- 해결법

  1. `일대일`,`다대일`과 같은 단일 연관 필드는 페치 조인해도 페이징 가능.

  2. 컬렉션을 사용해야 하는 경우, `@BatchSize`를 사용하도록 한다.

     ```sql
     @BatchSize(size = 100)
     privvate List<Member> members = new ArrayList<>();
     ```

  → 실무에서는 @BatchSize 대신 글로벌 세팅을 사용한다.

  ```sql
  <property name = "hibernate.jdbc.default_batch_fetch_size" value = "100"/>
  ```

<br>

### (8) 페치 조인 - 정리

- 모든 것을 페치 조인으로 해결할 수 없다.
- 페치 조인은 객체 그래프를 유지할 때, 사용하면 효과적이다. (member.getTeam().~~)
- 여러 테이블을 조인해서 `엔티티가 가진 모양이 아닌 전혀 다른 결과`를 내야 하면, 페치 조인 보다는 `일반 조인`을 사용하고 필요한 데이터들만 조회해서 `DTO로 반환`하는 것이 효과적이다.

<br>

## 3. 다형성 쿼리

### (1) 다형성 쿼리

- 조회 대상을 특정자식 한정

  ex) Item 중에 Book, Movie를 조회하라.

[JPQL]

```sql
SELECT i FROM ITEM i WHERE type(i) IN (Book, Movie)
```

[SQL]

```sql
SELECT i FROM i WHERE i.DTYPE IN ('B', 'M')
```

### (2) Treat (JPA 2.1)

ex) 부모인 Item과 자식 Book이 있다.

[JPQL]

```sql
SELECT i FROM ITEM i WHERE TREAT(i as Book).author = 'kim';
```

[SQL]

```sql
SELECT i.* FROM ITERM i WHERE i.DTYPE='B' AND i.author = 'kim';
```

<br>

## 4. 엔티티 직접 사용 - 기본 키 값

- JPQL에서 `엔티티를 직접 사용`하면 SQL에서 해당 `엔티티의 기본 키 값`을 사용

→ `엔티티` 혹은 `식별자`를 전달해도 같은 결과를 반환한다.

[JPQL]

```sql
SELECT count(m.id) FROM Member m //엔티티의 아이디를 사용
SELECT count(m) FROM Member m //엔티티를 직접 사용
```

[SQL] → JPQL은 같은 쿼리가 실행된다

```sql
SELECT count(m.id) AS cnt FROM Member m
```

## 엔티티 직접 사용 - 외래키 값

```sql
Team team = em.find(Team.class, 1L);
String qlString = “select m from Member m where m.team = :team”; List resultList = em.createQuery(qlString)
.setParameter("team", team) .getResultList();
String qlString = “select m from Member m where m.team.id = :teamId”; List resultList = em.createQuery(qlString)
.setParameter("teamId", teamId) .getResultList();
```

[SQL]

```sql
select m.* from Member m where m.team_id=?
```

<br>

## 5. Named쿼리

- `@NamedQuery`를 선언하고 명칭을 부여하면 해당 쿼리를 선언할 때, 별칭을 통해 쿼리를 실행시킬 수 있다.

### (1) Named 쿼리 - 정적 쿼리

- 어노테이션, XML에 정의
- 애플리케이션 로딩 시점에 초기화 후 `재사용`
- 어플리케이션 로딩 시점(컴파일 시점)에서 `쿼리 검증`

<br>

### (2) In JpaRepository

```sql
@Query("SELECT u FROM User u WHERE u.emailAddress = ?1")
User findByEmailAddress(String emailAdderss);
```

→ 이런식으로 작성할 경우에도 파싱하여 해당 쿼리를 로딩시점에서 확인할 수 있다.

<br>

## 6. 벌크 연산

- SQL의 `update`, `delete`라고 생각하면 된다.

> 예제. 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?

​		→ JPA변경 감지 기능으로 실행하려면 너무 많은 SQL실행

(변경된 데이터가 `100건`이라면 `100번의 update sql` 실행;;)

```sql
1. 재고가 10개 미만인 상품 리스트를 조회한다.
2. 상품 엔티티의 가격을 10% 증가한다.
3. 트랜잭션 커밋시점에 변경감지가 동작한다.
```

<br>

### 1. excuteUpdate()의 결과는 영향받은 엔티티 수 반환

- `UPDATE`, `DELETE` 지원

- INSERT(insert into..., select, 하이버네이트 지원)

- 실행 결과

  ```java
  String sql = "UPDATE Member m SET m.age = 20";
  entityManager.createQuery(sql).executeUpdate();
  ```

  

  ```sql
  Hibernate: 
      /* UPDATE
          Member m 
      SET
          m.age = 20 */ update
              Member 
          set
              age=20
  ```

<br>

### 2. 벌크 연산 주의

- 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리
  - `벌크 연산`을 먼저 실행
  - 벌크 연산 수행 후, `영속성 컨텍스트 초기화`

→ persist 안하고 excuteUpdate 실행 후, getAge()를 조회하면 0

(영속성 컨텍스트에 반영되지 않는다. → 데이터 정확성이 안맞을 수도 있다.)

<br>

