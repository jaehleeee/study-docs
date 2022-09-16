## 제어의 역전
 * 구현객체를 누가 생성해주는가. 프로그램의 제어 흐름을 누가 제어하는가에 대한 부분이다.
 * 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 AppConfig 처럼 외부에서 관리하는 것을 제어의 역전이라고 한다.
 * 프레임워크 VS 라이브러리
 

## 의존관계 주입
 * 실행 시점에 외부에서 실제 구현 객체를 생성하고, 클라이언트에 전달해서 클라이언트와 서버의 실제 의존 관계가 연결 되는 것을 의존관계 주입 이라 한다.
 * 클라이언트 주입을 사용하면,
    * 클라이언트 코드를 변경하지 않고 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.
    * 정적인 클래스 의존관계를 변경하지 않고, 객체 인스턴스 의존관계를 쉽게 변경할 수 있다.
 * 정적인 클래스 의존관계
    * import 코드만 보고 쉽게 파악
    * 애플리케이션을 실행하지 않아도 분석 가능
 * 동적인(실행시점) 객체(인스턴스) 의존관계
    * 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다.

## 컨테이너 
 * AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해주는 것을 IOC 컨테이너 혹은 DI 컨테이너라고 한다.
 * 의존관계 주입에 초첨을 맞춰 최근에는 DI 컨테이너라고 한다.
 * 또는 어셈블러, 오브젝트 팩토리 등으로 불리기도 한다.

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










