#### [GO TO BACK](../README.md)

# Chapter2. 웹 애플리케이션 개발

> 모델 데이터를 브라우저에서 보여주기  
> 폼 입력 처리하고 검사하기  
> 뷰 템플릿 라이브러리 선택하기  

### 1. 정보 보여주기
- 도메인 클래스 (속성을 정의)
- 스프링 MVC 컨트롤러 클래스 (정보를 가져와서 뷰에 전달)
- 뷰 템플릿 (사용자의 브라우저)

#### 1.1 도메인 설정
- 도메인 클래스
```java
package tacos;

import lombok.Data;
import lombok.RequiredArgsConstructor;

@Data
@RequiredArgsConstructor
public class Ingredient {
    private final String id;
    private final String name;
    private final Type type;
   
    public static enum type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

### 1.2 컨트롤러 클래스 생성
```java
package tacos.web;
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {

    @GetMapping
    public String showDesignForm(Model model) {
        List<Ingredient> ingredients = Arrays.asList(
            new Ingredient("FLTO", "Flour Tortilla", Type.WRAP),
            new Ingredient("COTO", "Corn Tortilla", Type.WRAP)
        );

        Type[] types = Ingredient.Type.values();
        for(Type type : types) {
        	model.addAttributes(type.toString().toLowerCase(),
                filterByType(ingredients, type));
        }

        model.addAttribute("taco", new Taco());
        return "design";
    }

    private List<Ingredient> filterByType(List<Ingredient> ingredients, Type type) {
        return ingredients
                    .stream()
                    .filter(x -> x.getType().equals(type))
                    .collect(Collectors.toList());
    }
}
```
- @Controller
- @RequestMapping

### 1.3 뷰 디자인하기
##### 뷰 정의
- JSP
- Thymeleaf
- FreeMarker
- Mustache
- Groovy
```thymeleaftemplatesfragmentexpressions
<p th:text="${message}">placeholder message</p>
```

### 2. 폼 제출 처리
```java
@PostMapping
public String processDesign(TTaco design) {
    log.info("Processing" + design);

    return "redirect:/orders/current";
}
```

### 3. 폼 입력 유효성 검사하기
- 스프링 자바빈 유효성 검사
https://jcp.org/en/jsr/detail?id=303
##### 유효성 검사 규칙 추가
```java
@Data
public class Taco {
    @NotNull
    @Size(min=5, message="Name must be at least 5 characters long")
    private String name;

    @Size(min=1, message="You must choose at least 1 ingredient")
    private List<String> ingredients;
}
```
```java
@Data
public class Order {
    @NotBlank(message="Name is required")
    private String deliveryName;

    @NotBlank(message="Street is required")
    private String deliveryStreet;

    @CreditCardNumber(message="Not a valid credit card number")
    private String ccNumber;

    @Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$",
            message="Must be formatted MM/YY")
    private String ccExpiration;

    @Digits(integer=3, fraction=0, message="Invalid CVV")
    private String ccCVV;
}
```

#### 3.2 폼과 바인딩될 때 유효성 검사 수행
```java
@PostMapping
public String proccessDesign(@Valid Taco design, Errors errors) {
    if(errors.hasErrors()) {
        return "design";
    }

    log.info("Processing design: " + design);
    return "redirect:/orders/current";
}
```

#### 3.3 유효성 에러 출력
```html
<label for="ccNumber">Credit Card #: </label>
<input type="text" th:field="*{ccNumber}" />
<span class="validationErrors"
    th:if="${#fields.hasErrors('ccNumber')}"
    th:errors="*{ccNumber}">CC Num Error</span>
```

### 4. 뷰 컨트롤러로 작업

###### 뷰 컨트롤러 선언 
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```
- ViewController는 HomeController를 대체 할 수 있다.

### 5. 뷰 템플릿 라이브러리 선택
|템플릿|스프링 부트 스타터 의존성|
|-----|-----------------|
|FreeMarker|spring-boot-starter-freemarker|
|Groovy 템플릿|spring-boot-starter-groovy-templates|
|JavaServer Pages(JSP)|없음(Tomcat, Jetty 서블릿컨테이너에서 자체 제공|
|Mustache|spring-boot-starter-mustache|
|Thymeleaf|spring-boot-starter-thymeleaf|

#### 5.1 템플릿 캐싱 
|템플릿|캐싱 속성|
|----|-------|
|FreeMarker|spring.freemarker.cache|
|Groovy Templates|spring.groovy.template.cache|
|Mustache|spring.mustache.cache|
|Thymeleaf|spring.thymeleaf.cache|
```properties
spring.thymeleaf.cache=false
```

