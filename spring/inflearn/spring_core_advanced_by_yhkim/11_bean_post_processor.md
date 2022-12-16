# 빈 후처리기 (Bean PostProcessor)
 * 스프링은 스프링 빈을 등록하면, 대상 객체를 생성하고 스프링 컨테이너 내부의 빈 저장소에 등록한다.
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

#### AtoBProcessor 정의
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

## [참조] 빈 생명주기 콜백
> 스프링 컨테이너 생성 ->  빈 생성 -> 의존관계 주입  ->  초기화 콜백 -> 사용  -> 소멸전 콜백 -> 스프링 종료

### 콜백의 종류
1. 인터페이스(InitializingBean, DisposableBean) 사용. 아래 2가지 메서드 지원.
    * afterPropertiesSet(): 의존관계 주입이 끝난 후 호출
    * destroy(): 빈이 죽기 직전에 호출
2. 설정 정보 초기화 메서드, 종료 메서드 지정
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

