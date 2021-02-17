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
##### mergeWith()
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
##### zip()
```java
Flux<String> characterFlux = Flux
    .just("Garfield", "Kojak", "Barbossa");
Flux<String> foodFlux = Flux
    .just("Lasagna", "Lollipops", "Apples");

Flux<Tuple2<String, String>> zippedFlux = 
    Flux.zip(characterFlux, foodFlux);
StepVerifier.create(zippedFlux)
    .expectNextMatches(p ->
        p.getT1().equals("Garfield") && p.getT2().equals("Lasagna"))
    .expectNextMatches(p ->
        p.getT1().equals("Kojak") && p.getT2().equals("Lollipops"))
    .expectNextMatches(p ->
        p.getT1().equals("Barbossa") && p.getT2().equals("Apples"))
    .verifyComplete();
```
```java
Flux<String> characterFlux = Flux
    .just("Garfield", "Kojak", "Barbossa");
Flux<String> foodFlux = Flux
    .just("Lasagna", "Lollipops", "Apples");

Flux<String> zippedFlux = 
    Flux.zip(characterFlux, foodFlux, (c, f) -> c + " eats " + f);

StepVerifier.create(zippedFlux)
    .expectNext("Garfield eats Lasagna")
    .expectNext("Kojak eats Lollipops")
    .expectNext("Barbossa eats Apples")
    .verifyComplete();
```
##### first()
```java
Flux<String> slowFlux = Flux.just("tortoise", "snail", "sloth")
    .delaySubscription(Duration.ofMillis(100));
Flux<String> fastFlux = Flux.just("here", "cheetah", "squirrel");

Flux<String> firstFlux = Flux.first(slowFlux, fastFlux);

StepVerifier.create(firstFlux)
    .expectNext("here")
    .expectNext("cheetah")
    .expectNext("squirrel")
    .verifyComplete();
```
#### 리액티브 스트림의 변환과 필터링
##### skip()
: 지정된 수 만큼 건너뜀
```java
Flux<String> skipFlux = Flux.just("one", "two", "skip a few", "ninety nine", "one hundred")
    .skip(3);

StepVerifier.create(skipFlux)
    .expectNext("ninety nine", "one hundred")
    .verifyComplete(); 
```
: 일정 시간을 건너뜀
```java
Flux<String> skipFlux = Flux.just("one", "two", "skip a few", "ninety nine", "one hundred")
    .delayElements(Duration.ofSeconds(1))
    .skip(Duration.ofSeconds(4));

StepVerifier.create(skipFlux)
    .expectNext("ninety nine", "one hundred")
    .verifyComplete();
```
##### take()
```java
Flux<String> nationalParkFlux = Flux.just("Yellowtone", "Yosemite", "Grand Canyon", "Zion", "Grand Teton")
    .take(3);

StepVerifier.create(nationalParkFlux)
    .expectNext("Yellowtone", "Yosemite", "Grand Canyon")
    .verifyComplete();
```
```java
Flux<String> nationalParkFlux = Flux.just("Yellowtone", "Yosemite", "Grand Canyon", "Zion", "Grand Teton")
    .delayElements(Duration.ofSeconds(1))
    .take(Duration.ofMillis(3500));

StepVerifier.create(nationalParkFlux)
    .expectNext("Yellowtone", "Yosemite", "Grand Canyon")
    .verifyComplete();
```
##### filter()
```java
Flux<String> nationalParkFlux = Flux.just("Yellowstone", "Yosemite", "Grand Canyon", "Zion", "Grand Teton")
    .filter(np -> !np.contains(" "));

StepVerifier.create(nationalParkFlux)
    .expectNext("Yellowtone", "Yosemite", "Zion")
    .verifyComplete();
```
##### distinct()
: 중복 제거
```java
Flux<String> animalFlux = Flux.just("dog", "cat", "bird", "dog", "bird", "anteater")
    .distinct();

StepVerifier.create(animalFlux)
    .expectNext("dog", "cat", "bird", "anteater")
    .verifyComplete();
```
##### map()
```java
Flux<Player> playerFlux = Flux
    .just("Michael Jordan", "Scottie Pippen", "Steve Kerr")
    .map(n -> {
        String[] split = n.split("\\s");
        return new Player(split[0], split[1]);
    });

StepVerifier.create(playerFlux)
    .expectNext("Michael", "Jordan")
    .expectNext("Scottie", "Pippen")
    .expectNext("Steve", "Kerr")
    .verifyComplete();
```
##### flatMap()
```java
Flux<Player> playerFlux = Flux
    .just("Michael Jordan", "Scottie Pippen", "Steve Kerr")
    .fluxMap(n -> Mono.just(n)
        .map(n -> {
            String[] split = n.split("\\s");
            return new Player(split[0], split[1]);
        })
        .subscribeOn(Schedulers.parallel())
    );

List<Player> playerList = Arrays.asList(
    new Player("Michael", "Jordan"),
    new Player("Scottie", "Pippen"),
    new Player("Steve", "Kerr")
);

StepVerifier.create(playerFlux)
    .expectNextMatches(p -> playerList.contains(p))
    .expectNextMatches(p -> playerList.contains(p))
    .expectNextMatches(p -> playerList.contains(p))
    .verifyComplete();
```
- .parallel(): 고정풀에서 가져온 작업 스레드에서 구독을 실행
- .immediate(): 현재 스레드에서 구독 실행
- .single(): 단일 재사용 가능한 스레드에서 구독 실행.
- .newSingle(): 매 호출마다 전용 스레드에서 구독 실행
- .elastic(): 무한 신축성 스레드에서 구독 실행.

#### 리액티브 스트림의 데이터 버퍼링하기
##### buffer()
```java
Flux<String> fruitFlux = Flux.just("apple", "orange", "banana", "kiwi", "strawberry");

Flux<List<String>> bufferedFlux = fruitFlux.buffer(3);

StepVerifier.create(bufferedFlux)
    .expectNext(Arrays.asList("apple", "orange", "banana"))
    .expectNext(Arrays.asList("kiwi", "strawberry"))
    .verifyComplete();
```
```java
Flux.just("apple", "orange", "banana", "kiwi", "strawberry")
    .buffer(3)
    .flatMap(x -> 
        Flux.fromIterable(x)
            .map(y -> y.toUpperCase())
            .subscribeOn(Schedulers.parallel())
            .log()
        ).subscribe();
```
##### collectList()
```JAVA
Flux<String> fruitFlux = Flux.just("apple", "orange", "banana", "kiwi", "strawberry");

Mono<List<String>> fruitListMono = fruitFlux.collectList();

StepVerifier.create(fruitListMono)
    .expectNext(Arrays.asList("apple", "orange", "banana", "kiwi", "strawberry"))
    .verifyComplete();
```
##### collectMap()
```java
Flux<String> animalFlux = Flux.just("aardvark", "elephant", "koala", "eagle", "kangaroo");

Mono<Map<Character, String>> animalMapMono = animalFlux.collectMap(a -> a.charAt(0));

StepVerifier.create(animalMapMono)
    .expectNextMatches(map -> {
        return
            map.size == 3 &&
            map.get('a').equals("aardvark") &&
            map.get('e').equals("eagle") &&
            map.get('k').equals("kangaroo");
    })
    .verifyComplete();
```
#### 리액티브 타입에 로직 오퍼레이션 수행하기
##### all()
```java
Flux<String> animalFlux = Flux.just("aardvark", "elephant", "koala", "eagle", "kangaroo");
Mono<Boolean> hasAMono = animalFlux.all(a -> a.contains("a"));
StepVerifier.create(hasAMono)
    .expectNext(true)
    .verifyComplete();
```
##### any()
```java
Flux<String> animalFlux = Flux.just("aardvark", "elephant", "koala", "eagle", "kangaroo");
Mono<Boolean> hasZMono = animalFlux.any(a -> a.contains("z"));
StepVerifier.create(hasTMono)
    .expectNext(false)
    .verifyComplete();
```
