#### [GO TO BACK](../README.md)

# Chapter19. 스프링 배포하기 
> 스프링 애플리케이션을 WAR나 JAR 파일로 빌드하기  
> 스프링 애플리케이션을 클라우드 파운드리에 푸시하기  
> 스프링 애플리케이션을 도커 컨테이너에 패키징 하기  

### 19.1 배포 옵션
- IDE에서 애플리케이션을 빌드하고 실행한다.
- 메이븐 springboot:run, 그래들: bootRun 명령으로 실행 빌드
- 메이블, 그래들을 사용해서 실행 가능한 jar 파일을 생성
- 메이븐, 그래들을 사용해서 war 파일 생성

#### 플랫폼 별 배포
- 자바 애플리케이션 서버에 배포하기: Tomcat, WebSphere, WebLogic 자바 애플리케이션 서버면 WAR
- 클라우드에 배포하기: Cloud Foundry, Amazon Web Service, Azure, Google Cloud Platform면 JAR

### 19.2 WAR 파일 빌드하고 배포하기
#### 자바로 스프링 웹 애플리케이션 활성화하기
```java
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;public class IngredientServiceServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(IngredientServiceApplication.class);
    }   
}
```
```xml
#pom.xml
<packaging>war</packaging>
```
```json
#gradle
apply plugin: "war"
```

### 19.3 클라우드 파운드리에 JAR 파일 푸시하기
- http://run.pivotal.io 
- PWS(pivotal web services)
```shell script
$ cf login -a https://api.run.pivotal.io
API endpoint: https://api.run.pivotal.io

Email> {이메일}

Password> {비밀번호}

Authenticating...
OK
```
```shell script
$ cf push ingredient-service -p target/ingredient-service-0.0.19-SNAPSHOT.jar
```
```shell script
$ cf push ingredient-service -p target/ingredient-service-0.0.19-SNAPSHOT.jar --random-route 
```
```shell script
$ cf create-service mlab sandbox ingredientdb
$ cf bind-service ingredient-service ingredientdb
$ cf restage ingredient-service
```

### 19.4 도커 컨테이너에서 스프링 부트 실행하기
```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.3</version>
            <configuration>
                <repository>
                    ${docker.image.prefix}/${project.artifactId}
                </repository>
                <buildArgs>
                    <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>
</build>

<properties>
    <docker.image.prefix>tacocloud</docker.image.prefix>
</properties>
```
```
# Dockerfile
FROM openjdk:8-jdk-alpine
ENV SPRING_PROFILES_ACTIVE docker
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", \
            "-Djava.security.egd=file:/dev/./urandom",\
            "-jar", \
            "/app.jar"]
```
- FROM 에는 base 이미지 지정
- ENV 환경 변수 설정
- VOLUMN 컨테이너의 마운트 지점
- ARG 메이븐 <buildArgs> 동일한 이름
- COPY app.jar 도커 파일 복사
- ENTRYPOINT 컨테이너가 시작 될 때 실행 하기 위한 명령행 코드를 배열로 지정

```yaml
spring:
  profiles: docker
  data:
    mongodb:
      host: mongo
```

```shell script
$ mvnw package dockerfile:build
```

- AWS: https://aws.amazon.com/ko/getting-started/hands-on/deploy-docker-containers/
- Azure: https://docs.docker.com/cloud/aci-integration/
- Google: https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app
- PWS: https://docs.run.pivotal.io/devguide/deploy-apps/push-docker.html
- PKS: https://tanzu.vmware.com/kubernetes-grid
