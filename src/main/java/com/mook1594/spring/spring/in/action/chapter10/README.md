#### [GO TO BACK](../README.md)

# Chapter10. 리액터 개요
> 리액티브 프로그래밍 이해하기  
> 프로젝트 리액터  
> 리액티브 데이터 오퍼레이터  

- 명령형: 순차적으로 연속되는 작업. 한번에 하나씩. 이전작업 다음.
- 리액티브: 일련의 작업이 병렬 실행. 부분 집합의 데이터 처리. 일을 넘겨주고 다른일. (파이프라인-컨베이어벨트)

### 10.1 리액티브 프로그램 이해하기
#### 리액티브 스트림 정의하기
```java
// 발행자
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> subscriber);
}
// 구독자
public interface Subscriber<T> { 
    void onSubscribe(Subscription sub);
    void onNext(T item);
    void onError(Throwable ex);
    void onComplete();
}
// 구독
public interface Subscription {
    void request(long n);
    void cancel();
}
// 프로세서
public interface Processor<T, R>
    extends Subscriber<T>, Publisher<R> {}
```
- 구독 신청이 되면 발행자로 부터 이벤트 수신. 이벤트 들은 구독자 인터페이스의 메서드를 통해 전송. 

### 리액터 시작
```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
</dependency>
```
- 스프링 부트가 아닌경우 build 에 BOM 설정
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-bom</artifactId>
            <version>Bismuth-RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>    
    </dependencies>
</dependencyManagement>
```

### 10.3 리액티브 오퍼레이션 적용
- 생성 오퍼레이션
- 조합 오퍼레이션
- 변환 오퍼레이션
- 로직 오퍼레이션

#### 리액티브 타입 생성
```java
@Test
public void createAFlux_just() {
    Flux<String> fruitFlux = Flux.just("Apple", "Orange", "Grape", "Banana", "Strawberry");

    fruitFlux.subscribe(f -> System.out.println(f));

    StepVerifier.create(fruitFlux)
        .expectNext("Apple")
        .expectNext("Orange")
        .expectNext("Grape")
        .expectNext("Banana")
        .expectNext("Strawberry")
        .verifyComplete();
}
```
```java
@Test
public void createAFlux_fromArray() {
    String[] fruits1 = new String[] {"Apple", "Orange", "Grape", "Banana", "Strawberry"};
    List<String> fruits2 = Arrays.asList("Apple", "Orange", "Grape", "Banana", "Strawberry");
    Stream<String> fruits3 = Stream.of("Apple", "Orange", "Grape", "Banana", "Strawberry");

    Flux<String> fruitFlux = Flux.fromArray(fruits1);
    Flux<String> fruitFlux = Flux.fromIterable(fruits2);
    Flux<String> fruitFlux = Flux.fromStream(fruits3);

    StepVerifier.create(fruits)
        .expectNext("Apple")
        .expectNext("Orange")
        .expectNext("Grape")
        .expectNext("Banana")
        .expectNext("Strawberry")
        .verifyComplete();
}
```
#### Flux 데이터 생성
```java
Flux<Integer> intervalFlux = Flux.range(1, 5);
Flux<Long> intervalFlux = Flux.interval(Duration.ofSeconds(1))
    .take(5);
```
#### 리액티브 타입 결합
##### delayElements(), delaySubscription()
```java
Flux<String> characterFlux = Flux
    .just("Garfield", "Kojak", "Barbossa")
    .delayElements(Duration.ofMillis(500))
Flux<String> foodFlux = Flux
    .just("Lasagna", "Lollipops", "Apples")
    .delaySubscription(Duration.ofMillis(250))
    .delayElements(Duration.ofMillis(500));

Flux<String> mergedFlux = characterFlux.mergeWith(foodFlux);
StepVerifier.create(mergedFlux)
    .expectNext("Garfield")
    .expectNext("Lasagna")
    .expectNext("Kojak")
    .expectNext("Lollipops") 
    .expectNext("Barbossa") 
    .expectNext("Apples")
    .verifyComplete();
```
##### mergeWith()
```java
Flux<String> characterFlux = Flux
    .just("Garfield", "Kojak", "Barbossa");
Flux<String> foodFlux = Flux
    .just("Lasagna", "Lollipops", "Apples");

Flux<Tuple2<String, String>> zippedFlux = 
    Flux.zip(characterFlux, foodFlux);
StepVerifier.create(zippedFlux)
    .expectNextMatches(p ->
        p.getT1().)
```
