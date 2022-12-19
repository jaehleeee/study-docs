# @Aspect 프록시
 * 기존 : 포인트컷과 어드바이스로 구성된 어드바이저를 만들어서 스프링 빈으로 등록하면, 앞서 배운 자동 프록시 생성기가 모두 자동으로 처리해준다.
    * 자동 프록시 생성기가 스프링 빈으로 등록된 어드바이저들을 찾고 해당되는 스프링 빈들에 자동으로 프록시를 적용해준다.
 * Aspect 애노테이션으로 매우 편리하게 포인트컷과 어드바이스로 구성되어 있는 어드바이저 생성 기능을 지원한다.

#### 사용 방법
1. advisor 만들 클래스에 `@Aspect` 어노테이션을 붙인다.
2. advisor로 만들 클래스 하위의 advisor로 사용할 메서드에 `@Around` 어노테이션에 `execution(* package명..)` 파라미터를 넣어 메서드에 붙인다.
   * `@Around` 어노테이션이 붙은 메서드가 advice가 된다.
   * 이 메서드의 파리미터로 ProceedingJoinPoint를 넣고, 메서드 로직에서 joinPoint.proceed() 를 통해 핵심 로직을 호출한다.
      * ProceedingJoinPoint에 실제 호출 대상, 인자, 객체, 메서드 등에 대한 정보가 포함되어 있다.
   * joinPoint.proceed() 전후로 공통 로직을 추가한다.
3. 이렇게 만든 advisor  클래스를 스프링 빈으로 등록해주기만 하면 된다.
   * 그럼 나머지는 자동 프록시 생성기가 알아서 처리해준다.

```java
@Slf4j
@Aspect
public class LogTraceAspect {

    private final LogTrace logTrace;

    public LogTraceAspect(LogTrace logTrace) {
        this.logTrace = logTrace;
    }

    @Around("execution(* hello.proxy.app..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        TraceStatus status = null;
        try {
            String message = joinPoint.getSignature().toShortString();
            status = logTrace.begin(message);

            //로직 호출
            Object result = joinPoint.proceed();

            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}

```
