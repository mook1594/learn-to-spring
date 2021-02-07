#### [GO TO BACK](../README.md)

# Chapter16. 스프링 부트 액추에이터 사용하기
> 스프링 부트 프로젝트에 액추에이터 활성화하기   
> 액추에이터 엔드포인트 살펴보기  
> 액추에이터 커스터마이징  
> 액추에이터 보안 처리하기  

##### 엑추에이터
- 시행중인 애플리케이션 내부를 볼 수 있는 방법 
- 어떻게 작동하는지, 건강 상태 확인, 실행에 영향주는 오퍼레이션 수행

### 16.1 액추에이터 개요
##### : 애플리케이션 내부 상태에 관한 것을 알 수 있음
- 애플리케이션 환경에서 사용할 수 있는 구성 속성들 
- 애플리케이션에 포함된 다양한 패키징의 로깅 레벨 
- 애플리케이션이 사용 중인 메모리 
- 지정된 엔드포인트가 받은 요청 횟수 
- 애플리케이션의 건강 상태 정보
##### dependency 추가 
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
- 액추에이터 엔드포인트 (prefix: /actuator)

|HttpMethod|endpoint|설명|초기화여부|
|---|---|--------|---|
|GET|/auditevent|호출된 감사(audit) 이벤트 리포트 생성|N|
|GET|/beans|컨텍스트의 모든 빈|N|
|GET|/conditions|성공,실패한 자동구성조건 내역|N|
|GET|/configprops|모든 구성 속성들을 현재값과 같이 알려줌|N|
|GET  POST  DELETE|/env|모든 속성 근원, 근원 속성|N|
|GET|/env/{toMatch}|특정 환겅의 속성값|N|
|GET|/health|앱 건강 상태 정보|Y|
|GET|/heapdump|힙(heap) 덤프를 다운로드|N|
|GET|/httptrace|최근 100개 요청에 대한 추적 기록|N|
|GET|/info|개발자가 정의한 애플리케이션 관한 정보 반환|Y|
|GET|/loggers|패키지 리스트 생성|N|
|GET POST|/loggers/{name}|로거레벨 반환|N|
|GET|/mappings|모든 HTTP 매핑과 핸들러 메서드 내역 제공|N|
|GET|/metrics|모든 메트릭 리스트 반환|N|
|GET|/metrics/{name}|지정된 메트릭 반화|N|
|GET|/scheduledtasks|스케줄링된 모든 태스크 내역 제공|N|
|GET|/threaddump|모든 애플리케이션 스레드 내역 반환|N|

#### 액추에이터 기본 경로 구성
- prefix: /actuator, 변경하려면
```yaml
management:
  endpoints:
    web:
      base-path: /management
```
#### 액추에이터 엔드포인트의 활성화와 비활성화
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
#        include: health,info,beans,conditions
        exclude: threaddump,heapdump
```
#### 액추에이터 엔드포인트 소비하기
- HATEOAS ??
#### 애플리케이션 기본 정보 가져오기
- /info, /health (누구, 상태는?)
##### /info
- 구성
```yaml
info:
  contact:
    eamil: support@tacocloud.com
    phone: 822-625-6831 
```
##### /health
- 응답: UP(작동중), DOWN(작동안함 or 접근불가), UNKNOWN(알수없음), OUT_OF_SERVICE(접근가능, 사용불가)
```yaml
management: 
  endpoint:
    health:
      slow-details: always # never, when-authorized
```
- 외부 데이터 (Cassandra, 구성서버, Couchbase, 유레카, Hysrix, JDBC, Elasticsearch, InfluxDB, JMS버 메시지 브로커, LDAP, 이메일 서버, Neo4j, Rabbit 메시지 브로커, Redis, Solr) 

#### 구성 상세 정보 보기
##### /beans: 빈 연결 정보
- 응답 최상위 요소는 contexts.
##### /conditions: 자동-구성 내역 (Autowired)
- 긍정 일치 (positive matches): 성공한 조건부 구성
- 부정 일치 (negative matches): 실패한 조건부 구성
- 조건 없는 클래스 (unconditional classes)
##### /env: 환경 속성과 구석 속성 살펴보기
- activeProfiles, propertySources, applicationConfig
##### /mappings: HTTP 요청-매핑 내역 보기
##### /loggers: 로깅 레벨 관리하기
- GET으로 조회
- POST로 configured 로깅 레벨 변경 `{"configuredLevel":"DEBUG"}`
- configuredLevel을 변경하면 effectiveLevel도 변경한다.
##### /heapdump: 힙 덤프
- 메모리나 스레드 문제를 찾는데 사용하 수 있는 gzip압축 형태의 HPROF 힙 덤프 파일을 다운로드 한다. `(힙 덤프 사용방법 보기)`
##### /httptrace: HTTP 요청 추적하기
- 가장 최근 100개의 요청
##### /threaddump: 스레드 모니터링
- 현재 실행중인 스레드 관한 스냅샷
- 스레드의 브록킹, 록킹, 스택 기록 
##### /metrics: 런타임 메트릭 활용하기
- 메모리, 프로세스, 가비지 컬렉션, HTTP 요청 관련 메트릭
- ex) /actuator/metrics/http.server.requests?tag=status:404&tag=uri:/**
- measurements 응답을 봐야함, availableTags로 태그를 추가로 넣을 수 있다.

### 16.3 액추에이터 커스터마이징
#### /info 엔드포인트에 정보 제공하기
- info에 동적 데이터 추가하기
```java
@Component
public class TacoCountInfoContributor implements InfoContributor {
    private TacoRepository tacoRepo;

    public TacoCountInfoContributor(TacoRepository tacoRepo) {
        this.tacoRepo = tacoRepo;
    }

    @Override
    public void contribute(Builder builder) {
        long tacoCount = tacoRepo.count();
        Map<String, Object> tacoMap = new HashMap<String, Object>();
        tacoMap.put("count", tacoCount);
        builder.withDetail("taco-stats", tacoMap);
    }
}
```
##### 빌드 정보 주입
- 빌드 정보 활성화
- build-info.properties 파일
```xml
# pom.xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
```json
# gradle
springBoot {
  buildInfo()
}
```
##### Git 커밋 정보 노출하기
- git.properties 노출
```xml
# pom.xml
<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>        
        </plugin>    
    </plugins>
</build>
```
```json
# gradle
plugins {
  id: "com.gorylenko.gradle-git-properties" version "1.4.17"
}
```
- 정보 상세화
```yaml
management:
  info:
    git:
      mode: full
```
#### 커스텀 건강 지표 정의하기
```java
@Component
public class WackoHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int hour = Calendar.getInstance().get(Calendar.HOUR_OF_DAY);
        if (hour > 12) {
            return Health
                .outOfService()
                .withDetail("reason", "I'm out oof service after lunchtime")
                .withDetail("hour", hour)
                .build();
        }
        if (Math.random() < 0.1) {
            return Health
                .down()
                .withDetail("reason", "I break 10% of the time")
                .build();
        }
        return Health
            .up()
            .withDetail("reason", "All is good!")
            .build();
    }
}
```
#### 커스텀 메트릭 등록히기
```
@Component
public class TacoMetrics extends AbstractRepositoryEventListener<Taco> {
    private MeterRegistry meterRegistry;
    public TacoMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @Override
    protected void onAfterCreate(Taco taco) {
        List<Ingredient> ingredients = taco.getIngredients();
        for (Ingredient ingredient : ingredients) {
            meterRegistry.counter("tacocloud", "ingredient", ingredient.getId()).increment();
        }
    }
}
```
#### 커스텀 엔드포인트 생성하기
```
@Component
//@Endpoint(id="notes", enableByDefault=true) // default 설정으로 include 안해됨
@WebEndpoint(id="notes", enableByDefault=true) // default 설정으로 include 안해됨
public class NotesEndPoint {
    private List<Note> notes = new ArrayList<>();

    @ReadOperation
    public List<Note> notes() {
        return notes;
    }

    @WriteOperation
    public List<Note> addNote(String text) {
        notes.add(new Note(text));
        return notes;
    }

    public List<Note> deleteNote(int index) {
        if(index < notes.size()) {
            notes.remove(index);
        }
        return notes;
    }

    @RequiredArgsConstructor
    public class Note {
        @Getter
        private Date time = new Date();

        @Getter
        private final String text;
    }
}
```
- @WebEndpoint: HTTP 엔드포인트로만
- @JmxEndpoint: MBean 엔드포인트로만

### 16.4 액추에이터 보안 처리하기
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .requestMatcher(EndpointRequest.to("beans", "threaddump","loggers").toAnyEndpoint()
                                        .excluding("health", "info"))
            .authorizeRequests()
                .anyRequest().hasRole("ADMIN")
        // .authorizeRequests()
        //     .antMattchers("/actuator/**").hasRole("ADMIN")
        .and()
        .httpBasic();
}
```
