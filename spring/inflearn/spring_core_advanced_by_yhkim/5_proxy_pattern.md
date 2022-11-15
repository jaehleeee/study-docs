# 프록시 패턴과 데코레이터 패턴

## 1. 예제 상황 : 스프링 빈 등록에 대한 여러 케이스
1) 인터페이스와 구현 클래스 - 스프링 빈 수동 등록
2) 인터페이스 없이 구현 클래스 - 스프링 빈 수동 등록
3) 컴포넌트 스캔으로 스프링 빈 등록
 
### 1) 인터페이스와 구현 클래스 - 스프링 빈 수동 등록
 * `@Import` : 클래스를 스프링 빈으로 등록한다. 일반적으로 설정파일을 등록할때 사용하지만, 스프링 빈을 등록할때 사용해도 된다. 
    * (테스트를 위해 import 설정을 추가했다. 원래는 componentScan 과정에서 @Configuration 클래스를 모두 빈으로 등록한다.)
 * `@SpringBootApplication(scanBasePackages = "hello.proxy.app")` : @ComponentScan 기능과 같다. 컴포넌트 스캔 시작 위치를 지정한다. 
    * 이 설정이 없다면, ProxyApplication 패키지와 하위 패키지를 스캔한다.

```java
@Import(AppV1Config.class)
@SpringBootApplication(scanBasePackages = "hello.proxy.app") //주의
public class ProxyApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProxyApplication.class, args);
	}

}

---

@Configuration
public class AppV1Config {

    @Bean
    public OrderControllerV1 orderControllerV1() {
        return new OrderControllerV1Impl(orderServiceV1());
    }

    @Bean
    public OrderServiceV1 orderServiceV1() {
        return new OrderServiceV1Impl(orderRepositoryV1());
    }

    @Bean
    public OrderRepositoryV1 orderRepositoryV1() {
        return new OrderRepositoryV1Impl();
    }

}


```



### 2) 인터페이스 없이 구현 클래스 - 스프링 빈 수동 등록



