# pointcut, advice, advisor
 * pointcut : 어디에(어떤 클래스 혹은 어떤 메서드) 부가 기능을 적용할지/적용하지 말지를 결정하는 필터링 로직
 * advice : 이전에 본 것처럼 프록시 호출하는 부가 기능
 * advisor : 하나의 포인트컷과 하나의 어드바이스를 가지고 있는 것, 조언자는 어디(pointcut)에 어떤 조언(advice)을 해야할지 알고 있다.


#### advisor 사용 예제

```java
@Test
@DisplayName("스프링이 제공하는 포인트컷")
void advisorTest3() {
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    
    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedNames("save");
    
    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, new TimeAdvice());
    proxyFactory.addAdvisor(advisor);
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    proxy.save();
    proxy.find();
}
```

#### 스프링 제공 포인트컷 종류
 * NameMatchMethodPointcut
 * JdkRegexpMethodPointcut : JDK 정규 표현식 기반 포인트컷 매칭
 * TruePointcut
 * AnnotationMatchingPointcut
 * AspecdtJExpressionPointcut : aspectJ 표현식으로 매칭 (가장 많이 사용됨.)

#### save 라는 이름의 메서드에만 적용하는 pointcut 만들어보기
 * (보통 스프링 제공해주는 pointcut을 사용하게 되기 때문에 직접 만들 일은 거의 없을 것임.)
 * 포인트컷은 클래스를 확인하는 ClassFilter와 메서드를 확인하는 MethodMatcher 2가지 요소로 이뤄져있다.

```java
static class MyPointcut implements Pointcut {

    @Override
    public ClassFilter getClassFilter() {
        return ClassFilter.TRUE;
    }

    @Override
    public MethodMatcher getMethodMatcher() {
        return new MyMethodMatcher();
    }
}

static class MyMethodMatcher implements MethodMatcher {

    private String matchName = "save";

    @Override
    public boolean matches(Method method, Class<?> targetClass) {
        boolean result = method.getName().equals(matchName);
        log.info("포인트컷 호출 method={} targetClass={}", method.getName(), targetClass);
        log.info("포인트컷 결과 result={}", result);
        return result;
    }

    // 이 값이 false이면 위에 args가 없는 matches 사용된다. (캐싱되므로 성능 향상), true 이면 아래 args가 있는 matches가 호출된다.
    @Override
    public boolean isRuntime() {
        return false;
    }

    @Override
    public boolean matches(Method method, Class<?> targetClass, Object... args) {
        return false;
    }
}

```


