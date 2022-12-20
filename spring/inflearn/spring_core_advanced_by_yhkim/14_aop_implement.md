# AOP 구현
 * 예제를 위해서 다음 의존성을 build.gradle에 추가. `implementation 'org.springframework.boot:spring-boot-starter-aop'`
 * 스프링 AOP에서는 AspectJ 문법을 차용하고 프록시 방식의 AOP를 제공한다. AspectJ를 직접 사용하는 것은 아니다.
 * `@Aspect` 는 컴포넌트 스캔 기능까지 가지고 있지 않다. AOP로 사용하려면 스프링 빈으로 꼭 등록해줘야 한다.

