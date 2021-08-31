# ♾ 영속성 관리

## 1. EntityManagerFactory와 EntityManager

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ea27285c-4e67-4f22-87b8-f73dac97f604/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041133Z&X-Amz-Expires=86400&X-Amz-Signature=d925e5a768d4af420b722dd767a721d73aa5c5a7926a9090eabd3f317072e992&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="80%" height="80%"/></center>

 EntityManagerFactory는 엔티티 매니저를 만드는 공장 역할입니다. 주로 데이터베이스를 하나만 사용하는 애플리케이션은 일반적으로 EntityManagerFactory를 하나만 생성합니다.

EntityManager는 엔티티를 필요에 따라 **데이터베이스와 동기화하는 역할**을 담당합니다. 만약 DB의 데이터에 접근하는 경우 반드시 EntityManager를 통해 영속성 컨텍스트의 엔티티를 취득하거나 새로 생성한 엔티티를 영속성 컨텍스트에 등록해야 합니다. 영속성 컨텍스트에 등록된 엔티티는 영속성 컨텍스트를 통해 관리받을 수 있습니다. 엔티티 매니저는 DB에 연결이 꼭 필요한 시점에 커넥션을 획득합니다.

EntityManagerFactory는 여러 쓰레드의 동시 접근에도 안전하지만 EntityManager는 여러 스레드의 동시 접근 시, 동시성 문제가 발생합니다. 그렇기 때문에 EntityManager는 스레드 간에 절대 공유하면 안됩니다.

<br>

## 2. 엔티티 생명주기

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b89f6433-3312-4a62-8c09-d726a07ff647/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T035214Z&X-Amz-Expires=86400&X-Amz-Signature=826fe3d52159d460e232b6dadf6edc3ae11c5c2ae9820ab9377246f19c0e79f8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="80%" height="80%"/>

- **비영속(new/transient)** : 객체를 생성하였지만 영속성 컨텍스트와 DB와 관계없는 상태를 말합니다.
- **영속(managed)** : 엔티티가 영속성 컨텍스트에 등록 및 저장된 상태를 말한다. 영속성 컨텍스트에 등록된 엔티티는 변경 감지, 지연 로딩, 동일성 보장, 1차 캐시 등 영속성 컨텍스트가 지원하는 기능을 사용할 수 있다.
- **준영속(detached)** : 영속성 컨텍스트에 관리하던 엔티티를 영속성 컨텍스트가 관리하지 않으면 준영속 상태가 된다. 준영속 상태가 될 경우, 변경 감지, 지연 로딩, 동일성 보장 등 영속성 컨텍스트 기능을 사용하지 못한다.
- **삭제(removed)** : 엔티티를 영속성 컨텍스트와 DB에서 삭제한 상태를 말합니다.

<br><br>

## 3. 영속성 컨텍스트의 특징

영속성 컨텍스트의 특징은 `1차 캐시`, `동일성 보장`, `트랜잭션 지원하는 쓰기 지원`,` 변경 감지(Dirty Checking)`, `지연 로딩(Lazy Loading)` 입니다. 

### (1) 1차 캐시

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/23f8a311-b2a3-4fcd-909e-6db0717bf354/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041242Z&X-Amz-Expires=86400&X-Amz-Signature=590a89dee2f071d28ef66b301520553c7464dea7ddffc3da5698764e755ddafa&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="70%" height="70%"/><br></center>

 1차 캐시는 키 값으로 식별자, 값을 Entity형태로 관리하며 EntityManager에서 사용하는 캐시입니다. 만약 엔티티를 조회한다면, 우선 1차 캐시에서 해당 엔티티를 검색합니다. 1차 캐시에 해당 엔티티가 존재할 경우, 해당 엔티티를 조회합니다. 하지만 해당 엔티티가 1차 캐시에 존재하지 않을 경우, DB에서 엔티티를 조회하고 1차 캐시에 등록합니다. 

<br>

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/be2380eb-f47c-4bf7-b17b-8ddab5a38e0f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041313Z&X-Amz-Expires=86400&X-Amz-Signature=6334ce8b9dc3f9cbb6ec6210a234d9d23d59da03f976cbb8a9123814bdbae294&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="70%" height="70%"/><br></center>

 만약 엔티티를 DB에 등록하고 싶은 경우, 엔티티를 영속성 컨텍스트에 등록(persist)하면서 1차 캐시에는 엔티티, 쓰기 지연 SQL 저장소에는 해당 객체를 삽입하는 쿼리(insert)를 등록합니다. 추가적으로 엔티티를 영속성 컨텍스트에 등록할 경우 또한 1차 캐시와 쓰기 SQL 저장소에 객체와 쿼리를 등록합니다. 그리고 마지막에 영속성 변경 내용을 반영하는 메서드 flush()를 호출할 경우, 쓰기 지연 SQL 저장소에 누적되었던 쿼리문들이 한번에 실행되어 해당 엔티티의 변경(등록, 수정, 삭제) 내역을 DB에 반영합니다.

<br><br>

### (2) 동일성 보장

영속성 컨텍스트는 동일성을 보장합니다. 영속성 컨텍스트는 1차 캐시에 존재하는 같은 엔티티 인스턴스를 반환하기 때문에 엔티티의 동일성을 보장합니다.

```
동일성(identity) : 실제 인스턴스의 주소값이 일치 여부를 확인한다.
동등성(equality) : 인스턴스의 값이 일치한다면 같은 객체로 취급하는 것을 의미한다.
```

<br>

### (3) 트랜잭션 지원하는 쓰기 지원

<center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2a089edd-8a68-42a6-a352-d825e593f25e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041448Z&X-Amz-Expires=86400&X-Amz-Signature=f84d80bda2d27a221175f2eb49008e0ca421193110360ef24253bafaef57f82e&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="70%" height="70%"/></center>

 변경에 관련한 쿼리를 즉시 DB에 보내지 않고 메모리에 저장해준다. 그리고 트랜잭션이 커밋 시에 모아둔 등록 쿼리를 보낸 후 커밋합니다.

<br>

### (4) 변경 감지 (dirty checking)

 JPA는 **트랜잭션이 끝나는 시점**에 변화가 있는 모든 엔티티 객체를 DB에 자동으로 반영하며 해당 엔티티는 영속 상태만 변경 감지합니다. JPA는 영속성 컨텍스트의 상태를 복사 및 저장해 스냅샷으로 관리하며 flush() 메서드를 사용할 때, 현재 시점과 비교하는 대조군으로 사용합니다. 자세한 영속성 컨텍스트 변경 감지 과정은 아래와 같습니다.

1. 트랜잭션 커밋 시, 엔티티 매니저 내부에서 flush() 메서드를 호출합니다.
2. 엔티티와 스냅샷을 비교해서 변경된 엔티티를 찾는다.
3. 변경된 엔티티가 존재할 경우, 수정 쿼리를 생성해 쓰기 지연 SQL 저장소에 보낸다.
4. 쓰기 지연 저장소의 SQL을 DB에 보낸다.
5. DB 트랜잭션을 커밋한다.

JPA는 변경 감지로 생성되는 update 쿼리를 **모든 필드를 업데이트하는 방식**을 기본값으로 사용합니다. 전체 필드를 업데이트할 경우, **생성하는 쿼리가 같아 부트 실행시점에 미리 만들어 재사용**이 가능합니다. 그리고 **DB 입장에서는 동일한 쿼리를 받을 시, 이전에 파싱된 쿼리를 재사용**하기 때문에 쿼리 재사용의 이점이 존재합니다.

 만약 변경 부분만 update를 사용하고 싶을 경우, `@DynamicUpdate` annotation을 사용하면 변경 필드만 반영하도록 구현할 수 있습니다. (이전에 정규화가 잘못된 경우일 확률이 높다고 합니다.)

```java
@Getter
@NoArgsConstructor
@Entity
@DynamicUpdate // 변경한 필드만 대응
public class Pay {...}
```

[출처 : 더티 체킹이란?(이동욱님 블로그)](https://jojoldu.tistory.com/415)

<br><br>

## 4. Flush()

### (1) flush()의 세부 동작

 플러시는 영속성 컨텍스트의 변경 내용을 DB에 반영하는 것을 말합니다. 영속성 컨테스트에 보관된 엔티티를 지우지 않고 플러시는 영속성 컨텍스트에 보관된 **엔티티의 변경 내용을 DB와 동기화`**하는 작업입니다. 플러시를 실행 시, 구체적인 과정은 아래와 같습니다.

1. 변경 감지가 동작해 모든 엔티티를 스냅샷과 비교해 수정된 엔티티를 찾는다.
2. 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
3. 쓰리 지연 SQL 저장소의 쿼리를 DB에 전송한다.

<br>

### (2) flush 방법

1. em.flush() 직접 호출
2. 트랜잭션 커밋
3. JPQL 쿼리 실행

<br>

### (3) flush 옵션

```java
entityManager.setFlushMode(FlushModeType.AUTO);
```

- FlushModeType.AUTO : 커밋이나 쿼리를 실행할 때 플러시(기본 값)
- FlushModeType.COMMIT : 커밋할 때만 플러쉬

기본 값을 FlushModeType.AUTO 입니다. 트랜잭션 커밋이나 쿼리 실행 시 플러쉬를 자동으로 호출합니다. 대부분의 AUTO를 사용하지만 성능 최적화를 위해 COMMIT을 사용합니다.

<br><br>

## 5. 준영속 상태

 준영속 상태는 영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 상태를 말합니다. 준영속 엔티티는 영속성 컨텍스트에서 관리하지 엔티티이므로 **영속성 컨텍스트에서 제공하는 기능(1차 캐시, 동일성 보장, 동작 감지, 트랙잭션의 쓰기 기능)**을 사용할 수 없습니다.

### (1) 준영속 상태 만드는 방법

 영속성을 준영속 상태로 만드는 방법은 크게 3가지 입니다.

1. em.detach(entity) : 특정 엔티티를 준영속 상태로 전환합니다. 해당 메서드를 실행 시, 1차 캐시에 등록된 정보를 확인해 해당 정보를 제거하고 쓰기 지연 SQL 저장소에 등록한 쿼리문을 제거합니다.

   <center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a8c6d684-961c-457e-a4c8-aa89b253aa5c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041634Z&X-Amz-Expires=86400&X-Amz-Signature=0e3de2355ef8b70adaf3051b3866d6eb2490bb8d6af8256f1bcd368b2faeb388&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="50%" height="50%"/></center>

2. em.clear() : 영속성 컨텍스트를 완전히 초기화한다.해당 메서드 실행 시, 영속성 컨텍스트를 초기화해 영속성 컨텍스트가 관리하던 모든 엔티티를 준영속 상태로 만듭니다.

   <center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/492f6412-6c02-4068-a133-dfca80067a9b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041603Z&X-Amz-Expires=86400&X-Amz-Signature=9c949299b86b99e2e14cdb705878426031ae3979c526d06c3872e5b8e4daba42&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="50%" height="50%"/></center>

3. em.close() : 영속성 컨텍스트틑 종료한다. 영속성 컨텍스트가 종료되면서 관리하던 엔티티를 모두 준영속 상태가 됩니다.

   <center><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/55e644f4-dcbf-43d7-8295-b454af98e2e4/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210831%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210831T041619Z&X-Amz-Expires=86400&X-Amz-Signature=b2510f2f9dddd8672e5596e90dd337756213a3b57c0ec54b1e2887c0dfe04a9c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22"  width="50%" height="50%"/></center>

<br>

**🤔 Q. 영속성 엔티티를 준영속 상태를 변경한다면 DB에서 사라지나요?? NO!!**

 준영속 상태로 변경된다고 해서 DB의 저장된 정보가 사라지지는 않습니다. 말그대로 영속성 컨텍스트가 관리하지 않을 뿐이지 DB에 저장되어있는 값은 제거되지 않습니다.

<br>

### (2) 준영속 상태를 영속 상태로 : merge()

```java
public <T> T merge(T entity);
```

 Merge() 메서드를 이용해 준영속 상태의 엔티티의 정보를 통해 **새로운 영속 상태의 엔티티를 반환**합니다. 

<br>

### (3) merge 예시

``` Java
static void mergeMember(Member member) {
  EntityManager em2 = emf.createEntityManager();
  EntityTransaction tx2 = em2.getTransaction();
  
  tx2.begin();
  Member mergeMember = em2.merge(member);
  tx2.commit();
  
  System.out.println(em.contains(member)); //false
  System.out.println(em.contains(memgeMember)); //true
  
  em2.close();
}
```

