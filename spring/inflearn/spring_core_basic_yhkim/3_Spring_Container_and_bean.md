```
@Configuration
public class AppConfig {

   @Bean
   public MemberService memberService() {
      return new MemberServiceImpl(memberRepository());
   }
   
   @Bean
   public MemberRepository memberRepository() {
      return new MemoryMemberRepository();
   }

   ...

}
```

```
pulbic class MemberApplication {

   public static void main(String[] args) {
      ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
      MemberService merberService = ac.getBean("memberService", MemberService.class);
   
   }

}
```

## ApplicationContext : 스프링 컨테이너
 * 기존에는 AppConfig 를 생성해서 직접 DI를 수행했지만, 스프링에선 스프링 컨테이너를 통해서 사용 
 * 스프링 컨테이너는 `@Configuration` 이 붙은 AppConfig 를 구성 정보로 사용하며, `@Bean` 메서드를 모두 호출하여 반환되는 객체를 스프링 컨테이너에 등록한다. 등록된 객체를 스프링 빈 이라고 한다.
 * 스프링 빈은 `@Bean`이 붙은 메서드 명을 스프링 빈의 이름으로 사용한다. 

## 스프링 컨테이너 생성
 * ApplicationContext는 스프린 컨테이너. interface로 만들어져있다.
 * 인터페이스이기 때문에 Xml 기반으로 구현할 수도 있고, 어노테이션 기반으로 구현체를 만들수도 있다.
