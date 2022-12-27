# 프록시 패턴

## 1. 예제 상황 : 스프링 빈 등록에 대한 여러 케이스
1) 인터페이스와 구현 클래스 - 스프링 빈 수동 등록
2) 인터페이스 없이 구현 클래스 - 스프링 빈 수동 등록
3) 컴포넌트 스캔으로 스프링 빈 등록 (`@Controller`, `@Service` 어노테이션 사용)

#### 위 케이스에 대한 새로운 요구 사항
 1. 원본 코드 수정없이 로그 추적기 적용하자.
 2. 특정 메서드는 로그 출력되지 않아야 한다.
 3. 위 3가지 케이스 모두에 적용할 수 있어야 한다.

 
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


## 2. 프록시 패턴이란?
 * 클라이언트의 요청을 서버가 처리하기전에 미리하거나 보충해주는 역할을 하는 대리자.
 * 단 클라이언트는 서버가 처리한 것인지, 프록시가 처리한 것인지 몰라야한다. (DI를 사용하여 Server 대신 Proxy를 주입해준다.)

#### 프록시 기능
1. 캐시 : 엄마에게 라면을 사달라고 부탁했는데, 이미 집에 있었다.
2. 접근 제어 : 권한에 따른 접근 차단
3. 부가 기능 추가 : 아빠에게 주유를 부탁했는데 세차까지 해주셨다.
4. 프록시 체인 : 대리자가 또 다른 대리자를 부를 수도 있다. 동생에게 라면 구입을 부탁했는데, 동생이 엄마에게 부탁해서 라면을 구입했다.


#### (둘다 프록시를 사용하는 방법이지만) 프록시 패턴과 데코레이터 패턴은 목적에 따라 구분
 * 프록시 패턴 : 접근 제어가 목적
 * 데코레이터 패턴 : 새로운 기능 추가가 목적

### 프록시 패턴
 * client -> cacheProxy -> realTarget 런타임 객체 의존관계 형성
 * cacheProxy에 클라이언트가 원하는 타겟이 없을때만 realTarget을 호출한다. cacheProxy가 타겟을 들고 있으면 클라이언트에게 cacheProxy 단계에서 타겟을 응답해준다.
 * 이 패턴의 핵심은, cacheProxy 도입하기 위해 클라이언트와 realTarget의 수정이 필요없다는 점이다.
    * 클라이언트 입장에서는 프록시가 있는지 없는지 알 수 없고 상관쓰지도 않는다.

#### 클라이언트 사용 코드

```java
@Test
void cacheProxyTest() {
	RealSubject realSubject = new RealSubject();
	CacheProxy cacheProxy = new CacheProxy(realSubject);
	ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
	client.execute();
	client.execute();
	client.execute();
}
```
