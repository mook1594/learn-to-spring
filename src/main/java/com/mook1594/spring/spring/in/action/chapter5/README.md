#### [GO TO BACK](../README.md)

# Chapter4. 스프링 시큐리티
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
```
#### 구성 속성 메타데이터 선언
