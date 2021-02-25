#### [GO TO BACK](../README.md)

# Chapter17. 스프링 관리하기 
> 스프링 부트 Admin 설정하기 
> 클라이언트 애플리케이션 등록하기  
> 액추에이터 엔드포인트 소비하기  
> Admin 서버의 보안  

### 17.1 스프링 부트 Admin 사용하기
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency> 
```
```java
@SpringBootApplication
@EnableAdminServer
public class BootAdminServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(BootAdminServerApplication.class, args);
    }
}
```
#### Admin 클라이언트 등록하기
- 각 애플리케이션이 자신을 Admin 서버에 등록한다.
- Admin 서버가 유레카 서비스 레지스트리를 통해서 서비스를 찾는다.
##### Admin 클라이언트 애플리케이션 구성하기
```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
</dependency>
```
```yaml
spring:
  application:
    name: ingredient-service
  boot:
    admin:
      client:
        url:http://localhost:9090
```
##### Admin 클라리언트 찾기
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
- 유레카 서비스로
```yaml
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka1.tacocloud.com:8761/eureka
```

### 17.2 Admin 서버 살펴보기

### 17.3 Admin 서버의 보안
- Admin 서버 로그인 활성화 하기
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
```yaml
spring:
  security:
    user:
      name: admin
      password: 53cr3t
  boot:
    admin:
      client:
        url: http://localhost:9090
        instance:
          metadata:
            user.name: ${spring.security.user.name}
            user.password: ${spring.security.user.password}
eureka:
  instance:
    metadata-map:
      user.name: admin
      user.password: password
```
### Admin-Server
```java
@Configuration
public class SecuritySecureConfig extends WebSecurityConfigurerAdapter {
	private final AdminServerProperties adminServer;

	public SecuritySecureConfig(AdminServerProperties adminServer) {
		this.adminServer = adminServer;
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		SavedRequestAwareAuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();
		successHandler.setTargetUrlParameter("redirectTo");
		successHandler.setDefaultTargetUrl(this.adminServer.path("/"));

		http.authorizeRequests()
			.antMatchers(this.adminServer.path("/assets/**")).permitAll()
			.antMatchers(this.adminServer.path("/login")).permitAll()
			.anyRequest().authenticated()
			.and()
		.formLogin().loginPage(this.adminServer.path("/login")).successHandler(successHandler).and()
			.logout().logoutUrl(this.adminServer.path("/logout")).and()
			.httpBasic().and()
			.csrf()
				.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
				.ignoringAntMatchers(
					this.adminServer.path("/instances"),
					this.adminServer.path("/actuator/**")
				);

	}
}

```
### Admin-Client
```java
@Override
@Order(2)
protected void configure(final HttpSecurity security) throws Exception {
//requestMatchers을 사용하면 일치하지 않는 요청이 나머지 다른 체인에 의해 확인됩니다.
//Actuator 외의 요청은 기존 보안 규칙을 따라야 하기 떄문에 requestMatchers를 사용해서 Actuator 보안설정만 진행한다.
security.requestMatcher(EndpointRequest.toAnyEndpoint())
    .authorizeRequests()
    .antMatchers("/actuator/health").permitAll() // /actuator/health 만 인증없이 접근할 수 있도록 함
    .anyRequest().authenticated()
    .and()
    .httpBasic();

}
```
```yaml
spring:
  security:
    user:
      name: "admin"
      password: "1234"
  application:
    name: "dev"
  boot:
    admin:
      client:
        url: "http://localhost:8080"
        username: "${spring.security.user.name}"
        password: "${spring.security.user.password}"
        instance:
          service-url: "http://localhost:8000"
          user:
            name: "${spring.security.user.name}"
            password: "${spring.security.user.password}"
management:
  security:
    enabled: true
  endpoints:
    web:
      exposure:
        include: "*"
    health:
      show-details: "always"

```
