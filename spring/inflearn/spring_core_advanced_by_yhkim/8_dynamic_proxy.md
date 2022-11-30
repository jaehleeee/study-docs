# 동적 할당

## 1. JDK 동적 프록시
 * 이름 그대로, 개발자가 직접 프록시를 만들 필요없이, 런타임에 동적으로 프록시를 만들어준다.
 * JDK 동적 프록시는 인터페이스가 필수다. (한계점)
 * JDK 동적 프록시에 적용할 로직은 `InvocationHandler` 인터페이스를 구현해서 직접 작성해야 한다.

#### 시간 측정하는 프록시 예제 코드
```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}

```
 * `Object target` : 동적 프록시가 호출할 대상
 * `method.invoke(target, args)` 리플렉션을 사용해서 `target`인스턴스의 메서드를 실행한다. 


### 동적 프록시 실행 클라이언트 예제 코드
```java
@Test
void dynamicA() {
    AInterface target = new AImpl(); 
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    // 메서드 파라미터 : 어떤 클래스 로더, 어떤 인터페이스 기반 프록시인지, 프록시가 수행할 로직
    AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);

    proxy.call();
    
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());
}
```


## 2. 실제 로그 추적기에 적용해보기

#### InvocationHander 구현
```java
public class LogTraceBasicHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;

    public LogTraceBasicHandler(Object target, LogTrace logTrace) {
        this.target = target;
        this.logTrace = logTrace;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        TraceStatus status = null;
        try {
            String message = method.getDeclaringClass().getSimpleName() + "." +
                    method.getName() + "()";
            status = logTrace.begin(message);

            //로직 호출
            Object result = method.invoke(target, args);
            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

#### 수동 빈 등록
```java
@Configuration
public class DynamicProxyBasicConfig {

    @Bean
    public OrderControllerV1 orderControllerV1(LogTrace logTrace) {
        OrderControllerV1 orderControllerV1 = new OrderControllerV1Impl(orderServiceV1(logTrace));
        OrderControllerV1 proxy = (OrderControllerV1) Proxy.newProxyInstance(OrderControllerV1.class.getClassLoader(),
                new Class[]{OrderControllerV1.class},
                new LogTraceBasicHandler(orderControllerV1, logTrace));
        return proxy;
    }


    @Bean
    public OrderServiceV1 orderServiceV1(LogTrace logTrace) {
        OrderServiceV1 orderServiceV1 = new OrderServiceV1Impl(orderRepositoryV1(logTrace));
        OrderServiceV1 proxy = (OrderServiceV1) Proxy.newProxyInstance(OrderServiceV1.class.getClassLoader(),
                new Class[]{OrderServiceV1.class},
                new LogTraceBasicHandler(orderServiceV1, logTrace));
        return proxy;
    }

    @Bean
    public OrderRepositoryV1 orderRepositoryV1(LogTrace logTrace) {
        OrderRepositoryV1 orderRepository = new OrderRepositoryV1Impl();

        OrderRepositoryV1 proxy = (OrderRepositoryV1) Proxy.newProxyInstance(OrderRepositoryV1.class.getClassLoader(),
                new Class[]{OrderRepositoryV1.class},
                new LogTraceBasicHandler(orderRepository, logTrace));
        return proxy;
    }
}

```


## 3. CGLIB
 * Code Generator Library
 * 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리
 * JDK 동적프록시의 한계는 인터페이스가 필수라는 점이다. 이 한계점을 극복해주는 것이 CGLIB 기술이다.
 * CGLIB는 인터페이스 구현 없이, 구체 클래스의 상속을 통해 프록시를 생성한다.
 * 단, 이후 배울 ProxyFactory가 이 기술을 편하게 사용하게 도와주기 때문에 , 실제로 CGLIB를 직접 사용할 경우는 거의 없다. 개념만 이해하자!

#### CGLIB 코드
 * JDK동적프록시에서 `InvocationHandler`를 제공했듯이, CGLIB에서는 `MethodInterceptor`를 제공함.
 * obj: CGLIB가 적용된 객체, methodProxy: 메서드 호출에 사용
```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}

```

#### CGLIB 사용하는 클라이언트 코드 예제
 * Enhancer : CGLIB는 이걸 이용해서 프록시를 생성한다.  
```java
@Slf4j
public class CglibTest {

    @Test
    void cglib() {
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();
        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());

        proxy.call();

    }
}

```
