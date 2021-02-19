#### [GO TO BACK](../README.md)

# Chapter12. 리액티브 데이터 퍼시스턴스 
> 스프링 데이터의 리액티브 리퍼지터리  
> 카산드라와 몽고DB의 리액티브 리퍼지터리 작성하기  
> 리액티브가 아닌 리퍼지터리를 리액티브 사용에 맞추어 조정하기  
> 카산드라를 사용한 데이터 모델링  

- 블로킹이 있는 명령행 코드
- 블로킹이 없는 리액티브 코드

### 12.1 스프링 데이터의 리액티브 개념 이해하기
- 아쉽게도 관계형 데이터베이스를 리액티브하게 사용하기 위한 지원이 되지 않는다.
- 최근에 r2dbc 가 나옴
```xml
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-pool</artifactId>
    <version>0.8.0.RELEASE</version>
</dependency>

<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-pool</artifactId>
    <version>0.8.0.BUILD-SNAPSHOT</version>
</dependency>

<dependency>
    <groupId>dev.miku</groupId>
    <artifactId>r2dbc-mysql</artifactId>
    <version>0.8.1.RELEASE</version>
</dependency>
```
#### 스프링 데이터 리액티브 개요
```java
Flux<Ingredient> findByType(Ingredient.Type type);
Flux<Taco> saveAll(Publisher<Taco> tacoPublisher);
```
#### 리액티브와 리액티브가 아닌 타입간 변환
- 명령 to 리액티브
```java
List<Order> orders = repo.findByUser(someUser);
Flux<Order> orderFlux = Flux.fromIterable(orders);

Order order = repo.findById(Long id);
Mono<Order> orderMono = Mono.just(order);
```
- 리액티브 to 명령
```
Taco taco = tacoMono.block();
tacoRepo.save(taco);

Iterable<Taco> tacos = tacoFlux.toIterable();
tacoRepo.saveAll(tacos);
```
- .block(), .toIterable() 할 때 블로킹이 되므로 리액티브 프로그래밍 모델을 벗어난다. (지양)
- 조금 더 리액티브한 방법 (save()는 여전히 블로킹 오퍼레이션 이지만, 데이터를 소비할때 리액티브 방식을 사용하는게 일괄처리보다 더 바람직)
```java
tacoFlux.subscribe(taco -> {
    tacoRepo.save(taco);
});
```
#### 리액티브 리퍼지터리 개발
- 스프링 데이터는 런타임시에 자동으로 구현

### 12.2 리액티브 카산드라 리퍼지터리 사용
- 카산드라: 분산처리, 고성능, 상시 가용, 궁극정인 일관성 갖는 NoSQL 데이터베이스 (분산노드 분할-한 노드가 전체 데이터를 갖지 않음)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra-reactive</artifactId>
</dependency>
```
```shell script
cqlsh> create keyspace tacocloud
    ... with replication={'class':'SimpleStrategy', 'replication_factor':1}
    ... and durable_writes=true;
```
```yaml
spring:
  data:
    cassandra:
      keyspace-name: tacocloud
      schema-action: recreate-drop-unused # 개발에 적합, 테이블 재생성
      contact-points:
      - casshost-1.tacocloud.com
      - casshost-2.tacocloud.com
      - casshost-3.tacocloud.com
      port: 9043
      username: tacocloud
      password: s3cr3tP455w0rd
      
```
