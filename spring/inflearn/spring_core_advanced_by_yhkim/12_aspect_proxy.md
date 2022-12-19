# @Aspect 프록시
 * 기존 : 포인트컷과 어드바이스로 구성된 어드바이저를 만들어서 스프링 빈으로 등록하면, 앞서 배운 자동 프록시 생성기가 모두 자동으로 처리해준다.
    * 자동 프록시 생성기가 스프링 빈으로 등록된 어드바이저들을 찾고 해당되는 스프링 빈들에 자동으로 프록시를 적용해준다.
 * Aspect 애노테이션으로 매우 편리하게 포인트컷과 어드바이스로 구성되어 있는 어드바이저 생성 기능을 지원한다.
 * 이러한 기능을 사용할 수 있게 해주는 것도 자동 프록시 생성기인 (AnnotationAwareAspectJAutoProxyCreator) 의 역할 중 하나다. (이름에서 알 수 있드시)
    * `@Aspect` 를 찾아서 Advisor로 만들어주는 역할을 한다.

#### 사용 방법
1. advisor 만들 클래스에 `@Aspect` 어노테이션을 붙인다.
2. advisor로 만들 클래스 하위의 advisor로 사용할 메서드에 `@Around` 어노테이션에 `execution(* package명..)` 파라미터를 넣어 메서드에 붙인다.
   * `@Around` 어노테이션이 붙은 메서드가 advice가 된다.
   * `execution(* package명..)` 파라미터가 포인트컷이 된다.
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

### `@Aspect`를 advisor로 변환하는 과정
1. 스프링 애플리케이션 로딩 시점에 자동 프록시 생성기 호출
2. 스프링 컨테이너에서 모든 `@Aspect` 빈 조회
3. 어드바이저 생성 : `@Aspect` 어드바이저 빌더를 통해 `@Aspect` 애노테이션 정보를 기반으로 어드바이저를 생성.
4.  `@Aspect` 기반 어드바이저 저장 : 생성한 어드바이저를 `@Aspect` 어드바이저 빌더 내부에 저장.

####  `@Aspect` 어드바이저 빌더
 * BeanFactoryAspectJAdvisorsBuilder 클래스.
 * `@Aspect` 의 정보를 기반으로 포인트컷, 어드바이스, 어드바이저를 생성하고 보관하는 것을 담당.
 * 내부 저장소에 캐시가 있고, 이미 만들어져있는 어드바이저를 캐시해서 사용함.


### 어드바이저를 기반으로 프록시를 생성하는 총 과정

![image](https://user-images.githubusercontent.com/48814463/208339950-98b7ad2b-56ad-482a-a52b-2b6389e66fb3.png)

