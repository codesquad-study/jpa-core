# 1. 다양한 연관관계 매핑

## 0. 연관관계 매핑 시, 고려할 3가지

### (1) 다중성

- @ManyToOne
- @OneToMany
- @OneToOne
- @ManyToMany (사용 금지)

<br>

### (2) 단방향, 양방향

- 객체 : 참조형 필드가 있는 쪽만 참조 가능하다.
- DB : 외래키(FK) 하나로 양쪽 조인이 가능하다.
- 양방향은 실제로 `서로 다른 두 단방향 관계`를 말한다.
- **즉, 외래키를 관리할 주인을 결정해야 한다.**

<br>

### (3) 연관관계 주인

- 외래키를 관리하는 관리하는 주체
- 주로 `외래키(FK)`를 포함하고 있는 테이블이 연관관계의 주인이 된다.

<br><br>

## 1. 다대일[N:1]

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/37d15433-39ca-4640-b331-1a9f96a76921/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T095222Z&X-Amz-Expires=86400&X-Amz-Signature=34da7d9a0fa5e1737ab2f4c4beb5ce4b785b0b5e79457b7a10af88835f008bb8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="70%">

- 다대일 관계에서는 주로 Many부분에 외래키(FK)를 할당한다.

- 양쪽을 서로 참조해서 개발하도록 하자.

  - 연관 관계 주인 : 외래키 관리 주체
  - mappedBy : 조회 기능 (외래키에 영향을 미치지 않는다.)

  ```java
  @Entity
  public class Team {
  		@OneToMany(mappedBy = "team")
  		private List<Member> members = new ArrayList<>();
    	...
  }
  
  @Entity
  public class Member {
  		@ManyToOne
  		@JoinColumn(name = "team_id")
  		private Team team;
    	...
  }
  ```

<br><br>

## 2. 일대다[1:N]

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/996627a8-2c2e-4195-bc5d-e8953ef30f72/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T095312Z&X-Amz-Expires=86400&X-Amz-Signature=ff80563fa6d66df15cab373532b7eaafd4a6d5bb2b3f0545a6ea13476cc1fb76&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="70%">

### (1) One에 외래키(FK)를 할당할 때의 단점

- 외래키(FK)의 주인이 Member이므로 `updateSQL`이 추가 작동한다. 다른 테이블의 외래키를 변경해야 하기 때문에 번거롭다.
- 실무에서 수십, 수백개의 테이블이 존재하는 경우, 쿼리에 섞여있는 `update SQL`는 파악하기도 어려우며 운영에 불편함을 유발한다.
- 결론 : ***다대일 관계를 이용하자.*** 연관관계 주인을 `@ManyToOne`, 조회용으로 `@OneToMany(mappedBy ="")`를 사용하도록 하자.

<br>

### (2) 그래도 사용하고 싶다면?

```java
Entity
public class Team {
	@ManyToOne
	private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {
	@OneToMany
	@JoinColumn (name = "TEAM_ID", insertable = false, updatable = false)
	private Team team;
}
```

- joinColumn에 `insertable  = false`,` updatable = false` 설정 시, 조회용으로만 사용할 수 있다.

<br><br>

## 3. 일대일[1:1]

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e17f7611-0dbb-4e73-8b05-b74f63148180/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T095336Z&X-Amz-Expires=86400&X-Amz-Signature=4eb9e24c93a47bda80a315b05a1c1ae1f8f1635dd49acdd6e036350b02c413af&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="70%">

- 주 테이블에 외래키(FK)를 선언하는 방법을 사용하도록 하자. (주 테이블 : 해당 객체를 통해 조회를 많이 하는 테이블)

- DB 입장에서 일대일 관계에서 FK를 누구에게 주든 상관없다. 결국 객체 입장에서의 트레이드 오프이다.

- 주 테이블이 외래키(FK)를 가지고 있는 것이 성능과 여러가지 면에 이점이 있다.

  (대상 테이블에 연관 관계 주인을 할당할 경우, 지연 로딩으로 설정해도 즉시 로딩 방식으로 작동한다.)

<br><br>



## 4. 다대다[N:M]

### (1) 다대다(@ManyToMany)의 한계

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/575b0976-1b45-4905-99ff-8e3229e73be8/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T095423Z&X-Amz-Expires=86400&X-Amz-Signature=d52002fefa2123ff1cf1716428cd4482977434dc37fa252e56ffdb672486762f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="70%">

- 중간 테이블이 숨겨져 있어 예상하지 못하는 SQL이 발생할 수 있다.

- 추가적인 컬럼을 추가할 수 없다. (ex. 주문 시간, 주문 수량)

  ```java
  @Entity
  public class Member {
  	@ManyToMany
  	@JoinTable = (name = "MEMBER_PRODUCT")
  	private List<Product> products; //컬럼 추가 못함...ㅠ
  }
  ```

<br>

### (2) 해결책 : @OneToMany & @ManyToOne 변경해서 사용하자!

1. 연결 테이블용 엔티티를 추가하자! (ex. MemberProduct)

   ```java
   @Entity
   public class MemberProdcut{
     @ManyToOne
     @JoinTable(name = "MEMBER_ID")
     private Member member;
   
     @ManyToOne
     @JoinTable(name = "PRODUCT_ID")
     private Product product; 
     ...
   }
   
   ```

   - 객체 입장

     <img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/94f40feb-002a-45b4-87e9-230a9f4146fb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T095450Z&X-Amz-Expires=86400&X-Amz-Signature=6e001f23c97c82ae26c3084a581e41a1ff90be403db64767f9b821e3e90b2f24&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="100%">

   - DB 입장

     <img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1908d76f-a67b-4849-91c8-a927f6ba2c1e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T095515Z&X-Amz-Expires=86400&X-Amz-Signature=c2745fb4c4306d1367d8ebc159944e9342c15d1b0cc2679f5e9e7ab8caee3103&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="100%">

   <br>

2. 만약 기본키(PK)를 사용할 경우,  의미없는 값을 사용하도록 하자.
   - 연관성이 있는 기본키(PK)값을 사용할 경우, 확장 시, 복잡한 경우가 발생할 수도 있으니 사용하지 말자.

3. 그래도 이왕이면 `외래키(FK)`만 사용하는 것을 추천한다.

<br><br>

# 2. 고급 매핑

## 1. 상속관계 매핑

### (1) 상속관계 매핑을 사용하는 이유

- 객체는 상속 관계가 존재하지만, RDB에서는 상속 관계를 지원하지 않는다.
- 객체 상속과 유사한 슈퍼타입, 서브 타입 관계 존재하지만 실제와는 다르다.
- 상속관계 매핑 : `객체의 상속 구조`와 `DB의 슈퍼타입 서브타입 관계`를 매핑한다.

<br>

### (2) 물리 모델 구현 방법 3가지

1. 조인 전략 : 공통된 슈퍼타입의 서브 타입의 테이블을 생성한다.

   <img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0f88c216-09d1-40f4-bf90-23bfccaf81e1/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T100814Z&X-Amz-Expires=86400&X-Amz-Signature=b5c73435278476ef9f50c44d5238699d4903d8616f55490c44f2fb8d16e31bf5&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="80%">

   ```java
   @Entity
   @Inheritance(strategy = InheritanceType.JOINED)
   @DiscriminatorColumn(name = "DTYPE")
   public abstract class Item {}
   ```

   - 장점 :  1. 테이블 정규화 / 2. 외래키 참조 무결설 조건 사용 가능 / 3. 저장 공간 효율화

     → 기본적으로 선택하는 전략이다.

   - 단점 : 다중 조인으로 인한 성능 저하  / 데이터 추가 시, insert sql 2번 호출 / 복잡한 조회 쿼리

     (성능 저하 : 조인을 잘 사용하면 성능 저하 거읭 없음 / 2번의 쿼리 호출이 단점이라고 생각하지 않음.)

     <br>

2. 단일 테이블 전략 : 통합 테이블로 변환(`JPA의 기본 전략`)

   <img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/23809fe9-5073-47bc-9df2-c78e5d1913d3/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T100858Z&X-Amz-Expires=86400&X-Amz-Signature=95e2b8376d1dd9a71d8ff47871d8b27658ec795e11214ae1ab931e608bf3cc45&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="70%">

   ```java
   @Entity
   @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
   @DiscriminatorValue(name = "Book")
   @PrimaryKeyJoinColumn(name = "BOOK_ID")
   public class Book {}
   ```

   - 장점 : 빠른 조회 성능(조인이 없다) / 단순한 조회 쿼리

   - 단점 : 자식 엔티티의 컬럼은 모두 nullable / 테이블의 크기로 인한 성능저하

     <br>

3. 구현 클래스마다 테이블 전략 : 각각의 서브 타입 테이블로 변환.
   - 장점 : 서브 타입을 명확하게 구분해서 관리 가능 / not null 제약조건 사용가능
   - 단점 : UNION SQL을 사용하여 하위 테이블 조회 시, 성능이 느려짐 /  자식 테이블을 통합해서 쿼리의 어려움
   - ORM 전문가와 데이터베이스 설계자 둘 다 좋아하지 않는 전략 → 그냥 하지 말 것.

<br><br>

### (3) @DiscrminatorColumn(), @DiscrminatorValue()

1. `@DiscriminatorColumn`

   ```java
   @Entity
   @Inheritance(strategy = InheritanceType.JOINED)
   @DiscriminatorColumn(name = "DTYPE")
   public abstract class Item {}
   ```

   - 부모 클래스에 구분 컬럼을 지정한다. 이 컬럼으로 저장된 자식 테이블을 구분할 수 있다. 기본값이 DTYPE이므로 @DiscriminatorColumn으로 사용해도 된다.

   <br>

2. `@DiscrminatorValue()`

   ```java
   @Entity
   @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
   @DiscriminatorValue(name = "book")
   @PrimaryKeyJoinColumn(name = "BOOK_ID")
   public class Book {}
   ```

   - 엔티티를 저장할 때 구분 컬럼에 입력한 값을 지정한다. 만약 영화 엔티티를 저장하면 구분 컬럼임 DTYPE에 book이 저장된다.

- `단일 테이블 전략`의 경우, row의 값의 타입을 파악하기 어려우므로 꼭 선언하도록 하자! (ex. 누가 Album, Book인지 알 수 없다.)

<br><br>

## 2. Mapped Superclass - 매핑 정보 상속

### (1) 사용 이유

- 공통 매핑 정보가 필요할 때 사용(ex. createBy, lastModifiedDate, createdDate)

  ```java
  @MappedSuperclass
  abstract class BaseEntity {
    private String createBy;
    private LocalDateTime lastModifiedDate;
  }
  
  @Entity
  public class Question extends BaseEntity {
    //createBy, lastModifiedDate 컬럼이 추가됨.
  }
  ```

  <br>

### (2) @MappedSuperclass 기본 특징

- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할

  (상속관계 매핑과는 관련이 없고, 테이블과 매핑하지 않으면 엔티티에도 속하지 않는다.)

- 별도로 조회, 검색이 불가하다.

- 직접 생성해서 사용할 일이 없으므로 `추상 클래스`로 사용하는 것을 권장한다.

- @Entity 클래스는 엔티티나 @MappedSuperclass로 지정한 클래스만 상속 가능

<br><br>

# 3. 프록시와 연관관계 관리

## 1. 프록시

### (1) 프록시 패턴

```java
import java.util.*;

interface Image {
    public void displayImage();
}

//on System A
class RealImage implements Image {
    private String filename;
    public RealImage(String filename) {
        this.filename = filename;
        loadImageFromDisk();
    }

    private void loadImageFromDisk() {
        System.out.println("Loading   " + filename);
    }

    @Override
    public void displayImage() {
        System.out.println("Displaying " + filename);
    }
}

//on System B
class ProxyImage implements Image {
    private String filename;
    private Image image;

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    @Override
    public void displayImage() {
        if (image == null)
           image = new RealImage(filename);

        image.displayImage();
    }
}

class ProxyExample {
    public static void main(String[] args) {
        Image image1 = new ProxyImage("HiRes_10MB_Photo1");
        Image image2 = new ProxyImage("HiRes_10MB_Photo2");

        image1.displayImage(); // loading necessary
        image2.displayImage(); // loading necessary
    }
}
```

```
Loading    HiRes_10MB_Photo1
Displaying HiRes_10MB_Photo1
Loading    HiRes_10MB_Photo2
Displaying HiRes_10MB_Photo2
```

- 구체적으로 인터페이스를 사용하고 실행시킬 클래스에 대한 객체가 들어갈 자리에 `대리자 객체`를 대신 투입
- 프록시는 비서역할이므로, **흐름제어만 할 뿐 결과값을 조작하거나 변경시키면 안됨**
- 실제 클래스를 상속 받아서 만들어지고 프록시 객체는 `실제 객체의 참조`.

<br>

### (2) 프록시 사용 이유

- 상황에 따라서 Member만 조회하고 싶은 경우가 있고, 때로는 Member와 Team을 다함께 불러올 수가 있다.

  → 이를 해결하기 위해서는 JPA는 프록시 객체를 통해서 이해를 해야 한다.

<br>

### (3) 프록시 기초

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7c25a4b5-3788-44d6-80f0-3f67bf1585e0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210914T100610Z&X-Amz-Expires=86400&X-Amz-Signature=53ca97060b02e56503c9e619ac3641c3927b8fc165d7807997e0fb760399bd7e&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="70%">

- 프록시 객체 초기화 : 영속성 컨텍스트를 거쳐 DB조회를 통해 실제 엔티티를 참조하도록 하는 과정
  1. 데이터를 조회 시 데이터가 존재하지 않다면, 영속성 컨텍스트에 초기화를 요청한다.
  2. 영속성 컨텍스트는 DB로 데이터를 받아와 엔티티를 생성 또는 조회하여 해당 엔티티를 반환한다.
  3. 반환된 엔티티의 참조 변수를 프록시의 필드에 할당하여 프록시는 엔티티 데이터에 접근이 가능하다.

<br>

### (4) 프록시 주의사항

- 객체를 비교할 때는 `instanceof`을 사용 해야 한다.(프록시 객체, 엔티티 객체 어떤 것이 올지 모름.)
- 준영속 엔티티는 영속성 컨텍스트의 관리를 받지 못해,`LazyInitializationException`을 반환한다.

<br>

## 2. 즉시 로딩과 지연 로딩

### 3. 결론

- 가급적이면 `Lazy Loading`을 사용하라. (특히 실무에서!!!)

- Eager Loading을 사용할 경우, 예상치 못한 SQL이 발생

- 즉시 로딩은 JPQL에서 `N + 1 문제`를 발생 시킨다.

  - 즉시 로딩은 로딩 시에 모든 값이 호출된 상태여야 한다.  그래서 JPQL을 사용할 경우, 멤버 조회 시, team의 값을 초기화하기 위해 team을 조회하는 쿼리를 추가 실행한다. (`select * from Member` + `select * from Team`)

  - 해결 방법 : 모든 로딩 전략을 Laz Loading으로 사용하고 JPQL의 fetch join으로 변경한다.

    ```java
    em.createQuery("select m from Member m join fetch m.team", Member.class);
    ```

- `@ManyToOne & @OneToOne`은 기본이 `즉시 로딩` → `지연 로딩`으로 설정한다. `(@OneToMany` 기본 값은 `지연 로딩`)

- ***실무에서는 모두 지연 로딩을 하도록 하자!***

<br><br>

## 3. 영속성 전이(CASCADE)와 고아 객체

### 1. 사용 이유

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶은 경우

<br>

### 2. 고아 객체

- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제 (DELETE SQL이 실행)
- `@OneToMany(orphanRemoval = true)`
- `CascadeType.ALL + orphanRemove = true`
  - 두 옵션 모두 활성 시, 부모 엔티티를 통해서 자식의 생명 주기를 관리할 수 있음
  - DDD의 Aggregate Root개념을 구현할 때 유용하다.

<br><br>

## Reference

- [WikiPedia]프록시 패턴 : https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%ED%8C%A8%ED%84%B4
- [Design_Pattern] 프록시 패턴(Proxy Pattern) : https://limkydev.tistory.com/79