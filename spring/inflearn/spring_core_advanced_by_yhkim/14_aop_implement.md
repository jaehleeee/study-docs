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

### advice 순서 적용
 * 어드바이스는 기본적으로 순서를 보장하지 않는다.
 * `@Order` 어노테이션을 이용해서 순서를 정렬해야하는데, 이 어노테이션은 클래스 단위로 설정된다. 즉 한 클래스 내에 있는 여러 메서드의 순서는 이 어노테이션으로 설정되지 않는다.
 * 그럼 하나의 클래스에 있는 여러 메서드의 순서는?
    * advice를 메서드가 아니라 클래스 내부에 있는 static 내부 클래스로 만들면 된다. 그럼 `@Order`를 적용할 수 있다.


## advice의 종류
1. `@Around` : 메서드 호출 전후에 수행. 아래 4가지를 모두 넣을 수 있음.  핵심로직 실행 코드도 함께 넣어줘야한다.
2. `@Before` : 조인포인트 실행 이전에 실행됨. 핵심로직 실행은 자동으로 실행된다.
3. `@After Returning` : 메서드가 정상 완료 후 실행됨. 핵심로직 실행은 자동으로 실행된다.
4. `@After Throwing` : 메서드 예외를 던지는 경우 실행됨. 핵심로직 실행은 자동으로 실행된다.
5. `@After` : 조인 포인트가 정상 또는 예외에 관계없이 실행. (finally) 핵심로직 실행은 자동으로 실행된다.

#### 참고
 * `@Around` 는 첫번째 파라미터에 ProceedingJointPoint 파라미터가 필수 (핵심기능을 호출해야 하기 때문)
 * 나머지 advice들은 첫번째 파라미터에 JointPoint 파라미터를 넣을 수 있는데, 필수는 아님.

<img width="830" alt="image" src="https://user-images.githubusercontent.com/48814463/209092595-0cdb7045-7df4-4cc0-9a69-316e45e080dd.png">

