## 1. ApplicationContext : 스프링 컨테이너
 * 기존에는 AppConfig 를 생성해서 직접 DI를 수행했지만, 스프링에선 스프링 컨테이너를 통해서 사용 
 * 스프링 컨테이너는 `@Configuration` 이 붙은 AppConfig 를 구성 정보로 사용하며, `@Bean` 메서드를 모두 호출하여 반환되는 객체를 스프링 컨테이너에 등록한다. 등록된 객체를 스프링 빈 이라고 한다.
 * 스프링 빈은 `@Bean`이 붙은 메서드 명을 스프링 빈의 이름으로 사용한다. 

### 스프링 컨테이너 생성
 * ApplicationContext는 스프링 컨테이너, interface로 만들어져있다.
 * 인터페이스이기 때문에 Xml 기반으로 구현할 수도 있고, 어노테이션 기반으로 구현체를 만들수도 있다.

### 스프링 빈 의존관계 설정
 * 스프링 컨테이너에 빈들이 등록되면, 설정정보를 참고하여 빈들끼리의 의존관계를 자동으로 주입해준다.

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



## 2. BeanFactory와 ApplicationContext
 * BeanFactory : 최상위 인터페이스 <- ApplicationContext : 인터페이스 <- AnnotationConfigApplicationContext 구현체
 *  BeanFactory 나 ApplicationContext를 스프링 컨테이너라고 한다.

### BeanFactory
 * 스프링 빈을 관리하고 조회하는 역할
 * getBean 제공

### ApplicationContext
 * BeanFactory 기능을 모두 상속받아 제공하기 때문에 BeanFactory을 직접 쓰지 않고 주로 ApplicationContext 사용한다.
 * 상속받은 빈 관리하는 기능 외에 다양한 부가기능 제공
    * MessageSource 메세지 소스를 활용한 국제화 기능
    * EnvironmentCapable : 환경변수 - 로컬, 개발, 운영 등 구분해서 처리
    * ApplicationEventPublisher : 애플리케이션 이벤트 - 이벤트 발행 및 구독하는 모델을 편리하게 지원
    * ResorceLoader : 편리한 리소스 조회 (file, classPath, 외부 리소스)


## 3. 스프링 빈 설정 메타 정보 : BeanDefinition
 * 스프링은 어떻게 이런 다양한 설정 형식(java, xml 등)을 지원할 수 있을까? -> BeanDefinition 이라는 추상화 활용
 * 어떤 형식이든 결국 그 형식을 읽어서 BeanDefinition 을 만들면 된다.
    * `AnnotationConfigApplicationContext`는 `AnnotatedBeanDefinitionReader`를 사용해서 `AppConfg.class`를 읽고 `BeanDefinition` 을 생성한다.
    * 새로운 형식이 추가되면, XxxxxConfigApplicationContext는 XxxxxedBeanDefinitionReader를 사용해서 AppConfg.xxxx 읽고 BeanDefinition 을 생성한다.
 * 스프링 컨테이너는 이 BeanDefinition 이라는 빈 설정 메타 정보를 기반으로 스프링 빈을 생성한다.


### BeanDefinition 정보
 * BeanClassName : 생성할 빈의 클래스명
 * factoryBeanName : 팩토리 역할의 빈을 사용하는 경우, ex) appConfig
 * factoryMethodName : 빈을 생성할 팩토리 메서드 지정, ex) memberService
 * Scope : 싱글톤 (디폴트)
 * LazyInit : 스프링 컨테이너를 생성할때 빈을 생성하는게 아니라, 빈을 사용할때까지 최대한 생성을 지연처리하는지 여부
 * InitMethodName : 빈을 생성하고 의존관계를 적용한 뒤 호출되는 초기화 메서드
 * DestroyMethodName : 빈 생명주기가 끝나서 제거하기 직전 호출되는 메서드
 * Constructor arguments, Properties : 의존관계 주입에서 사용
