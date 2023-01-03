# 프록시 팩토리
 * JDK 동적 프록시는 인터페이스가 있을때, CGLIB는 구체 클래스가 있을때 사용할 수 있다. 그럼 상황 상황마다 바꿔가면서 써야하나? 불편하게?
   * 이러한 불편을 해결해주는 공통 로직이 있으면 좋겠다.
 * 스프링은 이러한 부분을 통합해서 편리하게 해주는 `프록시 팩토리` 라는 기능을 제공한다.
   * 인터페이스가 있으면 JDK 동적 프록시를 생성해주고, 구체 클래스만 있다면 CGLIB를 사용하며 이러한 설정도 커스텀하게 변경 가능하다.
 * 그럼 InvocationHander와 MethodInterceptor는 어떻게 해결할까? 이 두 개념을 추상화하여 `Advice` 라는 개념을 만들었다. 
 * 개발자는 `Advice`에 공통 로직을 만들고, 결과적으로 InvocationHander와 MethodInterceptor는 `Advice`를 호출하도록 스프링이 세팅한다.

![image](https://user-images.githubusercontent.com/48814463/204924136-a0822caf-50d7-4928-afca-1dd4e4a5cd12.png)

![image](https://user-images.githubusercontent.com/48814463/204924601-e03c3dcb-d8e1-4402-b12f-f94c615e1f87.png)


#### 특정 조건에 맞을 때 프록시 로직을 적용하는 기능도 공통으로 제공되었으면?
 * 스프링은 `Pointcut` 이라는 개념을 도입해서 이 문제를 일관성있게 해결한다.

### advice 정의 예제
 * MethodInterceptor의 패키지를 주의하자. `import org.aopalliance.intercept.MethodInterceptor;` 라이브러리를 써야한다.
 * 파라미터로 들어온 MethodInvocation invocation에서 메서드명 확인, 메서드 실행 모두 가능하다.
    * advice 실행시점에 핵심 로직이 들어있는 target 객체를 넣어준다. 

```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

### advice 사용 예제
 * ProxyFactory는
    * 인터페이스가 있으면 JDK 동적 프록시 사용
    * 구체 클래스만 있으면 CGLIB 사용
    * ProxyTargetClass 옵션을 사용하면 인터페이스가 있어도 CGLIB를 사용하고, 클래스 기반 프록시 사용
       * 참고로 스프링부트는 AOP를 적용할 때 기본적으로 ProxyTargetClass = true를 설정해서 

```java
@Test
@DisplayName("인터페이스가 있으면 JDK 동적 프록시 사용")
void interfaceProxy() {
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    proxyFactory.addAdvice(new TimeAdvice());
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());

    proxy.save();

    assertThat(AopUtils.isAopProxy(proxy)).isTrue();
    assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue();
    assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
}

@Test
@DisplayName("구체 클래스만 있으면 CGLIB 사용")
void concreteProxy() {
    ConcreteService target = new ConcreteService();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    proxyFactory.addAdvice(new TimeAdvice());
    ConcreteService proxy = (ConcreteService) proxyFactory.getProxy();
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());

    proxy.call();

    assertThat(AopUtils.isAopProxy(proxy)).isTrue();
    assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse();
    assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
}

@Test
@DisplayName("ProxyTargetClass 옵션을 사용하면 인터페이스가 있어도 CGLIB를 사용하고, 클래스 기반 프록시 사용")
void proxyTargetClass() {
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    proxyFactory.setProxyTargetClass(true);
    proxyFactory.addAdvice(new TimeAdvice());
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());

    proxy.save();

    assertThat(AopUtils.isAopProxy(proxy)).isTrue();
    assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse();
    assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
}
```

#### 남은 문제
1. 너무 많은 설정
   * 프록시 적용 코드가 스프링 빈마다 필요하다.
2. 컴포넌트 스캔
 * 실제 객체들은 이미 컴포넌트 스캔으로 스프링 컨테이너에 스프링 빈으로 모두 등록되어버렸기 때문에 프록시 적용이 불가능하다.

-> 이러한 문제는 다음에 배울 `빈후처리기` 에서 해결할 수 있다.
