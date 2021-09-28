## 값 타입



### 엔티티 타입

@Entity 로 정의하는 객체

- 데이터가 변해도 식별자로 지속해서 추적 가능



### 값 타입

단순히 값으로 사용하는 자바 기본타입이나 객체

- 식별자가 없고 값만 있으므로 변경시 추적 불가
- 생명주기를 엔티티에 의존
  - 회원 삭제 ->  회원 이름, 나이 삭제
- 값 타입은 공유하면 X
  - 회원 이름 변경  ->  다른 회원 이름 변경 안돼



#### 🚨값 타입 공유 참조 주의 : 반드시 불변 객체로

- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 사이드 이펙트 발생할 수도

- **Solution**

  - 불변 객체(생성 시점 이후 절대 값을 변경할 수 없는 객체)
  - 생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 됨

  

#### 🚨 값 타입 비교

- 값 타입은 equals()를 통해 동등성 비교를 해야한다
  - 값들을 비교해야 한다

> 동일성 (identity) : 인스턴스 참조 비교 (`==`)
>
> 동등성(equivalence) : 인스턴스 값 비교 (`equals()`)



### 값 타입 분류 1 - **기본값 타입**

#### 자바 기본타입, 래퍼 클래스, String

- 자바의 기본 타입(primitive type) 은 공유 X 

  - 값을 복사하기 때문

  - 👍 **사이드 이펙트가 없다**

</br>

### 값 타입 분류 2 - **임베디드 타입**(embedded type, 복합 값 타입)

새로운 값 타입을 직접 정의할 수 있음

```java
@Embedded
private Address homeAddress;

// 임베디드 타입(값 타입) 객체
@Embeddable
public class Address { 
    private String city;
    private String street;
    private String zipcode;
}
```



<img src="./imgs/임베디드.png" alt="임베디드" style="zoom:50%;" />

#### 특징

- 테이블은 값 타입으로 묶든 안묶든 똑같이 필드를 가진다
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음

#### 임베디드 타입 연관관계

- 값 타입 객체 has 값 타입
- 값 타입 객체 has 엔티티 객체

#### @AttributeOverrid : 속성 재정의

- 한 엔티티에서 같은 값 타입을 사용하기위해 사용

- ```java
  @Embedded
  private Address homeAddress;
  
  @Embedded
  @AttributeOverrides({
          @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
          @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
          @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
  })
  private Address workAddress;
  ```

  - **실행 결과** 👉  city, street, zipcode 필드와 별개로 WORK_CITY, WORK_STREET, WORK_ZIPCODE 필드가 새로 생성된다

</br>

### 값 타입 분류 3 - **컬렉션 값 타입**(collection value type)

값 타입을 하나 이상 저장하려면 컬렉션에 저장할 때 사용

- @ElementColletion, @CollectionTable 사용
  - @CollectionTable 생략 시 기본 값을 사용해 매핑 ({엔티티이름}_{컬렉션 속성 이름})
- 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없기 때문에 별도의 테이블 생성해서 구현



#### 예시 1.  String 타입 컬렉션

```java
@ElementCollection
@CollectionTable(name = "FAVORITE_FOODS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
@Column(name = "FOOD_NAME")
private Set<String> favoriteFoods = new HashSet<String>();
```

```sql
create table FAVORITE_FOODS (
       MEMBER_ID bigint not null,
       FOOD_NAME varchar(255)  // @Column 미작성 시 필드 이름 "favoriteFoods"
    )
    
```



#### 예시 2.  Address 타입 컬렉션

```java
@ElementCollection
@CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
private List<Address> addressHistory = new ArrayList<Address>();
```

```sql
create table ADDRESS (
       MEMBER_ID bigint not null, // PK, FK
        city varchar(255),  	// PK
        street varchar(255),  // PK
        zipcode varchar(255)  // PK
    )
```



> ? 왜 모든 Address 테이블 필드가 PK인지

1. 저장

   - 값 타입 컬렉션의 생명 주기는 멤버에 의존적

   - 때문에 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다

   - ```java
     Member member1 = new Member("member1", 20);
     member1.getFavoriteFoods().add("치킨");
     member1.getFavoriteFoods().add("족발");
     em.persist(member1); // member 객체를 persist 할 때, 추가했던 값 타입 컬렉션들도 한꺼번에 반영
     ```

     

2. 조회

   - 값 타입 컬렉션도 지연 로딩 전략 사용

     ```java
     Member findMember =  em.find(Member.class, member1.getId());
     
     final List<Address> addressHistory = findMember.getAddressHistory();
     for(Address address : addressHistory){
         System.out.println("address = " + address.getCity());
     }
     
     final Set<String> favoriteFoods = findMember.getFavoriteFoods();
     for (String favoriteFood: favoriteFoods) {
         System.out.println("favoriteFood = " + favoriteFood);
     }
     ```

     ```sql
     Hibernate: 
         select
             addresshis0_.MEMBER_ID as MEMBER_I1_0_0_,
             addresshis0_.city as city2_0_0_,
             addresshis0_.street as street3_0_0_,
             addresshis0_.zipcode as zipcode4_0_0_ 
         from
             ADDRESS addresshis0_ 
         where
             addresshis0_.MEMBER_ID=?
     address = city1
     address = city2
     Hibernate: 
         select
             favoritefo0_.MEMBER_ID as MEMBER_I1_1_0_,
             favoritefo0_.FOOD_NAME as FOOD_NAM2_1_0_ 
         from
             FAVORITE_FOODS favoritefo0_ 
         where
             favoritefo0_.MEMBER_ID=?
     favoriteFood = 족발
     favoriteFood = 치킨
     favoriteFood = 피자
     ```

     

3. 삭제

   - 값 타입은 엔티티와 다르게 식별자 개념이 없다 → 값을 변경하면 추적이 어렵다

   - 값 타입 컬렉션에 변경사항이 발생하면, **주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다**

   - 때문에 값 타입 컬렉션을 매핑하는 테이블은 **모든 컬럼을 묶어서 기본 키를 구성해야 한다**

     - null 입력 X, 중복저장 X

   - ```java
     findMember.getAddressHistory().remove(new Address("city1", "street1", "10000"));
     findMember.getAddressHistory().add(new Address("new city1", "new street1", "10000"));
     ```

     ```sql
     Hibernate: 
         /* delete collection Member.addressHistory */ delete 
             from
                 ADDRESS 
             where
                 MEMBER_ID=?
     Hibernate: 
         /* insert collection
             row Member.addressHistory */ insert 
             into
                 ADDRESS
                 (MEMBER_ID, city, street, zipcode) 
             values
                 (?, ?, ?, ?)
     Hibernate: 
         /* insert collection
             row Member.addressHistory */ insert 
             into
                 ADDRESS
                 (MEMBER_ID, city, street, zipcode) 
             values
                 (?, ?, ?, ?)
     ```



#### 삭제에서 발생하는 문제를 해결하기 위해서

- 값 타입 컬렉션 대신 일대다 관계를 고려

  ```java
  @Entity
  public class AddressEntity {
  
      @Id
      @GeneratedValue
      private Long id;
  
      public AddressEntity() {}
  
  ```

  

</br>



### JPQL

- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 유사한 문법
  - SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원

- JPQL은 엔티티 객체를 대상으로 쿼리(객체 지향 SQL)
  - SQL을 추상화해서 특정 데이터베이스 SQL에 의존 X
  - SQL은 데이터베이스 테이블을 대상으로 쿼리

#### 기본 JPQL 문법

```sql
select_문 :: =
        select_절
        from_절
        [where_절]
        [groupby_절]
        [having_절]
        [orderby_절]

update_문 :: = update_절 [where_절]
delete_문 :: = delete_절 [where_절]
```

- 엔티티와 속성은 대소문자 구분O (Member, age)
- JPQL 키워드는 대소문자 구분X (SELECT, FROM, where)
- 엔티티 이름 사용
  - 테이블 이름이 아님(Member)
- 별칭은 필수(m)
  - as는 생략가능

#### 결과 조회 API

**query.getResultList()**

결과가 하나 이상일 때 리스트 반환

- 결과가 없으면 빈 리스트 반환
- NPE 걱정 없다

**query.getSingleResult()**

결과가 정확히 하나일 때 단일 객체 반환

- Exception 발생 우려(결과 없거나 둘 이상일 경우)



### 파라미터 바인딩 - 이름 기준, 위치 기준

- 위치 기준
  - 파라미터 순서 바뀌면 사이드 이펙트 발생 우려
- 이름 기준
  - 이걸 사용하자

</br>

### 프로젝션 (SELECT)

- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상 : 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자 등 기본 데이터 타입)
- DISTINCT로 중복 제거



#### 엔티티 프로젝션

```sql
SELECT m FROM Member m
SELECT m.team FROM Member m
```

- 조회된 모든 엔티티들이 영속성 컨텍스트에서 다 관리가 된다

  ```java
  Member member = em.createQuery("select m from Member m", Member.class)
  									.getResultList();
  Member findMember = result.get(0);
  findMember.setAge(20); // 영속성 컨텍스트에서 관리하므로 자동으로 수정사항이 반영된다
  ```



#### 임베디드 타입 프로젝션

```sql
SELECT m.address FROM Member m
```



#### 스칼라 타입 프로젝션

```sql
SELECT m.username, m.age FROM Member m
```



### Join 문법

- 조인은 명시적으로 하자, 예상하지 못한 결과가 나올 수도 있으므로!

```java
//  모두 똑같은 쿼리문 나옴
select t from Member m // 묵시적 조인 
select t from Member m join [m.team](<http://m.team>) t // 명시적 조인
```



#### 여러 값 조회

1. **Query 타입으로 조회**

```java
List resultList = em.createQuery("select m.usernaem, m.age from Member m")
										.getResultList(); // 타입을 모르기 때문에 제너릭 없이 List 형태로 받음
Object o = resultList.get(0);
Object[] result = (Object[]) o;
```

2. **Object[] 타입으로 조회**

```sql
List<Object[]> result = em.createQuery("select m.usernaem, m.age from Member m")
										.getResultList(); // 타입을 모르기 때문에 제너릭 없이 List 형태로 받음
```

3. **new 명령어로 조회**

- 단순 값을 DTO로 바로 조회

  ```java
  SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m
  ```

- 패키지명을 포함한 전체 클래스명 입력

- 순서와 타입이 일치하는 생성자 필요

</br>

### 페이징 API

- 페이징을 API로 추상화
- DB 방언에 따라 다른 SQL 문을 만들어 수행

```java
//페이징 쿼리
String jpql = "select m from Member m order by m.name desc"; 
List<Member> resultList = em.createQuery(jpql, Member.class)
																	.setFirstResult(10)
																	.setMaxResults(20) 
																	.getResultList();
```

**setFirstResult(int startPosition)**

- 조회 시작 위치

**setMaxResults(int maxResult)**

- 조회할 데이터 수

</br>

### 조인 종류

1. **내부 조인**

```sql
SELECT m FROM Member m [INNER] JOIN m.team t
```

- Member 있고 Team 없다 → 데이터 안가져옴

2. **외부 조인**

```sql
SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
```

- ​	Member 있고 Team 없다 → Team null, Member 데이터 가져옴

3. **세타 조인**

```sql
select count(m) from Member m, Team t where m.username = t.name
```

- 연관관계가 없는 두 테이블을 조인



### 조인 - ON 절

JPA 2.1 부터 지원

1. **조인 대상 필터링**

   예) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인

   ```java
   // JPQL
   SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A' 
   
   // SQL
   SELECT m.*, t.* FROM 
   Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and t.name='A'
   ```

2. **연관관계 없는 엔티티 외부 조인**

   예) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

   ```java
   // JPQL
   SELECT m, t FROM
   Member m LEFT JOIN Team t on m.username = t.name 
   
   // SQL
   SELECT m.*, t.* FROM 
   Member m LEFT JOIN Team t ON m.username = t.name
   ```

> ? 그럼 세타 조인이랑 연관관계 없는 엔티티의 외부 조인 복잡도가 같은건가?



### 그밖의 JPQL 타입 표현

- **문자**: ‘HELLO’, ‘She’’s'

- **Boolean**: TRUE, FALSE

- **숫자**: 10L(Long), 10D(Double), 10F(Float)

- **ENUM**: jpabook.MemberType.Admin (패키지명 포함)

  - ```java
    String query = "select m.username, 'HELLO', true from Member m where m.type = :userType";
    
    List<Object[]> result = em.createQuery(query)
    														.setParameter("userType", MemberType.ADMIN)
    														.getResultList();
    ```

    

- **엔티티** **타입**: TYPE(m) = Member (상속 관계에서 사용)

  - ```java
    // 상속관계 매핑 시
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn
    public abstract class Item { }
    
    @Entity
    @DiscriminatorValue("BB")
    public class Book extends Item { }
    
    List<Object[]> result = em.createQuery("select i from Item i where type(i) = Book", Iteam.class).getResultList();
    ```

- SQL과 문법이 같은 식

</br>

### 조건식 - CASE 식

**JPA 문법**

- COALESCE: 하나씩 조회해서 null이 아니면 반환

  - 예) 사용자 이름이 없으면 이름 없는 회원을 반환

    ```java
    select coalesce(m.username,'이름 없는 회원') from Member m
    ```

- NULLIF : 두 값 이 같으면 null 반환, 다르면 첫번째 값 반환

  - 예) 사용자 이름이 ‘관리자’면 null을 반환하고 나머지는 본인의 이름을 반환

    ```java
    select NULLIF(m.username, '관리자') from Member m
    ```



### 경로 표현식

- 점을 찍어 객체 그래프를 탐색하는 것

### 필드 종류

1. **상태 필드(state field)**

   - 단순히 값을 저장하기 위한 필드

   - (ex: m.username)

2. **연관 필드(association field)**

   - 연관관계를 위한 필드

   2-1. **단일 값 연관 필드**

   - @ManyToOne, @OneToOne, 대상이 엔티티
   - (ex: m.team)

   2-2. **컬렉션 값 연관 필드**

   - @OneToMany, @ManyToMany, 대상이 컬렉션
   - (ex: m.orders)



### 경로 표현식 특징

**상태 필드(state field)**

- 경로 탐색의 끝, 탐색X
  - ex) `m.username`
  - → 더 이상 점을 찍어 다른 객체 그래프를 탐색할 수 없다

**단일 값 연관 경로**

- 묵시적 내부 조인(inner join) 발생, 탐색O
  - 쿼리 튜닝이 어렵다, 예상치 못한 조인 쿼리문이 발생할 수 있기 때문
  - ex) `select m.team from Member m`

**컬렉션 값 연관 경로**

- 묵시적 내부 조인 발생, 탐색X

  - ex) `select t.member from Team t`

  ```java
  String query = "select t.members from Team t";
  
  Collection result = em.createQuery(query, Collection.class)
  										.getResultList();
  ```

- FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색 가능

  - ex) `select m.address, m.name from Team t join t.members m`
  - 별칭 m을 통해 그래프 탐색 가능해진다

### 묵시적 조인 주의 사항

- 항상 내부 조인
- 컬렉션은 경로 탐색의 끝이므로, 명시적 조인으로 별칭을 얻어야 함
- 경로 탐색은 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 JOIN절에 영향을 준다



### 실무 조언

- 가급적 명시적 조인 사용
- 조인은 SQL 튜닝에 매우 중요
- 묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어렵다



### JPQL - 페치 조인(fetch join)

연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능

```java
페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로
```

- JPQL에서 성능 최적화를 위해 제공 ( SQ에서 제공하는 조인의 종류가 아님)
- join fetch 명령어 사용



### 엔티티 페치 조인

```java
// JPQL 쿼리문
select m from Member m join fetch m.team

// 실제 나가는 SQL 쿼리문
SELECT M.*, T.* FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

- 회원을 조회하면서 연관된 **팀도 함께 조회**(SQL 한 번에)

- 다시 말해, 페치 조인을 사용하면 Member를 조회할 때

  -> Team객체를 프록시가 아닌 **실제 Team 객체로 조회가 가능**하다

> 만약 페치 조인을 하지 않을 경우 
>
> - 회원1, 팀 A(SQL) 
> - 회원2, 팀 A(1차캐시) 
> - 회원3, 팀 B(SQL)  
> - 이렇게 .. 회원 100번 조회할 경우.. N + 1 문제 발생

### 컬렉션 페치 조인

- 일대다 관계, 컬렉션 페치 조인

```sql
[JPQL]
select t
from Team t 
join fetch t.members where t.name = ‘팀A'

[SQL]
SELECT T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID WHERE T.NAME = '팀A'
```



실행 결과 중복된 값이 발생한다

```sql
teamname = 팀A, team = Team@0x100
-> username = 회원1, member = Member@0x200 
-> username = 회원2, member = Member@0x300 

teamname = 팀A, team = Team@0x100
-> username = 회원1, member = Member@0x200 
-> username = 회원2, member = Member@0x300
```



### 페치 조인과 DISTINCT

DISTINCT : 중복된 결과를 제거

```sql
select distinct t
	from Team t 
	join fetch t.members where t.name = ‘팀A’
```

일반적인 SQL문을 실행하면, DISTINCT를 추가하더라도 데이터가 다르므로 SQL 결과에서 중복제거 실패

BUT, JPA 레벨에서 추가적인 중복 제거 단계를 거치기 때문에

```sql
teamname = 팀A, team = Team@0x100
-> username = 회원1, member = Member@0x200 
-> username = 회원2, member = Member@0x300
```

>  다대일은 데이터 뻥튀기 안된다 걱정말자



### 페치 조인과 일반 조인의 차이

- 일반 조인 실행시 **연관된 엔티티를 함께 조회하지 않음**
- JPQL은 결과를 반환할 때 연관관계 고려X → fetch를 통해 알려줘!
- 페치 조인을 사용할 때만 **연관된 엔티티도 함께 조회(즉시 로딩)**
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념



### 페치 조인의 특징과 한계

#### 페치 조인 대상에는 별칭을 줄 수 없다

- 하이버네이트는 가능, 가급적 사용X

- 예시 ) team 아래 member가 5명 있는데, 아래 SQL 결과는 member 3명을 가진 team 객체

  이 상태로 team 아래있는 모든 5명의 멤버에게 일괄적으로 변경을 주고 싶더라도, team 객체가 가진 3명의 멤버들 정보만 바뀌는 등 예상치못한 상황이 발생할 수 있다. (데이터 무결성 문제)

  - 우리는 코드 설계 시 Team 내부에 모든 member들이 있다고 가정하고 설계한다는 점 명심!
  - 차라리 쿼리를 두 개 날려라

  ```sql
   select t from Team t join fetch t.members m where m.age > 10
  ```



#### 둘 이상의 컬렉션은 페치 조인 할 수 없다

- 1:N:M 일 경우 카티시안 곱 만들어질 수 있다 (1 * N * M)



#### 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다**.**

- 팀A한테 member가 5명 있다고 가정했을 때 페이징 단위를 3으로 할 경우
  - 팀A는 3명 member만 있다고 생각하고 작업을 진행하는 위험이 있을 수 있음
- 해결법
  - 일대다(컬렉션)일 경우 방향을 바꿔서 조회하면 가능
    - 일대일, 다대일 같은 단일값 연관 필드들은 페치 조인해도 페이징 가능



#### 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화

- 혹은 특정 갯수만큼 데이터를 조회하는 방법도 있다

  - `@BatchSize` : 연관된 엔티티를 조회할 때 지정된 size 만큼 SQL의 IN절을 사용해서 조회

  ```sql
  @BatchSize(100)
  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();
  ```

  - 글로벌 세팅 : persistence에 미리 엔티티 조회 size 지정

  ```sql
  // persistence.xml
  <property name = "hibernate.jdbc.default_batch_fetch_size" value = "100"/>
  ```



#### 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함

- `@OneToMany(fetch = FetchType.LAZY)` //글로벌 로딩 전략
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩



>  여러 테이블을 조인해서 **엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하는 경우**? 
>
> - 페치 조인 보다는 일반 조인을 사용해 필요한 데이터들만 조회해서 **DTO로 반환하는 것이 효과적**
> - 객체그래프를 유지할 때는 페치 조인!



</br>

### 다형성 쿼리 : Type

엔티티 상속 구조에서 **조회 대상을 특정 자식 타입으로 한정**할 때

예를 들어, item 중에 Book과 Movie를 조회하고 싶다면

```sql
// [JPQL]
select i from Item i
where type(i) IN (Book, Movie)

// [SQL]
select i from i
where i.DTYPE in (‘B’, ‘M’)
```

- type(i) → i.DTYPE으로 치환

### 다형성 쿼리 : Treat (JPA 2.1 이상)

상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용

- 자바의 타입 캐스팅과 유사

```sql
// [JPQL]
select i from Item i
where treat(i as Book).auther = ‘kim’

// [SQL]
select i.* from Item i
where i.DTYPE = ‘B’ and i.auther = ‘kim’
```

- treat()을 사용해 부모타입 item을 자식 타입 Book으로 다룬다
- 덕분에 auther 필드 접근 가능



</br>

### 엔티티 직접 사용 - 기본 키 값

- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기 본 키 값을 사용

```sql
// [JPQL]
select count(m.id) from Member m //엔티티의 아이디를 사용
select count(m) from Member m //엔티티를 직접 사용 -> ⭐️ 결국 엔티티의 기본 키인 아이디를 사용
[SQL](JPQL 둘다 같은 SQL 실행) 
select count(m.id) as cnt from Member m
```



### 엔티티 직접 사용 - 외래 키 값

```sql
Team team = em.find(Team.class, 1L);

// [JPQL] - 엔티티 직접 사용
String qlString = “select m from Member m where m.team = :team”; 

List resultList = em.createQuery(qlString)
											.setParameter("team", team).getResultList();

// [JPQL] - 외래키 값 사용
String qlString = “select m from Member m where m.team.id = :teamId”; 
List resultList = em.createQuery(qlString)
											.setParameter("teamId", teamId) .getResultList();
[SQL](JPQL 둘다 같은 SQL 실행) 
select m.* from Member m where m.team_id=?
```

</br>

### Named 쿼리

- 미리 정의해서 이름을 부여하고 사용하는 정적 JPQL → 재사용 가능
- 어노테이션 혹은 XML에 정의 (XML 우선권 가진다)
  - 어플리케이션 운영 상황에 따라 다른 쿼리를 가져가야할 경우 갈아 끼우기만 하면 되는 XML이 편리
- 어플리케이션 로딩 시점에 **JPQL 문법을 체크**, 초기화한 후 **재사용**

### 사용 예시 1 - 어노테이션 정의

```sql
@Entity
@NamedQuery(
				name = "Member.findByUsername",
				query="select m from Member m where m.username = :username")
public class Member {...
}
List<Member> resultList =
					em.createNamedQuery("Member.findByUsername", Member.class)
							.setParameter("username", "회원1").getResultList();
```

Repository에서 정의하는 **@Query**는 이름없는 Named 쿼리

### 사용 예시 2 - XML에 정의

그런 방법도 있다..ㅎㅎ



</br>

### 벌크 연산

쿼리 한 번으로 여러 테이블 로우 변경(엔티티)

- UPDATE, DELETE 지원
- executeUpdate() : 영향을 받은 엔티티 수 반환
- JPA가 벌크 연산보다는 실시간 연산에 조금 더 최적화 되어있긴 하지만 지원하긴 한다
- INSERT(insert into .. select, 하이버네이트 지원)

```sql
String qlString = "update Product p " +
											"set p.price = p.price * 1.1 " +
		                  "where p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)
                    .setParameter("stockAmount", 10)
                    .executeUpdate();
```



### 주의할 점!

- 벌크 연산은 **영속성 컨텍스트를 무시**하고 데이터베이스에 직접 쿼리

### 해결책

- 벌크 연산을 먼저 실행
- 벌크 연산 수행 후 영속성 컨텍스트 초기화
  - 예전에 조회했던 Account 엔티티는 5천만원 들고 있는데 벌크 연산할 때, 6천만원으로 바뀌었다
  - 근데, 벌크 연산 반영이 안되기 때문에
    - 영속성 컨텍스트 안의 Account 엔티티(5천만원) ≠ DB의 Account 데이터(6천만원)