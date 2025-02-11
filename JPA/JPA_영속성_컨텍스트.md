# JPA 영속성 컨텍스트

JPA를 사용하면서 영속성 컨텍스트와 로딩 전략에 대해 공부할 필요성을 느낌.

 특히 변경 감지(Dirty Checking)와 로딩 전략이 성능에 미치는 영향을 이해해야 한다.


### 영속성 컨텍스트란?

> Entity를 영구 저장하는 환경. 
EntityManager로 Entity를 저장하거나 조회하면 영속성 컨텍스트에 Entity를 보관하고 관리한다.


### Entity의 생명주기
###### 1. 비영속 상태(new/transient):
- Entity 객체를 생성했지만 아직 영속성 컨텍스트에 저장하지 않은 상태
- JPA가 해당 Entity를 관리하지 않음
######  2. 영속 상태(managed):
- Entity가 영속성 컨텍스트에 저장되어 관리되는 상태
- persist() 또는 save() 메서드 호출 시 이 상태로 전환
######  3. 준영속 상태(detached):
- 영속성 컨텍스트에 저장되었다가 분리된 상태
- detach() 메서드 호출 시 이 상태로 전환
######  4. 삭제 상태(removed):
- Entity를 영속성 컨텍스트와 데이터베이스에서 삭제한 상태
- remove() 메서드 호출 시 이 상태로 전환


## 변경 감지(Dirty Checking)

JPA는 엔티티의 변경사항을 자동으로 감지하여 데이터베이스에 반영

```java
@Transactional
public void exampleUpdateMember(Long id, String newName) {
    // 엔티티 조회
    Member member = memberRepository.findById(id).get();
    
    // 엔티티 데이터 수정
    member.setName(newName);
    
    // 별도의 update() 호출이 필요없음
    // 트랜잭션 커밋 시점에 변경 감지가 동작하여 UPDATE SQL 실행
}
```

### 변경 감지의 동작 순서
1. 트랜잭션 시작시 엔티티의 최초 상태를 스냅샷으로 저장
2. 트랜잭션 커밋 시점에 스냅샷과 엔티티의 현재 상태를 비교
3. 변경된 엔티티가 있다면 UPDATE SQL 생성
4. 쓰기 지연 SQL 저장소에 UPDATE SQL 추가
5. flush 호출하여 변경 내용을 데이터베이스에 반영
6. 트랜잭션 커밋

---

## 즉시 로딩과 지연 로딩
### 지연 로딩 (LAZY)
- 연관된 엔티티를 실제 사용하는 시점에 로딩
- 프록시 객체를 사용하여 지연 로딩 구현

```java
@Entity
public class Member {
    @Id
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;
}

// 사용 예시
Member member = entityManager.find(Member.class, 1L);
Team team = member.getTeam(); // 이 시점에 Team 조회 쿼리 실행
```

### 즉시 로딩 (EAGER)
- 엔티티를 조회할 때 연관된 엔티티도 함께 조회
- JOIN을 사용하여 한 번에 연관된 엔티티까지 조회

```java
@Entity
public class Member {
    @Id
    private Long id;
    
    @ManyToOne(fetch = FetchType.EAGER)
    private Team team;
}

// 사용 예시
Member member = entityManager.find(Member.class, 1L);
// Member와 Team을 조인해서 한 번에 조회
```

### 로딩 전략의 장단점

LAZY 로딩
- 장점: 필요한 시점에만 로딩하여 메모리 효율적
- 단점: N+1 문제 발생 가능

EAGER 로딩  
- 장점: 연관 Entity를 한번에 조회하여 추가 쿼리 없음
- 단점: 불필요한 조인으로 성능 저하 가능

--- 

### 적용 시 주의 사항

###### 1. 즉시 로딩은 지양
- 예상하지 못한 SQL 발생 가능성
- JPQL에서 N+1 문제 발생 위험

###### 2. 연관관계를 지연 로딩으로 설정

```
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.LAZY) // 지연 로딩 사용
    private Team team;
    
    @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}
```

###### 3. 성능 최적화가 필요한 경우
- fetch join 활용

```java
@Query("select m from Member m join fetch m.team")
List<Member> findMemberFetchJoin();
```

지연 로딩을 기본으로 사용하고, 필요한 경우에만 fetch join을 사용하여 최적화하는 것이 바람직.