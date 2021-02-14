# 10. 객체지향 쿼리 언어

### 10.1 객체 지향 쿼리 소개
- 식별자 조회: EntityManager.find()
- 객체 그래프 탐색: a.getB().getC()

#### JPA 기능
- JPQL(java persistence query language)
    - 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
    - SQL을 추상화 해서 특정 데이터베이스 SQL에 의존하지 않음
- Criteria 쿼리: JPQL을 편리하게 도와주는 API 빌더 모음
- 네이티브 SQL
- QueryDSL: JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음
- JDBC 직접 사용

#### JPQL
- 엔티티 객체를 조회하는 객체 지향 쿼리
- SQL을 추상화 하여 특정 DB에 의존하지 않음
#### Criteria 쿼리
```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

Root<Member> m = query.from(Member.class);

CriteriaQuery<Member> cq = 
    query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();
```
- 메타 모델을 사용하여 하드코딩 대체
```java
m.get("username") -> m.get(Member_.username)
```
#### QueryDSL
- 오픈 소스. 대부분 지원
```java
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;

List<Member> members = 
    query.from(member)
        .where(member.username.eq("kim"))
        .list(member);
```
#### Native SQL
```java
String sql = "SELECT ID, AGE, TEAM_ID FROM MEMBER WHERE NAME = 'kim';"
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```

### 10.2 JQPL
#### 커스텀 반환타입 설정
- TypeQuery
```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m", Member.class);
```
- Query
```java
Query query = 
    em.createQuery("SELECT m.username, m.age from Member m");
List resultList = query.getResultList();

for (Object o : resultList) {
    Object[] result = (Object[]) o;
    System.out.println("username = " + result[0]);
    System.out.println("age = " + result[1]);
}
```
#### 데이터 바인딩
- 이름 기반
```java
TypedQuery<Member> query = 
    em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class);
query.setParameter("username", usernameParam);
List<Member> resultList = query.getResultList();
```
- 위치 기반
```java
List<Member> members =
    em.createQuery("SELECT m FROM Member where m.username = ?1", Member.class)
        .setParameter(1, usernameParam)
        .getResultList();
```
#### 페치조인
- 연관된 엔터티를 함께 조회 하여 호출 횟수를 줄여 성능 최적화 할 수 있음
- 최적화를 위해 즉시 로딩으로 설정하면, 일부는 빠르지만 전체로 보면 사용하지 않는 엔티티를 자주 로딩 하므로 오히려 악영향. 그래서 글로벌은 지연로딩을, 그다음 성능개선에 부분적인 패치를 적용한다. 
##### 단점
- 페치 조인 대상에는 별칭을 줄 수 없다.
- 둘 이상의 컬렉션을 페치할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.

#### 조건식
##### LIKE 식
```
where m.username like '%원%'
where m.username like '회원_'
where m.username like '__3'
where m.username like '회원\%' ESCAPE '/'
```
##### 컬렉션 식
```
where m.orders is not empty

//query
where exists (select .....)
```
##### COALESCE
```
SELECT coalesce(m.username, '이름 없는 회원') from Member m
```
##### NULLIF
```
SELECT nullif(m.username, '이름 없는 회원') from Member m
```

#### 다형성 쿼리
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) 
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item { ... }

@Entity
@DiscriminatorValue("B")
public class Book extends Item {
    ...
    private String author;
}
```
- inheritance 전력에 따라 join된 자식 엔티티까지 함께 조회 가능
- discriminatorColumn: 하위 클래스 구분용도 
- DiscriminatorValue: 상위 클래스 구분용도
##### TREAT
```java
SELECT i FROM Item i where TREAT(i as BOOK).author = 'kim'

// sql
select i.* from Item i where i.DTYPE = 'B' AND i.author = 'kim'
```

#### 사용자 정의 함수 호출
```
function_invocation::= FUNCTION(function_name (, function_arg)*)
select function('group_concat', i.name) from Item i
```
- 방언 클래스
```java
public class MyH2Dialect extends H2Dialect {
    public MyH2Dialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING))
    }
}
select group_concat(i.name) from name i
```

### 10.3 Criteria
#### 메타 모델 API
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <version>1.3.0.Final</version>
</dependency>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <congifuration>
                <source>1.6</source>
                <target>1.6</target>
                <compilerArguments>
                    <processor>org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor</processor>
                </compilerArguments>
            </congifuration>
        </plugin>
    </plugins>
</build>
```
```java
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetamodel(Member.class)
public abstract class Member_ {
    public static volatile SingularAttribute<Member, Long> id;
    public static volatile SingularAttribute<Member, String> username;
    public static volatile SingularAttribute<Member, Integer> age;
    public static volatile ListAttribute<Member, Order> orders;
    public static volatile SinglularAttribute<Member, Team> team;
}
```

### 10.4 QueryDSL
#### 설정
```xml
<dependency>
    <groupId>com.mysema.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>3.6.3</version>
</dependency>
<dependency>
    <groupId>com.mysema.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>3.6.3</version>
    <scope>provided</scope>
</dependency>

<build>
    <plugins>
        <plugin>
            <groupId>com.mysema.maven</groupId>
            <artifactId>apt-maven-plugin</artifactId>
            <version>1.1.3</version>
            <executions>
                <execution>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/generated-sources/java</outputDirectory>
                        <processor>com.mysema.query.apt.jpa.JPAAnnotationProcessor</processor>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<Item> list = query.from(item)
    .where(item.name.eq("좋은상품").and(item.price.gt(20000)))
    .list(item);
```
#### 결과 조회
- uniqueResult(): 없으면 null, 여러개면 NonUniqueResultException
- singleResult(): 결과중 첫번째 리턴
- list(): 결과가 하나 이상일때

#### 페이징 정렬
```java
QItem item = QItem.item;
query.from(item)
    .where(item.price.gt(20000))
    .orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    .list(item);
```
```java
QueryModifiers queryModifiers = new QueryModifiers(20L, 10L) // limit, offset
List<Item> list = 
    query.from(item)
        .restrict(queryModifiers)
        .list(item);

SearchResults<Item> result = 
    query.from(item)
        .where(item.price.gt(10000))
        .offset(10).limit(20)
        .listResults(item);
long total = result.getTotal();
long limit = result.getLimit();
long offset = result.getOffset();
List<Item> results = result.getResults();
```
#### 그룹
```java
query.from(item)
    .groupBy(item.price)
    .having(item.price.gt(1000))
    .list(item);
```
#### 조인
```java
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
    .join(order.member, member)
    .leftJoin(order.orderItems, orderItem).fetch()
    .on(orderItem.count.gt(2))
    .list(order);
```
#### 서브쿼리
```java
QItem item = QItem.item
QItem itemSub = new QItem("itemSub");

query.from(item)
    .where(item.in(
        new JPASubQuery().from(itemSub)
        .where(item.name.eq(itemSub.name))
        .list(itemSub)
    ))
    .list(item);
```
#### 결과 반환
```java
QItem item = QItem.item;
List<Tuble> result = query.from(item).list(item.name, item.price);

for(Tuple tuple : result) {
    System.out.println("name = " + tuple.get(item.name));
    System.out.println("price = " + tuple.get(item.price));
}
```
#### 빈 생성
```java
QItem item = QItem.item;
List<ItemDto> result = query.from(item).list(
    Projections.bean(ItemDTO.class, item.name.as("username"), item.price) // setter
    Projections.fields(ItemDTO.class, item.name.as("username"), item.price)
    Projections.constructor(ItemDTO.class, item.name, item.price)
);
```
#### 수정, 삭제 배치
- 수정
```java
QItem item = QItem.item;
JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
long count = updateClause.where(item.name.eq("개발자"))
    .set(item.price, item.price.add(100))
    .execute();
```
- 삭제
```java
QItem item = QItem.item;
JPADeleteClause deleteClause = new JPADeleteClause(em, item);
long count = deleteClause.where(item.name.eq("개발자"))
    .execute();
```
#### 동적 쿼리
```java
SearchParam param = new SearchParam();
param.setName("개발자");
param.setPrice(10000);

QItem item = QItem.item;

BooleanBuilder builder = new BooleanBuilder();
if(StringUtils.hasText(param.getName())) {
    builder.and(item.name.contains(param.getName()));
}
if(param.getPrice() != null) {
    builder.and(item.price.gt(param.getPrice()));
}
List<Item> result = query.from(item)
    .where(builder)
    .list(item);
```
#### 메소드 위임
```java
public class ItemExpression {
    @QueryDelegate(Item.class)
    public static BooleanExpression isExpensive(QItem item, Integer price) {
        return item.price.gt(price);
    }
    
    @QueryDelegate(String.class)
    public static BooleanExpression isHelloStart(StringPath stringPath) {
        return stringPath.startWith("Hello");
    }
}

public class QItem extends EntityPathBase<Item> {
    ...
    public com.mysema.query.types.expr.BooleanExpression isExpensive(Integer price) {
        return ItemExpression.isExpensive(this, price);
    }
}
```

### 10.6 객체 지향 쿼리 심화
- find는 영속성 영속상태
- JPQL은 항ㄹ상 데이터 베이스에서 실행

