#### [GO TO BACK](../README.md)

# Chapter4. 스프링 시큐리티

> 스프링 시큐리티(Spring Security) 자동-구성하기  
> 커스텀 사용자 스토리지 정의하기  
> 커스텀 로그인 페이지 만들기  
> CSRF 공격으로부터 방어하기  
> 사용자 파악하기

### 1. 스프링 시큐리티 활성화하기
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>    
    </dependency>
</dependencies>
```

### 2. 스프링 시큐리티 구성하기
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/design", "/orders")
                    .access("hasRole('ROLE_USER')")
                .antMatchers("/", "/**").access("permitAll")
            .and()
                .httpBasic();
    }

    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user1")
            .password("{noop}password1")
            .authorities("ROLE_USER")
            .and()
            .withUser("user2")
            .password("{noop}password2")
            .authorities("ROLE_USER");
    } 
}
```
- SecurityConfig: 사용자의 HTTP 요청 경로에 대해 접근 제한 같은 보안 처리 관련 설정

#### 2.1 인메모리 사용자 스토어
```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user1")
        .pssword("{noop}password1")
        .authorities("ROLE_USER")
        .and()
        .withUser("user2")
        .password("{noop}password2")
        .authorities("ROLE_USER");
}
```
#### 2.2 JDBC 기반의 사용자 스토어
```java
@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .jdbcAuthenticataion()
        .dataSource(dataSource)
        .usersByUsernameQuery(
            "SELECT username, password, enabled from users " +
            "WHERE username = ?")
        .authoritiesByUsernameQuery(
            "SELECT username, authority from authorities " +
            "WHERE username = ?")
        .passwordEncoder(new BCryptPasswordEncoder());
}
```
```sql
create table if not exists users (
    username varchar2(50) not null primary key,
    password varchar2(50) not null,
    enabled char(1) default '1');
)

create table if not exists authorities (
    username varchar2(50) not null,
    authority varchar2(50) not null,
    constraint fk_authorities_users
        foreign key(username) references users(username));
)

create unique index ix_auth_username on authorities(username, authority);
```
##### 패스워드 암호화
- BCryptPasswordEncoder: bcrypt 해싱 암호화
- NoOpPasswordEncoder: 암호화하지 않음
- Pbkdf2PasswordEncoder: PBKDF2 암호화
- SCryptPasswordEncoder: scrypt 해싱 암호화
- StandardPasswordEncoder: SHA-256 해싱 암호화

#### 2.3 LDAP 기반 사용자 스토어
#### 2.4 사용자 인증의 커스터마이징
```java
@Entity
@Data
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@RequiredArgsConstructor
public class User implements UserDetails {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private final String username;
    private final String password;
    private final String fullname;
    private final String street;
    private final String city;
    private final String state;
    private final String zip;
    private final String phoneNumber;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```
```java
public interface UserRepository extends CrudRepository<User, Long> {
    User findByUsername(Strinig username);
}
```
```java
@Service
public class UserRepositoryUserDetailsService implements UserDetailsService {
    private UserRepository userRepo;

    @Autowired
    public UserRepositoryUserDetailsService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepo.findByUsername(username);
        if(user != null) {
            return user;
        }
        throw new UsernameNotFoundException("User '" + username + "' not found");
    }
}
```
```java
import org.springframework.beans.factory.annotation.Autowired;public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserDetailsService userDetailService;

    @Bean
    public PasswordEncoder encoder() {
        return new BCryptPasswordEncode();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(encoder());
    }
}
```

#### 4.3 웹 요청 보안 처리하기
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        .antMatchers("/design", "/orders")
        .hasRole("ROLE_USER")
        .antMatcher("/", "/**").permitAll();
}
```
##### 요청 경로 보안 처리 방법
- access(String): 인자로 전달된 표현식이 true면 접근가능
- anonymous(): 익명의 사용자에게 접근을 허용
- authenticated(): 익명이 아닌 사용자로 인증된 경우 접근 허용
- denyAll(): 무조건 접근을 거부
- fullyAuthenticated(): 익명이 아니거나 remember-me가 아닌 사용자로 인증되면 접근 허용
- hasAnyAuthority(String...): 지정된 권한 중 어떤것이라도 사용자가 갖고 있으면 허용
- hasAnyRole: 지정된 역할 중 어느 하나라도 사용자가 갖고있으면 접근 허용
- hasAuthority(String): 지증된 권한을 사용자가 갖고있으면 허용
- hasIpAddress(String): 지정된 IP 주소로부터 요청이 오면 허용
- hasRole(String): 지정된 역할을 갖고 있으면 허용
- not(): 다른 접근 메서드들의 효력 무효화
- permitAll(): 무조건 접근을 허용
- rememberMe(): remember-me통해 인증된 사용자 접근 허용


