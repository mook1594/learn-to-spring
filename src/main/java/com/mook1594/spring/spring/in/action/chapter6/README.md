#### [GO TO BACK](../README.md)

# Chapter6. 스프링 시큐리티
> 스프링 MVC에서 REST엔드포인트 정의하기  
> 하이퍼링크 REST 리소스 활성화하기  
> 리퍼지터리 기반의 REST 엔드포인트 자동화  

### 6.1 REST 컨트롤러 작성하기
#### 스프링 MVC의 HTTP 요청 처리
- @GetMapping
- @PostMapping
- @PutMapping
- @PatchMapping
- @DeleteMapping
- @RequestMapping
#### 서버 데이터 가져오기
```java
@RestController
@RequestMapping(path="/design", produces="application/json")
@CrossOrigin(origins="*")
public class DesignTacoController {
    private TacoRepository tacoRepo;

    @Autowired
    EntityLinks entityLinks;

    public DesignTacoController(TacoRepository tacoRepo) {
        this.tacoRepo = tacoRepo;
    }

    @GetMapping("/recent")
    public Iterable<Taco> recentTacos() {
        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
        return tacoRepo.findAll(page).getContent();
    }
}
```
- @Controller 를 사용할 경우 @ResponseBody를 꼭 지정해야한다.

### 6.2 하이퍼미디어 사용하기
- HATEOAS: Hypermedia As The Engine Of Applicate State
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```
#### 하이퍼링크 추가하기
```java
@GetMapping("/recent")
public Resources<Resource<Taco>> recentTacos() {
    PageRequest page = PageRequest.of(0, 12, Sort.by("createdBy").descending());
    List<Taco> tacos = tacoRepo.findAll(page).getContent();
    Resources<Resource<Taco>> recentResouces = Resources.wrap(tacos);
    
    recentResources.add(new Link("http://localhost:8080/design/recent", "recents"));
    recentResuorces.add(
        ControllerLinkBuilder.linkTo(DesignTacoController.class)
            .slash("recent")
            .withRel("recents"));
    );
    recentResuorces.add(
        linkTo(methodOn(DesignTacoController.class).recentTacos())
            .withRel("recents"));
    )
    return recentResources;
}
```
```json
{
    "_links": {
      "recents": {
        "href": "http://localhost:8080/design/recent"
      }
    }
}
```
#### 리소스 어셈블러 생성
```java
public class TacoResource extends ResourceSupport {
    
    @Getter
    private final String name;
    
    @Getter
    private final Date createdAt;

    @Getter
    private final List<Ingredient> ingredients;

    public TacoResource(Taco taco) {
        this.name = taco.getName();
        this.createdAt = taco.getCreatedAt();
        this.ingredients = taco.getIngredients();
    }
}
```
```java
public class TacoResourceAssembler extends ResourceAssemblerSupport<Taco, TacoResource> {
    public TacoResourceAssembler() {
        super(DesignTacoController.class, TacoResource.class);
    }

    @Override
    protected TacoResource instantiateReource(Taco taco) {
        return new TacoResource(taco);
    }

    @Override
    public TacoResource toResource(Taco taco) {
        return createResourceWithId(taco.getId(), taco);
    }
}
```
```java
@Getmapping
public Resource<TacoResource> recentTacos() {
    PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").decending());
    List<Taco> tacos = tacoRepo.findAll(page).getContent();

    List<TacoResource> tacoResources = new TacoResourceAssembler().toResources(tacos);
    Resources<TacoResource> recentResources = new Resources<TacoResource>(tacoResources);

    recentResources.add(
        linkTo(methodOn(DesignTacoController.class).recentTacos())
        .withRel("recents"));
    
    return recentResources;
}
```
```java
class IngredientResourceAssembler extends ResourceAssemblerSuppert<Ingredient, IngredientResource> {
    public IngredientResourceAssembler() {
        super(IngredientController2.class, IngredientResource.class);
    }
    
    @Override
    public IngredientResource toResource(Ingredient ingredient) {
        return createResourceWithId(ingredient.getId(), ingredient);
    }

    @Override
    protected IngredientResource instantiateResource(Ingredient ingredient) {
        return new IngredientResource(ingredient);
    }
}
```
```java
public class IngredientResource extends ResourceSupport {
    
    @Getter
    private String name;

    @Getter
    private Type type;

    public IngredientResource(Ingredient ingredient) {
        this.name = ingredient.getName();
        this.type = ingredient.getType();
    }
}
```
```java
public class TacoResource extends ResourceSupport {
    private static final IngredientResourceAssembler ingredientAssembler = new IngredientResourceAssembler();

    @Getter
    private fnial String name;

    @Getter
    private final Date createdAt;

    @Getter
    private final List<IngredientResource> ingredients;

    public TacoResource(Taco toco) {
        this.name = taco.getName();
        this.createdAt = taco.getCreatedAt();
        this.ingredients = ingredientAssembler.toResource(taco.getIngredients());
    }
}
```

#### embedded 관계 이름 짓기
```json
{
    "_embedded": {
      "tacoResourceList": [
        ...
      ]
    }
}
```
```java
@Relation(value="taco", colletionRelation="tacos")
public class TacoResource extends ResourceSupport {
    ...
}
```

### 데이터 기반 서비스 활성화 하기
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```
