#### [GO TO BACK](../README.md)

# 15. 고급 주제와 성능 최적화

### 15.1 예외 처리

##### 스프링 프레임워크에 JPA 예외 변환기 적용
```java
@Bean
public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
    return new PersistenceExceptionTranslationPostProcessor();
}
```

### 15.2 엔티티 비교
- 영속성 컨텍스트를 통해 데이터를 저장하거나 조회하면 1차 캐시에 저장된다.
- 동일성 (identical): == 가 참
- 동등성 (equinalent): equals() 참
- 데이터베이스 동등성: @Id인 식별자가 같다

### 15.3 프록시 심화 주제
```java
@Test
public void 영속성컨텍스트와_프록시() {
    Member newMember = new Member("member1", "회원1");
    em.persist(newMember);
    em.flush();
    em.clear();

    Member refMember = em.getReference(Member.class, "member1); // 프록시로 조회
    Member findMember = em.find(Member.class, "member1");
}
```
- 원본엔티티를 ㅁ너저 조회하면 프록시를 반환하지 않고 원본데이터가 반환됨.
- 프록시 타입을 비교할 때는 instanceof 를 사용

### 15.4 성능 최적화
- N + 1 문제, fetch LAZY 로딩
- 페치 조인 사용, join fetch
- 하이버네이트 @BatchSize 설정
- 하이버네이트 @Fetch(FetchMode.SUBSELECT)

#### 읽기 전용 쿼리의 성능 최적화 (메모리 사용량 최적화)
- 스칼라 타입으로 조회: select o.id, o.name from Order o
- 읽기 전용 쿼리 힌트 사용
```java
TypedQuery<Query> query = em.createQuery("select o from Order o", Order.class);
query.setHint("org.hibernate.readOnly", true)
```
- 읽기 전용 트랜젝션 사용
`@Transaction(readOnly = true)`
- 트랜젝션 밖에서 읽기
`@Transaction(propagation = Propagation.NOT_SUPPORTED`
#### 배치 처리
- 여러작업은 메모리 이슈를 초래하므로 일정 단위마다 엔티티를 플러시하고 초기화 해야한다.
