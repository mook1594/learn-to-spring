#### [GO TO BACK](../README.md)

# Chapter11. 리액티브 API 개발하기
> 스프링 WebFlux 사용하기  
> 리액티브 컨트롤러와 클라이언트 작성하고 테스트하기  
> REST API 소비하기  
> 리액티브 웹 애플리케이션의 보안  

### 11.1 스프링 WebFlux 사용하기 
- 스프링 MVC: 서블릿 기반, 스레드 블로킹과 다중 스레드 수행: 요청이 처리될때 스레드 풀에서 작업 스레드를 가져와 요청을 처리하며 작업 스레드가 종료될 때까지 요청 스레드는 블락.
- 단점: 요청량의 증가에 확장이 사실상 어려움. 처리가 느린작업이 있다면 심각한 상황 발생.
- 이를 보완할 비동기 웹 프레임워크
- 이벤트 루핑을 적용하여 적은 수의 스레드(CPU 코어당 하나-일반적)로 높은 확장성

#### 스프링 WebFlux
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```
##### 리액티브 컨트롤러
```java
@RestController
@RequestMapping
public class DesignTacoController {

    @GetMapping("/recent")
    public Flux<Taco> recentTacos() {
        return Flux.fromIterable(tacoRepo.findAll()).take(12);
    }

//    @GetMapping("/recent")
//    public Iterable<Taco> recentTacos() {
//        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
//        return tacoRepo.findAll(page).getContent();
//    }
}
```
##### 리액티브 리퍼지터리
```java
public interface TacoRepository extends ReactiveCrudRepository<Taco, Long>{
    
}
```
##### 단일 값 반환
```java
// as-is
@GetMapping("/{id}")
public Taco tacoById(@PathVariable("id") Long id) {
    Optional<Taco> optTaco = tacoRepo.findById(id);
    if(optTaco.isPresent()) {
        return optTaco.get();
    }
    return null;
}

// to-be
@GetMapping("/{id}")
public Mono<Taco> tacoById(@PathVariable("id") Long id) {
    return tacoRepo.findById(id);
}
```
##### RxJava 타입 사용
: Flux, Mono 뿐아니라 Observable, Single, Completable, Flowable 사용가능
```java
@GetMapping("/recent")
public Observable<Taco> recentTaco() {
    return tacoService.getRecentTacos();
}
@GetMapping("/{id}")
public Single<Taco> tacoById(@PathVariable("id") Long id) {
    return tacoService.lookupTaco(id);
}
```
##### 리액티브하게 입력 처리
```java
// as-is
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
    return tacoRepo.save(taco);
}

// to-be
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public Mono<Taco> postTaco(@RequestBody Mono<Taco> tacoMono) {
    return tacoRepo.saveAll(tacoMono).next();
}
```

### 11.2 함수형 요청 핸들러 정의
- RequestPredicate: 처리될 요청 종류 선언
- RouterFunction: 일치하는 요청이 핸들러에 어떻게 전달되지 선언
- ServerRequest: HTTP 요청, 헤더와 몸체
- ServerResponse: HTTP 응답, 헤더와 몸체
```java
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;
import static reactor.core.publisher.Mono.just;

@Configuration
public class RouterFunctionConfig {
    
    public RouterFunction<?> helloRouterFunction() {
        return route(GET("/hello"), 
            request -> ok().body(just("Hello World!"), String.class))
            .andRout(GET("/bye"),
            request -> ok().body(just("See ya!"), String.class));
    }
}
```
#### 컨트롤러와 같은 리액티브 함수형 방식
```java
@Configuration
public class RouterFunctionConfig {

    @Autowired
    private TacoRepository tacoRepo;

    @Bean
    public RouterFunction<?> routerFunction() {
        return route(GET("/design/taco"), this::recents)
            .andRoute(POST("/design"), this::postTaco);
    }
    
    public Mono<ServerResponse> recents(ServerRequest request) {
        return ServerResponse.ok()
            .body(tacoRepo.findAll().take(12), Taco.class);
    }
    public Mono<ServerResponse> postTaco(ServerRequest request) {
        Mono<Taco> taco = request.bodyToMono(Taco.class);
        Mono<Taco> savedTaco = tacoRepo.save(taco);
        return ServerResponse.created(URI.create("http://localhost:8000/design/taco/" + savedTaco.getId()))
            .body(savedTaco, Taco.class);
    }
}
```

### 11.3 리액티브 컨트롤러 테스트하기 
#### GET 요청 테스트
```java
@Test
public void shouldReturnRecentTacos() {
    Taco[] tacos = {
        testTaco(1L), testTaco(2L), testTaco(3L), testTaco(4L), testTaco(5L), testTaco(6L) ...
    };
    Flux<Taco> tacoFlux = Flux.just(tacos);

    TacoRepository tacoRepo = Mockito.mock(TacoRepository.class);

    when(tacoRepo.findAll()).thenReturn(tacoFlux);
    
    WebTestClient testClient = WebTestClient.bindToController(
        new DesignTacoController(tacoRepo))
            .build();
    testClient.get().uri("/design/recent")
        .exchange() // 해당 요청 제출
        .expectStatus().isOk()
        .expectBody()
            .jsonPath("$").isArray()
            .jsonPath("$").isNotEmpty()
            .jsonPath("$[0].id").isEqualTo(tacos[0].getId().toString())
            .jsonPath("$[0].name").isEqualTo("Taco 1").jsonPath("$[1].id")
            .isEqualTo("Taco 2").jsonPath("$[11].id")
            .isEqualTo(tacos[11].getId().toString())
        ...
            .jsonPath("$[11].name").isEqualTo("Taco 12").jsonPath("$[12]")
        .doesNotExist()
        .jsonpath("$[12]").doesNotExist(); // 마지막 인덱스 체
}
```
- jsonPath가 많아져 복잡. json() 활용
```java
ClassPathResource recentsResource = new ClassPathResource("/tacos/recent-tacos.json");
String recentsJson = StreamUtils.copyToString(recentResource.getInputStream(), Charset.defaultCahrset());

testClient.get().uri("/design/recent")
    .accept(MediaType.APPLICATION_JSON)
    .exchange()
    .expectStatus().isOk()
    .expectBodyList(Taco.class)
    .contains(Arrays.copyOf(tacos, 12));
```
#### POST 요청 테스트
```java
@Test
public void shouldSaveATaco() {
    TacoRepository tacoRepo = Mockito.mock(TacoRepository.class);
    Mono<Taco> unsavedTacoMono = Mono.just(testTaco(null));
    Taco savedTaco = testTaco(null);
    savedTaco.setId(1L);
    Mono<Taco> savedTacoMono = Mono.just(savedTaco);

    when(tacoRepo.save(any())).thenReturn(savedTacoMono);

    WebTestClient testClient = WebTestClient.bindToController(
        new DesignTacoController(tacoRepo)).build();

    testClient.post()
        .uri("/design")
        .contentType(MediaType.APPLICATION_JSON)
        .body(unsavedTacoMono, Taco.class)
        .exchange()
        .expectStatus().isCreated()
        .expectBody(Taco.class)
        .isEqualTo(savedTaco);
}
```
#### 실행 중인 서버로 테스트
```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class DesignTestControllerWebTest {

    @Autowired
    private WebTestClient testClient;

    public void shouldReturnRecentTacos() throws IOException {
        testClient.get().uri("/design/recent")
            .accept(MediaType.APPLICATION_JSON).exchange()
            .expectStatus().isOk()
            .expectBody()
                .jsonPath("$[?(@.id == 'TACO1')].name")
                    .isEqualTo("Carnivore")
                .jsonPath("$[?(@.id == 'TACO2')].name")
                    .isEqualTo("Bovine Bounty")
                .jsonPath("$[?(@.id == 'TACO3')].name")
                    .isEqualTo("Veg-Out");
    }
}
```

### 11.4 REST API를 리액티브하게 사용
- RestTemplate, WebClient
##### GET
```java
Mono<Ingredient> ingredient = WebClient.create()
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Ingredient.class);

ingredient.subscribe(i -> { ... })
```

#### 기본 URI 요청
```java
@Bean
public WebClient webClient() {
    return WebClient.create("http://localhost:8080");
}
```
```java
@Autowired
WebClient webClient;
public Mono<Ingredient> getIngredientById(String ingredientId) {
    Mono<Ingredient> ingredient = webClient
        .get()
        .uri("/ingredients/{id}", ingredientId)
        .retrieve()
        .bodyToMono(Ingredient.class);
    ingredient.subscribe(i -> { ... })
}
```
#### 오래 실행되는 요청 타임아웃
```java
Flux<Ingredient> ingredients = WebClient.create()
    .get()
    .uri("http://localhost:8080/ingredients")
    .retrieve()
    .bodyToFlux(Ingredient.class);

ingredients
    .timeout(Duration.ofSeconds(1))
    .subscribe(
        i -> { ... },
        e -> {
            // handle timeout error
        })
```
##### PUT, POST
```java
Mono<Ingredient> ingredientMono = ...;
Mono<Ingredient> result = webClient
    .post()
    .uri("/ingredients")
    .body(ingredientMono, Ingredient.class)
    .retrieve()
    .bodyToMono(Ingredient.class);

result.subscribe(i -> { ... })
```
```java
Ingredient ingredient = ...;
Mono<Ingredient> result = WebClient
    .put()
    .uri("ingredients")
    .syncBody(ingredient)
    .retrieve()
    .bodyToMono(Void.class)
    .subscribe();
```
##### DELETE
```java
Mono<Void> result = webClient
    .delete()
    .uri("/ingredients/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Void.class)
    .subscribe();
```
#### 에러 처리
```java
Mono<Ingredient> ingredientMono = webClient
    .get()
    .uri("http://localhost:8080/ingredient/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Ingredient.class);

ingredientMono.subscribe(
    ingredient -> {
        // 식자재 데이터를 처리한다.
    }, error -> {
        // 에러를 처리한다.
    })
```
```java
Mono<Ingredient> ingredientMono = webClient
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .retrieve()
    .onStatus(HttpStatus::is4xxClientError, // status -> status == HttpStatus.NOT_FOUND,
        response -> Mono.just(new UnknownIngredientException()))
    .bodyToMono(Ingredient.class);
```
#### 요청 교환하기
##### exchange()
```java
Mono<Ingredient> ingredientMono = webClient
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .exchange()
    .flatMap(cr -> cr.bodyToMono(Ingredient.class));
```
##### retrieve()
```java
Mono<Ingredient> ingredientMono = webClient
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Ingredient.class);
```
##### exchange() 또다른 기능
```java
Mono<Ingredient> ingredientMono = webClient
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .exchange()
    .flatMap(cr -> {
        if(cr.headers().header("X_UNAVAILABLE").contains("true")) {
            return Mono.empty();
        }
        return Mono.just(cr);
    })
    .flatMap(cr -> cr.bodyToMono(Ingredient.class));
```

### 11.5 리액티브 웹 API 보안
- 스프링 시큐리티는 서블릿 필터 중심으로 만들어져서 리액티브 웹에 맞치 않음.
- 스프링의 WebFilter로 서플릿 필터를 대신함.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
#### 리액티브 웹 보안 구성
- as-is
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/design", "/orders").hasAuthority("USER")
            .antMatchers("/**").permitAll();
    }
}
```
- to-be
```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        return http
            .authorizeExchange()
            .pathMatchers("/design", "/orders").hasAuthority("USER")
            .anyExchange().permitAll()
            .and()
            .build();
    }
}
```
- configure을 오버라이드 하지않음.

#### 리액티브 사용자 명세 서비스 구성
```java
@Autowired
UserRepository userRepo;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .userDetailService(new UserDetailsService() {
            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                User user = userRepo.findByUsername(username)
                if(user == null) {
                    throw new UsernameNotFoundException(username);
                }
                return user.toUserDetails();
            }
        });
}
```
```java
@Service
public ReactiveUserDetailsService userDetailsService(UserRepository userRepo) {

    return new ReactiveUserDetailsService() {
        @Override
        public Mono<UserDetails> findByUsername(String username) {
            return userRepo.findByUsername(username)
                        .map(user -> {
                            return user.toUserDetails();
                        });
        }   
    };
}

```
