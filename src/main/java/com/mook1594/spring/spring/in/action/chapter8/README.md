#### [GO TO BACK](../README.md)

# Chapter8. 비동기 메시지 전송하기
> 비동기 메시지 전송  
> JMS, RabbitMQ, 카프카(Kafka)를 사용해서 전송하기  
> 브로커에서 메시지 가져오기  
> 메시지 리스닝하기  

#### 비동기 메시징
- 간접적으로 메시지를 전송하는 방법
- 통신하는 애플리케이션 간의 결합도를 낮추고 확장성을 높여준다.

### 8.1 JMS로 메시지 전송하기
#### JMS 설정하기
```xml
<dependency>
    <groupid>org.springframework.boot</groupid>
    <artiifactId>spring-boot-starter-activemq</artiifactId>
</dependency>
```
```yaml
spring:
    activemq:
      broker-url: tcp://activemq.tacocloud.com
      user: tacoweb
      password: l3tm3ln
```
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
```
- Artemis는 ActiveMQ를 새롭게 다시 구현한 차세대 브로커.
- 스프링은 Artemis 브로커가 localhost:61616 포트 리스닝
#### Artemis 브로커의 위치와 인증 정보를 구성하는 속성
- spring.artemis.host: 브로커의 호스트
- spring.artemis.port: 브로커의 포트
- spring.artemis.user: 브로커를 사용하기 위한 사용자(선택 속성)
- spring.artemis.password: 브로커를 사용하기 위한 사용자 암호(선택)
```yaml
spring:
    artemis:
      host: artemis.tacocloud.com
      port: 61617
      user: taccoweb
      password: l3tmd3ln
```
#### JmsTemplate을 사용해서 메시지 전송하기
```java
// 원시 메시지 전송
void send(MessageCreator messageCreator) throws JmsException;
void send(Destination destination, MessageCreator messageCreator) throws JmsException;
void send(String destinationName, MessageCreator messageCreator) throws JmsException;

// 객체로 부터 변환된 메시지를 전송한다
void convertAndSend(Object message) throws JmsException;
void convertAndSend(Destination destination, Object message) throws JmsException;
void convertAndSend(String destinationName, Object message) throws JmsException;

// 객체로부터 변환되고 전송에 앞서 후처리되는 메시지를 전송
void convertAndSend(Object message, MessagePostProcessor postProcessor) throws JmsException;
void convertAndSend(Destination destination, Object message, MessagePostProcessor postProcessor) throws JmsException;
void convertAndSend(String destinationName, Object message, MessagePostProcessor postProcessor) throws JmsException;
```
##### send()를 사용해서 주문 데이터 전송하기
```java
@Service
public class JmsOrderMessagingService implements OrderMessagingService {
    private JmsTemplate jms;

    @Autowired
    public JmsOrderMessagingService(JmsTemplate jms) {
        this.jms = jms;
    }

    @Override
    public void sendOrder(Order order) {
        jms.send(new MessageCreator()) {
            @Override
            public Message createMessage(Session session) throws JMSException {
                return session.createObjectMessage(order);
            }
        }
    }
}
```
```java
@Override
public void sendOrder(Order order) {
    jms.send(session -> session.createObjectMessage(order));
}
```
- 도착지 설정
```yaml
spring:
    jms: 
      template:
        default-destination: tacocloud.order.queue
```
```java
@Bean
public Destination orderQueue() {
    return new ActiveMQQueue("tacocloud.order.queue");
}
```
```java
private Destination orderQueue;

@Autowired
public JmsOrderMessagingService(JmsTemplate jms, Destination orderQueue) {
    this.jms = jms;
    this.orderQueue = orderQueue;
}
...
@Override
public void sendOrder(Order order) {
    // jms.send(orderQueue, session -> session.createObjectMessage(order));
    jms.send("tacocloud.order.queue", session.createObjectMessage(order));
}
```
