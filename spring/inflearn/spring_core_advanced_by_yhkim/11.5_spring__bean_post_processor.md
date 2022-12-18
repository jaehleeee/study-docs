## 스프링이 제공하는 빈 후처리기
 * gradle `implementation 'org.springframework.boot:spring-boot-starter-aop'` 필수
 * 이는 자동 프록시 생성기(== 후처리기)를 활성화한다.
    * AnnotationAwareAspectJAutoProxyCreator 라는 후처리기가 스프링 빈에 자동으로 등록된다.
    * 이 빈 후처리기는 스프링 빈으로 등록된 Advisor 들을 자동으로 찾아서 프록시가 필요한 곳에 자동으로 프록시를 적용해준다.
    * Advisor에는 이미 포인트컷이 있어서 어떤 스프링빈에 프록시를 적용할지 알 수 있다. 여기서 advice라는 부가기능만 적용하면 된다.

#### 포인트컷의 2가지 기능
1. 프록시 적용 여부 판단
2. 프록시가 호출되었을때, 어드바이스 적용 여부 판단

#### 자동 프록시 생성기 작동 과정
 1. 스프링 빈으로 만들 대상 객체를 생성
 2. 빈 후처리기로 전달
 3. 모든 advisor 빈 조회
 4. 프록시 적용 대상 체크 : 앞서 조회한 advisor에 포함된 포인트컷을 이용하여 해당 객체가 프록시 적용 대상인지 클래스와 메서드 모두 보고 판단. 10개의 메서드 중 하나라도 포인트컷 조건에 만족하면 프록시 적용 대상이 된다.
 5. 프록시 생성 : 프록시 적용 대상이라면 프록시를 생성한다.
 6. 빈 등록 : 프록시 적용대상이라 프록시가 생성되었다면 프록시를 스프링 빈으로 등록하고, 대상이 아니면 기존 객체를 등록한다. 

![image](https://user-images.githubusercontent.com/48814463/208245072-9b6dc5eb-6883-46d4-80d3-662eeaf91c34.png)

## 스프링이 제공하는 빈 후처리기2 - `AspectJExpressionPointcut`
 * 포인트컷을 정밀하게 설정하지 않으면 불필요한 프록시가 많이 생성된다.
 * `AspectJExpressionPointcut` 는 AOP에 특화된 포인트컷 표현식. (실무에서 많이 사용됨.)
 * execution 내 문법
    * `*` : 모든 반환 타입
    * `hello.proxy.app..` : 해당 패키지와 그 하위 패키지
    * `*(..)`  : `*` 모든 메서드 이름, `(..)` 파라미터는 상관없음.

```java
 @Bean
 public Advisor advisor2(LogTrace logTrace) {
     //pointcut - *: 모든 반환타입 & hello.proxy.app 하위 클래스만 해당
     AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
     AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
     pointcut.setExpression("execution(* hello.proxy.app..*(..))");
     //advice
     LogTraceAdvice advice = new LogTraceAdvice(logTrace);
     return new DefaultPointcutAdvisor(pointcut, advice);
 }

 @Bean
 public Advisor advisor3(LogTrace logTrace) {
     //pointcut - noLog는 제외
     AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
     pointcut.setExpression("execution(* hello.proxy.app..*(..)) && !execution(* hello.proxy.app..noLog(..))");
     //advice
     LogTraceAdvice advice = new LogTraceAdvice(logTrace);
     return new DefaultPointcutAdvisor(pointcut, advice);
 }

```

## advisor가 여려개인 경우
 * 포인트컷 조건에 따라 프록시가 여러개 생성된다고 생각할수도 있지만,그렇지 않다.
 * 하나의 프록시에 여러 advisor를 적용할 수 있으므로, 포인트컷을 만족하는 advisor를 여러개 포함하는 프록시 1개가 생성된다.
