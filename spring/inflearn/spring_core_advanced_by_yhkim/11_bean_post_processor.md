# 빈 후처리기 (Bean PostProcessor)
 * 스프링의 빈 등록 절차
    1. 스프링이 빈이 될 클래스를 생성
    2. 스프링 컨테이너에 등록하기 전, 후처리기 프로세스 진행
    3. 초기화 전에 할지, 후에 할지는 메서드로 선택 가능
    4. 해당 클래스를 스프링 컨테이너에 등록
 * 그리고 이후엔 컨테이너에 등록한 스프링 빈을 조회해서 사용한다.
 * 스프링 빈 저장소에 등록할 목적으로 생성한 객체를 빈 저장소에 등록하기 직전에 조작하고 싶다면 빈 후처리기를 사용할 수 있다.
 * 빈 후처리기는 빈을 생성한 후에 무언가를 처리하는 용도인데, 객체를 조작할 수 있고 완전히 다른 객체로 바꿔치기도 가능하다.

![image](https://user-images.githubusercontent.com/48814463/207981280-23fe700e-c9b5-4a67-a67b-2e8f5d7445d1.png)


## 빈 후처리기 사용 방법
 * BeanPostProcessor 인터페이스를 구현하는 후처리기 클래스를 만들고, 스프링 빈으로 등록하면 된다.
 * BeanPostProcessor에는 2가지 메서드가 있다.
   1. postProcessBeforeInitialization : 객체 생성 이후에 @PostConstruct 같은 초기화가 발생하기 전에 호출
   2. postProcessAfterInitialization : 객체 생성 이후에 @PostConstruct 같은 초기화가 발생한 다음 호출

## 빈후처리기 코드 예제

#### A클래스를 B클래스로 변환해주는 AtoBProcessor 정의
 * 아래처럼 정의한 AtoBProcessor 를 Bean으로 등록하면, 스프링 컨테이너의 모든 bean에 대해 후처리기가 동작한다.
```java
@Slf4j
static class AtoBProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        if (bean instanceof ClassA) {
            return new ClassB();
        }

        return bean;
    }
}
```

#### 특정 패키지의 하위 클래스에 ProxyFactory를 적용하는 프로세서 예제

```java
public class PackageLogTracePostProcessor implements BeanPostProcessor {

    private final String basePackage;
    private final Advisor advisor;

    public PackageLogTracePostProcessor(String basePackage, Advisor advisor) {
        this.basePackage = basePackage;
        this.advisor = advisor;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        log.info("param beanName={} bean={}", beanName, bean.getClass());

        //프록시 적용 대상 여부 체크
        //프록시 적용 대상이 아니면 원본을 그대로 진행
        String packageName = bean.getClass().getPackageName();
        if (!packageName.startsWith(basePackage)) {
            return bean;
        }

        //프록시 대상이면 프록시를 만들어서 반환
        ProxyFactory proxyFactory = new ProxyFactory(bean);
        proxyFactory.addAdvisor(advisor);

        Object proxy = proxyFactory.getProxy();
        log.info("create proxy: target={} proxy={}", bean.getClass(), proxy.getClass());
        return proxy;
    }
}

```


## [참조] 빈 생명주기 콜백
#### 
> 스프링 컨테이너 생성 -> 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> bean 사용 start
> bean 사용 finish -> 소멸전 콜백 -> 스프링 종료

### 콜백의 종류
1. 콜백을 추가하고자 하는 클래스에 인터페이스(InitializingBean, DisposableBean)를 구현하여 사용. 아래 2가지 메서드를 override 하여 적용.
    * afterPropertiesSet(): 의존관계 주입이 끝난 후 호출
    * destroy(): 빈이 죽기 직전에 호출
2. bean 설정 정보(xml 에서 init-method, destroy-met 로 사용할 메서드 이름을 지정해야함) 초기화 메서드, 종료 메서드 지정
    *  init-method, destroy-method로 지정한 메서드를 만들어야함.
3. (추천) @PostConstruct, @PreDestroy 사용 - 초기화 혹은 소멸전 메서드 위에 붙이기만 하면 되기 때문에 가장 편하게 사용 가능

### @PostConstruct, @PreDestroy 란?
 * @PostConstruct : 빈 생성 및 의존성 주입이 이루어진 후 초기화를 수행하는 메서드
 * @PreDestroy : 빈소멸전 호출되는 메서드
 * @PostConstruct가 붙은 메서드는 클래스가 service(로직을 탈 때? 로 생각 됨)를 수행하기 전에 발생한다.
 * 이 메서드는 다른 리소스에서 호출되지 않는다해도 수행된다.
 * 빈 생성 이후 딱 1번만 호출됨을 보장한다.


#### 
```java
@Service
public class BusinessServiceImpl implements BusinessService {
 
    @Autowired
    DataDAO dataDAO;
 
    private ParamDTO paramDTO;
 
    @PostConstruct
    public void initialize(){
        paramDTO = new ParamDTO();
    }
}

```

![image](https://user-images.githubusercontent.com/48814463/208243632-aa05c274-d777-4775-accf-92b2372ff269.png)


