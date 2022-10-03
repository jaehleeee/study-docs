# 빈 스코프

### 스프링은 다양한 스코프를 지원한다.
 * 싱글톤 : 기본 스코프로, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
 * 프로토타입 : 스프링 컨테이너가 빈의 생성과 의존주입까지만 관여하고, 더는 관리하지 않는 매우 짧은 범위의 스코프
 * 웹 관련 스코프
    * `request` : 웹 요청이 들어오고 나갈때까지 유지
    * `session` : 웹 세션이 생성되고 종료될때까지 유지
    * `application` : 웹의 서블릿 컨텍스트와 같은 범위 

### 스코프 지정 : @Scope 애노테이션
``` java
@Scope("prototype")
@Component
public class HelloBean {}
```

``` java
@Scope("prototype")
@Bean
PrototypeBean HelloBean {
  return new HelloBean();
}
```
