#### [GO TO BACK](../README.md)

# Chapter18. JMX로 스프링 모니터링하기 
> 액추에이터 엔드포인트 MBeans 사용하기  
> 스프링 빈을 MBeans로 노출하기  
> 알림 발행(전송)하기  

### 18.1 액추에이터 MBeans 사용하기
```yaml
management:
  endpoints:
    jmx:
      exposure:
        include: health,info,bean,conditions
        exclude: env,metrics
```

### 18.2 우리의 MBeans 생성하기
```java
@Service
@ManagedResource
public class TacoCounter extends AbstractRepositoryEventListener<Taco> {

    private AtomicLong counter;

    public TacoCounter(TacoRepository tacoRepo) {
        long initialCount = tacoRepo.count();

        this.counter = new AtomicLong(initialCount);
    }

    @Override
    protected void onAfterCreate(Taco entity) {
        counter.incrementAndGet();
    }

    @ManagedAttribute
    public long getTacoCount() {
        return counter.get();    
    }

    @ManagedOperation
    public long increment(long delta) {
        return counter.addAndGet(delta);
    }
}   
```

### 18.3 알림 전송하기
```java
@Service
@ManagedResource
public class TacoCounter extends AbstractRepositoryEventListener<Taco> implements NotificationPublisherAware {
    private AtomicLong counter;
    private NotificationPublisher np;
    ...

    @Override
    public void sendNotificationPublisher(NotificationPublisher np) {
        this.np = np;
    }
    ...

    @ManagedOperation
    public long increment(long delta) {
        long before = counter.get();
        long after = counter.addAndGet(delta);
        if((after / 100) > (before / 100)) {
            Notification notification = new Notification (
                "taco.count", this, before, after + "th taco created!");
            np.sendNotification(notification);
        }
        return after;
    }
}
```

### 18.4 TacoCounter MBeans 빌드 및 사용하기
