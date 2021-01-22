#### [GO TO BACK](../README.md)

# Chapter5. 구성 속성 사용하기
> 자동-구성되는 빈 조정하기  
> 구성 속성을 애플리케이션 컴포넌트에 적용하기  
> 스프링 프로파일 사용하기  

### 5.1 자동-구성 세부 조정하기
- 빈 연결(Bean wiring)
- 속성 주입(Property Injection)

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.H2)
        .addScript("schema.sql")
        .addScripts("user_data.sql", "ingredient_data.sql")
        .build();
}
```
#### 스프링 환경 추상화 이해
- 모든 속성을 한 곳에서 관리
```properties
server.port=9090
```
```yaml
server:
    port: 9090
```
#### 데이터 소스 구성
```yaml
spring:
    datasource:
      url: jdbc:mysql://localhost/tococloud
      username: taco
      password: tacopassword
      driver-class-name: com.mysql.jdbc.Driver
```
#### 내장 서버 구성
#### 로깅 구성
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
            </pattern>
        </encoder>
    </appender>
    <logger name="root" level="INFO" />
    <root level="INFO">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```
```yaml
logging:
    path: /var/logs/
    file: TacoCloud.log
    level:
      root: WARN
      org:
        springframework:
          security: DEBUG
```
#### 다른 속성의 값 가져오기
```yaml
greeting:
    welcome: ${spring.application.name}
```

### 5.2 우리의 구성 속성 생성하기
```java
@GetMapping
public String ordersForUser (
    @AuthenticationPrincipal User user, Model model) {
    model.addAttribute("orders",
        orderRepo.findByUserOrderByPlacedAtDesc(user));

    return "orderList";   
}
```
#### 구성 속성 홀더 정의
```java
@Component
@ConfigurationProperties(prefix="taco.orders")
@Data
public class OrderProps {
    private int pageSize = 20;
}

@Controller
@RequestMapping("/orders")
@SessionAttributes("order")
public class OrderController {
    private OrderProps props;
    private OrderController(OrderRepository orderRepo,
                OrderProps props) {
        this.orderRepo = orderRepo;
        this.props = props;
    }

}
```
#### 구성 속성 메타데이터 선언
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>
        spring-boot-configuration-processor
    </artifactId>
    <optional>true</optional>
</dependency>
```
##### taco-cloud/src/main/resources/META-INF/additional-spring-configuration-metadata.json
```
{
    "properties": [
        {
            "name": "taco.orders.page-size",
            "type": "int",
            "description": "Sets the maximum number of orders to display in a list"
        }
    ]
}
```

### 5.2 프로파일 사용해서 구성하기
#### 프로파일 특정 속성 정의
- 파일 이름 규칙을 따라야한다
- application-{프로파일 이름}.yml
- application-{프로파일 이름}.properties
#### 프로파일을 사용해서 조건별로 빈 생성
- @Profile 애노테이션 활용
```java
@Bean
@Profile("dev")
public CommandLineRunner dataLoader(IngredientRepository repo,
    UserRepository userRepo, PasswordEncoder encoder) {
    ...
}   
```
```java
@Bean
@Profile({"dev", "qa"})
public CommandLineRunner dataLoader(IngredientRepository repo,
     UserRepository userRepo, PasswordEncoder encoder) {
     ...
 }  
```
```java
@Bean
@Profile("!prod")
public CommandLineRunner dataLoader(IngredientRepository repo,
     UserRepository userRepo, PasswordEncoder encoder) {
     ...
 }  
```
```java
@Profile("!prod", "!qa")
@Configuration
public class DevelopmentConfig {
    @Bean
    public CommandLineRunner dataLoader(IngredientRepository repo,
        UserRepository userRepo, PasswordEncoder encoder)   
}
```
