#### [GO TO BACK](../README.md)

# 12. 스프링 데이터 JPA

##### 스프링 데이터 JPA의 QueryDSL 지원
- org.springframework.data.querydsl.QueryDslPredicateExecutor
- org.springframework.data.querydsl.QueryDslRepositorySupport

#### QueryDslPredicateExecutor 사용
```java
public interface ItemRepository 
    extends JpaRepository<Item, Long>, QueryDslPredicateExecuor<Item> {
}
```
#### QueryDslRepositorySupport 사용
```java
public interface CustomOrderRepository {
    public List<Order> search(OrderSearch orderSearch);
}
public class OrderRepositoryImpl extends QueryDslRepositorySupport implements CustomOrderRepository {
    public OrderRepositoryImpl() {
        super(Order.class);
    }

    @Override
    public List<Order> search(OrderSearch orderSearch) {
        QOrder order = QOrder.order;
        QMember member = QMember.member;
        
        JPQLQuery query = from(order);
        
        if(StringUtils.hasText(orderSearch.getMemberName())) {
            query.leftJoin(order.member, member)
                .where(member.name.contains(orderSearch.getMemberName()));
        }
        if(orderSearch.getOrderStatus() != null) {
            query.where(order.status.eq(orderSearch.getOrderStatus()));
        }
        return query.list(order);
    }
}
```
