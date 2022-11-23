# 동적 프록시

#### 지금까지의 개선과 문제
 * 지금까지는 기존 코드를 변경하지 않고, 로그 추적기라는 부가 기능을 적용하는 것으로 발전시켜 왔다.
 * 하지만 문제가 있다. 대상이 되는 클래스 수 만큼 로그 추적기를 위한 프록시 클래스를 만들어줘야 한다는 점이다.
 * 이를 해결하면, 동적 프록시 기술을 사용해야 한다.
 * 자바가 제공하는 JDK 동적 프록시 기술이나 CGLIB 같은 프록시 생성 오픈소스 기술을 활용하면 가능하다.
 * 그리고 동적 프록시 기술을 이해하기 위해선 자바의 리플렉션 기술을 이해해야 한다.

## 1. 리플렉션

#### 이해를 위한 예시
 * 공통 로직1,2를 보면, target.callA()와 target.callB()만 다르다.
 * target.callA()와 target.callB()만 뭔가 동적으로 처리할 수 있으면 좋을 것 같다.
```java
@Test
void reflection0() {
    Hello target = new Hello();

    //공통 로직1 시작
    log.info("start");
    String result1 = target.callA(); //호출하는 메서드가 다음
    log.info("result={}", result1);
    //공통 로직1 종료

    //공통 로직2 시작
    log.info("start");
    String result2 = target.callB(); //호출하는 메서드가 다음
    log.info("result={}", result2);
    //공통 로직2 종료
}
```

#### 리플렉션 사용1
 * 리플렉션은 클래스나 메서드의 메타정보를 사용해서 동적으로 호출하는 메서드를 변경할 수 있다.
 * 하지만 여전히 그냥 직접 함수를 호출하는 것과 편리함 측면에서 나아진 점은 없어보인다. 아래 예시에서 리플렉션을 좀 더 활용해보겠다.
```java
@Test
void reflection1() throws Exception {
    //클래스 정보
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    Hello target = new Hello();
    //callA 메서드 정보
    Method methodCallA = classHello.getMethod("callA");
    Object result1 = methodCallA.invoke(target);
    log.info("result1={}", result1);

    //callB 메서드 정보
    Method methodCallB = classHello.getMethod("callB");
    Object result2 = methodCallB.invoke(target);
    log.info("result2={}", result2);
}
```

#### 리플렉션 사용2
```java
@Test
void reflection2() throws Exception {
    //클래스 정보
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    Hello target = new Hello();
    Method methodCallA = classHello.getMethod("callA");
    dynamicCall(methodCallA, target);

    Method methodCallB = classHello.getMethod("callB");
    dynamicCall(methodCallB, target);
}

private void dynamicCall(Method method, Object target) throws Exception {
    log.info("start");
    Object result = method.invoke(target);
    log.info("result={}", result);
}
```
