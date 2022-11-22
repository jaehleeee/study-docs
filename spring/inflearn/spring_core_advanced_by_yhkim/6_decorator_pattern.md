# 데코레이터 패턴
 * 형태는 프록시이지만, 새로운 부가 기능 추가가 목적

![image](https://user-images.githubusercontent.com/48814463/203172528-30242376-e410-4664-8c91-40824c19039c.png)

 * 기존 핵심 기능 역할을 하는, RealComponent가 있고
 * RealComponent 의존성을 가진 MessageDecorator가 있다.
 * MessageDecorator에서 부가기능을 추가하고, RealComponent의 핵심 기능을 실행해줌으로서 프록시 역할을 하게 된다.
 * 프록시 패턴과 마찬가지로, 클라이언트 입장에서는 프록시 여부를 알 수 없고 관심도 없다.

```java
public class MessageDecorator implements Component {

    private Component component;

    public MessageDecorator(Component component) {
        this.component = component;
    }

    @Override
    public String operation() {
        log.info("MessageDecorator 실행");

        //data -> *****data*****
        String result = component.operation();
        String decoResult = "*****" + result + "*****";
        log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);
        return decoResult;
    }
}
```
 * 이러한 체인 구조를 이용하면, 계속해서 부가기능을 추가할 수 있다.
    * client -> TimeDecorator -> MessageDecorator -> RealComponent

#### 클라이언트 사용 구조
```java
@Test
void decorator2() {
    Component realComponent = new RealComponent();
    Component messageDecorator = new MessageDecorator(realComponent);
    Component timeDecorator = new TimeDecorator(messageDecorator);
    DecoratorPatternClient client = new DecoratorPatternClient(timeDecorator);
    client.execute();
}
```

### 핵심 로직이 들어있는 코드는 수정이 없지만, 클라이언트가 사용할때 일일이 프록시들을 조립해줘야 하는가..?
-> 이는 configuration bean 설정에서 미리 조립해둘 수 있다.

#### logTrace 기능을 Controller, Service, Repository 프록시에 추가하는 케이스에 대한 bean 조립 예시
 * xxxxxxInterfaceProxy 클래스에서 logTrace 부가 기능이 포함되어 있고, 핵심 기능이 있는 클래스(Controller, Service, Repository) 의존성을 가지고 있어서 핵심 기능도 실행시켜준다.
 * 프록시를 실제 객체 대신 스프링 빈으로 등록한다. (프록시가 실제 객체를 참조하고 있기 때문에 실제 객체가 없어지는 것은 아니다. 프록시에 의해 참조될 뿐이다.)
```java
@Configuration
public class InterfaceProxyConfig {

    @Bean
    public OrderControllerV1 orderController(LogTrace logTrace) {
        OrderControllerV1Impl controllerImpl = new OrderControllerV1Impl(orderService(logTrace));
        return new OrderControllerInterfaceProxy(controllerImpl, logTrace);
    }

    @Bean
    public OrderServiceV1 orderService(LogTrace logTrace) {
        OrderServiceV1Impl serviceImpl = new OrderServiceV1Impl(orderRepository(logTrace));
        return new OrderServiceInterfaceProxy(serviceImpl, logTrace);
    }

    @Bean
    public OrderRepositoryV1 orderRepository(LogTrace logTrace) {
        OrderRepositoryV1Impl repositoryImpl = new OrderRepositoryV1Impl();
        return new OrderRepositoryInterfaceProxy(repositoryImpl, logTrace);
    }

}
```

#### 구체클래스 기반의 proxy 조립 예시
```java
@Configuration
public class ConcreteProxyConfig {

    @Bean
    public OrderControllerV2 orderControllerV2(LogTrace logTrace) {
        OrderControllerV2 controllerImpl = new OrderControllerV2(orderServiceV2(logTrace));
        return new OrderControllerConcreteProxy(controllerImpl, logTrace);
    }

    @Bean
    public OrderServiceV2 orderServiceV2(LogTrace logTrace) {
        OrderServiceV2 serviceImpl = new OrderServiceV2(orderRepositoryV2(logTrace));
        return new OrderServiceConcreteProxy(serviceImpl, logTrace);
    }

    @Bean
    public OrderRepositoryV2 orderRepositoryV2(LogTrace logTrace) {
        OrderRepositoryV2 repositoryImpl = new OrderRepositoryV2();
        return new OrderRepositoryConcreteProxy(repositoryImpl, logTrace);
    }
}
```


### 인터페이스 기반 프록시와 클래스 기반 프록시에 대하여
 * 클래스 기반 프록시는 해당 프록시만 적용할 수 있지만, 인터페이스 기반은 인터페이스만 같으면 모든 곳에 적용 가능하다.
 * 클래스 기반 프록시는 상속을 사용해야하기 때문에 몇가지 제약이 존재
    * 부모 클래스의 생성자를 호출해야 한다.
    * 클래스 final 키워드가 붙으면 상속 불가
    * 메서드 final 키워드가 붙으면 오버라이딩 불가
 * 인터페이스 기반 프록시의 단점은 인터페이스가 필요하다는 점 정도다.
