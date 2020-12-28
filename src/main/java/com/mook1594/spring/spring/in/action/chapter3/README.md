#### [GO TO BACK](../README.md)

# Chapter3. 데이터로 작업하기

> 스프링 JdbcTemplate 사용하기  
> SimpleJdbcInsert를 사용해서 데이터 추가하기  
> 스프링 데이터(Spring Data)를 사용해서 JPA 선언하고 사용하기
> persistence(저장 및 지속성 유지)

### 1. JDBC를 사용해서 데이터 읽고 쓰기
- JdbcTemplate을 사용해서 데이터베이스 쿼리하기
```java
private JdbcTemplate jdbc;

@Override
public Ingredient findById(String id) {
    return jdbc.queryForObject(
        "select id, name, type from Ingredient where id=?",
        this::mapRowToIngredient, id);
    );
}

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
    return new Ingredient(
        rs.getString("id"),
        rs.getString("name"),
        Ingredient.Type.valueOf(rs.getString("type"))
    );
}
```

#### JdbcTemplate 사용하기
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>testt</scope>
    </dependency>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
...
<properties>
    ...
    <h2.version>1.4.196</h2.version>
</properties>
...
```
- jdbc 의존썽 추가
- h2 내장 데이터베이스 사용

##### JDBC 리퍼지터리 정의하기
- 전체 데이터 조회 
- id 하나로 데이터 조회
- 데이터 저장 
```java
public interface IngredientRepository {
    Iterable<Ingredient> findAll();
    Ingredient findById(Striing id);
    Ingredient save(Ingredient ingredient);
}
```
```java
@Repository
public class JdbcIngredientRepository implements IngredientRepository {
    private JdbcTemplate jdbc;
    
    @Autowired
    public JdbcIIngredientRepository(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }

    @Override
    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type from Ingredient",
            this::mapRowToIngredient);
    }

    @Override
    public Ingredient findById(String id) {
        return jdbc.queryForObject("select id, name, type from Ingredient where id=?",
            this::mapRowToIngredient, id);
    } 
    
    @Override
    public Ingredient save(Ingredient ingredient) {
        jdbc.update(
            "insert into Ingredient (id, name, type) values (?, ?, ?)",
            ingredient.getId(),
            ingredient.getName(),
            ingredient.getType().toString()
        );
        return ingredient;
    }
    
    private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
        return new Ingredient(
            rs.getString("id"),
            rs.getString("name"),
            Ingredient.Type.valueOf(rs.getString("type")));
    }
}
```
- @Repository: 스테레오타입 어노테이션 (@Controller, @Component), 스프링 컴포넌트에서 검색하여 SAC의 빈으로 자동생성
- 스테레오타입 어노테이션: 스프링의 역할 그룹을 나타내는 어노테이션, @Component, @Repository, @Controller, @Service

##### @SessionAttributes
```java
@Slf4j
@Controller
@RequestMapping("/design")
@SessionAttributes("order")
public class DesignTacoController {
    ...

    @ModelAttributte(name = "order")
    public Order order() {
        return new Order();
    }
    
    @ModelAttribute(name = "taco")
    public Taco taco() {
        return new Taco();
    }

    @PostMapping
    public String processDesign(
        @Valid Taco design,
        Errors errors, @ModelAttribute Order order
    ) {
        if(errors.hasErrors()) {
            return "design";
        }
    
        Taco saved = tacoRepo.save(design);
        order.addDesign(saved);
        return "redirect:/orders/current";
    }
}
```
- @ModelAttribute: 객체가 모델에 생성되도록 한다.
- @SessionAttributes: 세션에 계속 보존되면섯 다수 요청에 걸처 사용가능 하다

#### SimpleJdbcInsert를 사용해ㅐ서 데이터 추가하기
```java
@Repository
public class JdbcOrderRepository implements OrderRepository {
    private SimpleJdbcInsert orderInserter;
    private SimpleJdbcInsert orderTacoInserter;
    private ObjectMapper objectMapper;

    public JdbcOrderRepository(JdbcTemplate jdbc) {
        this.orderInserter = new SimpleJdbcInsert(jdbc)
            .withTableName("Taco_Order")
            .usingGeneratedKeyColumns("id");

        this.orderTacoInserter = new SimpleJdbcInsert(jdbc)
            .withTableName("Taco_Order_Tacos");

        this.objectMapper = new ObjectMapper();
    }

    @Override
    public Order save(Order order) {
        order.setPlaceAt(new Date());
        long orderId = saveOrderDetails(prder);
        order.setId(orderId);
        List<Taco> tacos = order.getTacos();

        for(Taco taco : tacos) {
            saveTacoToOrder(taco, orderId);
        }
        return order;
    }

    private long saveOrderDetails(Order order) {
        @SuppressWarnings("unchecked")
        Map<String, Object> values = 
            objectMapper.convertValue(order, Map.class);
        values.put("placeAt", order.getPlaceAt());

        long orderId = 
            orderInserter
                .executeAndReturnKey(values)
                .longValue();
        return orderId;
    }

    private void saveTacoToOrder(Taco taco, long orderId) {
        Map<String, Object> values = new HashMap<>();
        values.put("tacoOrder", orderId);
        values.put("taco", taco.getId());
        orderTacoInserter.execute(values);
    }
}
```
```java
@Component
public class IngredientByIdConverter implements Converter<String, Ingredient> {
    private IngredientRepository ingredientRepo;

    @Autowired
    public IngredientByIdConverter(IngredientRepository ingredientRepo) {
        this.ingredientRepo = ingredientRepo;
    }

    @Override
    public Ingredient convert(String id) {
        return ingredientRepo.findById(id);
    }
}
```

### 2. 스프링 데이터 JPA를 사용해서 데이터 저장하고 사용하기
- 스프링 데이터 JPA: Relation DB
- 스프링 데이터 MongoDB: Mongo Document DB
- 스프링 데이터 Neo4: Neo4j 그래프 DB
- 스프링 데이터 Redis: Redis key-value Store
- 스프링 데이터 Cassandra: Cassandra DB

#### 스프링 데이터 JPA 프로젝트 추가
```xml
<dependencies>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <!-- Hibernate 제외 -->
        <exclusions>
            <artifactId>hibernate-entitymanager</artifactId>
            <groupId>org.hibernate</groupId>
        </exclusions>
    </dependency>
</dependencies>
```

#### 도메인 객체에 애노테이션 추가
```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@Entity
public class Ingredient {

    @Id
    private final String id;
    ...
}
```
- @NoArgsConstructor: JPA는 개체가 인자 없는 생성자를 가져야 한다. 인자 없는 생성자의 사용을 원치 않으면 access 속성을 PRIVATE로 설정하면 된다.
    그리고 Ingredient에는 초기화가 필요한 final 속성이 있으므로 force를 true로 하였다. Lombok은 final을 모두 null로 초기화 한다.
- @RequiredArgsConstructor: 인자가 있는 생성자를 갖는다.
 
 ```java
@Data
@Entity
public class Taco {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private Date createdAt;

    @NotNull
    @Size(min=5, message="Name must be at least 5 characters long")
    private String name;

    @ManyToMany(targetEntity=Ingredient.class)
    @Size(min=1, message="You must choose at least 1 ingredient")
    private List<Ingredient> ingredients;

    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}
```
- @PrePersist: 객체가 저장되지 전에 createdAt()이 호출된다.

#### JPA 리퍼지터리 선언
```java
public interface IngredientRepository extends CrudRepository<Ingredient, String> {

}
```

#### JPA 리퍼지터리 커스터마이징
- IsAfter, After, IsGreaterThan, GreaterThan
- IsGreaterThanEqual, GreaterThanEqual
- IsBefore, Before, IsLessThan, LessThan
- IsLessThanEqual, LessThanEqual
- IsBetween, Between
- IsNull, Null
- IsNotNull, NotNull
- IsIn, In
- IsNotIn, NotIn
- IsStartingWith, StartingWith, StartsWith
- IsEndingWith, EndingWith, EndsWith
- IsContaining, Containing, Contains
- IsLike, Like
- IsNotLike, NotLike
- IsTrue, True
- IsFalse, False
- Is, Equals
- IsNot, Not
- IgnoringCase, IgnoresCase
- AllIgnoringCase, AllIgnoresCase

##### @Query
```java
@Query("Order o where o.deliveryCity='Seattle'")
List<Order> readOrdersDeliveredInseattle();
```
