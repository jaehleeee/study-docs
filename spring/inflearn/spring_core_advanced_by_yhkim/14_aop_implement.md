# AOP 구현
 * 예제를 위해서 다음 의존성을 build.gradle에 추가. `implementation 'org.springframework.boot:spring-boot-starter-aop'`
 * 스프링 AOP에서는 AspectJ 문법을 차용하고 프록시 방식의 AOP를 제공한다. AspectJ를 직접 사용하는 것은 아니다.
 * `@Aspect` 는 컴포넌트 스캔 기능까지 가지고 있지 않다. AOP로 사용하려면 스프링 빈으로 꼭 등록해줘야 한다.

#### [참고] 스프링 빈 등록방법
1. `@Bean` 어노테이션으로 직접 등록
2. `@Component` 컴포넌트 스캔을 사용해서 자동 등록
3. `@Import`로 클래스를 임포트하면 빈으로 등록된다. (Import는 보통 설정파일 추가할 때 사용 된다.)


### AOP 구현 예제 코드1

```java
@Slf4j
@Aspect
public class Aspect1 {

    @Around("execution(* hello.aop.order..*(..))")
    public Object doLog(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        log.info("doLog {}", proceedingJoinPoint.getSignature());
        return proceedingJoinPoint.proceed();
    }
}

```


### AOP 구현 예제 코드2 - pointcut과 advice 분리
 * pointcut을 분리하면, 여러 advice에 붙여 쓸 수 있어 활용도가 높다.
 * pointcut 분리시 주의
    * 코드 내용은 비우고, 리턴 타입은 void 여야 한다. 
```java
@Slf4j
@Aspect
public class AspectV2 {

    //hello.aop.order 패키지와 하위 패키지
    @Pointcut("execution(* hello.aop.order..*(..))")
    private void allOrder(){} //pointcut signature

    @Around("allOrder()")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature()); //join point 시그니처
        return joinPoint.proceed();
    }

}
```
