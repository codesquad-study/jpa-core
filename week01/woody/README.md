



## 의존성

------

### Hibernate-entityManger

- JPA는 인터페이스
- 그리고 인터페이스 구현체로 hibernate을 선택한 것!

그 아래 `javax.persistence-api` 의존성 존재

- 하이버네이트가 JPA 인터페이스를 가지고있는 것

### h2 Version

h2 의존성 버전과 다운받은 h2 버전을 맞추는 것이 좋다!

### Dependency Version

현재 스프링 부트 버전에서는 어떤 디펜던시 버전이 가장 적절할까?

[Spring Boot 공식 문서 - Dependency Version](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html#dependency-versions) 를 참조하면 디펜던시마다 추천하는 버전이 명시되어있다.

이를 참고해서 의존성을 설정하면 좋다!

## JPA 설정

------

- persistence.xml로 JPA 관련 설정을 추가
- 위치는 `resources/META-INF/persistence.xml` 로 정해져있음

```java
<?xml version="1.0" encoding="UTF-8"?>
// JPA 버전은 2.2
<persistence version="2.2" 
             xmlns="<http://xmlns.jcp.org/xml/ns/persistence>" xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
             xsi:schemaLocation="<http://xmlns.jcp.org/xml/ns/persistence> <http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd>">
    <persistence-unit name="hello">
        <properties>
						// 데이터베이스 접근 정보
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
						// 어떤 데이터베이스 방언을 사용할지 설정
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

- 필수 속성
  - javax.persistence
    - 하이버네이트를 사용하지 않고, 다른 JPA 구현 라이브러리를 사용하더라도 사용 가능 (표준)
  - hibernate.dialect
    - 하이버네이트 전용 옵션 (다른 구현 라이브러리 사용 시 변경해야할 부분)

### 데이터베이스 방언

- 데이터베이스 방언 = 표준을 지키지 않는 특정 DB의 고유한 기능

  - 예를 들면 페이징을 할 때, MySQL에서 LIMIT을 Oracle ROWNUM 사용

- JPA는 DB에 종속적이지 않다 → Dialect에 사용하는 방언들을 갈아끼우기만 하면 된다!

  → persistence.xml에서 `hibernate.dialect` 속성 아래에 데이터베이스 방언을 명시하면 끗!

  ```java
  // 데이터베이스 벤더 설정
  <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
  ```



## JPA 사용

-----

### JPA 구동 방식

1. JPA는 persistance.xml의 정보를 조회해  EntityManagerFactory를 생성
2. EntityManagerFactory를 통해 필요할 때마다 EntityManager를 만들어 사용



### EntityMangerFactory 선언

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = 
							Persistence.createEntityManagerFactory("hello"); // 👈👈 이름이 일치해야 한다
    }
}

// persistence.xml
<persistence-unit name="hello"> // 👈👈 이름이 일치해야 한다
```



### persistence.xml 에서 하이버네이트 관련 옵션

</br>

- `hibernate.show_sql` : hibernate가 DB에 날리는 쿼리를 출력

- `hibernate.format_sql` : 깔끔하게 쿼리 출력 (show_sql이 false면 보여지지 않음)
- `hibernate.use_sql_comments` :  추가적인 주석 출력 ex) JPQL
- `hibernate.hbm2ddl.auto` : 애플리케이션 실행 시 DDL 구문을 자동 실행



### 주의

1. 엔티티 매니저 팩토리는 DB 당 하나만 생성

   - 애플리케이션 전체가 공유

2. 엔티티 매니저는 쓰레드간에 공유X

   - 한 번 사용하고 난 뒤 close!

3. JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

   - 데이터 변경 로직은 `EntityTransaction`으로 감싸야 한다.

   

</br>



## 영속성 컨텍스트의 이점

----

### 1차 캐시

- 한 트랜잭션 안에서 의미가 있다.(여러 트랜잭션이 공유하는 캐시는 2차 캐시)
- 우리가 생각하는 캐시만큼의 이점을 주진 않는다
- 엔티티 조회할 때, 1차 캐시에 이미 존재하는 엔티티면 DB까지 접근하지 않고 1차 캐시에서 가져옴

### 동일성 보장

- 반복 가능한 읽기 가능
- 같은 id로 두 번 조회할 경우 둘 다 같은 엔티티(==)다.

동일성(identity) : 실제 인스턴스가 같다 (==) 동등성(equality) : 실제 인스턴스는 다를 수 있지만 값이 같다 (equals())

### 트랜잭션 지원 쓰기 지연

- 영속성 컨텍스트 내부에 `쓰기 지연 SQL 저장소` 사용
- persist()를 여러 번 실행하더라도 바로 Insert 쿼리문이 DB로 가는게 아니라 `쓰기 지연 SQL 저장소` 에 차곡차곡 쌓인다
- 트랜잭션이 완료되면 그때 `쓰기 지연 SQL 저장소` 에 있는 insert 쿼리문들을 모두 DB로 날린다

### 변경 감지(Dirty Checking)

- 영속성 컨텍스트 내부에 `쓰기 지연 SQL 저장소` 사용
- 수정 사항을 개발자가 일일이 업데이트 시켜줄 필요가 없다
- 엔티티에 수정사항이 발생하면 엔티티와 스냅샷을 비교한 뒤 update 쿼리를 `쓰기 지연 SQL 저장소` 에 넣는다

### 지연 로딩

- 정말 필요할 때만 불러오기 때문에 마찬가지로 DB 오버헤드를 줄일 수 있다



</br>



## Flush

---------

영속성 컨텍스트의 변경내용을 데이터베이스에 반영

- 플러시 발생 ≠ 데이터베이스 트랜잭션 커밋

### 언제 발생하는가?

- 변경 감지(Dirty Checking)
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- `쓰기 지연 SQL 저장소`의 쿼리를 데이터베이스에 전송

### 호출 방법

- **em.flush()** 직접 호출
- **트랜잭션 커밋 -** 플러시 자동 호출
- **JPQL 쿼리 실행 -** 플러시 자동 호출



**em.flush()** - It saves the entity immediately to the database with in a transaction to be used further and it can be rolled back.

**em.getTransaction().commit** - It marks the end of transaction and saves all the chnages with in the transaction into the database and it can't be rolled back.





