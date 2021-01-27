#### [GO TO BACK](../README.md)

# Chapter7. REST 서비스 사용하기
> RestTemplate을 사용해서 REST API 사용하기  
> Traverson을 사용해서 하이퍼미디어 API 이동하기  

### 7.1 RestTemplate으로 REST 엔드포인트 사용하기
- delete: HTTP DELETE 요청
- exchange: ResponseEntity 반환
- execute: 객체 반환
- getForEntity: HTTP GET 요청, ResponseEntity 반환
- getForObject: HTTP GET 요청, 객체 반환
- headForHeaders: HTTP HEAD 요청, 헤더 반환
- optionsForAllow: HTTP OPTIONS 요청, Allow 헤더 반환
- patchForObject: HTTP PATCH 요청, 객체 반환
- postForEntity: HTTP POST, ResponseEntity 반환
- postForLocation: HTTP POST, 리소스 Url 반환
- postForObject: HTTP POST, 객체 반환
- put: HTTP PUT

```java
RestTemplate rest = new RestTemplate();
```
```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```
#### 리소스 가져오기 (GET)
```java
public Ingredient getIngredientById(String infredientId) {
    return rest.getForObject("http://localhost:8080/ingredients/{id}",
            Ingredient.class, ingredientId);
}
```
```java
public Ingredient getIngredientById(String ingredientId) {
    Map<String, String> urlVariables = new HashMap<>();
}
```
```java
public Ingredient getIngredientById(String ingredienId) {
    Map<String, String> urlVariables = new HashMap<>();
    urlVariable.put("id", ingredientId);
    URI url = UriComponentsBuilder
        .fromHttpUrl("http//localhost:8080/ingredients/{id}")
        .build(urlVariables);
    return rest.getForObject(url, Ingredient.class);
}
```
```java
public Ingredient getIngredientById(String ingredientId) {
    ResponseEntity<Ingredient> responseEntity = 
        rest.getForEntity("http://localhost:8080/ingredients/{id}",
            Ingredient.class, ingredientId);

    reurn responseEntity.getBody();
}
```
#### 리소스 쓰기 (PUT)
```java
public void updateIngredient(Ingredient ingredient) {
    rest.put("http://localhost:8080/ingredients/{id}",
            ingredient, ingredient.getId());
}
```
#### 리소스 삭제하기(DELETE)
```java
public void deleteIngredient(Ingredient ingredient) {
    rest.delete("http://localhost:8080/ingredients/{id}",
            ingredient.getId());
}
```
#### 리소스 데이터 추가하기(POST)
```java
public Ingredient createIngredient(Ingredient ingredient) {
    return rest.postForObject("http://localhost:8080/ingredients",
                ingredient, Ingredient.class);
}
```
```java
public URI createIngredient(Ingredient ingredient) {
    return rest.postForLocation("http://localhost:8080/ingredients", ingredient);
}
```
```java
public Ingredient createIngredient(Ingredient ingredient) {
    ResponseEntity<Ingredient> responseEntity = rest.postForEntity("http://localhost:8080/ingredients",
        ingredient, Ingredient.class);
    return responseEntity.getBody();
}
```

### 7.2 Traverson으로 REST API 사용하기
```java
Traverson traverson = new Traverson(
    URI.create("http://localhost:8080/api"), MediaTypes.HAL_JSON);
```
```java
ParameterizedTypeReference<Resources<Ingredient>> ingredientType = 
    new ParameterizedTypeReference<Resources<Ingredient>>() {};

Resource<Ingredient> ingredientRes = 
    tracerson
        .follow("ingredients")
        .toObject(ingredientType);

Collection<Ingredient> ingredients = ingredentRes.getContent();
```
