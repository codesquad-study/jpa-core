# Chapter 6 다양한 연관관계 매핑

## 연관관계 매핑 시 고려할 점

- 다중성(Multiplicity)

  - 다대일
  - 일대다
  - 일대일
  - 다대다  - 실무 사용 X

- 단방향, 양방향

  - 테이블은 외래 키 하나로 양쪽 조인이 가능하기 때문에 방향이라는 개념이 없다.
  - 그러나 객체는 참조용 필드가 있는 쪽으로만 참조가 가능하기 때문에 단방향, 양방향으로 나뉜다.
    - 양방향은 단방향이 두 개가 있다는 의미
    - A->B, B->A 두 가지 참조 중 외래 키를 관리할 곳을 지정해야 한다. 관리하는 참조가 연관관계의 주인이다.

- 연관관계의 주인

  - 연관관계의 주인은 mappedBy 속성을 사용하지 않는다. 
  - 다중성은 왼쪽을 연관관계의 주인으로 정한다.

  

## 다대일

- 데이터베이스 테이블의 외래 키는 항상 다 쪽에 있다. 

### 다대일 양방향

- 연관관계의 주인: Member.team (Team.members 아님)
- 양방향 연관관계는 항상 서로를 참조해야 한다. 
- 연관관계 편의 메서드는 양쪽 다 작성할 수 있지만 무한루프에 주의해야 한다.

```java
@Entity
public class Member {
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
    
    public void setTeam(Team team) {
        this.team = team;
        
        // 무한 루프 방지
        if (!team.getMembers().contains(this)) {
            team.getMembers().add(this);
        }
    }
}
```

```java
@Entity
public class Team {
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>(); // 조회를 위한 JPQL이나 객체 그래프 탐색 시 사용
    
    public void addMember(Member member) {
        this.members.add(member);
        
        // 무한 루프 방지
        if (member.getTeam() != this) {
            member.setTeam(this);
        }
    }
}
```





## 일대다

- 데이터베이스 테이블의 외래 키는 항상 다 쪽에 있다. 
- Collection, List, Set, Map 중 하나 사용 가능

### 일대다 단방향

- Team.members가 회원테이블의 team_id 외래키를 관리

```java
@Entity
public class Team {
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    @OneToMany
	@JoinColumn(name = "TEAM_ID")
	private List<Member> members = new ArrayList<>();
}

```

- `@JoinColumn`을 명시하지 않으면 JoinTable 전략이 기본으로 적용됨 (tem_member 테이블이 생김)
- 관리하는 외래 키가 다른 테이블(멤버 테이블)에 있기 때문에 멤버를 업데이트 하고 팀을 persist하면 member 테이블에 update 쿼리가 날아감

```java
em.persist(member1); // insert member1
em.persist(member2); // insert member2
em.persist(team1); // insert team1, update member1 & 2
```

1. 엔티티가 관리하는 외래 키가 다른 테이블에 있다.

2. 연관 관리를 위한 update sql 실행한다.

&rarr; 두 가지를 고려할 때 일대다 단방향 대신 다대일 양방향 매핑을 사용하는 것이 좋다. 



### 일대다 양방향

- 없다.

- 일대다 단방향 매핑 반대편에 같은 외래 키를 사용하는 읽기전용 다대일 단방향 매핑을 추가하여 흉내낼 수 있다.
  - 다대일 join column에 insertable=false, updatable=false 설정



## 일대일

- 주 테이블, 대상 테이블 모두 외래 키를 가질 수 있다.

#### 주 테이블에 외래 키 양방향

- 외래 키를 객체 참조와 비슷하게 사용

- 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 확인할 수 있음
- 값이 없으면 외래 키에 null을 허용한다.

```java
public class Member {
    
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
}
```

```java
@Entity
public class Locker {
    @Id @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "locker")
    private Member member;
}
```



#### 대상 테이블에 외래 키 양방향

- Locker에 외래 키

- 테이블 관계가 일대일에서 일대다(한 member가 여러 개의 locker 사용 가능)로 변경되어도 테이블 구조를 유지할 수 있다.

  - member_id 외래키에 걸려있는 unique 조건만 해제하면 된다. 

- 프록시 기능의 한계로 지연 로딩으로 설정하더라도 즉시 로딩된다.

  

## 다대다

- `@JoinTable`

```java
@Entity
public class Member {
    
    @ManyToMany
	@JoinTable(
        name = "MEMBER_PRODUCT", 
        joinColumns = @JoinColumn(name = "MEMBER_ID"),
        inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID")
    )
    private List<Product> products = new ArrayList<>();

}
```

```java
SELECT * FROM MEMBER_PRODUCT MP
INNER JOIN PRODUCT P ON MP.PRODUCT_ID=P.PRODUCT_ID // MEMBER_PRODUCT와 PRODUCT 조인
WHERE MP.MEMBER_ID=?
```

- ManyToMany 어노테이션을 사용하는 대신 일대다, 다대일 관계로 해결하자
- 복합 기본키 `@IdClass`
  - Serializable을 구현하는 별도의 식별자 클래스를 만들기
  - equals and hashcode 메서드 구현해야 함
  - 기본 생성자 있어야 함
  - public class여야 함

```java
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {
    ...
}
```

```java
public class MemberProductId implements Serializable {
    
    private String member;
    private String product;
    
    // default constructor
    
    // equals and hashcode overriding
    
}
```

- MemberProduct는 Member, Product와 식별 관계로 연결되어 있다.
  - 식별 관계: 부모 테이블의 기본키를 받아서 자신의 기본 키 + 외래 키로 사용하는 것

- 새롭게 Long 타입 id를 하나 지정해주는 것이 좋다(비식별 관계).
  - 비즈니스가 변경되더라도 기본 키가 변하지 않기 때문
  - 비식별관계: 부모 테이블의 기본키가 자식 테이블의 일반 컬럼이나 외래 키로 사용되는 것, 즉 받아온 식별자는 외래 키로만 사용하고 새로운 식별자를 추가하는 것
    - 필수적 비식별 관계: 외래 키에 null 금지, 연관관계 필수
      - 언제나 관계가 존재하는 것을 보장하기 때문에 내부 조인 사용
    - 선택적 비식별 관계: 외래 키에 null 허용, 연관관계 여부 선택 가능
      - 외부 조인 사용



# Chapter 7 고급 매핑

## 상속 관계 매핑

- 객체의 상속 구조와 데이터베이스의 슈퍼타입, 서브타입 관계를 매핑하는 것
  - 관계형 데이터베이스에는 상속이라는 개념이 없기 때문에 ORM([Object–relational mapping](https://en.wikipedia.org/wiki/Object–relational_mapping), 객체와 관계형 데이터베이스의 데이터를 자동으로 연결해주는 것)에서의 상속 관계 매핑은 객체의 상속 구조와 데이터베이스의 슈퍼 타입 - 서브 타입 관계를 매핑하는 것이다.

#### JOINED 전략 (정석)

- 엔티티 각각을 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략
- 조회할 때 주로 사용
- 테이블에는 타입이 없기 때문에 타입을 구분하는 컬럼을 추가해야 한다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 부모 클래스에 명시
@DiscriminatorColumn // 부모 클래스에 구분 컬럼 지정, 기본값 = DTYPE
public class Animal {
    @Id
    private long animalId;
    private String species;

    // constructor, getters, setters 
}

@Entity
@DiscriminatorValue("A") // 엔티티를 저장할 때 구분 컬럼에 입력할 값, Pet 엔티티를 저장하면 구분 컬럼인 DTYPE 값에 A가 저장된다, 기본값은 엔티티 이름
@PrimaryKeyJoinColumn(name = "petId") // 자식테이블의 기본 키 컬럼명은 부모 테이블의 id 컬럼명을 기본으로 사용하지만, 변경하고 싶다면 이 어노테이션을 사용하면 된다.
public class Pet extends Animal {
    private String name;

    // constructor, getters, setters
}
```

#### 장점

- 테이블이 정규화 된다.
- 외래 키 참조 무결성 제약 조건을 활용할 수 있다.
- 저장 공간을 효율적으로 사용한다.

#### 단점

- 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
- 조회 쿼리가 복잡하다.
- 테이블을 등록할 때 insert sql을 두 번 실행한다. 



#### SINGLE_TABLE 전략

- 전략 설정 안하면 기본적으로 JPA에 의해 선택되는 전략이다.
- 테이블을 하나만 사용하고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분하는 전략 - 구분 컬럼 필수
- 조인을 사용하지 않으므로 속도가 가장 빠르다. But, 단일 테이블이 너무 커지만 상황에 따라 조회 성능이 느려질 수도 있다. 
- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다. 
  - 자식 엔티티에서 사용하지 않는 컬럼에는 null이 들어가기 때문
- @DiscriminatorValue를 지정 안 하면 엔티티 이름이 값으로 설정된다. 

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class MyProduct {
    @Id
    private long productId;
    private String name;

    // constructor, getters, setters
}
```



#### TABLE_PER_CLASS 전략

- 자식 엔티티마다 테이블을 만드는 전략인데 잘 사용 안함
- 구분 컬럼을 사용하지 않는다.
- 자식 테이블을 통합해서 쿼리하기 어렵고 여러 자식 테이블을 함께 조회할 때 성능이 저하됨
  - union으로 모든 테이블 다 뒤져야 한다. (모든 테이블 select)

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item { // 추상 클래스로 만들었기 때문에 Item 테이블은 생성되지 않는다. 
    (...)
}

@Entity
public class Album extends Item {...}
```

- https://www.baeldung.com/hibernate-inheritance

#### 

## @MappedSuperclass

- 부모 클래스는 테이블과 매핑하지 않고 부모 클래스를 상속 받는 자식 클래스에게 매핑 정보만 제공하고 싶을 때 사용

```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    private LocalDateTime createdDateTime;

    @LastModifiedDate
    private LocalDateTime modifiedDateTime;
    
}
```

- Entity가 아니기 때문에 테이블과 매핑할 수 없다. 
- 조회, 검색 불가 (BaseEntity 타입을 검색할 수 없음)
- 직접 생성해서 사용할 일이 없으므로 추상 클래스로 만드는 것이 좋다. 
- `@AttributeOverride`
  - 부모에게 물려받은 매핑 정보 재정의
- `@AssociationOverride`
  - 연관관계 재정의

- 엔티티는 `@Entity`와 `@MappedSuperclass`만 오버라이딩 할 수 있다.



### 복합키 비식별관계 매핑

- JPA는 엔티티의 식별자를 키로 영속성 컨텍스트에 엔티티를 저장하는데 식별자 필드가 2개 이상이면 equals and hashcode로 동등성을 비교하는데 문제가 생긴다 &rarr; 별도의 식별자 클래스를 만들고 해당 클래스의 모든 필드를 equals and hashcode를 구현해야 한다.  
- 복합키에는 `@GeneratedValue`를 사용할 수 없다.
- IdClass - 데이터베이스, EmbeddedId - 객체지향

`IdClass`

- 식별자 클래스 속성 명 = 엔티티에서 사용하는 식별자 속성명
- Serializable을 구현
- equals and hashcode 메서드 구현해야 함
- 기본 생성자 있어야 함
- public class여야 함

```java
Parent parent = new Parent();
parent.setId1("myId1");
parent.setId2("myId2");
parent.setName("parentName");
em.persist(parent);
```

- em.persist()를 호출할 때 Parent.id1과 Parent.id2 값으로 ParentId 클래스가 내부적으로 만들어진 후 영속성 컨텍스트의 키로 사용된다. 



`@EmbeddedId`

- `@Embeddable` 어노테이션을 붙여야 함
- Serializable을 구현
- equals and hashcode 메서드 구현해야 함
- 기본 생성자 있어야 함
- public class여야 함

```java
@Entity
public class Parent {
    
    @EmbeddedId
    private ParentId id;
    
    private String name;
    
}
```

```java
@Embeddable
public class ParentId implements Serializable {
    
    @Column(name = "PARENT_ID1") // 식별자 클래스에 기본 키 직접 매핑
    private String id1;
    
    @Column(name = "PARENT_ID2")
    private String id2;
    
    // equals and hashcode
}
```



# Chapter 8 프록시와 연관관계 관리

## 프록시

- 실제 클래스를 상속받아서 만들기 때문에 실제 클래스와 겉모양이 같다. 
  - Entity target - 실제 객체에 대한 참조
  - 프록시 객체의 메서드 호출 시 프록시 객체가 실제 객체의 메서드를 호출한다. 
- 객체 그래프 탐색 시 프록시 기술 사용
- 연관된 객체를 처음부터 DB에서 조회하지 않고 실제 사용하는 시점에 조회할 수 있게 해줌

- JPA표준 명세는 지연 로딩 구현을 JPA 구현체에 위임했는데, 구현체인 하이버네이트는 프록시 사용과 바이트코드 수정 두 가지 방법을 제공한다. 

#### EntityManger.find()

- 영속성 컨텍스트에 엔티티가 없으면 데이터베이스를 조회

#### EntityManager.getReference()

- 데이터베이스 접근을 위임한 프록시 객체를 반환 (DB 조회 x, 엔티티 생성 x)
- 영속성 컨텍스트에 이미 엔티티가 존재한다면 프록시가 아닌 실제 엔티티를 반환

```java
Team team = em.getReference(Team.class, "team1"); // 식별자 값 보관
team.getId(); // 엔티티 접근 방식이 @Access(AccessType.PROPERTY)로 설정되어 있을 경우 프록시 초기화 X
```

- `getId()` 메서드 호출 시 `@Access(AccessType.FIELD)`일 경우 프록시 객체가 초기화 됨
  - `setTeam(team)`과 같이 연관관계를 설정할 때는 식별자만 사용하므로 프록시 초기화 X



### 프록시 객체의 초기화

- 프록시 객체는 실제 사용될 때 DB를 조회해서 실제 엔티티 객체를 생성한다.
- 최초사용 시 한 번만 초기화된다. 
- 준영속 상태의 프록시를 초기화하면 org.hibernate.LazyInitializationException이 발생한다. 

#### 과정

1. getName() 메서드 실행
2. 프록시 객체가 영속성 컨텍스트에 실제 엔티티 생성 요청(초기화)
3. 영속성 컨텍스트는 DB를 조회해서 실제 엔티티 객체 생성
4. 프록시 객체는 생성된 실제 엔티티 객체의 참조를 target 멤버변수에 보관
5. 프록시 객체가 실제 엔티티 객체의 getName() 메서드를 호출해서 결과 반환

#### PersistenceUnitUtil.isLoaded(Object entity)

- 프록시 인스턴스 초기화 여부 확인 가능
- 초기화, 프록시 인스턴스 x &rarr; true
- 프록시 인스턴스 &rarr; false



## 즉시 로딩

- 엔티티를 조회할 때 연관된 엔티티를 함께 조회
- `@ManyToOne(fetch = FetchType.EAGER)`
- 최적화를 위해 가능하면 조인 쿼리가 사용된다. 
- 외래 키에 not null 제약 조건을 걸어주면 내부 조인만 사용 가능
  - `@JoinColumn(nullable = false)`
  - `@ManyToOne(fetch = FetchType.EAGER, optional = false)`



## 지연 로딩

- 연관된 엔티티를 실제 사용할 때 조회
- `@ManyToOne(fetch = FetchType.LAZY)`

- 멤버 객체 안에 팀이 들어있고 em.find로 팀을 조회할 경우 처음에는 멤버만 조회되고, 멤버의 team 멤버변수에는 프록시 객체가 들어감
- 팀 객체를 실제 사용될 때 데이터베이스에서 팀을 조회해서 프록시 객체를 초기화 함



## 컬랙션 래퍼

- 하이버네이트가 제공하는 내장 컬렉션
- 하이버네이트는 엔티티 내부의 컬렉션을 추적, 관리하기 위해 엔티티를 영속화할 때 원본 컬렉션을 컬렉션 래퍼로 변경한다. 
  - `org.hibernate.collection.internal.PersistentBag`
- 컬렉션 래퍼도 컬렉션에 대한 프록시 역할을 한다. 
- `member.getOrders()`로는 컬렉션이 초기화되지 않고 `memger.getOrders().get(0)`과 같이 실제 데이터를 조회해야 컬렉션이 초기화 된다. 

- 컬렉션은 필드에서 바로 초기화 하는 것이 안전하다.

  (하이버네이트는 엔티티를 영속화 할 때 컬랙션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경하기 때문)

```java
    @ManyToMany
    @JoinTable(name = "category_item",
            joinColumns = @JoinColumn(name = "category_id"),
            inverseJoinColumns = @JoinColumn(name = "item_id"))
    private List<Item> items = new ArrayList<>();
```

```java
    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();
```





## JPA 기본 페치 전략

즉시 로딩: `@ManyToOne`, `@OneToOne`

-  `@ManyToOne`, `@OneToOne`은 optional이 false일 땐 내부 조인, optional이 true일 때 외부 조인
-  `@OneToMany`, `@ManyToMany`은 항상 외부 조인

- 컬렉션을 하나 이상 즉시 로딩하지 않기

  - A 테이블을 N, M 두 테이블과 일대다 조인할 경우 SQL 실행 결과가 N*M이 되어 너무 많은 데이터가 반환될 수 있다.
  - 컬렉션 즉시 로딩은 항상 외부 조인을 사용한다. 
    - Team:Member가 일대다 관계로 조인할 때 회원이 없는 팀을 내부 조인하면 팀도 조인되지 않는다.
    - 이를 방지하기 위해 JPA 즉시로딩은 항상 외부 조인 사용

  

지연 로딩: `@OneToMany`, `@ManyToMany`, 컬렉션

- 모든 연관관계에 지연 로딩을 사용하자
- 필요한 곳만 즉시 로딩으로 바꾸기





## Cascade

- JPA는 CASCADE 옵션으로 영속성 전이(transitive persistence) 기능 제공

- 라이프 사이클이 동일하고, 다른 엔티티가 참조하지 않을 때 쓰는 것이 좋다. 
- 여러 엔티티가 참조하는 경우에는 별도의 repository를 생성하는 것이 좋다.
- persist, remove는 바로 전이가 되지 않고 플러시 호출할 때 영속성이 전이된다.
- type
  - all: 상위 엔티티에서 하위 엔티티로 모든 작업을 전파
  - persist: 하위 엔티티까지 영속성 전달, 부모와 자식 엔티티 한 번에 영속화
  - merge: 하위 엔티티까지 병합 작업을 지속
  - remove: 상위 엔티티를 제거하면 연결된 하위 엔티티까지 제거
  - lock: 상위 엔티티와 하위 엔티티를 영속성 컨텍스트에 다시 관리하게 함
  - detach: 영속성 컨텍스트에서 연결된 하위 엔티티의 영속성까지 제거
  - refresh: 데이터베이스로부터 인스턴스의 값을 다시 읽어옴



## 고아 객체

- orphanRemoval: 부모테이블와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제

  &rarr; 부모 텐티티 컬렉션에서 자식 엔티티의 참조를 제거하면 자식 엔티티가 자동으로 삭제된다. 

- 영속성 컨텍스트를 플러시할 때 적용된다. 

  - 플러시 시점에 delete sql 실행

- 삭제된 엔티티를 다른곳에서 참조하면 문제가 발생하기 때문에 `@OneToOne`, `OneToMany`에서만 사용

- 부모 제거 &rarr; 자식 제거 &rarr; CascadeType.REMOVE와 동일한 효과





