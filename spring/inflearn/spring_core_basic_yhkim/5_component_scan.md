# 컴포넌트 스캔과 의존관계 자동 주입
 * 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
 * 또, 의존관계도 자동으로 주입하는 `@Autowired` 기능도 제공한다.

## 1. `@ComponentScan`
 * @Configuration 과 함께 설정 클래스에 붙여서 사용
    * 스프링부트에서는 Application 클래스 파일에 자동으로 붙는 `@SpringBootApplication`에 포함되어 있어서 수동으로 추가핦 필요없다.
 * `@Component` 어노테이션이 붙은 클래스를 모두 스프링 빈으로 등록한다.
 * 탐색위치는 `basePackages = "{package 위치}"` 파라미터를 통해 탐색 시작 위치를 설정할 수 있다. (설정하지 않으면, 디폴트는 ComponentScan 어노테이션이 붙은 클래스의 패키지부터 시작한다.)
 * 특정 component를 스캔에 포함시키려면, includeFilter 파라미터 사용
    * @MyIncludeComponent 어노테이션이 붙은 클래스를 포함하는 예시 : includeFilter = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class)
 * 특정 component를 스캔에서 제외하려면, excludeFilter 파라미터 사용
    * @MyExcludeComponent 어노테이션이 붙은 클래스를 제외하는 예시 : excludeFilter = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
 * 탐색 대상
    * @Component
    * @Controller(스프링 MVC 컨트롤러로 인식하는 기능도 있음)
    * @Service
    * @Repository(스프링 데이터 계층으로 인식, 데이터 계층의 예외를 스프링 예외로 변환해준다.)
    * @Configuration (스프링 설정 정보로 인식)


## 2. 의존관계 주입 with `@Autowired`
 * (주의) 스프링 빈 클래스에서만 동작하는 어노테이션이다

#### 4가지 사용 케이스
 * 생성자 주입
    * 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
    * 불변, 필수인 의존관계일때 사용된다. (`private final`)
    * 생성자가 1개만 있으면 `@Autowired` 생략 가능
 * 수정자 주입 (setter)
    * 선택적으로 가변 가능성이 있는 의존관계에 사용
    * 생성자 주입과 비슷하게, 스프링 빈 준비 단계에서 호출되어 의존관계 주입된다.
    * @Autowired 주입 시점에 주입할 대상이 없으면 오류가 발생한다. 주입할 대상이 없어도 동작하게 하려면 required = false 설정 필요
 * 필드 주입 (권장하지 않음.)
    * 코드가 간결해서 과거에 많이 사용했었음.
    * 외부에서 변경이 불가능해서 테스트하기 힘들다는 치명적인 단점이 있다. 
 * 일반 메서드 주입 (권장하지 않음.)
    * 한번에 여러 필드를 주입 받을 수 있다.
    * 일반적으로 거의 사용되지 않음.

#### 스프링 빈 주입을 선택적으로 하고 싶을 때
 *  `@Autowired(required = false)`
 *  `@Nullable`
 *  `Optional`

```
@Autowired(required = false)
public void setNoBean1(A a){} // 호출되지 않음.

@Autowired
public void setNoBean2(@Nullable A a){} // null 호출

@Autowired(required = false)
public void setNoBean3(Optional<A> a){} // Optional.empty 호출

```

#### 최신 트렌드 lombok 활용
 * 의존관계 클래스를 private final 필드로 세팅해두고, `@RequiredArgsConstructor` 어노테이션만 세팅하면, 생성자 의존성 자동 주입 완성

#### 같은 타입의 빈이 2개 이상인 경우
 * `@Autowired` 는 타입으로 빈을 조회한다. 
 * 그래서 하나의 interface 의존관계의 상속된 하위 타입이 2개 이상인 빈이 존재한다면 NoUniqueBeanDefinitionException이 발생한다.
    * DiscountPolicy 라는 의존관계 interface가 있고, 하위에 RateDiscountPolicy와 FixDiscountPolicy 2가지 빈이 존재하는 경우
####  해결방안은? `@Qualifier` `@Primary`
 1. @Autowired 필드명 매칭
 ```
 @Autowired
 private DiscountPolicy rateDiscountPolicy
 ```
 
 2. `@Qualifier` 끼리 매칭
     1. `@Component` 어노테이션과 함께 `@Qualifier`를 붙여서 구분자 추가
     2. `@Autowired` 주입하는 부분에서 `@Qualifier` 붙여서 구분자 매칭


 3. `@Primary` 사용 : 높은 우선순위 부여
     1. `@Component` 어노테이션과 함께 `@Qualifier`를 붙여서 자동 주입시 높은 우선순위 부여 추가
