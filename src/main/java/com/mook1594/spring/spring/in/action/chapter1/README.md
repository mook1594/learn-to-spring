#### [GO TO BACK](../README.md)

# Chapter1. 스프링 기초

## Part1. 스프링과 스프링 부터 핵심 사항
### 1. 스프링이란?
#### 어플리케이션 구성
- 많은 컴포넌트로 구성
- 컴포넌트의 생성과 상호간 인지 관리 필요

#### 스프링
- 스프링 애플리케이션 컨텍스트라는 컨테이너 제공 (String application context): 컴포넌트들을 생성하고 관리
- 컴포넌트 or 빈(Bean)들은 컨텍스트 내부에서 서로 연결되어 어플리케이션을 구성.

#### Bean의 연결
- 의존성 주입(Dependency Injection, DI) 패턴 기반으로 수행
- 스프링 별도의 컨테이너가 빈의 생성, 관리를 한다. 이 개체에서 모든 컴포넌트를 생성 관리하고 필요한 곳에 주입한다.
  
#### 스프링 컨테이너
- 핵심 컨테이너 외에도 관련 라이브러리를 포함하면 웹, 보안 프레임워크, 런타임 모니터링, 마이크로서비스, 리액티브 프로그래밍 모델등 지원

#### 스프링 컴포넌트 관계
- 기존엔 XML로 관리했으나 요즘엔 코드로 관리하는 추세
```java
@Configuration
public class ServiceConfiguration {
    @Bean
    public InventoryService inventoryService() {
        return new InventoryService();
    }

    @Bean
    public ProductService productService() {
        return new ProductService(inventoryService());
    }
}
```
- @Configuration 어노테이션: 컨텍스트에 제공하는 구성 클래스라는 걸 표시
- @Bean 어노테이션: 컨텍스트의 빈으로 추가 되어야 함을 표시 (메서드명이 ID로 정의됨) 
- 자바 기반 구성은 더 강화된 타입 안전과 향상된 리펙토링 기능을 포함.

#### 컴포넌트 자동 구성
- autowiring & component scanning: 컴포넌트를 자동으로 구성할 수 있는 스프링 기법
- Spring Boot: 스프링 프레임워크를 확장하여 생선성에 향상. 환경변수(classpath)를 기준으로 컴포넌트 구성 및 연결

### 2. 스프링 애플리케이션 초기 설정
#### Spring Initializr
- https://start.spring.io 에서 생성
- 명령행 curl, spring boot cli 사용
- String Tool Suite(STS) IDE에서 새 프로젝트 생성  
(STS는 이클립스 기반 IDE, String Boot Dashboard 기능 제공)
- Intellij IDEA IDE에서 새 프로젝트 생성
- NetBeans IDE에서 새 프로젝트 생성  

 
### 3. 스프링 애플리케이션
#### 스프링 부트 DevTools 
- 코드가 변경될때 자동 재시작
- 브라우저 리소스 변경시 자동 새로고침
- 템플릿 캐시 자동 비활성화
- H2 사용중이면 콘솔 활성

### 4. 스프링 살펴보기
#### 스프링 프레임워크
- 스프링 MVC
- JDBC 지원
- 리액티브 프로그래밍(Webflux)

#### 스프링 부트
- Actuator: 런타임시 살펴볼 수 있는 기능 (metric, thread, status)
- CLI 제공

#### 스프링 데이터
- JPA
- Mongo
- Neo4j

#### 스프링 시큐리티
- 보안 프레임워크
- authentication (인증)
- authorization (허가)

#### 스프링 통합과 배치
- 컴포넌트 통합, 배치 기반
- 트리거 적용

#### 스프링 클라우드
- 마이크로서비스 구조 가능
- 스프링 마이크로서비스 코딩 공작소 (Spring Microservices in Action)

